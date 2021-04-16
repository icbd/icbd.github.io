---
layout: post
title:  Package Script and Iterperter by Chef Omnibus
Author: CBD
tags:   [Omnibus, GitLab]
---

This is a demo project to show how to package Ruby script and Ruby interpreter with [`chef/Omnibus`](https://github.com/chef/omnibus).

## Development env

Development env is our local env, I'm on MacOS.

1. Preparation

    We need to prepare Ruby ( `>=2.6` ) in the development environment, and then

    ```sh
    gem install bundler:2.2.3
    gem install omnibus
    ```

2. Generate Omnibus code template

    ```sh
    omnibus new permutations
    cd omnibus-permutations/
    ```

3. Setup dependency and package scripts

    * Setup project info in `config/projects/`
    * Add dependency to `config/software/`
    * Setup package scripts in `package-scripts/`
    * Commit code and add a version tag

    More details: [https://gitlab.com/icbd/omnibus-permutations.git](https://gitlab.com/icbd/omnibus-permutations.git)

## Building env

Omnibus determines the platform for which to build an installer based on the platform it is currently running on.

So, we'll run the build task on a target platform.

In this demo, we use `CentOS-8`, so also need to install `rpmbuild` by:

```sh
dnf install -y rpm-build
```

And we also need Git and Ruby ( `>=2.6` ) on Building env.

```sh
git clone https://gitlab.com/icbd/omnibus-permutations.git
git tag
cd omnibus-permutations

bundle install --binstubs
./bin/omnibus build permutations --log-level debug
```

After packing, we'll get RPM file in `pkg/`:

```text
pkg/
├── permutations-1.0.4+20210415005454-1.el8.x86_64.rpm
├── permutations-1.0.4+20210415005454-1.el8.x86_64.rpm.metadata.json
└── version-manifest.json
```

## Test env

Let's download that RPM to locale Desktop, and install it on a new clean `CentOS-8` VM by Docker.

```sh
docker pull centos:8
docker run -v ~/Desktop/pkg/permutations-1.0.4+20210415005454-1.el8.x86_64.rpm:/install-pkg.rpm -it centos:8 bash
```

```sh
rpm -i install-pkg.rpm
source /etc/profile
permutations
```

Output:

```text
"ruby 2.7.3p183 (2021-04-05 revision 6847ee089d) [x86_64-linux]\n"
[["A", "B", "C"], ["A", "C", "B"], ["B", "A", "C"], ["B", "C", "A"], ["C", "A", "B"], ["C", "B", "A"]]
```
