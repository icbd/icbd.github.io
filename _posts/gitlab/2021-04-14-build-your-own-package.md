---
layout: post
title:  Build your own GitLab package
Author: CBD
tags:   [Omnibus, GitLab]
---

In this article, we will introduce the principles and steps of manual package building.

## Tools

1. [Chef/Omnibus](https://github.com/chef/omnibus)

    > Easily create full-stack installers for your project across a variety of platforms.

    Read more: [https://icbd.github.io/omnibus-demo/](https://icbd.github.io/omnibus-demo/)

    Demo project: [https://gitlab.com/icbd/omnibus-permutations.git](https://gitlab.com/icbd/omnibus-permutations.git)

2. [gitlab-org/Omnibus](https://gitlab.com/gitlab-org/omnibus/)

    > GitLab fork of the omnibus project. Required for faster responses to issues with omnibus-gitlab.

    [Merge Request Demo ( Feature Branch Flow )](https://gitlab.com/gitlab-org/omnibus/-/merge_requests/21)

3. [gitlab-org/omnibus-gitlab](https://gitlab.com/gitlab-org/omnibus-gitlab)

    > This project creates full-stack platform-specific downloadable packages for GitLab.

    [Merge Request Demo ( Master Branch Flow )](https://gitlab.com/gitlab-org/omnibus-gitlab/-/merge_requests/5154)

## Steps

Let's show how to build a package for `CentOS-8` .

According to `Chef/Omnibus` introduction, we know that the build script should be executed on `CentOS-8` environment,
and we use Docker to provide an `CentOS-8` environment.

```sh
docker pull registry.gitlab.com/gitlab-org/gitlab-omnibus-builder/centos_8
git clone https://gitlab.com/gitlab-org/omnibus-gitlab.git ~/omnibus-gitlab
cd ~/omnibus-gitlab
docker run -v ~/omnibus-gitlab:/omnibus-gitlab -it registry.gitlab.com/gitlab-org/gitlab-omnibus-builder/centos_8 bash
```

```bash
# in docker container

export ALTERNATIVE_SOURCES=true
export COMPILE_ASSETS=true

cd /omnibus-gitlab
bundle install
bundle binstubs --all

# The next rake  requires the git work area to be clean.
git checkout -b compile-assets-jh
git add .
git commit -m "Update Gemfile.lock"

bundle exec rake build:project
```

The packages will be built and available in the `~/omnibus-gitlab/pkg` directory.

## Details

* ALTERNATIVE_SOURCES

    ```yaml
    # .custom_sources.yml

    gitlab-rails:
    remote: "git@dev.gitlab.org:gitlab/gitlabhq.git"
    alternative: "https://gitlab.com/gitlab-org/gitlab-foss.git"
    gitaly:
    remote: "git@dev.gitlab.org:gitlab/gitaly"
    alternative: "https://gitlab.com/gitlab-org/gitaly"
    # ...
    ```

    In file `.custom_sources.yml`, the repo URL of each GitLab component is listed.

    By default, `remote` is used, if the flag `ALTERNATIVE_SOURCES=true` is specified, then the build will use `alternative`.

    You can also use `GITLAB_ALTERNATIVE_REPO` to specify your own repo.

    Read more: [https://docs.gitlab.com/13.10/omnibus/build/team_member_docs.html](https://docs.gitlab.com/13.10/omnibus/build/team_member_docs.html#i-want-to-use-a-specific-mirror-or-fork-of-various-gitlab-components-in-my-build)

* COMPILE_ASSETS

    Compiling frontend resources is a time-consuming step.

    And the frontend resources are cross-platform, which means that their results can be shared between various Linux release.

    Usually, this frontend compilation process is completed in the CI of the [GitLab Rails project](https://gitlab.com/gitlab-org/gitlab/).

    1. GitLab Rails CI compile frontend resources and push compilation result;
    2. Pull compilation result and use `docker -v` to mount it;
    3. Set `ASSET_PATH` env variable in building environment.

    Read more: [https://docs.gitlab.com/13.10/omnibus/build/build_package.html#fetch-upstream-assets](https://docs.gitlab.com/13.10/omnibus/build/build_package.html#fetch-upstream-assets)
