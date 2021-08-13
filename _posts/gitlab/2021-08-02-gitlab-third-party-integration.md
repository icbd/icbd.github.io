---
layout: post
title:  GitLab third-party integration
Author: CBD
tags:   [GitLab]
---

以集成禅道为例, 展示 GitLab 第三方集成所需的步骤.

## 配置部分

禅道的配置是 project 级别, 每个 GitLab 的 project maintainer 有权限配置该项目的禅道集成.

routes:

|Page|URL|
|---|---|
|Project|[http://localhost:3000/dev/zentao1](http://localhost:3000/dev/zentao1)|
|Integration Settings|[http://localhost:3000/dev/zentao1/-/settings/integrations](http://localhost:3000/dev/zentao1/-/settings/integrations)|
|Edit|[http://localhost:3000/dev/zentao1/-/services/zentao/edit](http://localhost:3000/dev/zentao1/-/services/zentao/edit)|

> services 或者 service_id 等含有 `service` 字样的问题, 源自于早起的 `service` 命名风格. \
[Epic: Rename "Services" to "Integrations"](https://gitlab.com/groups/gitlab-org/-/epics/2504) \
跨越多个 milestone, 从 `12.8` 到 `14.1`, 目前还在持续进行.

1. create_table :zentao_tracker_data

     1.1 `$ rails g migration create_zentao_tracker_data`

    ```ruby
    def change
      create_table :zentao_tracker_data do |t|
        # t.references :service, ...
        t.references :integration, foreign_key: { on_delete: :cascade }, type: :integer, index: true, null: false
        t.timestamps_with_timezone

        # ...

        t.text :encrypted_api_token
        t.text :encrypted_api_token_iv
      end
    end
    ```

    注意点:
    * 需要 `foreign_key`, 有 job 检查;
    * 推荐使用 `text` 而不是 `string`, 避免长度限制.

    1.2 `$ rails db:migrate` :

    * 往数据库的 `schema_migrations` 表中写入最新的迁移时间戳;
    * 往目录中提交时间戳及对应的 hash: `/db/schema_migrations/20210728101742` ;
    * 更新 `db/structure.sql`

    三个文件都需要提交.

2. Create Model: `Integrations::ZentaoTrackerData`

    2.1 加密保存敏感字段

    ```ruby
    include BaseDataFields

    attr_encrypted :api_token, encryption_options
    ```

    2.2 添加 relation to `HasDataFields`

    ( `Integration` include `HasDataFields` )

    ```ruby
    has_one :zentao_tracker_data, autosave: true, inverse_of: :integration, 
            foreign_key: :integration_id, class_name: 'Integrations::ZentaoTrackerData'
    ```

3. 扩展 Params permit 参数

    新参数的 key 加入: `Integrations::Params` 中的 `ALLOWED_PARAMS_CE` .

4. 让 `Integration` 识别到 zentao

    对于项目级别的集成, 把 `zentao` 添加到 `PROJECT_SPECIFIC_INTEGRATION_NAMES`, 影响的接口:

    * `available_integration_names`
    * `project_specific_integration_names`

5. 为单表继承添加新的类

    `Gitlab::Integrations::StiType#namespaced_integrations`

6. 实现 model: `Integrations::Zentao`

    6.1 Add relation with `Project`

    ```ruby
    has_one :zentao_integration, class_name: 'Integrations::Zentao'
    ```

    6.2 `class Zentao < Integration`

    需要实现的接口:
    * data_fields
    * title
    * description
    * self.to_param
    * self.supported_events
    * self.supported_event_actions
    * test (Auth 测试)
    * fields (显示在配置页面的字段,类型和提示信息)

7. 更新 GraphQL 文档

    `$ rake gitlab:graphql:check_docs`

    `doc/api/graphql/reference/index.md`

8. 更新 `gitlab.pot`

    `$ rake gettext:regenerate`

    `locale/gitlab.pot`

9. 更新 `API::Helpers::IntegrationsHelpers`

    `API::Services < ::API::Base`

    * integrations
    * integration_classes

10. 