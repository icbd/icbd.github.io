---
layout: post
title:  Deploy GitLab instance
date:   2021-04-12
Author: CBD
tags:   [GitLab]
---

## Story background

[https://about.gitlab.com/handbook/engineering/infrastructure/production/architecture/supporting-architecture.html](https://about.gitlab.com/handbook/engineering/infrastructure/production/architecture/supporting-architecture.html)

## Installer Scipt

0. Setup environment variables

1. Install GitLab instance, latest EE version

2. Setup SMTP sender

3. Setup Oauth2: Goole Workspace and GitLab.com

```bash
#!/bin/bash

# Base on Centos 8.2

export EXTERNAL_URL=<MASK>

cat << VARS >> /etc/profile

#-- GitLab Instance
export EXTERNAL_URL="$EXTERNAL_URL"
export REGISTRY_EXTERNAL_URL="$EXTERNAL_URL:5050"

export GOOGLE_OAUTH2_APP_ID=<MASK>
export GOOGLE_OAUTH2_APP_SECRET=<MASK>

export GITLABCOM_OAUTH2_APP_ID=<MASK>
export GITLABCOM_OAUTH2_APP_SECRET=<MASK>

export GITLAB_EMAIL_FROM=<MASK>
export GITLAB_SMTP_DOMAIN=<MASK>
export GITLAB_SMTP_PASSWORD=<MASK>
#-- GitLab Instance
VARS
source /etc/profile


dnf install -y curl policycoreutils openssh-server perl postfix
systemctl enable sshd postfix
systemctl start sshd postfix


curl https://packages.gitlab.com/install/repositories/gitlab/gitlab-ee/script.rpm.sh | sudo bash
dnf install -y gitlab-ee


cat << CONF >> /etc/gitlab/gitlab.rb

registry_external_url "$REGISTRY_EXTERNAL_URL"

#-- Alicloud SMTP
gitlab_rails['gitlab_email_from'] = ENV["GITLAB_EMAIL_FROM"]
gitlab_rails['smtp_enable'] = true
gitlab_rails['smtp_address'] = "smtpdm.aliyun.com"
gitlab_rails['smtp_port'] = 80
gitlab_rails['smtp_user_name'] = ENV["GITLAB_EMAIL_FROM"]
gitlab_rails['smtp_password'] = ENV["GITLAB_SMTP_PASSWORD"]
gitlab_rails['smtp_domain'] = ENV["GITLAB_SMTP_DOMAIN"]
gitlab_rails['smtp_authentication'] = "login"
#-- Alicloud SMTP

#-- OAuth2
gitlab_rails['omniauth_enabled'] = true
gitlab_rails['omniauth_allow_single_sign_on'] = ['google_oauth2']
gitlab_rails['omniauth_sync_email_from_provider'] = 'google_oauth2'
gitlab_rails['omniauth_sync_profile_from_provider'] = ['google_oauth2']
gitlab_rails['omniauth_sync_profile_attributes'] = ['name', 'email', 'location']
# gitlab_rails['omniauth_auto_sign_in_with_provider'] = 'google_oauth2'
gitlab_rails['omniauth_block_auto_created_users'] = false
gitlab_rails['omniauth_auto_link_user'] = ['google_oauth2']
gitlab_rails['omniauth_allow_bypass_two_factor'] = ['google_oauth2']
gitlab_rails['omniauth_providers'] = [
    {
        "name" => "google_oauth2",
        "app_id" => ENV["GOOGLE_OAUTH2_APP_ID"],
        "app_secret" => ENV["GOOGLE_OAUTH2_APP_SECRET"],
        "args" => { "access_type" => "offline", "approval_prompt" => "" }
    },
    {
        "name" => "gitlab",
        "app_id" => ENV["GITLABCOM_OAUTH2_APP_ID"],
        "app_secret" => ENV["GITLABCOM_OAUTH2_APP_SECRET"],
        "args" => { "scope" => "api" }
    }
]
#-- OAuth2
CONF

gitlab-ctl reconfigure

```

### References

1. SMTP

    [https://docs.gitlab.com/omnibus/settings/smtp.html#aliyun-direct-mail](https://docs.gitlab.com/omnibus/settings/smtp.html#aliyun-direct-mail)

    ```shell
    $ gitlab-rails c
    ```

    ```ruby
    > Notify.test_email('<YOUR_TEST_EMAIL>', 'Hello World', 'This is a test message').deliver_now
    ```

2. OAuth

    [https://docs.gitlab.com/ee/integration/omniauth.html](https://docs.gitlab.com/ee/integration/omniauth.html)

    [https://docs.gitlab.com/ee/integration/google.html](https://docs.gitlab.com/ee/integration/google.html)

    [https://docs.gitlab.com/ee/integration/gitlab.html](https://docs.gitlab.com/ee/integration/gitlab.html)

    Integrating `Google OAuth` is to login to GitLab instance, and use mail domain to restrict login permissions.

    Integrating `gitlab.com` is to use facilitate the use of the repo on it, including `clone` and `mirror` .

    (Need to combine Web UI configuration.)

## Manual Steps

1. Reset root password and first login

2. Sign-up restrictions

[https://docs.gitlab.com/ee/user/admin_area/settings/sign_up_restrictions.html#disable-new-sign-ups](https://docs.gitlab.com/ee/user/admin_area/settings/sign_up_restrictions.html#disable-new-sign-ups)

[https://docs.gitlab.com/ee/user/admin_area/settings/sign_up_restrictions.html#allowlist-email-domains](https://docs.gitlab.com/ee/user/admin_area/settings/sign_up_restrictions.html#allowlist-email-domains)

[https://docs.gitlab.com/ee/user/admin_area/appearance.html](https://docs.gitlab.com/ee/user/admin_area/appearance.html)

**`Goto: Admin Area > Settings > General > Sign-up restrictions`**

* Uncheck `Sign-up enabled`
* Uncheck `Require admin approval for new sign-ups`
* Check `Enable email restrictions for sign ups`
* Fill `Allowed domains for sign-ups`

Then `Save changes` .

**`Goto: Admin Area > Settings > General > Sign-in restrictions`**

`Enabled OAuth sign-in sources`

* Uncheck `GitLab.com`

Then `Save changes` .

**`Goto: Admin Area > Appearance`**

* Upload `Header logo`
* Fill `Sign in/Sign up pages`

Then `Update appearance settings` .

![dev-ops-login-page](/images/dev-ops-login-page.png)
