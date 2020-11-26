---
layout: post
title:  A Review of Rails Transaction Processing
date:   2020-11-26
Author: CBD
tags: [Rails, transaction]
---

解决这个事务相关的问题走了些弯路, 特此小结.

需求是给某个 Model 写钩子方法, 让他在创建/更新/删除之后, 通知一个特殊的 Chewy 对象, 创建/更新/删除 Elasticsearch 的索引.

## globalize

项目基于 Rails 4.2 , 用 [globalize 5.1](https://github.com/globalize/globalize) 做国际化.

业务里有一个叫 `Region` 的 Model 类, 根据 globalize 的约定, 他有一个翻译表 `region_translations`, 翻译表没有对应的类.

globalize 会自动为 `Region` 添加一个 `Translation` 类, 即 `Region::Translation`.

详见: [https://github.com/globalize/globalize/blob/master/lib/globalize/active_record/class_methods.rb#L51-L66](https://github.com/globalize/globalize/blob/master/lib/globalize/active_record/class_methods.rb#L51-L66)

我们可以通过 `Region.has_associated_audits` 方便地查看 Region 的关联关系:

> `Region.has_associated_audits["translations"]`

```text
=> #<ActiveRecord::Reflection::HasManyReflection:0x00007fc0b09666d0
 @active_record=Region(id: integer, region_type: integer, parent_id: integer, created_at: datetime, updated_at: datetime, abbr: string, position: integer, search_count: integer),
 @association_scope_cache={},
 @automatic_inverse_of=false,
 @constructable=true,
 @foreign_type="translations_type",
 @klass=nil,
 @name=:translations,
 @options={:class_name=>"Region::Translation", :foreign_key=>"region_id", :dependent=>:destroy, :extend=>Globalize::ActiveRecord::HasManyExtensions, :autosave=>true, :inverse_of=>:globalized_model},
 @plural_name="translations",
 @scope=nil,
 @scope_lock=#<Thread::Mutex:0x00007fc0b09664a0>,
 @type=nil>
```

重点是多对一关系中的 `:dependent=>:destroy`, 当 Region 的对象 destroy 的时候, 会级联地删除 Region::Translation 对应的对象.

## before_destroy

[https://api.rubyonrails.org/classes/ActiveRecord/Callbacks.html](https://api.rubyonrails.org/classes/ActiveRecord/Callbacks.html)

重点看 `#Ordering callbacks` 的章节, 文档给的例子正是这里遇到的情况!

默认情况下, destroy `Region` 对象, 会先级联删除翻译, 再进入 `before_destroy` 回调方法. 此时的回调方法正处于一个事务中, 级联删除的 SQL 已经执行但没有提交.

需求是删除 `Region` 以及翻译的同时, 删除他们对应的 Elasticsearch 索引, 但是现在已经读不到对应的翻译了. 正如上面文档所说, 正确的方法是:

```ruby
before_destroy :callback_meth, prepend: true
```

`prepend` 会改变回调的顺序, 先触发回调方法, 此时还没进入事务, 当然就可以读取翻译对象.

## 弯路

弯路之一是修改数据库查询的隔离级别:

```ruby
Region.transaction(isolation: :read_uncommitted) do
  # some query
end
```

遗憾的是, 在没使用 `prepend` 的时候, 我们已经处在一个事务内部了, 没法在事务中修改隔离级别, 会直接抛异常.

更错的路是新开一个线程, 在新线程内新开一个事务, 利用可重复读查询到结果, 利用 ConditionVariable 在线程之间通知唤醒. 这听起来就是一条歧路...

## 小结

Sample data:

```ruby
sh = Region.create!(region_type: :municipality,translations_attributes: [{ locale: 'zh-CN', name: '上海市' },{ locale: 'en', name: 'Shanghai' }])
```

Callback Method:

```ruby
  after_commit :sync_search_tags, on: [:create, :update]
  before_destroy :sync_search_tags, prepend: true
  def sync_search_tags
     destroyed = ActiveRecord::Base.connection.open_transactions > 0
    items = translations.map do |tr|
      index_id = "#{self.class_name.snakecase}:#{id}:#{tr.locale}"
      SearchTagsIndex::SearchTag.new(region_type, tr.name, tr.locale, {}, id: index_id, destroyed: destroyed)
    end
    SearchTagsIndex.type('default').import(items)
  end
```

`ActiveRecord::Base.connection.open_transactions`, 只要他大于 0 就说明他正处于一个事务之中, 在这个场景下, 他处于 destroy 的事务中.

`SearchTagsIndex` 是项目中的一个 Chewy 类, 靠一个普通 Ruby 类来管理索引.

通常一个 Chewy 类会跟一个 Model 类对应, Chewy 提供了现成的钩子方法, 当 Model 的对象修改后触发索引修改.

[https://github.com/toptal/chewy#index-definition](https://github.com/toptal/chewy#index-definition)

`id` 和 `destroyed` 是我这次加的:

* 指定 id 为了实现索引更新的幂等, id 缺失时 Elasticsearch 会自动填充 id (否则多次导入会产生重复数据);
* 如果 chewy 读到 `destroyed?` 为 true, 会删除对应的索引 (之前没有索引删除功能).

[https://github.com/toptal/chewy/blob/f9c02dddbcc4fc836f73330d8db331145c1ff012/lib/chewy/type/adapter/object.rb#L62-L66](https://github.com/toptal/chewy/blob/f9c02dddbcc4fc836f73330d8db331145c1ff012/lib/chewy/type/adapter/object.rb#L62-L66)