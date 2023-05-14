# How to add an AI feature to GitLab

---

## Debug in GDK

[Example: Generate test in MR](https://gitlab.com/gitlab-org/gitlab/-/merge_requests/118365)

---

## OpenAI key

Client Gem: [ruby-openai](https://github.com/alexrudall/ruby-openai)

```ruby
# https://platform.openai.com/account/api-keys

Gitlab::CurrentSettings.update(openai_api_key: "<open-ai-key>")
```

---

## pgvector

https://github.com/pgvector/pgvector

```sh
gdk config set gitlab.rails.databases.embedding.enabled true
gdk config set pgvector.enabled true
gdk reconfigure

```

---

## Feature flags

General FF:

```ruby
Feature.enable(:ai_related_settings)
Feature.enable(:openai_experimentation)
```

UI FF (Optional):

```ruby
Feature.enable(:ai_assist_ui)
Feature.enable(:ai_assist_flag)

```

Special FF:

```ruby
Feature.enable(:generate_test_file_flag)
```

---

## Namespace Settings

[Ref MR: Add namespace settings around AI features](https://gitlab.com/gitlab-org/gitlab/-/merge_requests/118222)

- Ultimate Group
- Public Project
- Top Group: Settings > General > Permissions and group features

- ✅ Use Experiment features
- ✅ Use third-party AI services

```ruby
group = Group.find 126
group.update(experiment_features_enabled: true, third_party_ai_features_enabled: true)
```

---

![Flow](/images/slides/add-new-ai-feature-to-gitlab/abstraction-layer-flow.png)

---

## Mutations::Ai:Action

> ee/app/graphql/mutations/ai/action.rb

POST `/api/graphql`

Requst:

```json
[
  { "operationName": "generateTestFile",
    "variables": {
      "resourceId": "gid://gitlab/MergeRequest/78",
      "filePath": "app/helpers/posts_helper.rb"
    },
    "query": "
      mutation generateTestFile($resourceId: AiModelID!, $filePath: String!) {
        aiAction(
          input: {generateTestFile: {resourceId: $resourceId, filePath: $filePath}, markupFormat: HTML}
        ) {
          errors
          __typename
        }
      }
    "
  }
]
```

Return the response immediately:

```json
[
  {
    "data":{
      "aiAction":{
        "errors":[],
        "__typename":"AiActionPayload"
      }
    }
  }
]
```

---

Mutations perform async job through `Llm::ExecuteMethodService`:

```ruby
params = [
  1, # user.id
  78, # resource.id
  "MergeRequest", # resource.class.name
  "generate_test_file", # action_name
  { # options
    "markup_format" => "html",
    "file_path" => "app/models/post.rb",
    "request_id" => "5f7163bc-4409-479c-be14-8605f4731837"
  }
]

Llm::CompletionWorker.new.perform *params

```

```log
% tree -P '*input_type.rb' ee/app/graphql/types/ai/
ee/app/graphql/types/ai/
├── base_method_input_type.rb
├── explain_code_input_type.rb
├── explain_vulnerability_input_type.rb
├── generate_description_input_type.rb
├── generate_test_file_input_type.rb
├── summarize_comments_input_type.rb
└── tanuki_bot_input_type.rb

```

---

## CompletionWorker

TODO

---

## How to implement a new action

- Register a new method in `ExecuteMethodService` class
- Create new Service in folder `ee/app/services/llm/`
- Authorization with FF and `send_to_ai?`
- Add new provider client in folder `ee/lib/gitlab/llm/`
- Add GraphQL components
- Add new GraphQL Completion in folder `ee/lib/gitlab/llm/open_ai/completions/`
- Add new Prompt template in folder `ee/lib/gitlab/llm/open_ai/templates/`

---

## Other Details

- `ExponentialBackoff` and `CircuitBreaker` ( error with `Maximum number of retries`)
- todo

---

## Reference

- https://about.gitlab.com/blog/2023/05/03/gitlab-ai-assisted-features
- https://about.gitlab.com/blog/2023/04/27/merge-request-suggest-a-test
- https://gitlab.com/gitlab-org/gitlab/-/blob/master/doc/development/ai_features.md
- https://gitlab.com/gitlab-org/gitlab/-/merge_requests/118365
- https://github.com/alexrudall/ruby-openai
- https://gitlab.com/gitlab-org/gitlab/-/merge_requests/118222
