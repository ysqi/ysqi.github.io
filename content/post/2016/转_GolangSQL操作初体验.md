
---
date: 2016-12-31T11:33:05+08:00
title: "GolangSQL操作初体验"
description: ""
disqus_identifier: 1485833585102818701
slug: "Golang-SQL-cao-zuo-chu-ti-yan"
source: "https://segmentfault.com/a/1190000006885571"
tags: 
- mysql 
- sql 
- golang 
topics:
- 编程语言与开发
---

https://segmentfault.com/a/

简介
----

Golang 提供了 **database/sql** 包用于对 SQL 的数据库的访问, 在这个包中,
最重要的自然就是 **sql.DB** 了.\
对于 **sql.DB**, 我们需要强调的是, `它并不代表一个数据库连接`,
它是一个已存在的数据库的抽象访问接口. sql.DB 为我们提供了两个重要的功能:

-   sql.DB 通过数据库驱动为我们管理底层数据库连接的打开和关闭操作.

-   sql.DB 为我们管理数据库连接池

有一点需要注意的是, 正因为 sql.DB 是以连接池的方式管理数据库连接,
我们每次进行数据库操作时, 都需要从连接池中取出一个连接,
当操作任务完成时, 我们需要将此连接返回到连接池中,
因此如果我们没有正确地将连接返回给连接池, 那么会造成 db.SQL
打开过多的数据库连接, 使数据库连接资源耗尽.

MySQL 数据库的基本操作
----------------------

### 数据库驱动的导入

有过数据库开发经验的朋友就知道了,
我们需要借助于一个数据库驱动才能和具体的数据库进行连接. 这在 Golang
中也不例外. 例如以 MySQL 数据库为例:

    import (
        "database/sql"
        _ "github.com/go-sql-driver/mysql"
    )

需要注意的是, 通常来说, 我们不应该直接使用驱动所提供的方法, 而是应该使用
sql.DB, 因此在导入 mysql 驱动时, 我们使用了匿名导入的方式(在包路径前添加
**\_**).\
当导入了一个数据库驱动后, 此驱动会自行初始化并注册自己到 Golang 的
database/sql 上下文中, 因此我们就可以通过 database/sql
包提供的方法访问数据库了.

### 数据库的连接

当导入了 MySQL 驱动后, 我们打开数据库连接:

    func main() {
        db, err := sql.Open("mysql",
            "user:password@tcp(127.0.0.1:3306)/test")
        if err != nil {
            log.Fatal(err)
        }
        defer db.Close()
    }

通过 sql.Open 函数, 可以创建一个数据库抽象操作接口, 如果打开成功的话,
它会返回一个 sql.DB 指针.\
sql.Open 函数的签名如下:

    func Open(driverName, dataSourceName string) (*DB, error)

它接收两个参数:

-   driverName, 使用的驱动名. 这个名字其实就是数据库驱动注册到
    database/sql 时所使用的名字.

-   dataSourceName, 第二个数据库连接的链接.
    这个链接包含了数据库的用户名,
    密码, 数据库主机以及需要连接的数据库名等信息.

`需要注意的是, golang 对数据库的连接是延时初始化的(lazy init), 即 sql.Open 并不会立即建立一个数据库的网络连接, 也不会对数据库链接参数的合法性做检验, 它仅仅是初始化一个 sql.DB 对象.`
当我们进行第一次数据库查询操作时, 此时才会真正建立网络连接.\
如果我们想立即检查数据库连接是否可用, 那么可以利用 sql.DB 的 Ping 方法,
例如:

    err = db.Ping()
    if err != nil {
        log.Fatal(err)
    }

> sql.DB 的最佳实践:\
> sql.DB 对象是作为长期生存的对象来使用的, 我们应当避免频繁地调用 Open()
> 和 Close(). 即一般来说, 我们要对一个数据库进行操作时, 创建一个 sql.DB
> 并将其保存起来, 每次操作此数据库时, 传递此 sql.DB 对象即可,
> 最后在需要对此数据库进行访问时, 关闭对应的 sql.DB 对象.

### 数据库的查询

数据库查询的一般步骤如下:

-   调用 db.Query 执行 SQL 语句, 此方法会返回一个 Rows 作为查询的结果

-   通过 rows.Next() 迭代查询数据.

-   通过 rows.Scan() 读取每一行的值

-   调用 db.Close() 关闭查询

例如我们有如下一个数据库表:

    CREATE TABLE `user` (
      `id` bigint(20) NOT NULL AUTO_INCREMENT,
      `name` varchar(20) DEFAULT '',
      `age` int(11) DEFAULT '0',
      PRIMARY KEY (`id`)
    ) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4

我们向其中插入一条记录:

    func insertData(db *sql.DB) {
        rows, err := db.Query(`INSERT INTO user (id, name, age) VALUES (1, "xys", 20)`)
        defer rows.Close()
        if err != nil {
            log.Fatalf("insert data error: %v\n", err)
        }

        var result int
        rows.Scan(&result)
        log.Printf("insert result %v\n", result)
    }

通过调用 db.Query, 我们执行了一条 INSERT 语句插入了一条数据.
当执行完毕后, 首先需要做的是检查语句是否执行成功, 当没有错误时, 就通过
rows.Scan 获取执行的结果. 因为 INSERT 返回的是插入的数据的行数,
因此我们打印的语句就是 "insert result 0".

