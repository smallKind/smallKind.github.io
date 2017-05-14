---
layout: post
title: 怎么本地搭建jekyll
date: 2017-05-14 20:03:01 +8000
category: 分享
tags: jekyll
---

* content
{:toc}

### 怎么在本地搭建Jekyll

#### Windows :

1. 安装 Ruby

   1. 前往 [http://rubyinstaller.org/downloads/](http://rubyinstaller.org/downloads/)
   2. 在 “RubyInstallers” 部分，选择某个版本点击下载。例如， Ruby 2.0.0-p451 (x64) 是适于64位 Windows 机器上的 Ruby 2.0.0 x64 安装包。
   3. 通过安装包安装
      * 最好保持默认的路径 例： C:\Ruby200-x64， 因为安装包明确提出 “请不要使用带有空格的文件夹 (如： Program Files)”。
      * 勾选 “Add Ruby executables to your PATH”，这样执行程序会被自动添加至 PATH 而避免不必要的头疼。
   4. 打开一个命令提示行并输入以下命令来检测 Ruby 是否成功安装。
    
       ruby -v
       
       输出示例：

       ruby 2.0.0p451 (2014-02-24) [x64-mingw32]
  
2. 安装 DevKit

  1. 再次前往 http://rubyinstaller.org/downloads/
  2. 载同系统及 Ruby 版本相对应的 DevKit 安装包。 例如，DevKit-mingw64-64-4.7.2-20130224-1432-sfx.exe 适用于64位 Windows 系统上的 Ruby 2.0.0 x64。
  3. 运行安装包并解压缩至某文件夹，如 C:\DevKit
  4. 通过初始化来创建 config.yml 文件。在命令行窗口内，输入下列命令：
     
     cd “C:\DevKit”
     
     ruby dk.rb init
     
     notepad config.yml

  5. 在打开的记事本窗口中，于末尾添加新的一行（第一步ruby的默认路径，如有可不添加）例： - C:\Ruby200-x64，保存文件并退出。
  6. 回到命令行窗口内，审查（非必须）并安装。
  
     ruby dk.rb review
     
     ruby dk.rb install
     
3. 安装 Jekyll
 
  1. 确保 gem 已经正确安装
    
     gem -v
     
     输出示例：
     
     2.0.14
     
  2. 安装 Jekyll gem     

     gem install jekyll

#### Linux:

1. 安装依赖软件

   yum install ruby
    
   yum install ruby-devel
   
   yum install gcc
   
2. 修改gem源

   gem source -l
   
   gem sources --remove https://rubygems.org/
 
   gem sources -a https://ruby.taobao.org

   gem source -l

   https://ruby.taobao.org/

   gem sources -u  
   
3. 安装Jekyll

   gem install json_pure
   
   gem install jekyll  
   
输入：jekyll --version  输出类似如下：jekyll 3.1.2 则安装成功

### Jekyll的基本用法

[Jekyll的基本用法](http://jekyllcn.com/docs/usage/) 

[Jekyll的主题](http://jekyllthemes.org/)    

后台运行，任意主机访问启动命令： jekyll serve build -B -H 0.0.0.0

