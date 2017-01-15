---
layout: post
title:  "Java Cryptography Architecture Part 2"
date:   2017-01-14 23:53:58 +0800
categories: my2829
---

### SecureRandom
#### 生成随机数
```java
SecureRandom secureRandom = SecureRandom.getInstance("SHA1PRNG");
byte[] randomBytes = new byte[16];
secureRandom.nextBytes(randomBytes);

// e.g. 924cc3f787fc6baf75869ede25dd88d1
System.out.println(Hex.encodeHex(randomBytes));
```

### MessageDigest
#### 生成信息摘要
```java
MessageDigest messageDigest = MessageDigest.getInstance("md5");
byte[] bytes = "hello world.".getBytes();
messageDigest.update(bytes);
byte[] digest = messageDigest.digest();
// 3c4292ae95be58e0c58e4e5511f09647
System.out.println(Hex.encodeHex(digest));
```

### Signature
#### 签名
> A cryptographically secure signature algorithm takes arbitrary-sized input and a private key and generates a relatively short (often fixed-size) string of bytes, called the signature, with the following properties:
- Only the owner of a private/public key pair is able to create a signature. It should be computationally infeasible for anyone having a public key to recover the private key.
- Given the public key corresponding to the private key used to generate the signature, it should be possible to verify the authenticity and integrity of the input.
- The signature and the public key do not reveal anything about the private key.

```java
String message = "this message should be sent with it's signature.";

// 生成公钥和私钥
KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("DSA");
keyPairGenerator.initialize(1024);
KeyPair keyPair = keyPairGenerator.generateKeyPair();

Signature signature = Signature.getInstance("DSA");

// Signing
signature.initSign(keyPair.getPrivate());
signature.update(message.getBytes());
byte[] sigBytes = signature.sign();
// 302c0214603346bc415073cb9cc7921bb9df679a6e3abded0214321261086278e2fb016d94c1bc87d048518c184d
System.out.println(Hex.encodeHex(sigBytes));

// Verifying
signature.initVerify(keyPair.getPublic());
signature.update(message.getBytes());
if (signature.verify(sigBytes)) {
    System.out.println("the message is valid.");
}
```

### Cipher
#### 加密
```java
String clearText = "this message should be encrypted.";

// 使用generator生成key
// KeyGenerator keyGenerator = KeyGenerator.getInstance("AES");
// SecretKey secretKey = keyGenerator.generateKey();

// 使用自己的密码，密码需要是16位
// String password = "1234567890123456";
// SecretKeySpec secretKeySpec = new SecretKeySpec(password.getBytes(), "AES");

// 使用自己的密码，又不是16位，需要使用MessageDigest转成16位的
String mypassword = "mypassword";
MessageDigest messageDigest = MessageDigest.getInstance("SHA-1");
messageDigest.update(mypassword.getBytes());
byte[] passwordBytes = messageDigest.digest();
SecretKeySpec secretKeySpec = new SecretKeySpec(passwordBytes, 0, 16, "AES");

// Encrypt
Cipher cipher = Cipher.getInstance("AES/CBC/PKCS5Padding");
cipher.init(Cipher.ENCRYPT_MODE, secretKeySpec);
byte[] cipherBytes = cipher.doFinal(clearText.getBytes());
System.out.println(Hex.encodeHex(cipherBytes));

// 加密的时候没有提供IV，会自动生成一个，通过getIV获取自动生成的IV
byte[] ivBytes = cipher.getIV();

// Decrypt
IvParameterSpec ivParameterSpec = new IvParameterSpec(ivBytes);
cipher.init(Cipher.DECRYPT_MODE, secretKeySpec, ivParameterSpec);
byte[] clearBytes = cipher.doFinal(cipherBytes);
System.out.println(new String(clearBytes));
```


