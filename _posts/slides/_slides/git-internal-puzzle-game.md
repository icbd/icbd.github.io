# Git Internal Puzzle Game

---

## How to learn Git

![progit2](https://git-scm.com/images/progit2.png)

Git is a version control system.

Git is a file object database.

---

<table>
<tr><th>Database</th><th>Git</th></tr>
<tr><td>C</td><td>Create Git Objects</td></tr>
<tr><td>R</td><td>Fetch Git Objects / Index</td></tr>
<tr><td>U</td><td>Update branch / tag</td></tr>
<tr><td>D</td><td>Git Garbage Collection</td></tr>
</table>

---

## `.git`

```shell
git init repo
tree .git
```

```
 ./
 └── .git/
    ├── branches/
    ├── config
    ├── description
    ├── HEAD
    ├── hooks/
    │  ├── applypatch-msg.sample*
    │  ├── commit-msg.sample*
    │  ├── fsmonitor-watchman.sample*
    │  ├── post-update.sample*
    │  ├── pre-applypatch.sample*
    │  ├── pre-commit.sample*
    │  ├── pre-merge-commit.sample*
    │  ├── pre-push.sample*
    │  ├── pre-rebase.sample*
    │  ├── pre-receive.sample*
    │  ├── prepare-commit-msg.sample*
    │  ├── push-to-checkout.sample*
    │  ├── sendemail-validate.sample*
    │  └── update.sample*
    ├── info/
    │  └── exclude
    ├── objects/
    │  ├── info/
    │  └── pack/
    └── refs/
       ├── heads/
       └── tags/
```

---

## First commit

```shell
git commit --allow-empty -m "Init"

alias inflate="ruby -r zlib -e \"ARGV.each {|path| puts Zlib::Inflate.inflate File.read(path) }\""
inflate <object-path> | hexdump -C
```

---

## Git is not magic

```shell
rgit --help

```

https://icbd.github.io/git-cat-file-and-object-frame/
https://github.com/icbd/ruby-on-git/blob/master/lib/ruby_on_git/object/commit.rb

---

```shell
cat .git/objects/pack/pack-*.pack | git unpack-objects
```


---

## Real Game: https://learngitbranching.js.org

> An interactive git visualization and tutorial.

> Aspiring students of git can use this app to educate and challenge themselves towards mastery of git!

---



---

## Reference

- https://git-scm.com/book/zh/v2
- https://git-merge.com/
- [Derrick Stolee GitHub blogs](https://github.blog/author/dstolee/)
- https://shop.jcoglan.com/building-git/
- https://shafiul.github.io/gitbook/
- https://marklodato.github.io/visual-git-guide/index-zh-cn.html
