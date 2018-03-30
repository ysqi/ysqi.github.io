
---
date: 2016-12-31T11:33:16+08:00
title: "Golang实现RSA加密解密(附带php)"
description: ""
disqus_identifier: 1485833596878555243
slug: "Golang-shi-xian-RSA-jia-mi-jie-mi-(fu-dai-php)"
source: "https://segmentfault.com/a/1190000006156859"
tags: 
- base64 
- openssl 
- php 
- rsa 
- golang 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

安全总是很重要的，各个语言对于通用的加密算法都会有实现。前段时间，用Go实现了RSA和DES的加密解密，在这分享一下。（对于RSA和DES加密算法本身，请查阅相关资料）

在PHP中，很多功能经常是一个函数解决；而Go中的却不是。本文会通过PHP加密，Go解密；Go加密，PHP解密来学习Go的RSA和DES相关的API。

该文讨论Go RSA加密解密。所有操作在linux下完成。

### 一、概要

这是一个非对称加密算法，一般通过公钥加密，私钥解密。

在加解密过程中，使用openssl生产密钥。执行如下操作：

1）创建私钥：

    openssl genrsa -out private.pem 1024 //密钥长度，1024觉得不够安全的话可以用2048，但是代价也相应增大

2）创建公钥：

    openssl rsa -in private.pem -pubout -out public.pem

这样便生产了密钥。

一般地，各个语言也会提供API，用于生成密钥。在Go中，可以查看encoding/pem包和crypto/x509包。具体怎么产生，可查看《GO加密解密RSA番外篇：生成RSA密钥》。

加密解密这块，涉及到很多标准，个人建议需要的时候临时学习一下。

### 二、Go RSA加密解密

1、rsa加解密，必然会去查crypto/ras这个包

> Package rsa implements RSA encryption as specified in PKCS\#1.

这是该包的说明：实现RSA加密技术，基于PKCS\#1规范。

对于什么是PKCS\#1，可以查阅相关资料。PKCS（公钥密码标准），而\#1就是RSA的标准。可以查看：PKCS系列简介

从该包中函数的名称，可以看到有两对加解密的函数。

    EncryptOAEP和DecryptOAEP
    EncryptPKCS1v15和DecryptPKCS1v15

这称作加密方案，详细可以查看，PKCS \#1 v2.1 RSA 算法标准

可见，当与其他语言交互时，需要确定好使用哪种方案。

PublicKey和PrivateKey两个类型分别代表公钥和私钥，关于这两个类型中成员该怎么设置，这涉及到RSA加密算法，本文中，这两个类型的实例通过解析文章开头生成的密钥得到。

2、解析密钥得到PublicKey和PrivateKey的实例\
这个过程，我也是花了好些时间（主要对各种加密的各种东东不熟）：怎么将openssl生成的密钥文件解析到公钥和私钥实例呢？

在encoding/pem包中，看到了—–BEGIN
Type—–这样的字样，这正好和openssl生成的密钥形式差不多，那就试试。

在该包中，一个block代表的是PEM编码的结构，关于PEM，请查阅相关资料。我们要解析密钥，当然用Decode方法：

    func Decode(data []byte) (p *Block, rest []byte)

这样便得到了一个Block的实例（指针）。

解析来看crypto/x509。为什么是x509呢？这又涉及到一堆概念。先不管这些，我也是看encoding和crypto这两个包的子包摸索出来的。\
在x509包中，有一个函数：

    func ParsePKIXPublicKey(derBytes []byte) (pub interface{}, err error)

从该函数的说明：ParsePKIXPublicKey parses a DER encoded public key.
These values are typically found in PEM blocks with “BEGIN PUBLIC
KEY”。可见这就是解析PublicKey的。另外，这里说到了PEM，可以上面的encoding/pem对了。（PKIX是啥东东，查看这里
）

而解析私钥的，有好几个方法，从上面的介绍，我们知道，RSA是PKCS\#1，刚好有一个方法：

    func ParsePKCS1PrivateKey(der []byte) (key *rsa.PrivateKey, err error)

返回的就是rsa.PrivateKey。

3、解密解密实现\
通过上面的介绍，Go中RSA的解密解密实现就不难了。代码如下：

