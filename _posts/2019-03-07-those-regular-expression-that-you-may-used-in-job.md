---
layout: post
title: '工作中你也许能用到的正则表达式'
tags: [read]
---
## 目录

* TOC
{:toc}
查看原文：[原文](https://github.com/cdoco/common-regex)

## 邮箱

`gaozihang-001@gmail.com` 只允许英文字母、数字、下划线、英文句号、以及中划线组成

```regex
^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$
```

![email](../images/regularexpression/email.png)

`高子航001Abc@bowbee.com.cn` 名称允许汉字、字母、数字，域名只允许英文域名

```regex
^[A-Za-z0-9\u4e00-\u9fa5]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$
```

![email](../images/regularexpression/email2.png)

## 电话

`13012345678` 手机号

```regex
^1(3|4|5|6|7|8|9)\d{9}$
```

![phone](../images/regularexpression/phone.png)

`XXX-XXXXXXX` `XXXX-XXXXXXXX` 固定电话

```regex
(\(\d{3,4}\)|\d{3,4}-|\s)?\d{8}
```

![email](../images/regularexpression/phone2.png)

## 域名

`https://google.com/`

```regex
^((http:\/\/)|(https:\/\/))?([a-zA-Z0-9]([a-zA-Z0-9\-]{0,61}[a-zA-Z0-9])?\.)+[a-zA-Z]{2,6}(\/)
```

![domain-name](../images/regularexpression/domain-name.png)

## IP

`127.0.0.1`

```regex
((?:(?:25[0-5]|2[0-4]\d|[01]?\d?\d)\.){3}(?:25[0-5]|2[0-4]\d|[01]?\d?\d))
```

![ip](../images/regularexpression/ip.png)

## 帐号校验

`gaozihang_001` 字母开头，允许5-16字节，允许字母数字下划线

```regex
^[a-zA-Z][a-zA-Z0-9_]{4,15}$
```

![user](../images/regularexpression/userid.png)

## 字符校验

### 汉字

`高子航`

```regex
^[\u4e00-\u9fa5]{0,}$
```

![chinese](../images/regularexpression/chineses.png)

### 英文和数字

```regex
^[A-Za-z0-9]+$
```

![char](../images/regularexpression/char1.png)

### 长度为3-20的所有字符

```regex
^.{3,20}$
```

![char](../images/regularexpression/char2.png)

### 英文字符

#### 由26个英文字母组成的字符串

```regex
^[A-Za-z]+$
```

![char](../images/regularexpression/char3.png)

#### 由26个大写英文字母组成的字符串

```regex
^[A-Z]+$
```

![char](../images/regularexpression/char4.png)

#### 由26个小写英文字母组成的字符串

```regex
^[a-z]+$
```

![char](../images/regularexpression/char5.png)

#### 由数字和26个英文字母组成的字符串

```regex
^[A-Za-z0-9]+$
```

![char](../images/regularexpression/char6.png)

#### 由数字、26个英文字母或者下划线组成的字符串 

```regex
^\w+$
```

![char](../images/regularexpression/char7.png)

### 中文、英文、数字包括下划线

```regex
^[\u4E00-\u9FA5A-Za-z0-9_]+$
```

![char](../images/regularexpression/char8.png)

### 中文、英文、数字但不包括下划线等符号

```regex
^[\u4E00-\u9FA5A-Za-z0-9]+$
```

![char](../images/regularexpression/char9.png)

### 禁止输入含有%&',;=?$\"等字符

```regex
[^%&',;=?$\x22]+
```

![char](../images/regularexpression/char10.png)

### 禁止输入含有~的字符

```regex
[^~\x22]+
```

![char](../images/regularexpression/char11.png)

## 数字正则

### 整数

```regex
^-?[1-9]\d*$
```

![num](../images/regularexpression/num1.png)

#### 正整数

```regex
^[1-9]\d*$
```

![num](../images/regularexpression/num2.png)

#### 负整数

```regex
^-[1-9]\d*$
```

![num](../images/regularexpression/num3.png)

#### 非负整数

```regex
^[1-9]\d*|0$
```

![num](../images/regularexpression/num4.png)

#### 非正整数

```regex
^-[1-9]\d*|0$
```

![num](../images/regularexpression/num5.png)

### 浮点数

```regex
^-?([1-9]\d*\.\d*|0\.\d*[1-9]\d*|0?\.0+|0)$
```

![num](../images/regularexpression/num6.png)

#### 正浮点数

```regex
^[1-9]\d*\.\d*|0\.\d*[1-9]\d*$
```

![num](../images/regularexpression/num7.png)

#### 负浮点数

```regex
^-([1-9]\d*\.\d*|0\.\d*[1-9]\d*)$
```

![num](../images/regularexpression/num8.png)

#### 非负浮点数

```regex
^[1-9]\d*\.\d*|0\.\d*[1-9]\d*|0?\.0+|0$
```

![num](../images/regularexpression/num9.png)

#### 非正浮点数

```regex
^(-([1-9]\d*\.\d*|0\.\d*[1-9]\d*))|0?\.0+|0$
```

![num](../images/regularexpression/num10.png)
