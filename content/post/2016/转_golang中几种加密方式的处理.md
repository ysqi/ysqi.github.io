
---
date: 2016-12-31T11:34:03+08:00
title: "golang中几种加密方式的处理"
description: ""
disqus_identifier: 1485833643359952421
slug: "golangzhong-ji-chong-jia-mi-fang-shi-de-chu-li"
source: "https://segmentfault.com/a/1190000004151272"
tags: 
- golang 
topics:
- 编程语言与开发
---

缘由
----

在与第三方平台进行接入的时候，通常会存在一些签名或者加密的处理，在进行开发的时候，因为语言的\
不同，需要按照规范进行相应处理。

DES加解密
---------

DES：<https://en.wikipedia.org/wiki/Data_Encryption_Standard>

golang中的标准库crypto/des中有DES的实现，但是golang库的描述比较简单，如果不熟悉DES的加密规则，是不容易\
进行相应代码编写的，与第三方进行不同语言之间的加密与解密时，也容易混淆，出现错误。

DES区分为CBC和EBC加密模式，并且有不同的填充方式。

CBC（等）：<https://en.wikipedia.org/wiki/Block_cipher_mode_of_operation>\
Padding：<https://en.wikipedia.org/wiki/Padding_(cryptography)>

注意 **PKCS\#5 padding is identical to PKCS\#7 padding**

所以对不同的平台与语言进行DES加解密对接时，需要知道对方的是采用何种加密模式以及何种填充方式：

-   Windows 默认是CBC模式，CryptSetKeyParam函数，openssl
    函数名中直接表明

-   Java 中如果Cipher.getInstance()中不填写，默认是DES/ECB/PKCS5Padding

-   C\# 中默认是CBC模式，PKCS7Padding(PKCS5Padding)

golang默认提供的是CBC模式，所以对于ECB模式，需要自己编写代码

PKCS5Padding与PKCS5Unpadding

        func PKCS5Padding(ciphertext []byte, blockSize int) []byte {
            padding := blockSize - len(ciphertext)%blockSize
            padtext := bytes.Repeat([]byte{byte(padding)}, padding)
            return append(ciphertext, padtext...)
        }

        func PKCS5Unpadding(origData []byte) []byte {
            length := len(origData)
            unpadding := int(origData[length-1])
            return origData[:(length - unpadding)]
        }

ECB加密模式

            block, err := des.NewCipher(key)
            if err != nil {
                ...
            }
            bs := block.BlockSize()
            src = PKCS5Padding(src, bs)
            if len(src)%bs != 0 {
                ....
            }
            out := make([]byte, len(src))
            dst := out
            for len(src) > 0 {
                block.Encrypt(dst, src[:bs])
                src = src[bs:]
                dst = dst[bs:]
            }
            ...
        }

ECB下的解密

        block, err := des.NewCipher(key)
        if err != nil {
            ...
        }

        out := make([]byte, len(src))
        dst := out
        bs := block.BlockSize()
        if len(src)%bs != 0 {
            ...
        }

        for len(src) > 0 {
            block.Decrypt(dst, src[:bs])
            src = src[bs:]
            dst = dst[bs:]
        }
        out = PKCS5UnPadding(out)

RSA加解密
---------

与其他语言默认有更高级的封装不同，golang中需要依据不同的概念，自己组合进行封装处理，为此，需要先理解几个不同的概念。

PEM:
<https://en.wikipedia.org/wiki/Privacy-enhanced_Electronic_Mail>，通常是以.pem结尾的文件，在密钥存储和X.509证书体系中使用比较多，下面是一个X509证书下的PEM格式:

    -----BEGIN CERTIFICATE-----
        base64
    -----END CERTIFICATE-----

PKCS：<https://en.wikipedia.org/wiki/PKCS>，这是一个庞大的体系，不同的密钥采用不同的pkcs文件格式。如私钥采用pkcs8。

X.509：<https://en.wikipedia.org/wiki/X.509>，这是一个公钥管理基础（public
key infrastructure, pki)，在IETF中通常对应PKIX。

说明：

使用
openssl（如`openssl genrsa -out rsa_private_key.pem 1024`)生成的pem文件，就是符合PEM格式的，以`-----BEGIN RSA PRIVATE KEY-----`开头，`-----END RSA PRIVATE KEY-----`结尾。

也可以转换为pkcs8:

    openssl pkcs8 -topk8 -inform PEM -in rsa_private_key.pem -outform PEM -nocrypt

注意，虽然数据格式pkcs8格式，但是-outform也表明了，文件格式仍旧是符合PEM格式的，只是两个PEM文件是存在差异的。

清楚了上面几种概念与格式之后，编写golang对应的公钥与私钥加解密方式，就相对容易一些，首先是将pem文件解码，然后进行对应的密码解码为golang支持的结构体，再进行相应的处理。

如对于私钥，可以进行如下操作进行签名：

        block, _ := pem.Decode([]byte(key))
        if block == nil {       // 失败情况
            ....
        }

        private, err := x509.ParsePKCS8PrivateKey(block.Bytes)
        if err != nil {
            ...
        }

        h := crypto.Hash.New(crypto.SHA1)
        h.Write(data)
        hashed := h.Sum(nil)

        // 进行rsa加密签名
        signedData, err := rsa.SignPKCS1v15(rand.Reader, private.(*rsa.PrivateKey), crypto.SHA1, hashed)
        ...

通过私钥进行解密，代码格式如下；

        block, _ := pem.Decode([]byte(key))
        if block == nil {       // 失败情况
            ....
        }

        private, err := x509.ParsePKCS8PrivateKey(block.Bytes)
        if err != nil {
            ...
        }

        v, err := rsa.DecryptPKCS1v15(rand.Reader, private, data)
        ...

对于公钥对数据进行加密：

        block, _ := pem.Decode([]byte(key))
        if block == nil {       // 失败情况
            ....
        }

        pub, err := x509.ParsePKIXPublicKey(block.Bytes)
        if err != nil {
            ...
        }

        encryptedData, err := rsa.EncryptPKCS1v15(rand.Reader, pub.(*rsa.PublicKey), data)
        ...

小结
----

搞清楚具体的加密方式，然后再在golang里面编写，代码还是很清晰也不难看懂。

文章发布于 <https://segmentfault.com/a/1190000004151272>，作者：Damon

