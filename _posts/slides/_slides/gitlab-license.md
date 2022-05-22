# GitLab License 101

[@icbd](https://gitlab.com/icbd)

---

## Price

![price](/images/slides/gitlab-license/price.png)

---

## Activate license

1. validity of license
1. license details

---

## Activate license for self-managed without network

1. validity of license ( sign & verify )
1. license details ( encrypt & decrypt )

---

## Asymmetric cryptographic algorithm

![encrypt-decrypt](https://upload.wikimedia.org/wikipedia/commons/f/f9/Public_key_encryption.svg)

- Bob: GitLab LicenseDot 
- Alice: A GitLab instance installed by users in their intranet

---

## Asymmetric cryptographic algorithm

![sign-verify](https://upload.wikimedia.org/wikipedia/commons/7/78/Private_key_signing.svg)

- Bob: GitLab LicenseDot
- Alice: A GitLab instance installed by users in their intranet

---

## Gem

```sh
cd gitlab
bundle install
gem list --details gitlab-license
```

```text
*** LOCAL GEMS ***

gitlab-license (2.1.0, 2.0.0)
    Authors: Douwe Maan, Stan Hu, Tyler Amos
    Homepage: https://dev.gitlab.org/gitlab/gitlab-license
    License: MIT
    Installed at (2.1.0): /Users/cbd/.asdf/installs/ruby/2.7.4/lib/ruby/gems/2.7.0
                 (2.0.0): /Users/cbd/.asdf/installs/ruby/2.7.4/lib/ruby/gems/2.7.0

    gitlab-license helps you generate, verify and enforce software
    licenses.

gitlab-license_finder (6.14.2.1)
    Authors: Ryan Collins, Daniil Kouznetsov, Andy Shen, Shane
    Lattanzio, Li Sheng Tai, Vlad vassilovski, Jacob Maine, Matthew Kane
    Parker, Ian Lesperance, David Edwards, Paul Meskers, Brent Wheeldon,
    Trevor John, David Tengdin, William Ramsey, David Dening, Geoff
    Pleiss, Mike Chinigo, Mike Dalessio, Jeff Jun
    Homepage: https://github.com/pivotal/LicenseFinder
    License: MIT
    Installed at: /Users/cbd/.asdf/installs/ruby/2.7.4/lib/ruby/gems/2.7.0

    Audit the OSS licenses of your application's dependencies.
```

```sh
gem open gitlab-license
```

---

## Code & Demo

[ba026ab3](https://jihulab.com/gitlab-cn/gitlab/-/commit/ba026ab3224f1b4811c719dec6ac88137512c11f)

---

## License & GitlabSubscriptions::Features

- [ee/app/models/license.rb](https://jihulab.com/gitlab-cn/gitlab/-/blob/main-jh/ee/app/models/license.rb)
- [ee/app/models/gitlab_subscriptions/features.rb](https://jihulab.com/gitlab-cn/gitlab/-/blob/main-jh/ee/app/models/gitlab_subscriptions/features.rb)
- [jh/app/models/jh/gitlab_subscriptions/features.rb](https://jihulab.com/gitlab-cn/gitlab/-/blob/main-jh/jh/app/models/jh/gitlab_subscriptions/features.rb)
- [ee/spec/support/helpers/ee/license_helpers.rb](https://jihulab.com/gitlab-cn/gitlab/-/blob/main-jh/ee/spec/support/helpers/ee/license_helpers.rb)

[MR: Allows JH to independently control the feature license level](https://jihulab.com/gitlab-cn/gitlab/-/merge_requests/381/diffs)

---

## Tips

```ruby
License.feature_available?(:sast) # check feature in current plan

Gitlab::CurrentSettings.should_check_namespace_plan? # SaaS only

project.licensed_feature_available?(:sast)
namespace.licensed_feature_available?(:sast)

stub_licensed_features(sast: true) # in test
```

---

## More

- [CustomersDot](https://jihulab.com/gitlab-cn/internal/customers-jihulab-com)
- [LicenseDot](https://jihulab.com/gitlab-cn/internal/license-gitlab-cn)
