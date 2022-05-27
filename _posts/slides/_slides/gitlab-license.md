## GitLab License & Features 101

[@icbd](https://gitlab.com/icbd)

---

## Price

![price](/images/slides/gitlab-license/price.png)

---

## License Info

* licensee
* plan
* active_user_count
* starts_at
* expires_at

---

## Activate license

1. SaaS
1. Self-managed

---

## Activate license for self-managed without network

1. decrypt license info
1. can't reveal secret key

---

## OpenSSL::PKey::RSA

```ruby
  # rsa.public_encrypt(string)          => String
  # rsa.public_encrypt(string, padding) => String
  # 
  # Encrypt _string_ with the public key.  _padding_ defaults to PKCS1_PADDING.
  # The encrypted string output can be decrypted using #private_decrypt.
  def public_encrypt(*several_variants)
    # This is a stub, used for indexing
  end

  # rsa.private_decrypt(string)          => String
  # rsa.private_decrypt(string, padding) => String
  # 
  # Decrypt _string_, which has been encrypted with the public key, with the
  # private key.  _padding_ defaults to PKCS1_PADDING.
  def private_decrypt(*several_variants)
    # This is a stub, used for indexing
  end

  # rsa.private_encrypt(string)          => String
  # rsa.private_encrypt(string, padding) => String
  # 
  # Encrypt _string_ with the private key.  _padding_ defaults to PKCS1_PADDING.
  # The encrypted string output can be decrypted using #public_decrypt.
  def private_encrypt(*several_variants)
    # This is a stub, used for indexing
  end

  # rsa.public_decrypt(string)          => String
  # rsa.public_decrypt(string, padding) => String
  # 
  # Decrypt _string_, which has been encrypted with the private key, with the
  # public key.  _padding_ defaults to PKCS1_PADDING.
  def public_decrypt(*several_variants)
    # This is a stub, used for indexing
  end
```

---

## Asymmetric cryptographic algorithm

![encrypt-decrypt](https://upload.wikimedia.org/wikipedia/commons/f/f9/Public_key_encryption.svg)

---

## Asymmetric cryptographic algorithm

![sign-verify](https://upload.wikimedia.org/wikipedia/commons/7/78/Private_key_signing.svg)

---

## TLS Handshake (RSA)

![ssl handshake rsa](https://blog.cloudflare.com/content/images/2014/Sep/ssl_handshake_rsa.jpg)

---

## Bob Alice Duality

<img src="https://upload.wikimedia.org/wikipedia/commons/f/f9/Public_key_encryption.svg">
<img src="https://upload.wikimedia.org/wikipedia/commons/7/78/Private_key_signing.svg">

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

## RSA & AES

```ruby
# gitlab/license/encryptor.rb

  attr_accessor :key

  def initialize(key)
    raise KeyError, 'No RSA encryption key provided.' if key && !key.is_a?(OpenSSL::PKey::RSA)

    @key = key
  end

  def encrypt(data)
    raise KeyError, 'Provided key is not a private key.' unless key.private?

    # Encrypt the data using symmetric AES encryption.
    cipher = OpenSSL::Cipher::AES128.new(:CBC)
    cipher.encrypt
    aes_key = cipher.random_key
    aes_iv  = cipher.random_iv

    encrypted_data = cipher.update(data) + cipher.final

    # Encrypt the AES key using asymmetric RSA encryption.
    encrypted_key = key.private_encrypt(aes_key)

    encryption_data = {
      'data' => Base64.encode64(encrypted_data),
      'key' => Base64.encode64(encrypted_key),
      'iv' => Base64.encode64(aes_iv)
    }

    json_data = JSON.dump(encryption_data)
    Base64.encode64(json_data)
  end
```

---

## Code & Demo

[4dc413a6f1a391e73a3eb322aa3cf6d0b3091225](https://jihulab.com/gitlab-cn/gitlab/-/commit/4dc413a6f1a391e73a3eb322aa3cf6d0b3091225)

---

## License & GitlabSubscriptions::Features

- [ee/app/models/license.rb#feature_available?](https://jihulab.com/gitlab-cn/gitlab/-/blob/main-jh/ee/app/models/license.rb#L70)
- [ee/app/models/gitlab_subscriptions/features.rb](https://jihulab.com/gitlab-cn/gitlab/-/blob/main-jh/ee/app/models/gitlab_subscriptions/features.rb)
- [jh/app/models/jh/gitlab_subscriptions/features.rb](https://jihulab.com/gitlab-cn/gitlab/-/blob/main-jh/jh/app/models/jh/gitlab_subscriptions/features.rb)
- [ee/app/models/ee/project.rb#licensed_feature_available?](https://jihulab.com/gitlab-cn/gitlab/-/blob/main-jh/ee/app/models/ee/project.rb#L809)
- [ee/app/models/ee/namespace.rb#licensed_feature_available?](https://jihulab.com/gitlab-cn/gitlab/-/blob/main-jh/ee/app/models/ee/namespace.rb#L170)
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