接下来如法炮制, 我们从数据库中将插入的数据取出:

    func selectData(db *sql.DB) {
        var id int
        var name string
        var age int
        rows, err := db.Query(`SELECT * From user where id = 1`)
        if err != nil {
            log.Fatalf("insert data error: %v\n", err)
            return
        }
        for rows.Next() {
            rows.Scan(&id, &age, &name)
            if err != nil {
                log.Fatal(err)
            }
            log.Printf("get data, id: %d, name: %s, age: %d", id, name, age)
        }

        err = rows.Err()
        if err != nil {
            log.Fatal(err)
        }
    }

上面的代码的流程基本上没有很大的差别, 不过我们需要注意一点的是
**rows.Scan** 参数的顺序很重要, 需要和查询的结果的 column 对应. 例如
"SELECT \* From user where id = 1" 查询的行的 column 顺序是 "id, name,
age", 因此 rows.Scan 也需要按照此顺序 rows.Scan(&id, &name, &age),
不然会造成数据读取的错位.

> `注意`:
>
> 1.  对于每个数据库操作都需要检查是否有错误返回
>
> 2.  每次 db.Query 操作后, 都需要调用 rows.Close(). 因为 db.Query()
>     会从数据库连接池中获取一个连接, 如果我们没有调用
>     rows.Close(), 则此连接会一直被占用. 因此通常我们使用
>     **defer rows.Close()** 来确保数据库连接可以正确放回到连接池中.
>
> 3.  多次调用 rows.Close() 不会有副作用, 因此即使我们已经显示地调用了
>     rows.Close(), 我们还是应该使用 **defer rows.Close()** 来关闭查询.
>
完整的例子如下:

    func insertData(db *sql.DB) {
        rows, err := db.Query(`INSERT INTO user (id, name, age) VALUES (1, "xys", 20)`)
        defer rows.Close()

        if err != nil {
            log.Fatalf("insert data error: %v\n", err)
        }

        var result int
        rows.Scan(&result)
        log.Printf("insert result %v\n", result)
    }

    func selectData(db *sql.DB) {
        var id int
        var name string
        var age int
        rows, err := db.Query(`SELECT id, name, age From user where id = 1`)
        if err != nil {
            log.Fatalf("insert data error: %v\n", err)
            return
        }
        for rows.Next() {
            err = rows.Scan(&id, &name, &age)
            if err != nil {
                log.Fatal(err)
            }
            log.Printf("get data, id: %d, name: %s, age: %d", id, name, age)
        }

        err = rows.Err()
        if err != nil {
            log.Fatal(err)
        }
    }

    func main() {
        db, err := sql.Open("mysql", "root:root@tcp(127.0.0.1:3306)/test")

        defer db.Close()

        if err != nil {
            fmt.Printf("connect to db 127.0.0.1:3306 error: %v\n", err)
            return
        }

        insertData(db)

        selectData(db)
    }

### 预编译语句(Prepared Statement)

预编译语句(PreparedStatement)提供了诸多好处, 因此我们在开发中尽量使用它.
下面列出了使用预编译语句所提供的功能:

-   PreparedStatement 可以实现自定义参数的查询

-   PreparedStatement 通常来说, 比手动拼接字符串 SQL 语句高效.

-   PreparedStatement 可以防止SQL注入攻击

下面我们将上一小节的例子使用 Prepared Statement 来改写:

    func deleteData(db *sql.DB) {
        stmt, _ := db.Prepare(`DELETE FROM user WHERE id = ?`)

        rows, err := stmt.Query(1)
        defer stmt.Close()

        rows.Close()
        if err != nil {
            log.Fatalf("delete data error: %v\n", err)
        }

        rows, err = stmt.Query(2)
        rows.Close()
        if err != nil {
            log.Fatalf("delete data error: %v\n", err)
        }
    }

    func insertData(db *sql.DB) {
        stmt, _ := db.Prepare(`INSERT INTO user (id, name, age) VALUES (?, ?, ?)`)

        rows, err := stmt.Query(1, "xys", 20)
        defer stmt.Close()

        rows.Close()
        if err != nil {
            log.Fatalf("insert data error: %v\n", err)
        }

        rows, err = stmt.Query(2, "test", 19)
        var result int
        rows.Scan(&result)
        log.Printf("insert result %v\n", result)
        rows.Close()
    }

    func selectData(db *sql.DB) {
        var id int
        var name string
        var age int
        stmt, _ := db.Prepare(`SELECT * From user where age > ?`)

        rows, err := stmt.Query(10)

        defer stmt.Close()
        defer rows.Close()

        if err != nil {
            log.Fatalf("select data error: %v\n", err)
            return
        }
        for rows.Next() {
            err = rows.Scan(&id, &name, &age)
            if err != nil {
                log.Fatal(err)
            }
            log.Printf("get data, id: %d, name: %s, age: %d", id, name, age)
        }

        err = rows.Err()
        if err != nil {
            log.Fatal(err)
        }
    }

    func main() {
        db, err := sql.Open("mysql", "root:root@tcp(127.0.0.1:3306)/test")

        defer db.Close()

        if err != nil {
            fmt.Printf("connect to db 127.0.0.1:3306 error: %v\n", err)
            return
        }

        deleteData(db)

        insertData(db)

        selectData(db)
    }

