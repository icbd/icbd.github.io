---
layout: post
title:  Ruby CGI to Rack
date:   2020-08-23
Author: CBD 
tags: [Ruby, Server]
---

## 1974 年, TCP/IP 诞生;

## 1991 年 HTTP 0.9 发布;

## 1995 年 Apache HTTP Server;

Apache 脱胎于 NCSA 的 HTTPd, 是当时 Netscape Web Server 的开源替代产品. 期初的 Web 页面几乎只有文本.

我们在 macOS 15.5 上使用 Apache 2.4.43 来配置一个只能伺服静态资源的 vHost .

文档详见: [https://httpd.apache.org/docs/2.4/vhosts/name-based.html](https://httpd.apache.org/docs/2.4/vhosts/name-based.html)

在 `/etc/hosts` 中添加:

```
127.0.0.1 archeology.devel
127.0.0.1 www.archeology.devel
```

打开 `httpd.conf` 中的注释:

```
Include /usr/local/etc/httpd/extra/httpd-vhosts.conf
```

编辑 `extra/httpd-vhosts.conf`:

```
<VirtualHost *:80>
    ServerAdmin admin@archeology.devel
    DocumentRoot "/Users/cbd/Coding/ruby/archeology"
    ServerName archeology.devel
    ServerAlias www.archeology.devel
    ErrorLog "/usr/local/var/log/httpd/archeology.devel-error_log"
    CustomLog "/usr/local/var/log/httpd/archeology.devel-access_log" common
</VirtualHost>

<Directory "/Users/cbd/Coding/ruby/archeology">
    AllowOverride None
    Options None
    Require all granted
</Directory>
```

在 Document Root 的目录里添加静态文件:

```
.
├── favicon.ico
└── index.html

```

如此, [http://www.archeology.devel/](http://www.archeology.devel/) 就可以访问了.

## CGI

[https://en.wikipedia.org/wiki/Common_Gateway_Interface](https://en.wikipedia.org/wiki/Common_Gateway_Interface)

CGI 即 Common Gateway Interface, 1993 就出现在了 NCSA 的 HTTPd 中.

纯静态的页面已经不能满足需求, 大家希望能通过 Server 做定制化的工作, CGI 就是最朴素的解决方案.

CGI 实际上是一个协议: 
* URL 路径对应 CGI 脚本的路径;
* GET 请求参数映射到 Server 的环境变量;
* POST 请求参数映射到 `$stdin`;
* CGI 脚本的响应结果输出到 `$stdout` 然后返回给客户端;

文档详见: [https://httpd.apache.org/docs/2.4/howto/cgi.html](https://httpd.apache.org/docs/2.4/howto/cgi.html)

为了开启 CGI 功能, 需要调整一下配置. 

修改 `httpd.conf`, 打开注释:

```
LoadModule cgi_module lib/httpd/modules/mod_cgi.so
```

修改 `extra/httpd-vhosts.conf`:

允许 Apache 把 `.rb` 后缀的请求当做 CGI 来解析执行;

```
<Directory "/Users/cbd/Coding/ruby/archeology">
    AllowOverride None
    Options +ExecCGI
    AddHandler cgi-script .rb
    Require all granted
</Directory>
```

重启 Apache 并查看配置:

```sh
sudo httpd -k restart
httpd -S
```

添加 cgi 脚本:

```
.
├── cgi
│   └── cgi_server.rb
├── favicon.ico
└── index.html
```

> cgi_server.rb

```ruby
#!/usr/bin/env ruby

require "cgi"
require 'json'

cgi = CGI.new

info = {
  'Process.pid' => Process.pid,
  'Thread.current' => Thread.current,
  'Time.now' => Time.now,
}

puts "Content-Type:application/json\n\n"
puts info.merge(cgi.params).to_json
```

至此, 访问 [http://www.archeology.devel/cgi/cgi_server.rb](http://www.archeology.devel/cgi/cgi_server.rb) 就能得到动态的内容了. 

`cgi_server.rb` 其实只是一个脚本, 不限实现的语言. 该脚本也可以直接本地执行. ( Ruby CGI 会提示 `offline mode`, 键入参数后回车, `Ctrl+D` 结束输入. )

多次请求会发现每次的 pid 都不同, 也就是说, 每个新的请求都会新开启一个进程来执行脚本, 处理完之后就结束进程.

## Fast-CGI

CGI 的问题显而易见, 为每个请求都开启新进程是很大的开销. Fast-CGI 选择用 UNIX Socket 或 TCP 的方式跟一个CGI进程通信, 这个 CGI 进程需要常驻地来处理请求.

文档详见: [https://httpd.apache.org/docs/2.4/mod/mod_proxy_fcgi.html](https://httpd.apache.org/docs/2.4/mod/mod_proxy_fcgi.html)

编辑 `httpd.conf`, 打开注释:

```
LoadModule proxy_module lib/httpd/modules/mod_proxy.so
LoadModule proxy_fcgi_module lib/httpd/modules/mod_proxy_fcgi.so
```

```
.
├── cgi
│   ├── cgi_server.rb
│   └── fcgi_server.rb
├── favicon.ico
└── index.html
```

> fcgi_server.rb

```ruby
#!/usr/bin/env ruby
require "fcgi"
require 'json'

FCGI.each_cgi do |cgi|
  info = {
    'Process.pid' => Process.pid,
    'Thread.current' => Thread.current,
    'Time.now' => Time.now,
  }
  puts "Content-Type:application/json\n\n"
  puts info.merge(cgi.params).to_json
end
```

安装工具和依赖:

```
brew install fcgi spawn-fcgi
gem install fcgi
```

```
spawn-fcgi -u _www -U _www  -f /Users/cbd/Coding/ruby/archeology/cgi/fcgi_server.rb -p 3000
```

编辑 `extra/httpd-vhosts.conf`, 添加 ProxyPass 规则:

```
<VirtualHost *:80>
    ServerAdmin admin@archeology.devel
    DocumentRoot "/Users/cbd/Coding/ruby/archeology"
    ServerName archeology.devel
    ServerAlias www.archeology.devel
    ErrorLog "/usr/local/var/log/httpd/archeology.devel-error_log"
    CustomLog "/usr/local/var/log/httpd/archeology.devel-access_log" common

    ProxyPass "/fast-cgi/" "fcgi://localhost:3000/"
</VirtualHost>
```

至此, 访问 [http://www.archeology.devel/fast-cgi/](http://www.archeology.devel/fast-cgi/) 就能得到对应的动态内容了. 

response 中显示 pid 是固定的.


## 2005 年, Ruby on Rails 1.0 发布

Rails 初期更像是一组工具包, 还有专门的工具封装 CGI.

## 2007 年, Rack 发布

Rack 把大家从处理 CGI 的繁琐工作中解放出来, 使得各个组件更加模块化:

* Common Web Server (Apache/NGINX) 负责处理静态资源, 转发动态请求;

* Web Server (PUMA/Thin/Unicorn) 负责把请求调度给 Rack 对象;

* Rack 负责解析 HTTP 请求;

* Rack 中间件 (Rails) 负责加工请求, 组织各种功能的Gem;

* 具体的业务逻辑负责拼接功能模块;

每个模块各司其职, 也可以根据需求场景替换, 在一致的约定下有序配合工作.


写一个 Rack 支持的 App, 要符合两个要求:

* 能相应 `call` 方法, 参数为 env ;
* 返回一个数组, 满足排列 `[http_code, http_headers, http_body]`

```
.
├── cgi
│   ├── cgi_server.rb
│   └── fcgi_server.rb
├── favicon.ico
├── index.html
└── rack
    └── rack_server.rb
```

> rack/rack_server.rb

```ruby
#!/usr/bin/env ruby
require 'rack'
require 'rack/handler/puma'
require 'json'

app = Proc.new do |env|
  payload = {
    'Process.pid' => Process.pid,
    'Thread.current' => Thread.current,
    'Time.now' => Time.now,
    'env' => env
  }.to_json
  ['200', {'Content-Type' => 'application/json'}, [payload]]
end

Rack::Handler::Puma.run(app, {Verbose: true, Port: 3000, Threads: '3:3'})
```

编辑 `httpd.conf`, 打开注释:

```
LoadModule proxy_http_module lib/httpd/modules/mod_proxy_http.so
```

编辑 `extra/httpd-vhosts.conf`, 在 VirtualHost 中加入:

```
ProxyPass "/api/" "http://localhost:3000/"
```

至此, 访问 [http://archeology.devel/api/](http://archeology.devel/api/) 就能得到 Rack 的响应了.


## Demo Code

[https://gitlab.com/Cbd-Focus/ruby-cgi-to-rack](https://gitlab.com/Cbd-Focus/ruby-cgi-to-rack)


## 参考

[https://insights.thoughtworks.cn/ruby-web-server/](https://insights.thoughtworks.cn/ruby-web-server/)