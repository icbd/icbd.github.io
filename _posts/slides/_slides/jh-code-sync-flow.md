![](/images/slides/jh-code-sync-flow/pre-main-jh-code-sync-flow.png)

[POS file](/images/slides/jh-code-sync-flow/code-sync-flow.pos)

---

## 现在的方案有什么问题?

---

## 现在的方案有什么问题?

直接从 master 合并代码进 main-jh:

- main-jh 上存在未经验证的来自上游的新修改;
- 上游的新修改经常破坏 JH 的 Pipeline;
- JH 正常的 Feature MR 被阻塞;

---

## 几个备选方案

---


### 备选方案: 以 MR 合并

- 创建从 master 到 main-jh 的 MR,
- 触发分支上的 Pipeline,
- 为该 MR 提交 depended MR 来修复 Pipeline,
- 走 MR Approve 流程手动合并代码.

---

### 备选方案: pre-main-jh

- 把 master&main-jh 合并进 pre-main-jh,
- 触发分支上的 Pipeline,
- 在 pre-main-jh 上修复 Pipeline,
- 将验证过的的更新合并进 main-jh .

---

## 方案优劣

---

### 方案优劣: 以 MR 合并


<table>
<tr><th>优势</th><th>劣势</th></tr>
<tr><td>没有引入新概念</td><td>CodeSync MR 需要大量定制</td></tr>
<tr><td>Git Graph 跟现在保持一致</td><td>Fix pipeline 流程复杂</td></tr>
<tr><td>合并流程容易理解</td><td>自动合并需要额外开发</td></tr>
</table>

---

### 方案优劣:  pre-main-jh


<table>
<tr><th>优势</th><th>劣势</th></tr>
<tr><td>引入新的过度分支</td><td>很容易模拟 main-jh 的 Pipeline</td></tr>
<tr><td>Git Graph 发生变化</td><td>Fix pipeline 流程简单</td></tr>
<tr><td>合并流程略复杂</td><td>方便实现自动流程</td></tr>
</table>

---

## 最终选定: pre-main-jh

![](/images/slides/jh-code-sync-flow/pre-main-jh-code-sync-flow.png)

---

定时同步 master 到 pre-main-jh, 如果 Pipeline 通过则自动 merge

![](/images/slides/jh-code-sync-flow/s1.png)

---

如果 pre-main-jh 上的 Pipeline 失败, 则创建 Fix MR 来修复

![](/images/slides/jh-code-sync-flow/s2.png)

---

- 在 main-jh 上修复功能 MR 导致的 Pipeline 失败;
- 在 pre-main-jh 上修复跟 master 存在分叉导致的失败;

![](/images/slides/jh-code-sync-flow/s3.png)

---

- pre-main-jh 必须要包含 main-jh 的最新代码

![](/images/slides/jh-code-sync-flow/pre-main-jh-code-sync-flow.png)

---

## 总结

- master & main-jh 的更新带到 pre-main-jh 上验证;
- 分叉导致的失败在 pre-main-jh 上修复;
- JH 功能导致的失败在 main-jh 上修复;
- master 本身导致的失败在上游修复;
- 被验证过的代码才能合并入 main-jh.
