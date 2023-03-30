[如何书写高质量的 MR / 如何做优秀的 Code Review](https://gitlab.cn/handbook/engineering/code-review/how-to-write-great-mr-and-code-review)

---

## 缘起

Code Review 对于 MR 来说，不仅是一个工作流程，还是团队成员为了达到共同目标而一起努力的过程。

---

来自 GitLab Inc. 团队的一句话做了很好的总结：

> go through a code review process to ensure the code is effective, understandable, maintainable, and secure.

---

- **Effective**

  “有效的”要求 MR 是正确的，能解决问题的，并且也需要在性能上过关。 这是 MR 的底线，达不到这一点的 MR，Reviewer 是一定不可以通过的。

---

- **Understandable**

  “可理解的”是在“有效的”基础上的要求。我们希望刚进入团队的新人也可以很容易读懂过往的代码。
  Reviewer 需要换位思考，假设自己不了解背景的情况下，是否也能理解 MR 的改动。

---

- **Maintainable**

  “可维护的”的代码最能体现贡献者的编码水平。
  这里隐含了一些要求，代码的组织方式是否符合约定，方法的颗粒度如何，是否可重用，代码是否容易扩展，是否容易测试，异常处理是否合适等等。
  这需要 Reviewer 发挥自己的经验和智慧，引导贡献者向着更好的方向改进。

---

- **Secure**

  “安全的”要求贡献者保持良好的编码习惯，对可能暴露风险的场景添加测试。
  Reviewer 需要了解常见的漏洞和攻击手段，为贡献者做好最后一道把关。

---

## 创建 MR

创建 MR 有非常多种方式，我们并没有要求统一从议题或者从分支创建。好的 MR 一般有以下特点：

---

- 尽可能使用英语

  因为我们经常需要把 JiHu 的需求或者问题跟 Upstream 讨论，使用英语会大大提高跟 Upstream 的沟通效率。

---

- 清晰的 MR 标题

  标题应该简明扼要地概括出 MR 的改动。我们希望从标题中读出，这是一个新增功能、Bug 修复还是功能重构。

  请务必以大写字母开头，字符数控制在 3 到 72 之间，更多规则请注意 `danger-review` 的提示。

  在准备阶段使用 `Draft:` 标记，准备完成后去除该标记再添加 Reviewer。

---

- 详尽的 MR 描述

  请使用 MR **模板**，并根据模板填写描述的每个章节。

---

  好的功能 MR 应该包含以下描述信息：

    - JiHu Only?
    - SaaS / Self-Managed
    - 付费级别：Free / Premium / Ultimate
    - 相关联的 Issue 链接
    - 需要依赖的 MR
    - 相关的 MR
    - 该 MR 在技术实现上的背景
    - 如何本地调试，是否需要配置 Feature Flag 或者环境变量（不要暴露 Token）
    - MR 的效果，如果涉及 UI 请添加截图或演示视频
---

  好的 Bug Fix MR 应该包含以下描述信息：

    - 如何复现 Bug
    - Bug 的现象和分析
    - Bug 所影响的版本和用户
    - 引入 Bug 的来源
    - 修复 Bug 的方法

---

- 恰当的标记信息

  及时分配 Milestone 对追溯代码非常有好处，能方便地推断最早引入该 MR 的 release 是什么。

  添加充分的 `label`， 对复盘分析非常有用。
  比如通过 `fix:pipeline` 可以过滤出所有修复 Pipeline 的 MR，然后对它们进行分类聚类，指导我们制定的下一步优化方案。

  还有一些标记可以辅助调试，比如 `pipeline:run-in-ruby3` 和 `pipeline:run-all-jest`。

---

## 好的 MR 由好的提交组成

好的提交是有组织的。

每个提交都有明确的分工，能达到分别 review 的程度。多个提交之间能表达出修改的渐进和依赖关系。

单个 MR 的提交数目不宜过多，建议不超过 10 个，强制要求不超过 20 个。

越大的 MR 越难 Review，Review 的周期会变长，引入 Bug 的可能性会加大。
单个 MR 的修改文件数不宜过多，建议不超过 10 个。对于超过 20 个文件变更的 MR，需要确认这样做的合理性和必要性。

创建 MR 时勾选 `squash` 选项，尽可能保证主分支提交的干净整洁。

---

[提交信息编写指南](https://docs.gitlab.com/ee/development/contributing/merge_request_workflow.html#commit-messages-guidelines)
列举了一组提交信息规则，主要包含以下几点：

- 消息标题需要以大写字母开头
- 消息标题字符数在 3 到 72 之间
- 消息标题和消息体用一个空行分隔
- 消息体每行不超过 72 个字符
- 不能包含 Emoji
- 使用 Issue、Milestone、MR 的完整链接而不是简短引用

---

如果 MR 对用户的使用有影响，无论是新增去除或者修改功能，都应该书写 `Changelog`，
详见 [更新日志条目](https://docs.gitlab.cn/jh/development/changelog.html) 。

---

## MR Review 流程

我们有一篇文章专门介绍 [JiHu Code Review](https://gitlab.cn/handbook/engineering/code-review/) 流程，
这里要讲的是流程之外还有哪些做法能让 MR 更好。

---

- 讨论式的留言

  对于 Reviewer 来说，非常有把握的建议可以使用祈使句式，对于不了解或有疑问的修改，可以使用疑问句请求贡献者解答。
  在 Code Review 中，"不求甚解" 是个贬义词，"是什么" 和 "为什么" 的提问能帮我们更好地梳理逻辑。

  Reviewer 的提问会创建出一个 Thread，如果贡献者同意 Reviewer 的修改建议，需要在提交修改后 Thread 中标记本次修改的内容。
  对于没有争议的小问题，贡献者可以自行解决，其他情况一般由 Reviewer 确认后解决。

  如果贡献者不同意 Reviewer 的修改建议，需要明确当前方案的优势并展开讨论，直到跟 Reviewer 达成共识。
  如果当前方案跟 Reviewer 的修改建议并没有明显的优劣，同时双方也未达成共识，
  那么可以再邀请一位熟悉该逻辑的 Reviewer 帮忙裁决。

  若始终难以抉择，那么选择跟现有代码风格更相近的方案。

---

- 主动自我留言

  一般来说，贡献者比 Reviewer 更了解具体的逻辑和需求。
  在开始正式 Review 之前，推荐贡献者先进行一轮自我 Review，并对可能产生疑问的方法添加留言。
  这样 Reviewer 就相当于在阅读一份包含更多批注的代码，会加速 Review 的整体进程。

---

## 写好 MR 更多细节

### 好的方法名

方法名应该简洁、清晰明了，能够准确描述方法的功能和用途。
一个好的方法名能够提高代码的可读性和可维护性，也能够帮助其他开发者更好地理解你的代码。

名词，使用专有名词，减少通用词汇产生的误解。

---

在 Controller 或 Helper 中，通常需要定义某种动作和过程，可以使用动宾短语，比如：

```ruby
before_action :check_password_expiration, if: :html_request?
```

---

如果该过程会产生副作用，包括修改对象的数据，抛异常等，都应该使用叹号方法 `!` 命名, 比如：

```ruby
before_action :authenticate_user!, only: [:new, :create]
```

---

在 Model 中需要面向对象的高内聚的命名，一般以名词、形容词来呈现。

比如：

```ruby
def preferred_language
end

def is_admin?
end
```

---

得益于 Ruby 的灵活性，我们可以把对象内的变量和函数都视为对象的属性，以方法的形式来包装，既阅读通顺又灵活易改，比如：

```ruby
def access_level
end

def access_level=(new_level)
end
```

---

### 好的代码组织

Ruby on Rails 项目是典型的 MVC 结构，
又极力推崇约定优于配置的 [Rails 信条](https://ruby-china.org/wiki/the-rails-doctrine)，
在代码组织方面有比较明确的业界共识。

Ruby 这样的解释型语言，更有必要遵循约定的代码组织方式，
这样让人在不执行 Ruby 解释器的情况下，也可以方便并准确地找到方法的定义。

---

#### M Model

模型是对象的定义和描述。

模型不一定有对应的数据库表，能抽象成对象的都应该定义为模型。

---

#### V View

视图负责展示。

模板文件中存放样式，不应该包含复杂逻辑，如果需要逻辑判断，则应该提取到 Helper 中。

---

#### C Controller

控制器是内外沟通的桥梁。

它负责参数的安全过滤和流程的衔接，不应该包含复杂逻辑。

复杂逻辑应该在模型中实现，若果逻辑跨越多个模型，则可以创建更高抽象的模型。

多个控制器的公共部分，可以通过 concern 来重用。

---

### 好的单元测试

好的代码让单元测试的编写更加容易，如果方法有合适的颗粒度，测试会显得更加有层次，方便控制变量，也更加容易读懂。
所以可以通过单元测试来反向评判逻辑代码的好坏。

RSpec 以可读性好表达能力强著称，我们也希望充分发挥它的优势。
具体来说，用 `context` 描述测试的上下文，应该包含正例和反例，用 `it` 描述具体的测试结果，它应该是一句通顺的自然语言（特别注意英语语法的时态）。

核心的功能很可能会与其他的功能有交叉，使用 `shared_examples` 能帮助重用测试用例。