// 加密

    func RsaEncrypt(origData []byte) ([]byte, error) {
        block, _ := pem.Decode(publicKey)
        if block == nil {
            return nil, errors.New("public key error")
        }
        pubInterface, err := x509.ParsePKIXPublicKey(block.Bytes)
        if err != nil {
            return nil, err
        }
        pub := pubInterface.(*rsa.PublicKey)
        return rsa.EncryptPKCS1v15(rand.Reader, pub, origData)
    }

// 解密

    func RsaDecrypt(ciphertext []byte) ([]byte, error) {
        block, _ := pem.Decode(privateKey)
        if block == nil {
            return nil, errors.New("private key error!")
        }
        priv, err := x509.ParsePKCS1PrivateKey(block.Bytes)
        if err != nil {
            return nil, err
        }
        return rsa.DecryptPKCS1v15(rand.Reader, priv, ciphertext)
    }

其中，publicKey和privateKey是openssl生成的密钥，我生成的如下：

// 公钥和私钥可以从文件中读取

    var privateKey = []byte(`
    -----BEGIN RSA PRIVATE KEY-----
    MIICXQIBAAKBgQDZsfv1qscqYdy4vY+P4e3cAtmvppXQcRvrF1cB4drkv0haU24Y
    7m5qYtT52Kr539RdbKKdLAM6s20lWy7+5C0DgacdwYWd/7PeCELyEipZJL07Vro7
    Ate8Bfjya+wltGK9+XNUIHiumUKULW4KDx21+1NLAUeJ6PeW+DAkmJWF6QIDAQAB
    AoGBAJlNxenTQj6OfCl9FMR2jlMJjtMrtQT9InQEE7m3m7bLHeC+MCJOhmNVBjaM
    ZpthDORdxIZ6oCuOf6Z2+Dl35lntGFh5J7S34UP2BWzF1IyyQfySCNexGNHKT1G1
    XKQtHmtc2gWWthEg+S6ciIyw2IGrrP2Rke81vYHExPrexf0hAkEA9Izb0MiYsMCB
    /jemLJB0Lb3Y/B8xjGjQFFBQT7bmwBVjvZWZVpnMnXi9sWGdgUpxsCuAIROXjZ40
    IRZ2C9EouwJBAOPjPvV8Sgw4vaseOqlJvSq/C/pIFx6RVznDGlc8bRg7SgTPpjHG
    4G+M3mVgpCX1a/EU1mB+fhiJ2LAZ/pTtY6sCQGaW9NwIWu3DRIVGCSMm0mYh/3X9
    DAcwLSJoctiODQ1Fq9rreDE5QfpJnaJdJfsIJNtX1F+L3YceeBXtW0Ynz2MCQBI8
    9KP274Is5FkWkUFNKnuKUK4WKOuEXEO+LpR+vIhs7k6WQ8nGDd4/mujoJBr5mkrw
    DPwqA3N5TMNDQVGv8gMCQQCaKGJgWYgvo3/milFfImbp+m7/Y3vCptarldXrYQWO
    AQjxwc71ZGBFDITYvdgJM1MTqc8xQek1FXn1vfpy2c6O
    -----END RSA PRIVATE KEY-----
    `)
     
    var publicKey = []byte(`
    -----BEGIN PUBLIC KEY-----
    MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDZsfv1qscqYdy4vY+P4e3cAtmv
    ppXQcRvrF1cB4drkv0haU24Y7m5qYtT52Kr539RdbKKdLAM6s20lWy7+5C0Dgacd
    wYWd/7PeCELyEipZJL07Vro7Ate8Bfjya+wltGK9+XNUIHiumUKULW4KDx21+1NL
    AUeJ6PeW+DAkmJWF6QIDAQAB
    -----END PUBLIC KEY-----
    `)

4、使用例子

    package main
     
    import (
        "fmt"
    )

    func main() {
        data, err := RsaEncrypt([]byte("git@github.com/mrkt"))
        if err != nil {
            panic(err)
        }
        origData, err := RsaDecrypt(data)
        if err != nil {
            panic(err)
        }
        fmt.Println(string(origData))
    }

该例子是加密完git@github.com/mrkt后立马解密

### 三、跨语言加解密

语言内部正常，还得看看和其他语言是否一致，即：其他语言加密，Go语言得正确解密；Go语言加密，其他语言正确解密

1、PHP RSA加解密\
这里，我选择PHP，使用的是openssl扩展。PHP中加解密很简单，如下两个方法（这里只考虑用公钥加密，私钥解密）：

