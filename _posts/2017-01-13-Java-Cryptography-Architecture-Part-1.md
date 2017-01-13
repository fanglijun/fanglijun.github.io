---
layout: post
title:  "Java Cryptography Architecture Part 1"
date:   2017-01-13 23:56:58 +0800
categories: my2829
---

### Java Cryptography Architecture
![](https://mmbiz.qlogo.cn/mmbiz_png/SHQtibmBWibdxHF0mwOrWtKAL1jRmvBDLA2hoVUrSvIAWiaN2icF5oMFiaG3ofVTicuEK8w5sTiam9GBDNQsF16tPiamjg/0?wx_fmt=gif)

1. Application 调用 Engine Classes（e.g. MessageDigest，SecureRandom，Cipher）的getInstance方法，参数是想要使用的算法（e.g. MD5）
2. Engine Classes 查询 Provider 数据库，如果有Provider实现了请求的算法（e.g. MD5），则返回一个Engine Class的实例，它是该Provider的一层封装
3. Application 拿到一个Engine Class的实例后，即可按相应的接口调用算法

#### 一个算法（e.g. MD5）可能有多个Provider都实现了，怎么确定用哪个Provider？
1. 每个Provider会有一个preference值表示优先级，默认会按照这个优先级来查找Provider，返回优先级高的
2. 调用Engine Classes的getInstance方法时，可以传第二个参数指定使用某个Provider

#### 这种设计的好处
- implementation independence and interoperability
    - Application不依赖具体的算法实现，它依赖的是每种Engine Classes的接口
    - 各种算法的实现 可以互相替换，比如 MessageDigest 所使用的具体算法可以是MD5，也可以换成 SHA1
- algorithm independence and extensibility
    - Engine Classes 也不依赖与每种算法的实现（e.g. MD5Provider），它依赖的是Provider接口，它只需要知道哪个具体的Provider实现了MD5算法
    - 扩展性，可以自己覆盖已有算法（通过指定Provider优先级）或者增加算法

### Available Engine Classes
- **SecureRandom**: used to generate random or pseudo-random numbers.
- **MessageDigest**: used to calculate the message digest (hash) of specified data.
- **Signature**: initialized with keys, these are used to sign data and verify digital signatures.
- **Cipher**: initialized with keys, these used for encrypting/decrypting data. There are various types of algorithms: symmetric bulk encryption (e.g. AES, DES, DESede, Blowfish, IDEA), stream encryption (e.g. RC4), asymmetric encryption (e.g. RSA), and password-based encryption (PBE).
- **Message Authentication Codes (MAC)**: like MessageDigests, these also generate hash values, but are first initialized with keys to protect the integrity of messages.
- **KeyFactory**: used to convert existing opaque cryptographic keys of type Key into key specifications (transparent representations of the underlying key material), and vice versa.
- **SecretKeyFactory**: used to convert existing opaque cryptographic keys of type SecretKey into key specifications (transparent representations of the underlying key material), and vice versa. SecretKeyFactorys are specialized KeyFactorys that create secret (symmetric) keys only.
- **KeyPairGenerator**: used to generate a new pair of public and private keys suitable for use with a specified algorithm.
- **KeyGenerator**: used to generate new secret keys for use with a specified algorithm.
KeyAgreement: used by two or more parties to agree upon and establish a specific key to use for a particular cryptographic operation.
- 等等


### Generator 和 Factory 的区别
> A generator creates objects with brand-new contents, whereas a factory creates objects from existing material (for example, an encoding).

