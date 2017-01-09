---
layout: post
title:  "byte[], String and Base64"
date:   2017-01-09 23:54:58 +0800
categories: my2829
---

### byte[], String and Base64
- byte[] 表示二进制数据
- String 表示字符串
- Encoding 是字符串和二进制数据相互转换的规则（对应关系）
- byte[] 转 String 需要按一定的encoding，如果未提供，会按操作系统默认的encoding
- 如果byte[]里的二进制数据按照指定的encoding无法对应到一个字符，那后果是无法预料的

The "proper conversion" between byte[] and String is to explicitly state the encoding you want to use. If you start with a byte[] and it does not in fact contain text data, there is no "proper conversion". Strings are for text, byte[] is for binary data, and the only really sensible thing to do is to avoid converting between them unless you absolutely have to.

If you really must use a String to hold binary data then the safest way is to use Base64 encoding.

- Base64是一种基于64个可打印字符来表示二进制数据的表示方法。
- 二进制数据（byte[]）可以被Base64Encode成字符串，这是二进制数据用字符串显示的最安全的方法