> bool openssl\_public\_encrypt ( string \$data , string &\$crypted ,
> mixed\
> \$key \[, int \$padding = OPENSSL\_PKCS1\_PADDING \] ) bool\
> openssl\_private\_decrypt ( string \$data , string &\$decrypted ,
> mixed\
> \$key \[, int \$padding = OPENSSL\_PKCS1\_PADDING \] )

最后一个参数是加密方案（补齐方式）。由于Go中使用的是PKCS1而不是OAEP，所以，使用默认值即可。

PHP代码如下：

> \$privateKey = '-----BEGIN RSA PRIVATE KEY-----\
> MIICXQIBAAKBgQDZsfv1qscqYdy4vY+P4e3cAtmvppXQcRvrF1cB4drkv0haU24Y\
> 7m5qYtT52Kr539RdbKKdLAM6s20lWy7+5C0DgacdwYWd/7PeCELyEipZJL07Vro7\
> Ate8Bfjya+wltGK9+XNUIHiumUKULW4KDx21+1NLAUeJ6PeW+DAkmJWF6QIDAQAB\
> AoGBAJlNxenTQj6OfCl9FMR2jlMJjtMrtQT9InQEE7m3m7bLHeC+MCJOhmNVBjaM\
> ZpthDORdxIZ6oCuOf6Z2+Dl35lntGFh5J7S34UP2BWzF1IyyQfySCNexGNHKT1G1\
> XKQtHmtc2gWWthEg+S6ciIyw2IGrrP2Rke81vYHExPrexf0hAkEA9Izb0MiYsMCB\
> /jemLJB0Lb3Y/B8xjGjQFFBQT7bmwBVjvZWZVpnMnXi9sWGdgUpxsCuAIROXjZ40\
> IRZ2C9EouwJBAOPjPvV8Sgw4vaseOqlJvSq/C/pIFx6RVznDGlc8bRg7SgTPpjHG\
> 4G+M3mVgpCX1a/EU1mB+fhiJ2LAZ/pTtY6sCQGaW9NwIWu3DRIVGCSMm0mYh/3X9\
> DAcwLSJoctiODQ1Fq9rreDE5QfpJnaJdJfsIJNtX1F+L3YceeBXtW0Ynz2MCQBI8\
> 9KP274Is5FkWkUFNKnuKUK4WKOuEXEO+LpR+vIhs7k6WQ8nGDd4/mujoJBr5mkrw\
> DPwqA3N5TMNDQVGv8gMCQQCaKGJgWYgvo3/milFfImbp+m7/Y3vCptarldXrYQWO\
> AQjxwc71ZGBFDITYvdgJM1MTqc8xQek1FXn1vfpy2c6O\
> -----END RSA PRIVATE KEY-----'; \$publicKey = '-----BEGIN PUBLIC
> KEY-----\
> MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDZsfv1qscqYdy4vY+P4e3cAtmv\
> ppXQcRvrF1cB4drkv0haU24Y7m5qYtT52Kr539RdbKKdLAM6s20lWy7+5C0Dgacd\
> wYWd/7PeCELyEipZJL07Vro7Ate8Bfjya+wltGK9+XNUIHiumUKULW4KDx21+1NL\
> AUeJ6PeW+DAkmJWF6QIDAQAB\
> -----END PUBLIC KEY-----';

    function rsaEncrypt($data)
    {
        global $publicKey;
        openssl_public_encrypt($data, $crypted, $publicKey);
        return $crypted;

    }
    function rsaDecrypt($data)
    {
        global $privateKey;
        openssl_private_decrypt($data, $decrypted, $privateKey);
        return $decrypted;
    }

    function main()
    {

        $crypted = rsaEncrypt("git@github.com/mrk");
        $decrypted = rsaDecrypt($crypted);
        echo "encrypt and decrypt:" . $decrypted;

    }

main();\
这里也是用PHP加解密git@github.com/mrkt

2、Go和PHP一起工作\
这里要注意的一点是，由于加密后是字节流，直接输出查看会乱码，因此，为了便于语言直接加解密，这里将加密之后的数据进行base64编码。

3、使用\
示例中，php和Go版本都支持-d参数传入加密好的字符串，将其解密；不传时，会输出加密好并base64编码的串，可用于其他语言解密。

