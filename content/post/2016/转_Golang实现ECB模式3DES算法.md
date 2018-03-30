
---
date: 2016-12-31T11:33:15+08:00
title: "Golang实现ECB模式3DES算法"
description: ""
disqus_identifier: 1485833595549575852
slug: "Golangshi-xian-ECBmo-shi-3DESsuan-fa"
source: "https://segmentfault.com/a/1190000006183694"
tags: 
- 3des 
- golang 
topics:
- 编程语言与开发
---

简介
----

因项目需要使用ECB模式下的3DES算法加解密信息，golang默认只提供CBC模式，只能自己实现ECB模式。\
参考[](https://segmentfault.com/a/1190000004151272)[https://segmentfault.com/a/11...](https://segmentfault.com/a/1190000004151272)，文章对ECB模式的DES有解释，并实现了部分DES算法样例。这里把算法补全，提供3DES算法实现。

基础
----

**3DES**\
3DES算法就是采用一个长度为24字节的密钥，将密钥分成各8字节的3份子密钥：K1、k2、k3。使用这3个密钥对明文进行加密、解密处理，如下：\
*E(k,d)、D(k,d)分别表示使用密钥k对数据d进行加密或解密，返回加密或解密后的数据。*

3DES加密过程：\
`E(k3,D(k2,E(k1,d)))`\
意思为：将明文d先用k1加密，得到密文d1；对d1再用k2做解密处理，得到密文d2；再对d2用k3做加密处理，得到最终密文。\
3DES解密过程与加密相反：\
`D(k1,E(k2,D(k3,d)))`\
意思为：将密文d先用k3解密，得到密文d1；对d1再用k2做加密处理，得到密文d2；再对d2用k1做解密处理，得到最终明文。

**填充**\
填充方式采用PKCS5Padding，代码照搬如下：

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

填充过程就是要把明文长度凑成8的整数倍，少几个就填充几个对应的数字。如少4个字节才满8的倍数，那就填充4个0x04；如果明文刚好是8的倍数，就要再填充8个0x08。\
解密后，要把填充的数据删除，就取最后一个字节的值，按值删掉最后几个字节。(设计的很巧妙)

代码
----

完整的代码：

    package tripledesecb
    import (
        "bytes"
        "crypto/des"
        "errors"
        "fmt"
        "golang.org/x/crypto/pbkdf2"
    )

    //ECB PKCS5Padding
    func PKCS5Padding(ciphertext []byte, blockSize int) []byte {
        padding := blockSize - len(ciphertext)%blockSize
        padtext := bytes.Repeat([]byte{byte(padding)}, padding)
        return append(ciphertext, padtext...)
    }

    //ECB PKCS5Unpadding
    func PKCS5Unpadding(origData []byte) []byte {
        length := len(origData)
        unpadding := int(origData[length-1])
        return origData[:(length - unpadding)]
    }

    //Des加密
    func encrypt(origData, key []byte) ([]byte, error) {
        if len(origData) < 1 || len(key) < 1 {
            return nil, errors.New("wrong data or key")
        }
        block, err := des.NewCipher(key)
        if err != nil {
            return nil, err
        }
        bs := block.BlockSize()
        if len(origData)%bs != 0 {
            return nil, errors.New("wrong padding")
        }
        out := make([]byte, len(origData))
        dst := out
        for len(origData) > 0 {
            block.Encrypt(dst, origData[:bs])
            origData = origData[bs:]
            dst = dst[bs:]
        }
        return out, nil
    }

    //Des解密
    func decrypt(crypted, key []byte) ([]byte, error) {
        if len(crypted) < 1 || len(key) < 1 {
            return nil, errors.New("wrong data or key")
        }
        block, err := des.NewCipher(key)
        if err != nil {
            return nil, err
        }
        out := make([]byte, len(crypted))
        dst := out
        bs := block.BlockSize()
        if len(crypted)%bs != 0 {
            return nil, errors.New("wrong crypted size")
        }

        for len(crypted) > 0 {
            block.Decrypt(dst, crypted[:bs])
            crypted = crypted[bs:]
            dst = dst[bs:]
        }

        return out, nil
    }

    //[golang ECB 3DES Encrypt]
    func TripleEcbDesEncrypt(origData, key []byte) ([]byte, error) {
        tkey := make([]byte, 24, 24)
        copy(tkey, key)
        k1 := tkey[:8]
        k2 := tkey[8:16]
        k3 := tkey[16:]

        block, err := des.NewCipher(k1)
        if err != nil {
            return nil, err
        }
        bs := block.BlockSize()
        origData = PKCS5Padding(origData, bs)

        buf1, err := encrypt(origData, k1)
        if err != nil {
            return nil, err
        }
        buf2, err := decrypt(buf1, k2)
        if err != nil {
            return nil, err
        }
        out, err := encrypt(buf2, k3)
        if err != nil {
            return nil, err
        }
        return out, nil
    }

    //[golang ECB 3DES Decrypt]
    func TripleEcbDesDecrypt(crypted, key []byte) ([]byte, error) {
        tkey := make([]byte, 24, 24)
        copy(tkey, key)
        k1 := tkey[:8]
        k2 := tkey[8:16]
        k3 := tkey[16:]
        buf1, err := decrypt(crypted, k3)
        if err != nil {
            return nil, err
        }
        buf2, err := encrypt(buf1, k2)
        if err != nil {
            return nil, err
        }
        out, err := decrypt(buf2, k1)
        if err != nil {
            return nil, err
        }
        out = PKCS5Unpadding(out)
        return out, nil
    }

上面的代码，稍微改一下就可以开放出ecb模式的des算法，我用不上，就没放出来。

刚开始用golang，也是第一次在segmentfault上发文，轻拍！

