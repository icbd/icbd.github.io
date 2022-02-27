---
layout: post
title:  Git internal objects
date:   2022-02-16
Author: CBD
tags:   [Git]
---

## 示例项目

[https://jihulab.com/show/git_objects_demo](https://jihulab.com/show/git_objects_demo)

该项目共包含一个 commit, 包含的文件结构如下:

```text
.
├── doc
│   └── index.md
├── readme.md
└── wiki
    ├── help.md
    └── info.md

```

git objects 目录下, 共 8 个文件:

> `find .git/objects -type f`

```text
.git/objects/69/15089af93052dc6f67fd7245037230a6b1894b
.git/objects/67/89c99cbe4a25ecc9e69eb1e66df819ddfab2e5
.git/objects/d0/38440f5c27d52fed6b6c5eb7d180f1645ab724
.git/objects/f2/0b9ed4f8ba4e99df6a2491bbed85e8daf526d5
.git/objects/20/61bac5e1a292b380f98072ef5ffbc2fc3eefa9
.git/objects/11/ef4924f8f1e21e6d7412040973f5e868559ac7
.git/objects/16/796efecb4599c92244ac8bafb217e20009008e
.git/objects/53/33c1763419c89486091d80b6ba430d30b321a0
```

<table>
  <thead>
    <tr>
      <th>Hash</th>
      <th>Type</th>
      <th>Content</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>6915089af93052dc6f67fd7245037230a6b1894b</td>
      <td>commit</td>
      <td> 
<pre>
tree 6789c99cbe4a25ecc9e69eb1e66df819ddfab2e5
author Baodong <wwwicbd@gmail.com> 1645015448 +0800
committer Baodong <wwwicbd@gmail.com> 1645015448 +0800

Init
</pre>
      </td>
    </tr>
    <tr>
      <td>6789c99cbe4a25ecc9e69eb1e66df819ddfab2e5</td>
      <td>tree</td>
      <td>
<pre>
040000 tree d038440f5c27d52fed6b6c5eb7d180f1645ab724	doc
100644 blob 5333c1763419c89486091d80b6ba430d30b321a0	readme.md
040000 tree 11ef4924f8f1e21e6d7412040973f5e868559ac7	wiki
</pre>
      </td>
    </tr>
    <tr>
      <td>d038440f5c27d52fed6b6c5eb7d180f1645ab724</td>
      <td>tree</td>
      <td>
<pre>
100644 blob 2061bac5e1a292b380f98072ef5ffbc2fc3eefa9	index.md
</pre>
      </td>
    </tr>
    <tr>
      <td>f20b9ed4f8ba4e99df6a2491bbed85e8daf526d5</td>
      <td>blob</td>
      <td>
<pre>
# Info
</pre>
      </td>
    </tr>
    <tr>
      <td>2061bac5e1a292b380f98072ef5ffbc2fc3eefa9</td>
      <td>blob</td>
      <td>
<pre>
# Index

</pre>
      </td>
    </tr>
    <tr>
      <td>11ef4924f8f1e21e6d7412040973f5e868559ac7</td>
      <td>tree</td>
      <td>
<pre>
100644 blob 16796efecb4599c92244ac8bafb217e20009008e	help.md
100644 blob f20b9ed4f8ba4e99df6a2491bbed85e8daf526d5	info.md
</pre>
      </td>
    </tr>
    <tr>
      <td>16796efecb4599c92244ac8bafb217e20009008e</td>
      <td>blob</td>
      <td>
<pre>
# Help

</pre>
      </td>
    </tr>
    <tr>
      <td>5333c1763419c89486091d80b6ba430d30b321a0</td>
      <td>blob</td>
      <td>
<pre>
# Readme

```text
.
├── doc
│   └── index.md
├── readme.md
└── wiki
    ├── help.md
    └── info.md
```
</pre>
      </td>
    </tr>
  </tbody>
</table>

上面 8 个 Hash 中, 

有 1 个 `commit` 类型的, 对应 repo 中唯一的 commit;

有 3 个 `tree` 类型的, 对应 repo 中 3 个目录: 根目录 / doc 目录 / wiki 目录.

有 4 个 `blob` 类型的, 对应 repo 中 4 个 md 文件.

## 文件修改

在上面的 repo 的基础上, 对 readme 新加一行, 并提交到了新的 repo:

[https://jihulab.com/show/git_objects_demo_2](https://jihulab.com/show/git_objects_demo_2)

此时, objects 新增了三个 Hash:

```text
.git/objects/92/60de33334266d9ab1e8c8333ccc0898865748f
.git/objects/b3/c84f0cfa3b38c0bd9f8c1713536dc144077199
.git/objects/cb/e1f2b84138ece7363749e9a31193a62104262d
```

<table>
  <thead>
    <tr>
      <th>Hash</th>
      <th>Type</th>
      <th>Content</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>9260de33334266d9ab1e8c8333ccc0898865748f</td>
      <td>commit</td>
      <td> 
<pre>
tree cbe1f2b84138ece7363749e9a31193a62104262d
parent 6915089af93052dc6f67fd7245037230a6b1894b
author Baodong <wwwicbd@gmail.com> 1645018274 +0800
committer Baodong <wwwicbd@gmail.com> 1645018274 +0800

Add blog link
</pre>
      </td>
    </tr>

        <tr>
      <td>b3c84f0cfa3b38c0bd9f8c1713536dc144077199</td>
      <td>blob</td>
      <td> 
<pre>
# Readme

[https://icbd.github.io/git-internal-objects/](https://icbd.github.io/git-internal-objects/)

```text
.
├── doc
│   └── index.md
├── readme.md
└── wiki
    ├── help.md
    └── info.md
```
</pre>
      </td>
    </tr>

        <tr>
      <td>cbe1f2b84138ece7363749e9a31193a62104262d</td>
      <td>tree</td>
      <td> 
<pre>
040000 tree d038440f5c27d52fed6b6c5eb7d180f1645ab724	doc
100644 blob b3c84f0cfa3b38c0bd9f8c1713536dc144077199	readme.md
040000 tree 11ef4924f8f1e21e6d7412040973f5e868559ac7	wiki
</pre>
      </td>
    </tr>
  </tbody>
</table>


该 repo 共包含 11 个 git objects, 相比于之前新增了 3 个,

有 1 个 `commit` 类型的, 对应新增的 commit;

有 1 个 `tree` 类型的, 对应被修改的 `readme.md` 所在的目录 (目录下有文件发送变化, 目录本身的 tree 也发送变化);

有 1 个 `blob` 类型的, 对应被修改的 `readme.md` 文件 (虽然只修改了其中一行, blob 还是保存了完整的文件内容).
