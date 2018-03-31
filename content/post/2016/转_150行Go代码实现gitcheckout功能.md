
---
date: 2016-12-31T11:35:07+08:00
title: "150行Go代码实现gitcheckout功能"
description: ""
disqus_identifier: 1485833707574981000
slug: "150hang-Godai-ma-shi-xian-git-checkoutgong-neng"
source: "https://segmentfault.com/a/1190000000403067"
tags: 
- git 
- golang 
categories:
- 编程语言与开发
---

由于历史原由，git一直是被黑成比较难用的版本控制器。其实近年来git的用户界面已经被简化的非常简单了，配上github、bitbucket等[hosting](https://git.wiki.kernel.org/index.php/GitHosting),已接近完美。\
git其实挺简单的，本文用了约150行golang代码实现了git
checkout功能，阅读代码之前，您应该读过[《Git
Pro》中的git内部原理](http://git-scm.com/book/en/Git-Internals)一节。

1. 数据定义：
=============

    type blob struct {
        sha1     string
        filename string
    }
    type tree struct {
        b     []*blob
        name  string
        child []*tree
    }
    type commit struct {
        sha1   string
        tree   *tree
        parent *commit
    }

其中blob定义一个文件 ,sha1是文件的sha1值,filename是不包括路径的文件名。\
tree定义相当于目录，b是目录下的文件，name是当前目录名，不包括父路径，child是目录下的目录。\
commit是一次提交，sha1是提交的sha1值，tree指向一要树形的根节点，沿此根结点可以检出所有的文件。\
对照下面这副图就比较容易理解:\

2. 工具函数
===========

    func readSha1FileReader(sha1 string) (reader io.Reader, err error) {

        f, err := os.Open(getSha1FilePath(sha1))
        if err != nil{
            return
        }
        return zlib.NewReader(f)
    }

    func readSha1FileContent(sha1 string) (content []byte, err error) {

        if reader, err := readSha1FileReader(sha1);err == nil{
            buf := new(bytes.Buffer)
            buf.ReadFrom(reader)
            content = buf.Bytes()
        }
        return
    }

    func getSha1FileContentBody(content []byte) []byte {
        i := bytes.IndexByte(content, 0)
        return content[i+1:]
    }

    func getSha1FilePath(sha1 string) string {
        return ".git/objects/" + sha1[0:2] + "/" + sha1[2:]
    }

-   getSha1FilePath 根据sha1值取得对应的object路径。
-   readSha1FileReader
    根据sha1值读取object内容，注意原始内容是经过压缩的，调用zlib是为了对其解压。
-   readSha1FileContent 对readSha1FileReader的一层封装，返回的是byte数组
-   getSha1FileContentBody
    返回object的内容的body部分，header的内容我们直接忽略了

> 上面提到的object是位于路径.git/objects/路径下的文件

3. 构建树
=========

    func BuildTree(sha1 string) *tree {
        all, err := readSha1FileContent(sha1)
        if err != nil {
            log.Fatal("BuildTree error:", err)
            return nil
        }

        content := getSha1FileContentBody(all)
        start := 0
        tree := tree{}
        for i := 0; i < len(content); {
            if content[i] == 0 {
                line := content[start : i+21]
                _type := line[:6]
                id := line[i-start+1:]
                obj_sha1 := fmt.Sprintf("%x", id)
                switch string(_type[0:3]) {
                //BLOB
                case "100":
                    name := string(line[7 : i-start])
                    b := blob{sha1: obj_sha1, filename: name}
                    tree.b = append(tree.b, &b)
                    break
                //TREE
                case "400":
                    name := string(line[6 : i-start])
                    child := BuildTree(obj_sha1)
                    child.name = name
                    tree.child = append(tree.child, child)
                    break
                }
                i += 21
                start = i
            } else {
                i++
            }
        }
        return &tree
    }

以上便是检出git的库的核心函数，其入参是一次Commit的Sha1值。要理解这个函数，需要知道[tree文件的格式定义](http://icattlecoder.qiniudn.com/Git_Data_Formats.txt.zip)（《Git
Pro》一书中没有）：

    <TREE>
        :   _deflate_( <OBJECT_HEADER> <TREE_CONTENTS> )
        |   <COMPACT_OBJECT_HEADER> _deflate_( <TREE_CONTENTS> )
        ;

    <TREE_CONTENTS>
        :   <TREE_ENTRIES>
        ;

    <TREE_ENTRIES>
        # Tree entries are sorted by the byte sequence that comprises
        # the entry name. However, for the purposes of the sort
        # comparison, entries for tree objects are compared as if the
        # entry name byte sequence has a trailing ASCII '/' (0x2f).
        :   ( <TREE_ENTRY> )*
        ;

    <TREE_ENTRY>
        # The type of the object referenced MUST be appropriate for
        # the mode. Regular files and symbolic links reference a BLOB
        # and directories reference a TREE.
        :   <OCTAL_MODE> <SP> <NAME> <NUL> <BINARY_OBJ_ID>
        ;

通过`getSha1FileContentBody`函数即可取得`TREE_CONTENTS`,`TREE_CONTENTS`包括一个或多个`TREE_ENTRY`,`TREE_ENTRY`的格式如下：

    <OCTAL_MODE> <SP> <NAME> <NUL> <BINARY_OBJ_ID>

`OCTAL_MODE`的前三个字节定义了object类型，`"100"`为Blob，`"400"`为Tree，如果是Tree对像，则需要递归调用。

4. 检出文件
===========

BuildTree根据指定的Commit构建出所有文件形成的树型结构，有了它，就很容易检出文件。

    func (b *blob) checkout(prefix string) {
        if content, err := readSha1FileContent(b.sha1);err!=nil{
            log.Fatal("blob checkout error:", err)
        }else{
            body := getSha1FileContentBody(content)
            filename := prefix + "/" + b.filename
            log.Println("WriteFile:",filename)
            if err = ioutil.WriteFile(filename, body, 0644);err!=nil{
                log.Fatal("blob checkout error:", err)
            }
        }
    }

    func (t *tree) checkout(path string) {
        if _, err := os.Stat(path); os.IsNotExist(err) {
            log.Println("Mkdir:",path)
            if err := os.Mkdir(path, 0777); err != nil {
                log.Fatal("mkdir error:", err)
                return
            }
        }
        for _, v := range t.b {
            v.checkout(path)    //BLOB checkout
        }
        for _, v := range t.child {
            v.checkout(path + "/" + v.name)     //TREE checkout
        }
    }

    func (c *commit) CheckOut() {
        if pwd, err := os.Getwd();err==nil{
            c.tree.checkout(pwd)
        }else{
            log.Fatal("commit checkout error:", err)
        }
    }

以上三个函数的调用顺序为`commit.CheckOUt`-&gt;`tree.checkout`-&gt;`blob.checkout`.\
如果有目录，`tree.checkout`会生成目录。`blob.checkout`则会生成文件。

5. 示例
=======

完整的代码见[这里](http://github.com/icattlecoder/gogit.git)

编译
----

    ~/tmp$ git clone git@github.com:icattlecoder/gogit.git
    ~/tmp$ cd gogit
    ~/tmp$ go build gogit.go

检出示例库的代码
----------------

    ~/tmp$ git clone git@github.com:icattlecoder/jsfiddle.git
    ~/tmp$ cd jsfiddle
    ~/tmp$ rm -Rf ajaxupload/ formupload/ resumbleupload/ uptoken/
    ~/tmp$ mv ../gogit/gogit .
    ~/tmp$ ./gogit
    ~/tmp$ ls
    ajaxupload     formupload   resumbleupload uptoken

在运行gogit之前，删除了本地文件，而运行gogit后，所有文件又恢复了，因此实现了`git checkout`功能。

**注意：本文的git checkout不能处理压缩过的git库**

