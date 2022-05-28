---
layout: post
title:  New Mac Essential Tools
date:   2021-04-29
Author: CBD
tags:   [Tools]
---

## Terminal  tools

```zsh
# https://github.com/Homebrew/brew
# 🍺 The missing package manager for macOS (or Linux)
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# https://github.com/asdf-vm/asdf
# Extendable version manager with support for Ruby, Node.js, Elixir, Erlang & more
brew install coreutils curl git
brew install asdf
echo -e "\n. $(brew --prefix asdf)/asdf.sh" >> ${ZDOTDIR:-~}/.zshrc

# https://github.com/extrawurst/gitui
# Blazing 💥 fast terminal-ui for git written in rust 🦀
brew install gitui

# https://direnv.net/
# direnv is an extension for your shell.
brew install direnv

```

## Extensions

[https://plugins.jetbrains.com/plugin/8183-gitlink](https://plugins.jetbrains.com/plugin/8183-gitlink)

## App

- [ ] Chrome
- [ ] Weixin
- [ ] Slack
- [ ] iterm2
- [ ] Visual Studio Code
- [ ] RubyMine
- [ ] GoLand
- [ ] CopyClip 2
- [ ] 欧路词典
- [ ] ClashX
- [ ] Sip
- [ ] 印象笔记
- [ ] GitHub Desktop
- [ ] Docker
- [ ] Cyberduck
- [ ] Rectangle
- [ ] Karabiner
- [ ] Navicat
- [ ] Charles
- [ ] IINA
- [ ] direnv
