---
layout: post
title: 从零开始的查询优化
subtitle: record-query-upgrade
author: ymkNK
date: 2020-10-30 16:28:10 +0800
categories: database
tag: database
img: 5.jpg
---
### 背景
需要查询到一批部门以及子部门，最终拿到这些部门下的人员信息，我们能够进行的操作如下
- 通过部门的目录前缀（前缀树）的方式，能够查询到所有子部门
- 能够通过部门查询到这个部门（不包含子部门）的所有人员
- 已经添加索引

### 方法一

#### 思路
找到每一个需要排除的部门，以及每一个排除的部门的子部门，然后分别查出来需要排除的人,最后通过stream的flatMap进行聚合
```
           return allDepartment.stream().flatMap(department -> {
               List<EmployeeDetail> allByDepartmentId = employeeRepo.findAllByDepartmentId(department.getDepartmentId());
               return allByDepartmentId.stream();
           }).collect(Collectors.toList());
```

#### 耗费时间
`30616ms`



### 方法二

#### 思路
找到每一个需要排除的部门，将部门聚合，分别查询
```
           List<Department> allDepartment = findAllChildDepartments(byId.get());
           return employeeRepo.findAllByDepartmentIdIsIn(allDepartment.stream().map(Department::getDepartmentId).collect(Collectors.toList()));

           List<EmployeeDetail> excludeEmployees = excludeDepartments.stream().flatMap(department -> serviceV2.findAllEmployeesByDepartmentId(department.getDepartmentId(), 1).stream()).collect(Collectors.toList());
```

#### 耗费时间
`20057ms`


### 方法三

#### 思路
将所有需要排除的部门聚合，统一查询
```
    @Override
    public List<EmployeeDetail> findAllEmployeesByDepartmentsId(Iterable<String> departmentIds) {
        return employeeRepo.findAllByDepartmentIdIsIn(departmentIds);
    }
```
```
            List<Department> allExcludedDepartments = excludeDepartments.stream().flatMap(department -> serviceV2.findAllChildDepartments(department).stream()).collect(Collectors.toList());
            List<EmployeeDetail> excludeEmployees = serviceV2.findAllEmployeesByDepartmentsId(allExcludedDepartments.stream().map(Department::getDepartmentId).collect(Collectors.toList()));

```

#### 耗费时间
`26058ms`
时间耗费反而多了，因为对数据库的查询次数变多了

### 方法四

#### 思路
取出所有的排除的部门，构造出一个Set，然后通过filter取到需要的人员
```
        List<Department> excludeDepartments = serviceV2.findAllDepartmentsByName(exclude);
        log.info("excludeDepartments:{}", JSONObject.toJSONString(excludeDepartments.stream().map(Department::getPath).toArray(), true));
        StopWatch started = StopWatch.createStarted();

        List<Department> allExcludedDepartments = excludeDepartments.stream().flatMap(department -> serviceV2.findAllChildDepartments(department).stream()).collect(Collectors.toList());
        log.info("excludeDepartments size:{}", allExcludedDepartments.size());
        Set<String> excludeDepartmentIdSet = allExcludedDepartments.stream().map(Department::getDepartmentId).collect(Collectors.toSet());
        List<EmployeeDetail> allEmployees = serviceV2.findAllEmployees();
        log.debug("all employee size:[{}]", allEmployees.size());
        allEmployees.removeIf(employeeDetail -> {
            String departmentId = employeeDetail.getDepartmentId();
            return excludeDepartmentIdSet.contains(departmentId);
        });
        log.debug("filtered employee size:[{}]", allEmployees.size());
        started.stop();
        log.debug("cost time:[{}]ms", started.getTime(TimeUnit.MILLISECONDS));
```

#### 耗时
`14030ms`


### 方法五

#### 思路
将方法四的removeIf换成流的处理方式
```
        allEmployees = allEmployees.stream().filter(employeeDetail -> {
                    String departmentId = employeeDetail.getDepartmentId();
                    return !excludeDepartmentIdSet.contains(departmentId);
                }).collect(Collectors.toList());
```

#### 耗时
`14104ms`
几乎没有区别


### 方法六

#### 思路
将所有的部门聚合，直接从库中取出来目标数据
```
        List<Department> excludeDepartments = serviceV2.findAllDepartmentsByName(exclude);
        log.info("excludeDepartments:{}", JSONObject.toJSONString(excludeDepartments.stream().map(Department::getPath).toArray(), true));
        StopWatch started = StopWatch.createStarted();s
        List<Department> allExcludedDepartments = excludeDepartments.stream().flatMap(department -> serviceV2.findAllChildDepartments(department).stream()).collect(Collectors.toList());
        log.info("excludeDepartments size:{}", allExcludedDepartments.size());
        Set<String> excludeDepartmentIdSet = allExcludedDepartments.stream().map(Department::getDepartmentId).collect(Collectors.toSet());
        List<EmployeeDetail> allEmployees = serviceV2.findAllEmployeesNotInDepartmentIds(excludeDepartmentIdSet);
        log.debug("all employee size:[{}]", allEmployees.size());
        log.debug("filtered employee size:[{}]", allEmployees.size());
        started.stop();
        log.debug("cost time:[{}]ms", started.getTime(TimeUnit.MILLISECONDS));
```

#### 耗时
`8666ms`

### 结论
方法六最靠谱，在数据量越大的情况下，查询时间与数据量肯定呈现线性关系。在这种情况下，尽量减少IO的读取次数，这样的查询时间消耗最少


