# JiHu GitLab MR Code Owners Rules

https://jihulab.com/gitlab-cn/gitlab/-/issues/2791

---

![issue-from](/images/slides/mr-code-owners/mr-diff-1528.png)

---

![issue-from](/images/slides/mr-code-owners/qa-diff-1587.png)

---

https://gitlab.com/gitlab-org/gitlab/-/merge_requests/117020/diffs

```yaml
# .gitlab/CODEOWNERS

[JH Frontend] @jihulab/maintainers/frontend
  /jh/app/assets/
  /jh/**/*.scss
  /jh/**/*.js
  /jh/**/*.vue

  [JH Rails Backend] @jihulab/maintainers/rails-backend
  /jh/**/*.rb
  /jh/**/*.rake
  /jh/qa/ @jihulab/maintainers/quality

  [JH Technical Writer] @jihulab/maintainers/technical-writer
  /jh/doc/

```

---

```md
https://jihulab.com/jihulab/maintainers/frontend

@qk44077907
@jeremywu
@lxwan
@orozot

https://jihulab.com/jihulab/maintainers/rails-backend

@zhanglinjie
@lyb124553153
@icbd
@luzhiyuan.deer
@songhuang
@zhzhang

https://jihulab.com/jihulab/maintainers/quality

@TomHeng
@leiqi_jihulab

https://jihulab.com/jihulab/maintainers/technical-writer

@zhaoqi
@shuangzhang
```

---

## Setup

- Project with Premium Feature
- Add rules to `CODEOWNERS` file
- Create groups and assign roles (`Developer` at least)
- Invite groups to the project
- Protected branches: Turn on the `Require approval from code owners` toggle
- Configure `All eligible users` approval rule

---

![Groups as Code Owners](https://jihulab.com/gitlab-cn/gitlab/uploads/533d2925f84727e6ca52cf564e2a6a60/%E6%88%AA%E5%B1%8F2023-04-04_19.40.27.png)

---

## Ref Doc

- https://docs.gitlab.com/ee/user/project/codeowners/index.html
- https://docs.gitlab.com/ee/user/project/protected_branches.html#require-code-owner-approval-on-a-protected-branch
- https://docs.gitlab.com/ee/user/project/merge_requests/approvals/rules.html#code-owners-as-eligible-approvers
