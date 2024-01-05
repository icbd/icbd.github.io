---
layout: post
title:  Rack override HTML form method
date:   2024-01-05
Author: CBD
tags:   [Ruby, HTML, Rack]
---

根据 HTML 规范，HTML form 的 method 只接受: `get` `post` `dialog`.

method 属性不能为空, 非法值被认为是 `get`.

[https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#attr-fs-method](https://html.spec.whatwg.org/multipage/form-control-infrastructure.html#attr-fs-method)

在 Rails 的 Controller 中还是能正常处理 RESTFul 的 post/patch/put/delete 方法, 这是因为 Rack 帮我们做了一些处理.

如果当前是 POST 请求, 并且包含 form data,
那么 Rack 就尝试获取 `_method` 字段的值,
将其转为大写字母后赋值给 `env[REQUEST_METHOD]`.

允许通过 `_method` 重写的 HTTP method 包括: `GET HEAD PUT POST DELETE OPTIONS PATCH LINK UNLINK`, 其他方法会保留原样(跳过处理).

> [Rack::MethodOverride](https://github.com/rack/rack/blob/3897649e8740e560a5fa142f972121a119b26b5c/lib/rack/method_override.rb)

```ruby
# frozen_string_literal: true

require_relative 'constants'
require_relative 'request'
require_relative 'utils'

module Rack
  class MethodOverride
    HTTP_METHODS = %w[GET HEAD PUT POST DELETE OPTIONS PATCH LINK UNLINK]

    METHOD_OVERRIDE_PARAM_KEY = "_method"
    HTTP_METHOD_OVERRIDE_HEADER = "HTTP_X_HTTP_METHOD_OVERRIDE"
    ALLOWED_METHODS = %w[POST]

    def initialize(app)
      @app = app
    end

    def call(env)
      if allowed_methods.include?(env[REQUEST_METHOD])
        method = method_override(env)
        if HTTP_METHODS.include?(method)
          env[RACK_METHODOVERRIDE_ORIGINAL_METHOD] = env[REQUEST_METHOD]
          env[REQUEST_METHOD] = method
        end
      end

      @app.call(env)
    end

    def method_override(env)
      req = Request.new(env)
      method = method_override_param(req) ||
        env[HTTP_METHOD_OVERRIDE_HEADER]
      begin
        method.to_s.upcase
      rescue ArgumentError
        env[RACK_ERRORS].puts "Invalid string for method"
      end
    end

    private

    def allowed_methods
      ALLOWED_METHODS
    end

    def method_override_param(req)
      req.POST[METHOD_OVERRIDE_PARAM_KEY] if req.form_data? || req.parseable_data?
    rescue Utils::InvalidParameterError, Utils::ParameterTypeError, QueryParser::ParamsTooDeepError
      req.get_header(RACK_ERRORS).puts "Invalid or incomplete POST params"
    rescue EOFError
      req.get_header(RACK_ERRORS).puts "Bad request content body"
    end
  end
end
```
