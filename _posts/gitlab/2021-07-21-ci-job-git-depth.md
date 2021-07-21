---
layout: post
title:  GitLab CI Job GIT_DEPTH
Author: CBD
tags:   [GitLab, CIJobWiki]
---

## Failed Message

https://gitlab.com/gitlab-jh/gitlab/-/jobs/1437530056

```text
$ bundle exec danger --fail-on-errors=true --verbose
fatal: couldn't find remote ref refs/heads/danger_base
fatal: couldn't find remote ref refs/heads/danger_head
fatal: couldn't find remote ref refs/heads/danger_base
fatal: couldn't find remote ref refs/heads/danger_head
fatal: couldn't find remote ref refs/heads/danger_base
fatal: couldn't find remote ref refs/heads/danger_head
fatal: couldn't find remote ref refs/heads/danger_base
fatal: couldn't find remote ref refs/heads/danger_head
fatal: protocol error: bad pack header
bundler: failed to load command: danger (/builds/gitlab-org-forks/gitlab/vendor/ruby/2.7.0/bin/danger)
RuntimeError: Cannot find a merge base between danger_base and danger_head. If you are using shallow clone/fetch, try increasing the --depth
  /builds/gitlab-org-forks/gitlab/vendor/ruby/2.7.0/gems/danger-8.3.1/lib/danger/scm_source/git_repo.rb:138:in `find_merge_base'
  /builds/gitlab-org-forks/gitlab/vendor/ruby/2.7.0/gems/danger-8.3.1/lib/danger/scm_source/git_repo.rb:22:in `diff_for_folder'
  /builds/gitlab-org-forks/gitlab/vendor/ruby/2.7.0/gems/danger-8.3.1/lib/danger/danger_core/dangerfile.rb:274:in `setup_for_running'
  /builds/gitlab-org-forks/gitlab/vendor/ruby/2.7.0/gems/danger-8.3.1/lib/danger/danger_core/dangerfile.rb:284:in `run'
  /builds/gitlab-org-forks/gitlab/vendor/ruby/2.7.0/gems/danger-8.3.1/lib/danger/danger_core/executor.rb:29:in `run'
  /builds/gitlab-org-forks/gitlab/vendor/ruby/2.7.0/gems/danger-8.3.1/lib/danger/commands/runner.rb:73:in `run'
  /builds/gitlab-org-forks/gitlab/vendor/ruby/2.7.0/gems/claide-1.0.3/lib/claide/command.rb:334:in `run'
  /builds/gitlab-org-forks/gitlab/vendor/ruby/2.7.0/gems/danger-8.3.1/bin/danger:5:in `<top (required)>'
  /builds/gitlab-org-forks/gitlab/vendor/ruby/2.7.0/bin/danger:23:in `load'
  /builds/gitlab-org-forks/gitlab/vendor/ruby/2.7.0/bin/danger:23:in `<top (required)>'
```

## Why

[GitLab and GitLab Runner perform a shallow clone by default.](https://docs.gitlab.com/ee/ci/large_repositories/index.html#shallow-cloning)

> Ideally, you should always use GIT_DEPTH with a small number like 10. <br/>  
> This instructs GitLab Runner to perform shallow clones.

<br/>

[Limit the number of changes fetched during clone](https://docs.gitlab.com/ee/ci/pipelines/settings.html#limit-the-number-of-changes-fetched-during-clone)

> Under Git strategy, under Git shallow clone, enter a value. The maximum value is 1000. <br/>  
> To disable shallow clone and make GitLab CI/CD fetch all branches and tags each   time, keep the value empty or set to 0. <br/>  
> In GitLab 12.0 and later, newly created projects automatically have a default git depth value of 50. <br/>  
> This value can be overridden by the `GIT_DEPTH` variable in the `.gitlab-ci.yml` file. <br/>  

<br/>

```yml
# .gitlab-ci.yml

variables:
  RAILS_ENV: "test"
  NODE_ENV: "test"
  BUNDLE_WITHOUT: "production:development"
  BUNDLE_INSTALL_FLAGS: "--jobs=$(nproc) --retry=3 --quiet"
  NODE_OPTIONS: --max_old_space_size=3584
  GIT_DEPTH: "20"

```

## Source Code

```ruby
# app/presenters/ci/build_runner_presenter.rb

def git_depth
  if git_depth_variable
    git_depth_variable[:value]
  else
    project.ci_default_git_depth
  end.to_i
end

def git_depth_variable
  strong_memoize(:git_depth_variable) do
    variables&.find { |variable| variable[:key] == 'GIT_DEPTH' }
  end
end
```

```ruby
# app/services/ci/register_job_service.rb

# one step of execute method
def present_build!(build)
  # We need to use the presenter here because Gitaly calls in the presenter
  # may fail, and we need to ensure the response has been generated.
  presented_build = ::Ci::BuildRunnerPresenter.new(build)
  build_json = ::API::Entities::Ci::JobRequest::Response.new(presented_build).to_json
  Result.new(build, build_json, true)
end

```

```ruby
# lib/api/ci/runner.rb

# trigger
::Ci::RegisterJobService.new(current_runner).execute(runner_params)
```

## Ref

https://gitlab.com/gitlab-org/gitlab/-/issues/14040

https://gitlab.com/gitlab-org/gitlab-foss/-/merge_requests/28919
