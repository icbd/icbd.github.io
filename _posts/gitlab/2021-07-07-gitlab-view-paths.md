---
layout: post
title:  GitLab view paths
date:   2021-07-07
Author: CBD
tags:   [GitLab, Rails, view_paths]
---

我们使用这个脚手架创建一个示例项目: `rails g scaffold User name:string score:integer` .

添加这么两个 partial:

* `lib/views/users/_footer.html.erb`
* `app/views/users/_footer.html.erb`

我们模仿一下 GitLab 的需求, 对于大部分通用功能, 写在 `app/views/` 中. 对于 EE 版本的特殊功能, 写在一个独立目录中, 比如这里的 `lib/views/`.

而且希望先查找特殊目录的 views 文件, 如果不存在时再去 `app/views` 中查找.

最简单的办法是在 Controller 里添加 `prepend_view_path Rails.root.join("lib/views")`. 

如果需要在所有 Controller 里都应用这个效果, 可以这样配置:

```ruby
  class Application < Rails::Application
    config.load_defaults 6.1

    # ...

    config.paths['app/views'].unshift "#{config.root}/lib/views"
  end
```

Rails 在启动的时候会检查 `paths["app/views"]`, 如果我们这样配置它, 就自动把这个路径应该到所有 Controller 上:

[https://github.com/rails/rails/blob/7578e6f141e3c24fc6a1208e189eb1958d0b304f/railties/lib/rails/engine.rb#L618-L624](https://github.com/rails/rails/blob/7578e6f141e3c24fc6a1208e189eb1958d0b304f/railties/lib/rails/engine.rb#L618-L624)

```ruby
    initializer :add_view_paths do
      views = paths["app/views"].existent
      unless views.empty?
        ActiveSupport.on_load(:action_controller) { prepend_view_path(views) if respond_to?(:prepend_view_path) }
        ActiveSupport.on_load(:action_mailer) { prepend_view_path(views) }
      end
    end
```

```ruby
# File actionview/lib/action_view/view_paths.rb, line 122
def prepend_view_path(path)
  lookup_context.view_paths.unshift(*path)
end
```

通过 `lookup_context` 我们可以知道更多关于 view path 的细节.

比如像这样查询 partial 是否存在:

```ruby
  def partial_exists?(partial)
    lookup_context.exists?(partial, [], true)
  end
```

`lookup_context` 是一个对象, 它包含了查找 view 所需的所有信息.

## GitLab 是如何做的

存在这样的 helper method 用于条件渲染, 如果存在的话渲染给定的 partial, 不存在就跳过:

```ruby
def render_if_exists(partial = nil, **options)
    return unless partial_exists?(partial || options[:partial])

    if partial.nil?
      render(**options)
    else
      render(partial, options)
    end
  end

  def partial_exists?(partial)
    lookup_context.exists?(partial, [], true)
  end
```

存在这样的 helper method 用于给 EE 覆盖/扩展 CE 的 view:

```ruby
def render_ce(partial, locals = {})
      render template: find_ce_template(partial), locals: locals
    end

    # Tries to find a matching partial first, if there is none, we try to find a matching view
    # rubocop: disable CodeReuse/ActiveRecord
    def find_ce_template(name)
      prefixes = [] # So don't create extra [] garbage

      if ce_lookup_context.exists?(name, prefixes, true)
        ce_lookup_context.find(name, prefixes, true)
      else
        ce_lookup_context.find(name, prefixes, false)
      end
    end
    # rubocop: enable CodeReuse/ActiveRecord

    def ce_lookup_context
      @ce_lookup_context ||= begin
        ce_view_paths = lookup_context.view_paths.paths.reject do |resolver|
          resolver.to_path.start_with?("#{Rails.root}/ee")
        end

        ActionView::LookupContext.new(ce_view_paths)
      end
    end
```

关键在于 `lookup_context.view_paths.paths.reject`, 把 ee 的 path 从 `view_paths` 中去除, 以避免循环渲染导致的死循环.
