---
layout: post
title:  GitLab CI Job GraphQL Check
date:   2021-07-22
Author: CBD
tags:   [GitLab, CIJobWiki]
---

## Failed Message

https://gitlab.com/gitlab-jh/gitlab/-/jobs/1426384442

```text
# 1 GraphQL query out of 441 failed validation:
# - /builds/gitlab-org-forks/gitlab/ee/app/assets/javascripts/security_configuration/dast_profiles/graphql/dast_failed_site_validations.query.graphql (known failure)
#
##########
$ bundle exec rake gitlab:graphql:check_docs
##########
#
# GraphQL documentation is outdated! Please update it by running `bundle exec rake gitlab:graphql:compile_docs`.
#
##########
```

## Why

```yml
# .gitlab/ci/graphql.gitlab-ci.yml

graphql-verify:
  variables:
    SETUP_DB: "false"
  extends:
    - .default-retry
    - .rails-cache
    - .default-before_script
    - .graphql:rules:graphql-verify
  stage: test
  needs: []
  script:
    - bundle exec rake gitlab:graphql:validate
    - bundle exec rake gitlab:graphql:check_docs

```

## Source Code

```ruby
desc 'GitLab | GraphQL | Generate GraphQL docs'
task compile_docs: [:environment, :enable_feature_flags] do
  renderer = Tooling::Graphql::Docs::Renderer.new(GitlabSchema, render_options)

  renderer.write

  puts "Documentation compiled."
end

desc 'GitLab | GraphQL | Check if GraphQL docs are up to date'
task check_docs: [:environment, :enable_feature_flags] do
  renderer = Tooling::Graphql::Docs::Renderer.new(GitlabSchema, render_options)

  doc = File.read(Rails.root.join(OUTPUT_DIR, 'index.md'))

  if doc == renderer.contents
    puts "GraphQL documentation is up to date"
  else
    format_output('GraphQL documentation is outdated! Please update it by running `bundle exec rake gitlab:graphql:compile_docs`.')
    abort
  end
end
```

Before, we need to manually execute this rake task: `gitlab:graphql:compile_docs`.

Since the Upstream  Repo will also update this markdown file, there is a risk of conflict.

We plan to add a new step to `code-sync` Job for automatic execution:

* Delete JH markdown file
* Execute the rake script to generate a latest markdown file
* Commit the new markdown file

## Background Knowledge about Code Sync

1. Step One

    `gitlab-jh/gitlab` mirror sync code from `gitlab-org/gitlab`.
    So, the master branch of JH will automatically keep in sync with the master branch of Upstream.

2. Step Two

    Synchronizing code for JH projects: [gitlab-jh/code-sync](https://gitlab.com/gitlab-jh/code-sync)

    ```yml
    # .gitlab-ci.yml

      script:
        - git clone --filter=tree:0 "${REPOSITORY}" "${CI_JOB_NAME}"
        - cd ${CI_JOB_NAME}
        - git config user.email "${USER_EMAIL}"
        - git config user.name "${USER_NAME}"
        - git checkout "${MERGE_INTO}"
        - git merge "origin/${MERGE_FROM}"
        - git push origin "${MERGE_INTO}"

    ```

    We set up a set of scheduled tasks to execute this script periodically.
