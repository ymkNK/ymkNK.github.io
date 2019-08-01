task :default => :new

require 'fileutils'

desc "创建新 post"
task :new do
  puts "请输入要创建的文章文件文件名字："
    @url = STDIN.gets.chomp
    puts "请输入文章的 标题："
    @name = STDIN.gets.chomp
    puts "请输入文章的 子标题："
    @subtitle = STDIN.gets.chomp
    # puts "请输入文章的 分类，以空格分隔："
    # @categories = STDIN.gets.chomp
    # 现在这俩没啥区别了，就只用输入一个就行了
    puts "请输入文章的 标签："
    @tag = STDIN.gets.chomp

    @slug = "#{@url}"
    @slug = @slug.downcase.strip.gsub(' ', '-')
    @date = Time.now.strftime("%F")

    # 生成随机封面
    @img_num = rand(1...11)
    # 需要检查一下对应tag的目录是否存在
    @directory_name = @tag.downcase
    @directory_url = "_posts/#{@tag}"
    if File.directory?(@directory_url)
        puts "The directory: #{@directory_url} has existed."
    else
        FileUtils.mkdir(@directory_url)
        puts "The directory: #{@directory_url} has created."
    end

    # 对应分类的文件放进对应的文件夹当中
    @post_name = "#{@directory_url}/#{@date}-#{@slug}.md"

    if File.exist?(@post_name)
            abort("文件名已经存在！创建失败")
    end

    FileUtils.touch(@post_name)
    open(@post_name, 'a') do |file|
            file.puts "---"
            file.puts "layout: post"
            file.puts "title: #{@name}"
            file.puts "subtitle: #{@subtitle}"
            file.puts "author: ymkNK"
            file.puts "date: #{Time.now}"
            # tag和categories一致了，因为目前这俩没啥太大区别
            file.puts "categories: #{@tag}"
            file.puts "tag: #{@tag}"
            file.puts "img: #{@img_num}.jpg"
            file.puts "---"
    end
    puts "rake new successfully"
    exec "vim #{@post_name}"
end

task :git do
  puts "请输入git内容 m"
    @msg = STDIN.gets.chomp

    @slug = "#{@msg}"
    @slug = @slug.downcase.strip.gsub(' ', '-')
    @date = Time.now.strftime("%F")
    @finalmsg = "#{@date}-#{@slug}"
    system "git add ."
    system "git commit -m \"#{@finalmsg}\""
    system "git push"
    puts "git push \"#{@finalmsg}\" successfully"
end
