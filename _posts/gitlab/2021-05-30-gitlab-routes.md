---
layout: post
title:  GitLab Routes
date:   2021-05-30
Author: CBD
tags:   [GitLab]
---

## 路由基础回顾

[https://guides.rubyonrails.org/routing.html](https://guides.rubyonrails.org/routing.html)

类别 | 定义 | Path | Controller
---|---|---|---
复数资源 | `resources :users` | GET `/users/:id` | `UsersController#show`
单数资源 | `resource  :user` | GET `/user` | `UsersController#show`
嵌套资源 | `resources :teachers { resources :users }` | POST `/teachers/:tid/users` | `UsersController#create`
嵌套命名空间 | `namespace :admin { resources :users }` | GET `/admin/users/:id` | `Admin::UsersController#show`
定制模块前缀 | `scope module: 'admin' { resources :users }` | GET `/users/:id` | `Admin::UsersController#show`
定制路径前缀 | `scope '/admin' { resources :users }` | GET `/admin/users/:id` | `UsersController#show`

* Controller 总是复数的
* 不要嵌套两个以上的资源
* 不要嵌套 member action

```ruby
r = Rails.application.routes
r.recognize_path "/users/123"
```

## GitLab 路由


  