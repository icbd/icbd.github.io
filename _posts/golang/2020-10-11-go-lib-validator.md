---
layout: post
title:  Go Lib Validator
date:   2020-10-11
Author: CBD
tags: [Golang, lib]
---

validator 是一个验证器, 可以单独验证某个值, 也可以验证 struct 的字段. 验证器的原理并不复杂, 把参数值传入验证函数, 如果没有错误则验证通过. validator 会帮我们定义一系列常用的验证函数, 也支持自己注册验证函数. 可以对单个字段验证, 也可以对结构体的各个字段验证, 还支持嵌套验证. 验证函数可以一个或多个. 我们从最基本的变量验证开始, 看看 vlidator 是怎么组织的.

## 基本用法

```go
var validate = validator.New()

func main() {
	myEmail := "hi#gmail.com"
	if errs := validate.Var(myEmail, "required,email"); errs != nil {
		fmt.Println(errs) // Key: '' Error:Field validation for '' failed on the 'email' tag
	}
}
```

### New()

```go
// New returns a new instance of 'validate' with sane defaults.
func New() *Validate {

	tc := new(tagCache)
	tc.m.Store(make(map[string]*cTag))

	sc := new(structCache)
	sc.m.Store(make(map[reflect.Type]*cStruct))

	v := &Validate{
		tagName:     defaultTagName,
		aliases:     make(map[string]string, len(bakedInAliases)),
		validations: make(map[string]internalValidationFuncWrapper, len(bakedInValidators)),
		tagCache:    tc,
		structCache: sc,
	}

	// must copy alias validators for separate validations to be used in each validator instance
	for k, val := range bakedInAliases {
		v.RegisterAlias(k, val)
	}

	// must copy validators for separate validations to be used in each instance
	for k, val := range bakedInValidators {

		switch k {
		// these require that even if the value is nil that the validation should run, omitempty still overrides this behaviour
		case requiredIfTag, requiredUnlessTag, requiredWithTag, requiredWithAllTag, requiredWithoutTag, requiredWithoutAllTag:
			_ = v.registerValidation(k, wrapFunc(val), true, true)
		default:
			// no need to error check here, baked in will always be valid
			_ = v.registerValidation(k, wrapFunc(val), true, false)
		}
	}

	v.pool = &sync.Pool{
		New: func() interface{} {
			return &validate{
				v:        v,
				ns:       make([]byte, 0, 64),
				actualNs: make([]byte, 0, 64),
				misc:     make([]byte, 32),
			}
		},
	}

	return v
}
```

验证前需要解析字段的验证规则, 虽然验证是个高频操作, 但解析规则其实只需要一次, 为提高效率 validator 做了一层缓存, `validator.New()` 中会把预制好的验证函数加载到实例中.

其中, 利用 `atomic.Value` 实现了存取的原子性, 利用 `sync.Pool` 缓存了 validator 实例.

### ValidationFunc

```go
// BakedInValidators is the default map of ValidationFunc
	// you can add, remove or even replace items to suite your needs,
	// or even disregard and use your own map if so desired.
	bakedInValidators = map[string]Func{
		"required":             hasValue,
		"required_if":          requiredIf,
		"required_unless":      requiredUnless,
		"required_with":        requiredWith,
		"required_with_all":    requiredWithAll,
		"required_without":     requiredWithout,
		"required_without_all": requiredWithoutAll,
		"excluded_with":        excludedWith,
		"excluded_with_all":    excludedWithAll,
		"excluded_without":     excludedWithout,
		"excluded_without_all": excludedWithoutAll,
		"isdefault":            isDefault,
		"len":                  hasLengthOf,
		"min":                  hasMinOf,
		"max":                  hasMaxOf,
    "eq":                   isEq,
    // ...
  }
```

`bakedInValidators` 存储了预定义的验证名和验证函数的映射关系.

验证函数的类型是 `Func`, 接受 FieldLevel 为参数, 结果返回 bool.

### fetchCacheTag

这个例子中, `"required,email"` 对应着 `required` 和 `email` 两个验证函数, `fetchCacheTag` 先在缓存里找 `"required,email"` 对应的 `cTag`, 找不到就解析并缓存. `cTag` 是个链表, 把多个验证函数串起来, 验证时需要让每个验证函数都通过才算成功.

### traverseField

这是处理具体验证的方法, 经过一系列解析工作, `ct.fn(ctx, v)` 调用了具体的验证方法. 对于这个例子来说就是:

```go
// IsEmail is the validation function for validating if the current field's value is a valid email address.
func isEmail(fl FieldLevel) bool {
	return emailRegex.MatchString(fl.Field().String())
}
```

ct 上的 `fn` 是新建 validator 实例的时候设置好的,  经过 `wrapFunc`  把最终的验证函数(`Func` 类型)包装了成了 `FuncCtx`, 以提供 context 的支持:

```go
func wrapFunc(fn Func) FuncCtx {
	if fn == nil {
		return nil // be sure not to wrap a bad function.
	}
	return func(ctx context.Context, fl FieldLevel) bool {
		return fn(fl)
	}
}
```

## More

这里有更多更详细的例子: [https://github.com/go-playground/validator/tree/master/_examples](https://github.com/go-playground/validator/tree/master/_examples)

对于用户来说只需要关注 `Func` 函数的实现, validator 帮我们做验证的解析和调度.
