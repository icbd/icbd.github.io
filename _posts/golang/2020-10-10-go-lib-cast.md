---
layout: post
title:  Go Lib cast
date:   2020-10-10
Author: CBD
tags: [Golang, lib]
---

cast 是一个用来做类型转换的库, 基本用法如下:

```go
cast.ToString("mayonegg")         // "mayonegg"
cast.ToString(8)                  // "8"
cast.ToString(8.31)               // "8.31"
cast.ToString([]byte("one time")) // "one time"
cast.ToString(nil)                // ""

var foo interface{} = "one more time"
cast.ToString(foo)                // "one more time"

```

## 源码组织方式

cast 把源码拆到两个文件中, `cast.go` 中是类似这样的包装函数, 直接给用户使用:

```go
func ToBool(i interface{}) bool {
	v, _ := ToBoolE(i)
	return v
}
```

`caste.go` 中是具体的解析函数, 函数名以 `E` 结尾表示会携带 error 信息.
如果 `ToBoolE` 这样的函数产生了 error, 第一个返回值为该类型的零值, 所以 `ToBool` 这样的函数可以直接丢弃 error.

```go
// ToBoolE casts an interface to a bool type.
func ToBoolE(i interface{}) (bool, error) {
	i = indirect(i)

	switch b := i.(type) {
	case bool:
		return b, nil
	case nil:
		return false, nil
	case int:
		if i.(int) != 0 {
			return true, nil
		}
		return false, nil
	case string:
		return strconv.ParseBool(i.(string))
	default:
		return false, fmt.Errorf("unable to cast %#v of type %T to bool", i, i)
	}
}
```

如 `ToBoolE` 所示, 这个库的原理就是通过 `i.(type)` 检查接口类型, 然后调用对应的解析方法.

原理虽简单, 但还是有几个值得学习的亮点.

## indirect

```go
// From html/template/content.go
// Copyright 2011 The Go Authors. All rights reserved.
// indirect returns the value, after dereferencing as many times
// as necessary to reach the base type (or nil).
func indirect(a interface{}) interface{} {
	if a == nil {
		return nil
	}
	if t := reflect.TypeOf(a); t.Kind() != reflect.Ptr {
		// Avoid creating a reflect.Value if it's not a pointer.
		return a
	}
	v := reflect.ValueOf(a)
	for v.Kind() == reflect.Ptr && !v.IsNil() {
		v = v.Elem()
	}
	return v.Interface()
}
```

这个方法实际上摘抄自标准库, 通过反射查看 a 的类型, 如果是个指针, 就深度搜索直到取得原值.

```go
	if t := reflect.TypeOf(a); t.Kind() != reflect.Ptr {
		// Avoid creating a reflect.Value if it's not a pointer.
		return a
	}
```

这一段其实可以删掉, 他的作用是当 a 不是指针的时候可以立即返回.
`reflect.TypeOf(a).Kind()` 和 `reflect.ValueOf(a).Kind()` 都可以得到 a 的类型, 但 `TypeOf` 的开销会更小,

## indirectToStringerOrError

```go
func indirectToStringerOrError(a interface{}) interface{} {
	if a == nil {
		return nil
	}

	var errorType = reflect.TypeOf((*error)(nil)).Elem()
	var fmtStringerType = reflect.TypeOf((*fmt.Stringer)(nil)).Elem()

	v := reflect.ValueOf(a)
	for !v.Type().Implements(fmtStringerType) && !v.Type().Implements(errorType) && v.Kind() == reflect.Ptr && !v.IsNil() {
		v = v.Elem()
	}
	return v.Interface()
}
```

`errorType` 和 `fmtStringerType` 分别是 `error` 和 `Stringer` 的接口类型. 这里获得接口类型的方法很经典:

1. `(*error)(nil)` 空的指针;
2. `reflect.TypeOf((*error)(nil))` 获得指针的 Type;
3. `reflect.TypeOf((*error)(nil)).Elem()` 取得 Type 指向元素的 Type. **Type 的 `Elem()` 还是个 Type.**

相比之前的 indirect, 这里新加了两个停止条件:

```go
!v.Type().Implements(fmtStringerType) && !v.Type().Implements(errorType)
```

因为他们都有转为字符串的方法:

```go
	case fmt.Stringer:
		return s.String(), nil
	case error:
		return s.Error(), nil
```

## interface to map

```go
// ToStringMapIntE casts an interface to a map[string]int{} type.
func ToStringMapIntE(i interface{}) (map[string]int, error) {
	var m = map[string]int{}
	if i == nil {
		return m, fmt.Errorf("unable to cast %#v of type %T to map[string]int", i, i)
	}

	switch v := i.(type) {
	case map[interface{}]interface{}:
		for k, val := range v {
			m[ToString(k)] = ToInt(val)
		}
		return m, nil
	case map[string]interface{}:
		for k, val := range v {
			m[k] = ToInt(val)
		}
		return m, nil
	case map[string]int:
		return v, nil
	case string:
		err := jsonStringToObject(v, &m)
		return m, err
	}

	if reflect.TypeOf(i).Kind() != reflect.Map {
		return m, fmt.Errorf("unable to cast %#v of type %T to map[string]int", i, i)
	}

	mVal := reflect.ValueOf(m)
	v := reflect.ValueOf(i)
	for _, keyVal := range v.MapKeys() {
		val, err := ToIntE(v.MapIndex(keyVal).Interface())
		if err != nil {
			return m, fmt.Errorf("unable to cast %#v of type %T to map[string]int", i, i)
		}
		mVal.SetMapIndex(keyVal, reflect.ValueOf(val))
	}
	return m, nil
}
```

开始读还有些疑惑, 既然有了 `map[interface{}]interface{}`, 那还要其他的 case 做什么呢? 为什么需要反射呢?

```go
func main() {
	a := map[string]string{"hello": "123"}
	var b interface{} = a
	c := b.(map[interface{}]interface{})
	// panic: interface conversion: interface {} is map[string]string, not map[interface {}]interface {}

	fmt.Println(a, b, c)
}
```

虽然可以用 `interface{}` 接收任何类型的值, 但是不能用 `map[interface{}]interface{}` 来接收 `map[string]string`, 他们是不同的类型.

i 是个 `interface{}`, 我们还没有信息把他转为具体的 map, 或者说没必要穷举所有 map 的 key/value 类型情况.
i 不能直接迭代, 可以使用反射利用 `MapKeys` 和 `MapIndex` 在不转换 map 类型的情况下读取和设置.

## 优化

给 cast 交了一个 PR, 让 `ToStringMapString` 支持非字符串的 json:

[feat(ToStringMapStringE): Add support for non string JOSN value](https://github.com/spf13/cast/pull/104)

其他可优化项:

* 合并 case 条件, 减少冗余代码;
* 提取配置项, 让用户定制解析顺序和默认行为;
* 对 map slice 的转换时没有使用 `indirect` 来解开指针(或许是出于效率考虑).
