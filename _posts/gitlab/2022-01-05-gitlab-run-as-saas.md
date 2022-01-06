---
layout: post
title:  Let gitlab run locally in SaaS mode 
date:   2022-01-05
Author: CBD
tags:   [GitLab]
---

## Mock 本地方法

先直奔主题, GitLab 通过 `Gitlab.com?` 来判断当前是否是 SaaS 环境, 最直接的办法就是修改掉这个方法, 让他直接返回 `true` .

```diff
diff --git a/lib/gitlab.rb b/lib/gitlab.rb
index 2449554d3c0..35b52ce13b6 100644
--- a/lib/gitlab.rb
+++ b/lib/gitlab.rb
@@ -50,8 +50,7 @@ def self.revision
   HTTP_PROXY_ENV_VARS = %w(http_proxy https_proxy HTTP_PROXY HTTPS_PROXY).freeze
 
   def self.com?
-    # Check `gl_subdomain?` as well to keep parity with gitlab.com
-    Gitlab.config.gitlab.url == Gitlab::Saas.com_url || gl_subdomain?
+    true
   end
 
   def self.com
```

如果你只是需要 Debug 一些前端的显示, 这样就足够了.

## 内网穿透

如果还需要让它更像 SaaS 一些, 比如让它跟第三方 API 对接, 那么你需要配置内网穿透, 让它能接受来自公网的请求.

大家常用的工具是 [ngrok](https://ngrok.com/), 国内的替代产品是 [natapp](https://natapp.cn/) .

这里以 natapp 为例, `icbd.natapp1.cc` 是他为我分配的公网域名.

![内网穿透](/images/natapp.png)

因为本地 GDK 中的 Rails 默认工作在 3000 端口, 我们还需要在 natapp 的 Dashboard 配置 `本地端口`:

![natapp-dashboard](/images/natapp-dashboard.png)

接下来是 gitlab 的配置:

> config/gitlab.yml

```yml
production: &base
  #
  # 1. GitLab app settings
  # ==========================

  ## GitLab settings
  gitlab:
    ## Fill the value of: Gitlab.config.gitlab.url
    url: http://icbd.natapp1.cc
    ## Web server settings (note: host is the FQDN, do not include http://)
    host: icbd.natapp1.cc
    port: 3000
    https: false

    relative_url_root: ""
```

注意, 我们并没有在 `gdk.yml` 中修改 `hostname`.

这里修改 `url` 的目的是预先填充 `Gitlab.config.gitlab.url` 的值, 否则由 Settings 生成的 url 会携带额外的端口信息, 从而影响 `com?` 方法的判断. 修改 `url` 跟上面 mock `com?` 是等效的, 可以二选一.

> config/initializers/1_settings.rb

```ruby
Settings.gitlab['url'] ||= Settings.__send__(:build_gitlab_url)
```

然后修改 `com_url` 中的地址:

```diff
diff --git a/jh/lib/jh/gitlab/saas.rb b/jh/lib/jh/gitlab/saas.rb
index 793405c1916..603138b5b7c 100644
--- a/jh/lib/jh/gitlab/saas.rb
+++ b/jh/lib/jh/gitlab/saas.rb
@@ -10,7 +10,8 @@ module Saas
 
         override :com_url
         def com_url
-          'https://jihulab.com'
+          # 'https://jihulab.com'
+          'http://icbd.natapp1.cc'
         end
 
         override :staging_com_url
```

## 其他配置

以 root 用户登录, 来到页面 `/admin/application_settings/general#account-settings`,
勾选 `Allow use of licensed EE features`.

> Licensed Enterprise Edition features can be used if the project namespace's plan includes the feature, or if the project is public.
