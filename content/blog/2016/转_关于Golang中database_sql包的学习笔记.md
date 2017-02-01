
---
date: 2016-12-31T11:34:24+08:00
title: "关于Golang中database_sql包的学习笔记"
description: ""
disqus_identifier: 1485833664566288124
slug: "guan-yu-Golangzhong-database_sqlbao-de-xue-xi-bi-ji"
source: "https://segmentfault.com/a/1190000003036452"
tags: 
- sql 
- 数据库 
- golang 
topics:
- 编程语言与开发
---

> 因为最近在学习Go，所以找了revel这个框架来学习，感觉和php的面向对象有很大不同。revel没有提供db
> mapping的组件，所以在github上搜了很多ORM来学习，在`jmoiron/sqlx`中发现了一篇比较详细介绍`database/sql`这个包的文章，拿来和大家分享。本文并不是按字句的翻译，如果哪里表述不清楚建议阅读原文
> [原文地址](http://go-database-sql.org/index.html)

概述
----

`sql.DB`不是一个连接，它是数据库的抽象接口。它可以根据driver打开关闭数据库连接，管理连接池。正在使用的连接被标记为繁忙，用完后回到连接池等待下次使用。所以，如果你没有把连接释放回连接池，会导致过多连接使系统资源耗尽。

使用DB
------

### 导入driver

这里使用的是[MySQL drivers](https://github.com/go-sql-driver/mysql)

    import (
        "database/sql"
        _ "github.com/go-sql-driver/mysql"
    )

### 连接DB

    func main() {
        db, err := sql.Open("mysql",
            "user:password@tcp(127.0.0.1:3306)/hello")
        if err != nil {
            log.Fatal(err)
        }
        defer db.Close()
    }

`sql.Open`的第一个参数是driver名称，第二个参数是driver连接数据库的信息，各个driver可能不同。DB不是连接，并且只有当需要使用时才会创建连接，如果想立即验证连接，需要用`Ping()`方法，如下：

    err = db.Ping()
    if err != nil {
        // do something here
    }

sql.DB的设计就是用来作为长连接使用的。不要频繁Open,
Close。比较好的做法是，为每个不同的datastore建一个DB对象，保持这些对象Open。如果需要短连接，那么把DB作为参数传入function，而不要在function中Open,
Close。

### 读取DB

如果方法包含`Query`，那么这个方法是用于查询并返回rows的。其他情况应该用`Exec()`。

    var (
        id int
        name string
    )
    rows, err := db.Query("select id, name from users where id = ?", 1)
    if err != nil {
        log.Fatal(err)
    }
    defer rows.Close()
    for rows.Next() {
        err := rows.Scan(&id, &name)
        if err != nil {
            log.Fatal(err)
        }
        log.Println(id, name)
    }
    err = rows.Err()
    if err != nil {
        log.Fatal(err)
    }

上面代码的过程为：`db.Query()`表示向数据库发送一个query，`defer rows.Close()`非常重要，遍历rows使用`rows.Next()`，
把遍历到的数据存入变量使用`rows.Scan()`,
遍历完成后检查error。有几点需要注意：

1.  检查遍历是否有error
2.  结果集(rows)未关闭前，底层的连接处于繁忙状态。当遍历读到最后一条记录时，会发生一个内部EOF错误，自动调用`rows.Close()`，但是如果提前退出循环，rows不会关闭，连接不会回到连接池中，连接也不会关闭。所以手动关闭非常重要。`rows.Close()`可以多次调用，是无害操作。

### 单行Query

err在`Scan`后才产生，所以可以如下写：

    var name string
    err = db.QueryRow("select name from users where id = ?", 1).Scan(&name)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(name)

修改数据，事务
--------------

一般用Prepared Statements和`Exec()`完成`INSERT`, `UPDATE`,
`DELETE`操作。

    stmt, err := db.Prepare("INSERT INTO users(name) VALUES(?)")
    if err != nil {
        log.Fatal(err)
    }
    res, err := stmt.Exec("Dolly")
    if err != nil {
        log.Fatal(err)
    }
    lastId, err := res.LastInsertId()
    if err != nil {
        log.Fatal(err)
    }
    rowCnt, err := res.RowsAffected()
    if err != nil {
        log.Fatal(err)
    }
    log.Printf("ID = %d, affected = %d\n", lastId, rowCnt)

### 事务

`db.Begin()`开始事务，`Commit()` 或
`Rollback()`关闭事务。`Tx`从连接池中取出一个连接，在关闭之前都是使用这个连接。Tx不能和DB层的`BEGIN`,
`COMMIT`混合使用。

如果你需要通过多条语句修改连接状态，你必须使用Tx，例如：

-   创建仅对单个连接可见的临时表
-   设置变量，例如`SET @var := somevalue`
-   改变连接选项，例如字符集，超时

Prepared Statements
-------------------

### Prepared Statements and Connection

在数据库层面，Prepared
Statements是和单个数据库连接绑定的。客户端发送一个有占位符的statement到服务端，服务器返回一个statement
ID，然后客户端发送ID和参数来执行statement。

在GO中，连接不直接暴露，你不能为连接绑定statement，而是只能为DB或Tx绑定。`database/sql`包有自动重试等功能。当你生成一个Prepared
Statement

1.  自动在连接池中绑定到一个空闲连接
2.  `Stmt`对象记住绑定了哪个连接
3.  执行`Stmt`时，尝试使用该连接。如果不可用，例如连接被关闭或繁忙中，会自动re-prepare，绑定到另一个连接。

这就导致在高并发的场景，过度使用statement可能导致statement泄漏，statement持续重复prepare和re-prepare的过程，甚至会达到服务器端statement数量上限。

某些操作使用了PS，例如`db.Query(sql, param1, param2)`,
并在最后自动关闭statement。

有些场景不适合用statement：

1.  数据库不支持。例如Sphinx，MemSQL。他们支持MySQL wire protocol,
    但不支持"binary" protocol。
2.  statement不需要重用很多次，并且有其他方法保证安全。[例子](https://vividcortex.com/blog/2014/11/19/analyzing-prepared-statement-performance-with-vividcortex/)

### 在Transaction中使用PS

PS在Tx中唯一绑定一个连接，不会re-prepare。

Tx和statement不能分离，在DB中创建的statement也不能在Tx中使用，因为他们必定不是使用同一个连接使用Tx必须十分小心，例如下面的代码：

    tx, err := db.Begin()
    if err != nil {
        log.Fatal(err)
    }
    defer tx.Rollback()
    stmt, err := tx.Prepare("INSERT INTO foo VALUES (?)")
    if err != nil {
        log.Fatal(err)
    }
    defer stmt.Close() // danger!
    for i := 0; i < 10; i++ {
        _, err = stmt.Exec(i)
        if err != nil {
            log.Fatal(err)
        }
    }
    err = tx.Commit()
    if err != nil {
        log.Fatal(err)
    }
    // stmt.Close() runs here!

`*sql.Tx`一旦释放，连接就回到连接池中，这里stmt在关闭时就无法找到连接。所以必须在Tx
commit或rollback之前关闭statement。

处理Error
---------

### 循环Rows的Error

如果循环中发生错误会自动运行`rows.Close()`，用`rows.Err()`接收这个错误，Close方法可以多次调用。循环之后判断error是非常必要的。

    for rows.Next() {
        // ...
    }
    if err = rows.Err(); err != nil {
        // handle the error here
    }

### 关闭Resultsets时的error

如果你在rows遍历结束之前退出循环，必须手动关闭Resultset，并且接收error。

    for rows.Next() {
        // ...
        break; // whoops, rows is not closed! memory leak...
    }
    // do the usual "if err = rows.Err()" [omitted here]...
    // it's always safe to [re?]close here:
    if err = rows.Close(); err != nil {
        // but what should we do if there's an error?
        log.Println(err)
    }

### QueryRow()的error

    var name string
    err = db.QueryRow("select name from users where id = ?", 1).Scan(&name)
    if err != nil {
        log.Fatal(err)
    }
    fmt.Println(name)

如果id为1的不存在，err为sql.ErrNoRows，一般应用中不存在的情况都需要单独处理。此外，Query返回的错误都会延迟到Scan被调用，所以应该写成如下代码：

    var name string
    err = db.QueryRow("select name from users where id = ?", 1).Scan(&name)
    if err != nil {
        if err == sql.ErrNoRows {
            // there were no rows, but otherwise no error occurred
        } else {
            log.Fatal(err)
        }
    }
    fmt.Println(name)

把空结果当做Error处理是为了强行让程序员处理结果为空的情况

### 分析数据库Error

各个数据库处理方式不太一样，mysql为例：

    if driverErr, ok := err.(*mysql.MySQLError); ok { 
        // Now the error number is accessible directly
        if driverErr.Number == 1045 {
            // Handle the permission-denied error
        }
    }

`MySQLError`,
`Number`都是DB特异的，别的数据库可能是别的类型或字段。这里的数字可以替换为常量，例如这个包
[MySQL error numbers maintained by
VividCortex](https://github.com/VividCortex/mysqlerr)

### 连接错误

NULL值处理
----------

简单说就是设计数据库的时候不要出现null，处理起来非常费力。Null的type很有限，例如没有`sql.NullUint64`;
null值没有默认零值。

    for rows.Next() {
        var s sql.NullString
        err := rows.Scan(&s)
        // check err
        if s.Valid {
           // use s.String
        } else {
           // NULL value
        }
    }

未知Column
----------

`rows.Columns()`的使用，用于处理不能得知结果字段个数或类型的情况，例如：

    cols, err := rows.Columns()
    if err != nil {
        // handle the error
    } else {
        dest := []interface{}{ // Standard MySQL columns
            new(uint64), // id
            new(string), // host
            new(string), // user
            new(string), // db
            new(string), // command
            new(uint32), // time
            new(string), // state
            new(string), // info
        }
        if len(cols) == 11 {
            // Percona Server
        } else if len(cols) > 8 {
            // Handle this case
        }
        err = rows.Scan(dest...)
        // Work with the values in dest
    }

    cols, err := rows.Columns() // Remember to check err afterwards
    vals := make([]interface{}, len(cols))
    for i, _ := range cols {
        vals[i] = new(sql.RawBytes)
    }
    for rows.Next() {
        err = rows.Scan(vals...)
        // Now you can check each element of vals for nil-ness,
        // and you can use type introspection and type assertions
        // to fetch the column into a typed variable.
    }

关于连接池
----------

1.  避免错误操作，例如LOCK TABLE后用
    INSERT会死锁，因为两个操作不是同一个连接，insert的连接没有table
    lock。
2.  当需要连接，且连接池中没有可用连接时，新的连接就会被创建。
3.  默认没有连接上限，你可以设置一个，但这可能会导致数据库产生错误“too
    many connections”
4.  `db.SetMaxIdleConns(N)`设置最大空闲连接数
5.  `db.SetMaxOpenConns(N)`设置最大打开连接数
6.  长时间保持空闲连接可能会导致db timeout


