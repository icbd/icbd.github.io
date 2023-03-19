# Appreciation of DeclarativePolicy

---

## Introduction

official-repo

[gitlab.com/gitlab-org/ruby/gems/declarative-policy](https://gitlab.com/gitlab-org/ruby/gems/declarative-policy)

<br>

cbd-comment-repo

[gitlab.com/icbd-group/declarative-policy](https://gitlab.com/icbd-group/declarative-policy)

---

## Introduction

This library provides a DSL for writing authorization policies.

<br>

It can be used to separate logic from permissions, and has been
used at scale in production at GitLab.com.

<br>

The original author is Jeanine Adkisson, copyright is held by GitLab.

---

## Author

![Jeanine](/images/slides/appreciation-of-declarative-policy/jeanine.png)

---

## Pattern

`the user can drive the car?`

<br>

|Subject|Verb|Object|
|---|---|---|
| the user | own | the car |
| the user | too_young | <nil> |
| the car | can_work | <nil> |

<br>

How and where to define?

---

## Basic Usage

- [Debug by unit test](https://gitlab.com/icbd-group/declarative-policy/blob/doc/comments/spec/debug_spec.rb)
- [MergeRequestPolicy](https://gitlab.com/gitlab-org/gitlab/-/blob/master/app/policies/merge_request_policy.rb) < [IssuablePolicy](https://gitlab.com/gitlab-org/gitlab/-/blob/master/app/policies/issuable_policy.rb) < [BasePolicy](https://gitlab.com/gitlab-org/gitlab/-/blob/master/app/policies/base_policy.rb)
- delegate [ProjectPolicy](https://gitlab.com/gitlab-org/gitlab/-/blob/master/app/policies/project_policy.rb)
- built-in [DeclarativePolicy::Base](https://gitlab.com/icbd-group/declarative-policy/blob/doc/comments/lib/declarative_policy/base.rb#L373-377)

---

## Defining policies

`MergeRequest` => `MergeRequestPolicy`

<br>

```ruby

    # Place all policy definitions in `app/policies`.
    # Rails adds custom directories under `app` to the autoload paths automatically.


    policy = DeclarativePolicy.policy_for(current_user, merge_request)
    policy.allowed?(:approve_merge_request)
    
    
    DeclarativePolicy.configure do
      name_transformation { |name| "Policies::#{name}" }
    end

```

---

## Defining policies

`nil` => `NilClass`

<br>

```ruby

    policy = DeclarativePolicy.policy_for(nil, nil)
    policy.allowed?(:approve_merge_request) # always false

    # module DeclarativePolicy
    #   class NilPolicy < DeclarativePolicy::Base
    #     rule { default }.prevent_all
    #   end
    # end
    # 
    # 
    # module DeclarativePolicy
    #   class Base
    #     desc 'Unknown user'
    #     condition(:anonymous, scope: :user, score: 0) { @user.nil? }
    #
    #     desc 'By default'
    #     condition(:default, scope: :global, score: 0) { true }
    #   end
    # end
    # 
    # 
    # DeclarativePolicy.configure do
    #   nil_policy MyNilPolicy
    # end
    # 

```

---

## Defining policies

**Conditions** are facts about the state of the system.

```ruby

  desc 'Unknown user'
  condition(:anonymous, scope: :user, score: 0) { @user.nil? }
```

<br>

- conditions can be defined in any order;
- conditions have access to `@user` and `@subject`;
- conditions are evaluated at most once;
- conditions with low scores are prioritized (minimum 0);
- conditions with same scope are sharing the results;
- conditions are only used inside the policy;

---

## Defining policies

**Rules** are conclusions we can draw based on the facts.

```ruby

  # rule { own & ~too_young }.enable :drive
  rule { own }.enable :drive
  rule { too_young }.prevent :drive
  
  rule { can?(drive) }.policy do
    enable :travel_by_car
    prevent :drunk_driving
  end

```

<br>

- 
- 
- 
- 
- 

---

## DSL: `rule`

```ruby

  rule { own & ~too_young }.enable :drive
```

---

## DSL: `rule`

```ruby

    rule do
      obj = own & ~too_young
      obj # an instance of DeclarativePolicy::Rule::And
    end.enable(:drive)
```

---

## DSL: `rule`

```ruby

    rule do
      obj = cond(:own) & ~(cond(:too_young))
      obj
    end.enable :drive
```

---

## DSL: `rule`

```ruby

    rule do
      obj =
        cond(:own)
          .and(cond(:too_young).negate)
    
      obj
    end.enable :drive
```

---

## DSL: `rule`

```ruby

    rule_obj = rule do
      obj =
        DeclarativePolicy::Rule::Condition
          .new(:own)
          .and(
            DeclarativePolicy::Rule::Not.make(
              DeclarativePolicy::Rule::Condition.new(:too_young)
            )
          )
      obj
    end # DeclarativePolicy::PolicyDsl
    rule_obj.enable :drive
```

---

