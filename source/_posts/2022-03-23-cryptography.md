---
title: 密码学原理和实现（go版本）笔记
date: 2022-03-23 17:30:58
tags:
categories: [Program, go]
---

# 一、名词解释

## 1. HMAC

**全称**

- Hash-based Message Authentication Code
- 散列消息认证码

**说明**

- 使用密码散列函数，结合加密密钥，计算成的消息认证码，主要用于保证数据完整性，同时作为消息的身份认证

## 2. SHA256

**全称**

- Secure Hash Algorithm 256
- 安全散列算法256

**说明**

- 散列函数的一种，对任意长度的数字，计算一个32byte（256bit）的字符串（message digest）

# 二、算法描述和实现

## 1. HMAC-SHA256

- 使用SHA256生成hash值的HMAC算法

### 1.1. 算法步骤

摘自[从零入门HMAC-SHA256](https://blog.csdn.net/sdnyqfyqf/article/details/105534376)

<img src="2022-03-23-01.png" />

1. 密钥填充。若密钥比SHA-256算法的分组长度B（512-bit）短，则需在末尾填充0，直到其长度达到单向散列函数的分组长度为止。若密钥比分组长度长，则要用SHA-256算法求出密钥的散列值，然后将这个散列值作为新的密钥；
2. 内部填充。将填充后的密钥与被称为ipad的序列进行异或运算，所形成的值为ipadkey。ipad是将00110110这一序列不断循环反复直到达到分组长度；
3. 与消息组合。将ipadkey与消息组合，也就是将ipadkey附加在消息的开头。
4. 计算散列值。将3的结果输入SHA-256函数，并计算出散列值。
5. 外部填充。将填充后的密钥与被称为opad的序列进行异或运算，所形成的值为opadkey。opad是将01011100这一序列不断循环反复直到达到分组长度。
6. 与散列值组合。将4的散列值拼在opadkey后面。
7. 计算散列值。将6的结果输入SHA-256函数，并计算出散列值，这个散列值就是最终的摘要内容。

### 1.2. go的上层实现

```golang
package main

import (
	"crypto/hmac"
	"crypto/sha256"
	"encoding/hex"
	"fmt"
)

func main() {
	key := "aaaaa"
    str_to_sign := "bbbb"

	sig := hmac.New(sha256.New, []byte(key))
	sig.Write([]byte(str_to_sign))
	fmt.Println(hex.EncodeToString(sig.Sum(nil)))
}
```
