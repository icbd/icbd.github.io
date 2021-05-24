---
layout: post
title:  GitLab EE与JH的加载
date:   2021-05-24
Author: CBD
tags:   [GitLab]
---

GitLab CE 版本涵盖了 EE 版本的大部分功能, 现在 CE 的实例也是通过 EE 版本的按照包来安装, CE 版本也可称作 `EE Free` .

如果希望使用 EE 的功能, 只需要导入 License 即可激活 EE, 而不需要重新部署实例.

GitLab 的 CE 和 EE 版共用同一个代码仓库, 由于 CE 会提供很大部分的基础功能, 所以默认的命名空间也就对应 CE 的功能.

(CE 仍然保留了独立的仓库, 但是只读)

JH 是 EE 版本的增强, 在 EE 的基础上进一步的定制和扩展.

以一个具体的 Module 为例, 看一下 JH => EE => CE 的关系:

```ruby
# app/helpers/version_check_helper.rb
# 代码稍有简化

module VersionCheckHelper
  def link_to_version
      link_to Gitlab::VERSION, source_host_url + namespace_project_tag_path(source_code_group, source_code_project, "v#{Gitlab::VERSION}")
    end
  end

  def source_host_url
    Gitlab::COM_URL
  end

  def source_code_group
    'gitlab-org'
  end

  def source_code_project
    'gitlab-foss'
  end
end

VersionCheckHelper.prepend_mod

```

|版本|项目地址|
|---|---|
| CE | https://gitlab.com/gitlab-org/gitlab-foss |
| EE | https://gitlab.com/gitlab-org/gitlab |
| JH | https://gitlab.cn/gitlab-jh/gitlab |

可见:

* EE 跟 CE 的区别是 项目名;
* JH 跟 CE 的区别是 域名, 组名, 项目名;
* JH 跟 EE 的区别是 域名, 组名;

```ruby
# ee/app/helpers/ee/version_check_helper.rb

module EE
  module VersionCheckHelper
    def source_code_project
      'gitlab'
    end
  end
end

```

```ruby
# jh/app/helpers/jh/version_check_helper.rb

module JH
  module VersionCheckHelper
    def source_host_url
      Gitlab::CN_URL
    end

    def source_code_group
      'gitlab-jh'
    end
  end
end

```

**CE 部分的代码包含了最基础的公共功能, CE 不依赖 EE 的代码就能正确执行;**

**EE 在 CE 的基础上, 覆盖或添加额外的功能, EE 不依赖 JH 的代码能正确运行;**

**JH 在 EE 的基础上, 覆盖或添加额外的功能.**

## 实现原理

1. `EE` `JH` 模块名

    我们看到 `EE` 对应的模块是 `module EE`, 这里两个 `E` 都是大写, 是由于 `acronym` 的作用.

    参见: `config/initializers_before_autoloader/000_inflections.rb`

2. autoload

    [https://guides.rubyonrails.org/autoloading_and_reloading_constants.html](https://guides.rubyonrails.org/autoloading_and_reloading_constants.html)

    [https://guides.rubyonrails.org/autoloading_and_reloading_constants_classic_mode.html](https://guides.rubyonrails.org/autoloading_and_reloading_constants_classic_mode.html)

    开发环境下, Rails 的模块是按需加载的. 在使用某个模块的时候, [根据预先约定好的命名规则](https://github.com/fxn/zeitwerk#file-structure), 把模块名转换为文件名, 然后去到这个文件中加载这个模块.

    所以开发环境下,  `autoload_paths` 目录下所有文件内的模块都不需要显示地 `require`, 可以直接调用.

    生产环境下, Rails 在的会启用 `Eager Loading`, 把以后需要用的模块预先加载起来.

    所以生产环境下, `eager_load_paths` 目录下所有文件内的模块在 server 启动的时候就已经加载过了, 可以直接调用.

    综上, 我们需要做的是把 EE 和 JH 特有的文件添加到 `autoload_paths` 和 `eager_load_paths`.

    GitLab 的 `config/application.rb` 做了这么几项工作:

    1. 使用 `require_dependency` 加载前置的配置文件;
    2. 把通用的扩展目录添加到 `eager_load_paths` 中, 此时的 `eager_load_paths` 已经满足 CE 的需求了, 记作 `foss_eager_load_paths`;
    3. 把 `foss_eager_load_paths` 分别应用跟 `ee` 拼接 (如果是 JH 版本再跟 `jh` 拼接), 然后把每个新目录加入到 `eager_load_paths`;
    4. 把 `eager_load_paths` 加入到 `autoload_paths`

    ```ruby
      pp ActiveSupport::Dependencies.autoload_paths
    ```

3. Override

    默认情况下, EE 和 JH 目录的代码不会对 CE 的代码有任何影响.

    如果在文件末尾追加了如 `VersionCheckHelper.prepend_mod`, 会加载 `EE::VersionCheckHelper` 并 prepend 到当前模块, 从而达到 override 方法的效果.

    如果是 JH , 还在再加载 `JH::VersionCheckHelper` 并 prepend, 从覆盖 EE 和 CE 的方法.

    ( 这里的覆盖是增量覆盖, 只覆盖同名的方法, EE 中未声明的方法不会被移除. )

    `prepend_mod` 的实现见 `config/initializers/0_inject_enterprise_edition_module.rb`.

    最关键的方法为:

    ```ruby
      def prepend_mod(with_descendants: false)
        prepend_mod_with(name, with_descendants: with_descendants) # rubocop: disable Cop/InjectEnterpriseEditionModule
      end

      def prepend_mod_with(constant_name, namespace: Object, with_descendants: false)
        each_extension_for(constant_name, namespace) do |constant|
          prepend_module(constant, with_descendants)
        end
      end

      private

      def each_extension_for(constant_name, namespace)
        Gitlab.extensions.each do |extension_name|
          extension_namespace =
            const_get_maybe_false(namespace, extension_name.upcase)

          extension_module =
            const_get_maybe_false(extension_namespace, constant_name)

          yield(extension_module) if extension_module
        end
      end
    ```

    当 `VersionCheckHelper.prepend_mod` 时, `VersionCheckHelper#name` 得到 `"VersionCheckHelper"` 字符串,
    `each_extension_for` 遍历 `Gitlab.extensions`, 拼装得到 `EE::VersionCheckHelper` 和 `JH::VersionCheckHelper`, 然后执行 prepend .

4. ActiveSupport::Concern

    如果需要扩展模块方法/类方法, 参考:

    ```ruby
    module JH
      module VersionCheck
        extend ActiveSupport::Concern

        class_methods do
          extend ::Gitlab::Utils::Override

          override :host

          def host
            ENV['VERSION_APP_HOST'] || 'https://version.gitlab.cn'
          end
        end
      end
    end
    ```

    这里多出的 `extend ::Gitlab::Utils::Override` 和 `override :host` 并没有什么作用, 除非开启 `STATIC_VERIFICATION` .

## 扩展阅读

[https://docs.gitlab.com/ee/development/ee_features.html](https://docs.gitlab.com/ee/development/ee_features.html)