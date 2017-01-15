---
layout: post
title:  "Java Cryptography Architecture Part 3"
date:   2017-01-15 23:53:58 +0800
categories: my2829
---

### 两个接口：Key 、KeySpec
#### Key
- 用于Cipher的加密解密
- 是密钥的不透明（opaque）的表示形式，不能获取 用于密钥生成的参数
- 只定义了三个方法：getAlgorithm, getFormat, and getEncoded

#### KeySpec
- 是密钥的透明（transparent）的表示形式，可获取 用于密钥生成的参数
- 可用于指定参数，生成特定的密钥
- 仅用作分组，没有定义接口方法

#### e.g. DSAPrivateKeySpec
- DSAPrivateKeySpec 实现的是 KeySpec，提供了getX, getP, getQ, and getG方法，可以用于获取DSA算法生成密钥的参数 x, p, q, g
- 也可以指定 x,p,q,g 生成指定的密钥: `new DSAPrivateKeySpec(x,p,q,g)`

### Key 和 KeySpec 的生成和相互转换
```java
public static void key() throws Exception {
    // KeyPairGenerator 用于生成非对称加密的一对公钥和私钥
    KeyPairGenerator keyPairGenerator = KeyPairGenerator.getInstance("DSA");
    KeyPair keyPair = keyPairGenerator.generateKeyPair();

    // KeyGenerator 用于生成对称加密的密钥
    KeyGenerator keyGenerator = KeyGenerator.getInstance("DES");
    SecretKey secretKey = keyGenerator.generateKey();


    // Key 和 KeySpec 的相互转换
    // KeyFactory 用于公钥、私钥对的转换
    KeyFactory keyFactory = KeyFactory.getInstance("DSA");

    // Key 转 KeySpec
    // DSA的私钥，有参数x,p,q,g
    DSAPrivateKeySpec dsaPrivateKeySpec = keyFactory.getKeySpec(keyPair.getPrivate(), DSAPrivateKeySpec.class);
    BigInteger x = dsaPrivateKeySpec.getX();
    BigInteger p = dsaPrivateKeySpec.getP();
    BigInteger q = dsaPrivateKeySpec.getQ();
    BigInteger g = dsaPrivateKeySpec.getG();
    System.out.printf("x: %d %np: %d %nq: %d %ng: %d %n", x, p, q, g);
    // DSA的公钥，有参数y,p,q,g（p,q,g同私钥）
    DSAPublicKeySpec dsaPublicKeySpec = keyFactory.getKeySpec(keyPair.getPublic(), DSAPublicKeySpec.class);
    BigInteger y = dsaPublicKeySpec.getY();
    System.out.printf("y: %d %n", y);

    // KeySpec 转 Key
    PrivateKey privateKey = keyFactory.generatePrivate(dsaPrivateKeySpec);
    PublicKey publicKey = keyFactory.generatePublic(dsaPublicKeySpec);
    System.out.println("private key equal: " + privateKey.equals(keyPair.getPrivate()));
    System.out.println("public key equal: " + publicKey.equals(keyPair.getPublic()));

    // SecretKeyFactory 用于对称加密的密钥的转换
    SecretKeyFactory secretKeyFactory = SecretKeyFactory.getInstance("DES");
    KeySpec keySpec = secretKeyFactory.getKeySpec(secretKey, DESKeySpec.class);
    SecretKey secretKey1 = secretKeyFactory.generateSecret(keySpec);
    System.out.println("secret key equal: " + secretKey1.equals(secretKey));


    // 直接由 KeySpec 生成 Key
    // 生成非对称加密的密钥对
    DSAPublicKeySpec newDSAPublicKeySpec = new DSAPublicKeySpec(y, p, q, g);
    DSAPrivateKeySpec newDSAPrivateKeySpec = new DSAPrivateKeySpec(x, p, q, g);
    // 转成 Key 再比较，不能直接用KeySpec.equals比较，因为KeySpec没有覆盖equals方法仍然是用==比较
    PublicKey dsaPublicKey = keyFactory.generatePublic(newDSAPublicKeySpec);
    System.out.println("new dsa public key equal: " + dsaPublicKey.equals(keyPair.getPublic()));

    // 生成对称加密的密钥
    DESKeySpec desKeySpec = new DESKeySpec(secretKey.getEncoded());
    SecretKey secretKey2 = secretKeyFactory.generateSecret(desKeySpec);
    System.out.println("new des secret key equal: " + secretKey2.equals(secretKey));

    // AES比较特殊
    // JDK没有 AESKeySpec 和 AES SecretKeyFactory
    // 但是 SecretKeySpec 同时实现了 KeySpec 和 SecretKey (即 Key)
    String password = "1234567890123456";
    byte[] passwordBytes = password.getBytes();
    SecretKeySpec secretKeySpec = new SecretKeySpec(passwordBytes, "AES");

    // SecretKeySpec 可以当 Key 用
    // 这里输出原始密码 -> password
    System.out.println(new String(secretKeySpec.getEncoded()));
}

```


