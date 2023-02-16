---
layout:     post
title:      jekyll环境搭建
subtitle:   MacOS(M1)搭建jekyll环境
date:       2023-02-12
author:     Key
header-img: img/post-jekyll.jpg
catalog: true
tags:
    - 环境搭建
    - Mac OS
---

## 记一次MacOS(M1）搭建jekyll环境

​	需要系统中存在**gem**，mac上默认自带gem所以基本上不用关注这个点

```shell
// 检查gem版本 本机版本 3.2.3 
$ gem -v 
// 更新权限
$ gem update --system
```

   目前出现的问题，由于mac上开启了**csrutil**,所以这里用gem基本上是安装不了的，会提示无法将文件写入/usr/bin/下。这个时候要将csrutil关闭，这里只查到了部分机器关闭的方式(方式列入下方)。

      1. 将设备关机按住 Command + R，一直到机器有选择启动方式的界面，选择恢复与还原。
    
      2. 将设备关机后，按住电源键30秒左右，一直到机器有选择启动方式的界面，选择恢复与还原。

   目前本人尝试第二种方式可以正确进入，部分机器可能有不同。

   进入后，需要输入一次账户密码，然后点击左上角的选项，打开终端，然后输入**csrutil disable**，关闭csrutil后正常重启即可。

​	重启后可以打开终端。开始安装	

```shell
// 查询一下当前csrutil状态, 输出为 System Integrity Protection status: disabled.
$ csrutil status

// 进行环境安装 注意这一步需要用梯子，能访问国外网站才可以正常下载，各种提示LoadError的都是网络问题
$ gen install jekyll bundler
```

​	安装如果正常完成了。需要更新一下ruby版本。

```shell
  // 查询ruby的版本列表（输出比较多，往上翻一下可以看到ruby的版本）
 	$ rvm list known
 	//更新最新版本 (注意，这里是我选的版本)，我的机器(m1的机器很多都有这个问题)提示有openssl的问题。
 	$ rvm install 3.0.0
 	
 	// 下面解决opnssl的问题
  // 先查询一下当前设备中的openssl的版本
  $ brew info openssl
  // 重新安装openssl
  $ brew install openssl
  // 再次尝试安装最新ruby
  $ rvm install 3.0.0

```

正确安装后就可以尝试启动jekyll了，先进入我们博客项目，再运行一下命令

```shell
// 运行jekyll
$ bundler exec jekyll server
// 到这里还是会有报错，报错如下
// 注意看LoadError,其中前面的内容就是缺少的库，我的机器是缺少kramdown-parser-gfm，缺少其他的内容也可以用这种方式安装
// 安装一个kramdown-parser-gfm
$ brew install kramdown-parser-gfm
```

```shell
Configuration file: /Users/coderkey/code/coderkey.github.io/_config.yml
       Deprecation: You appear to have pagination turned on, but you haven't included the `jekyll-paginate` gem. Ensure you have `plugins: [jekyll-paginate]` in your configuration file. 
            Source: /Users/coderkey/code/coderkey.github.io
       Destination: /Users/coderkey/code/coderkey.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
                    ------------------------------------------------
      Jekyll 4.3.2   Please append `--trace` to the `serve` command 
                     for any additional information or backtrace. 
                    ------------------------------------------------
/Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/converters/markdown/kramdown_parser.rb:122:in `require': cannot load such file -- kramdown-parser-gfm (LoadError)
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/converters/markdown/kramdown_parser.rb:122:in `load_dependencies'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/converters/markdown/kramdown_parser.rb:89:in `initialize'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/converters/markdown.rb:37:in `new'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/converters/markdown.rb:37:in `get_processor'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/converters/markdown.rb:15:in `setup'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/converters/markdown.rb:84:in `convert'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/renderer.rb:105:in `block in convert'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/renderer.rb:104:in `each'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/renderer.rb:104:in `reduce'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/renderer.rb:104:in `convert'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/renderer.rb:84:in `render_document'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/renderer.rb:63:in `run'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/site.rb:572:in `render_regenerated'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/site.rb:557:in `block (2 levels) in render_docs'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/site.rb:556:in `each'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/site.rb:556:in `block in render_docs'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/site.rb:555:in `each_value'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/site.rb:555:in `render_docs'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/site.rb:210:in `render'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/site.rb:80:in `process'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/command.rb:28:in `process_site'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/commands/build.rb:65:in `build'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/commands/build.rb:36:in `process'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/command.rb:91:in `block in process_with_graceful_fail'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/command.rb:91:in `each'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/command.rb:91:in `process_with_graceful_fail'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/lib/jekyll/commands/serve.rb:86:in `block (2 levels) in init_with_program'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/mercenary-0.4.0/lib/mercenary/command.rb:221:in `block in execute'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/mercenary-0.4.0/lib/mercenary/command.rb:221:in `each'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/mercenary-0.4.0/lib/mercenary/command.rb:221:in `execute'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/mercenary-0.4.0/lib/mercenary/program.rb:44:in `go'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/mercenary-0.4.0/lib/mercenary.rb:21:in `program'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/gems/jekyll-4.3.2/exe/jekyll:15:in `<top (required)>'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/bin/jekyll:23:in `load'
	from /Users/coderkey/.rvm/gems/ruby-3.0.0/bin/jekyll:23:in `<main>'
	
```

​	

到这一步基本上就已经完成全部的安装了，可以尝试运行了

```shell
// 进入博客项目
$ cd coderkey.github.io

// init bundle 这一步会创建文件	Gemfile 和 Gemfile.lock
$ bundle init

// 修改Gemfile
$ vi GemFile

// gem "jekyll", "~> 4.0"   将该内容添加到文件中，保存关闭
//尝试运行jekyll，正常情况下会启动一个服务，如果还缺少依赖的话，根据上面LoadError提示安装环境即可
bundle exec jekyll server

// 浏览器访问 http://127.0.0.1:4000/查看博客页

```

