---
layout: post
title:  UTF-8 in Golang
date:   2020-09-27
Author: CBD
tags: [Golang]
---

Unicode 是字符集, UTF-8 是编码规则.

Unicode 给每个字符定义了唯一标号, 即码点. 每个码点用 4 个字节表示, UTF-32 就是一对一地将码点对应到字符.

UTF-32 解决了字符表示问题, 但是在实际使用中是及其浪费空间的, 因为它表示最常用的 ASCII 码在也需要占用 4 个字节.

UTF-8 采用变长的方式, 让常用的 ASCII 码只占用一个字节(并且兼容), 让东亚文字多占几个字节来容纳其庞大的字形.

```text
0xxxxxxx                             runes 0-127    (ASCII)
110xxxxx 10xxxxxx                    128-2047       (values <128 unused)
1110xxxx 10xxxxxx 10xxxxxx           2048-65535     (values <2048 unused)
11110xxx 10xxxxxx 10xxxxxx 10xxxxxx  65536-0x10ffff (other values unused)
```

在 Golang 中:

* 代码使用 UTF-8 编码;
* string 不可变, 内部是字节的组合;
* rune 使用 int32 存储, 代表一个码点;
* range string 时自动根据 rune 迭代.

```golang
func show(s string) {
	fmt.Printf("%q\n", s)

	fmt.Println("-----")
	fmt.Printf("UTF-8 Hex:\t0x%x\n", s)
	fmt.Println("range by byte:")
	for i, b := range []byte(s) {
		fmt.Printf("i:%d, 0x%x, 0b%b\n", i, b, b)
	}

	fmt.Println("-----")
	r, size := utf8.DecodeRune([]byte(s))
	fmt.Printf("Rune:\t%v, %d bytes\n", r, size)
	fmt.Println("range by rune:")
	for i, r := range s {
		fmt.Printf("i:%d, 0x%x\n", i, r)
	}
}
```

## `show("B")`

[Unicode 13.0, Basic Latin (ASCII)](https://www.unicode.org/charts/PDF/U0000.pdf)

```text
"B"
-----
UTF-8 Hex:      0x42
range by byte:
i:0, 0x42, 0b1000010
-----
Rune:   66, 1 bytes
range by rune:
i:0, 0x42

```

`"B"` 是 ASCII 码范围内的, 占用一个 byte 来存储, 用 `0x42` (十进制为 `66`) 表示. UTF-8 兼容 ASCII.

## `show("中")`

[Unicode 13.0, CJK Unified Ideographs (Han) (35MB)](https://www.unicode.org/charts/PDF/U4E00.pdf)

```text
"中"
-----
UTF-8 Hex:      0xe4b8ad
range by byte:
i:0, 0xe4, 0b11100100
i:1, 0xb8, 0b10111000
i:2, 0xad, 0b10101101
-----
Rune:   20013, 3 bytes
range by rune:
i:0, 0x4e2d
```

`中` 的码点是 `4e2d`, 二进制表示为:

```text
0100 1110 0010 1101
```

虽然 `中` 的码点只占用 16位, 但是要把 `中` 安排到 UTF-8 的编码中, 需要额外的字节.

```text
0b1110xxxx
0b10xxxxxx
0b10xxxxxx
```

这里的 x 是真正用来摆放码点的空间. `16 / 6 = 2 ... 4`,  三个字节的 UTF-8 刚好够用, 所以 `中` 的高位是 `1110`. 将码点内容 `0100 1110 0010 1101` 按照顺序依次替换 x .即得到了:

```text
0b11100100
0b10111000
0b10101101
```

Golang 中也可以使用 `"\u4e2d"` 来表示 `"中"`. 注意 `\u` 是小写, 表示用 16 位.

## `show("😆")`

[Emoji List, v13.1](https://unicode.org/emoji/charts/emoji-list.html)

```text
"😆"
-----
UTF-8 Hex:      0xf09f9886
range by byte:
i:0, 0xf0, 0b11110000
i:1, 0x9f, 0b10011111
i:2, 0x98, 0b10011000
i:3, 0x86, 0b10000110
-----
Rune:   128518, 4 bytes
range by rune:
i:0, 0x1f606

```

`😆` 的码点为 `1f606`, 20 位.

```text
0001
1111
0110
0000
0110
```

 `20 / 6 = 3 ... 2` . 最后余出了 2 位, 那就在最前面补 0 来对齐, 也就相当于把码点从后往前排入位子.

```text
0b11110000
0b10011111
0b10011000
0b10000110
```

所以常用的 Emoji 在 UTF-8 中占用 4 个字节.

在 Golang 中可以用 `"\U0001f606"` 来表示 `"😆"`. 注意 `\U` 是大写, 表示用 32 位.

## `show("👩‍🍳")`

```text
"👩\u200d🍳"
-----
UTF-8 Hex:      0xf09f91a9e2808df09f8db3
range by byte:
i:0, 0xf0, 0b11110000
i:1, 0x9f, 0b10011111
i:2, 0x91, 0b10010001
i:3, 0xa9, 0b10101001
i:4, 0xe2, 0b11100010
i:5, 0x80, 0b10000000
i:6, 0x8d, 0b10001101
i:7, 0xf0, 0b11110000
i:8, 0x9f, 0b10011111
i:9, 0x8d, 0b10001101
i:10, 0xb3, 0b10110011
-----
Rune:   128105, 4 bytes
range by rune:
i:0, 0x1f469
i:4, 0x200d
i:7, 0x1f373

```

`👩‍🍳` 这个表情的含义是女性厨师, 是由 `"👩"` 加  连接符 `"\u200d"` 加 `"🍳"` 这三个码点组合产生的. 还有其他很多根据性别或者肤色的组合.

支持新版 Emoji 的设备会显示一个图标, 不支持的会显示 3 个. Golang 认为这是 3 个 rune.

## 参考

[Ken Thompson invented UTF-8 in one evening](http://doc.cat-v.org/bell_labs/utf-8_history)

[Unicode Charts](http://www.unicode.org/charts/)

[Unicode Emoji Full List](https://unicode.org/emoji/charts/full-emoji-list.html)