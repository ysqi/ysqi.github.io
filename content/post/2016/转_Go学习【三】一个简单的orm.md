
---
date: 2016-12-31T11:33:23+08:00
title: "Go学习【三】一个简单的orm"
description: ""
disqus_identifier: 1485833603535052299
slug: "Goxue-xi-【san-】yi-ge-jian-chan-de-orm"
source: "https://segmentfault.com/a/1190000005887338"
tags: 
- golang 
- orm 
categories:
- 编程语言与开发
---

https://segmentfault.com/a/

**碎语：**（请自行跳过）

距离上篇文章发布也有半个月的时间了

然后这半个月 也没有用go写项目或继续学习 只能算简单的入门了
以后如果有需要的话 或许会深入的去了解一下这门语言 与各种常用的类库
精力有限 把自己之前尝试写一个简单orm的一些片段与大家分享

也许在月底会尝试用go去写一个爬虫 留待下篇文章分享

**前言：**

关于go的orm框架有许多不错的
为什么自己想写一个原因无非就是想通过写orm的过程来对自己学习的知识做一个阶段性的锻炼与检验
当然目前写的这个只能算是一个玩具 如果你能在这个玩具里有所收获
那便是最好的了

**正文：**

技术需求：对反射有一些了解

反射可以简单的划分为以下几步：\
1获取对象\
t := reflect.TypeOf(arg) \#获取类型\
v := reflect.ValueOf(arg) \#获取值\
2获取字段（值 或 名称）\
vf := v.Field(i)\
fv := v.Field(i).Interface() \#获取值

3设值\
vf.CanSet() \#判断是否可以设值\
vf.setxxx(xx)

然后插入 删除 更新 可以用相同的方法实现 只需要使用到 1 2 步

查询会用到第 3 步

有了上面的这些知识我们就可以尝试写出一个orm框架了 闲话不多说上代码

插入 删除 与 更新省略

    func insert(arg interface{}) (sql []byte, params []interface{}, kIdstr string, err error) {
        if arg == nil {
            err = errors.New("expected a pointer to a struct")
            return
        }
        var values []byte
        //获取字段
        paramsMap, tableName, kIdcolumn, kIdstr := elem(arg)
        //拼装sql语句
        sql = append(sql, []byte("INSERT "+tableName+" ( ")...)
        values = append(values, []byte(" VALUES (")...)
        for colum, v := range paramsMap {
            if colum != kIdcolumn {
                sql = append(sql, []byte(" "+colum+" ,")...)
                values = append(values, []byte(" ? ,")...)
                //获取对应参数
                params = append(params, v)
            }

        }
        //拼装成功
        sql = append(sql[:len(sql)-1], ')')
        values = append(values[:len(values)-1], ')')
        sql = append(sql, values...)
        log.Println("===>", string(sql), params)
        return
    }

    func elem(arg interface{}) (paramsMap map[string]interface{}, tableName, kIdcolumn, kIdFiled string) {
        t := reflect.TypeOf(arg)
        v := reflect.ValueOf(arg).Elem()
        //获取表名
        if t.Kind() == reflect.Ptr {
            t = t.Elem()
            tableName = t.Name()
            log.Println("===> tableName:", tableName)
        }
        //获取字段
        num := v.NumField()
        paramsMap = make(map[string]interface{}, num)
        for i := 0; i < num; i++ {
            //inteface 方法 非导出字段无法使用
            if v.Field(i).CanInterface() {
                var tn string
                //获取字段的值
                fv := v.Field(i).Interface()
                // 以后可以改为tag 进行更好的扩展
                tf := t.Field(i)
                dC := tf.Tag.Get(dbColumn)
                if dC == "" {
                    dC = tf.Tag.Get(dbID)
                    if dC == "" {
                        tn = tf.Name
                    } else {
                        kIdFiled = tf.Name
                        tn = dC
                        kIdcolumn = tn
                        dT := tf.Tag.Get(dbTableName)
                        if dT != "" {
                            tableName = dT
                        }
                    }
                } else {
                    tn = dC
                }
                paramsMap[tn] = fv
            } else {
                //此处省略判断类型进行匹配
                //....
            }
        }
        return
    }

    func (this *Mysql) Insert(obj interface{}) error {
        query, param, kIdstr, err := insert(obj)
        if err != nil {
            return err
        }
        result, err := this.Exec(string(query), param...)
        if err != nil {
            return err
        }
        num, err := result.LastInsertId()
        if err != nil {
            return err
        }
        v := reflect.ValueOf(obj).Elem()
        vv := v.FieldByName(kIdstr)
        if vv.CanSet() {
            vv.SetInt(num)
        }
        return nil

    }

查询（目前只支持查询单条数据 下一版会支持多条）

    func selectOne(arg interface{}) (sql []byte, params []interface{}, err error) {
        if arg == nil {
            err = errors.New("expected a pointer to a struct")
            return
        }
        //获取字段
        paramsMap, tableName, kIdcolumn, _ := elem(arg)
        var sqlWhere string
        //拼装sql语句
        sql = append(sql, []byte("SELECT ")...)
        for colum, v := range paramsMap {
            sql = append(sql, []byte(" "+colum+" ,")...)
            if colum == kIdcolumn {
                sqlWhere = " WHERE " + colum + " = ? "
                params = append(params, v)
            }

        }
        sql = sql[:len(sql)-1]
        sql = append(sql, []byte("FROM "+tableName)...)
        sql = append(sql, []byte(sqlWhere)...)
        //拼装成功
        log.Println("===>", string(sql), params)
        return
    }

    /*2016/06/19/22:35*/
    func (this *Mysql) selectOne(obj interface{}, query string, params ...interface{}) (*sql.Rows, error) {
        if len(params) == 0 {
            return nil, fmt.Errorf("params is nil")
        }
        tx, err := this.DB.Begin()
        if err != nil {
            return nil, err
        }
        rows, err := tx.Query(query, params...)
        if err != nil {
            return nil, err
        }
        //进行设值 字段与数据库对应关系
        filedCMap := filedColumnMapper(obj)
        //设值 需要更多详细操作
        setFiled(obj, rows, filedCMap)

        err = tx.Commit()
        if err != nil {
            return nil, err
        }
        return rows, nil
    }

    //设值字段与数据的映射关系
    func filedColumnMapper(obj interface{}) map[string]string {
        t := reflect.TypeOf(obj).Elem()
        v := reflect.ValueOf(obj).Elem()
        num := t.NumField()
        //获取 字段 对应关系 ----此处应拿到buil-sql中
        filedCMap := make(map[string]string, num)
        for i := 0; i < num; i++ {
            //inteface 方法 非导出字段无法使用
            if v.Field(i).CanInterface() {
                var tn string
                // 以后可以改为tag 进行更好的扩展
                tf := t.Field(i)
                kC := tf.Tag.Get(dbColumn)
                if kC == "" {
                    kC = tf.Tag.Get(dbColumn)
                    if kC == "" {
                        tn = tf.Name
                    } else {
                        tn = kC
                    }
                } else {
                    tn = kC
                }
                filedCMap[tn] = tf.Name
            } else {
                //此处省略判断类型进行匹配
                //....
            }
        }
        return filedCMap
    }

    //为字段设值
    func setFiled(obj interface{}, rows *sql.Rows, filedCMap map[string]string) {
        //获取键值对
        cols, _ := rows.Columns()
        buff := make([]interface{}, len(cols)) // 临时slice
        data := make([]string, len(cols))      // 存数据slice
        for i, _ := range buff {
            buff[i] = &data[i]
        }
        for rows.Next() {
            rows.Scan(buff...) // ...是必须的
        }
        t := reflect.TypeOf(obj).Elem()
        v := reflect.ValueOf(obj).Elem()
        for k, values := range data {
            //根据 colum获取字段名称
            filedName := filedCMap[cols[k]]
            //进行设值
            if _, ok := t.FieldByName(filedName); ok {
                vft := v.FieldByName(filedName)
                switch vft.Kind() {
                case reflect.Int, reflect.Int8, reflect.Int16, reflect.Int32, reflect.Int64:
                    val, err := strconv.ParseInt(values, 10, 64)
                    if err == nil {
                        vft.SetInt(val)
                    }
                case reflect.String:
                    vft.SetString(values)
                case reflect.Uint, reflect.Uint8, reflect.Uint16, reflect.Uint32, reflect.Uint64:
                    val, err := strconv.ParseUint(values, 10, 64)
                    if err == nil {
                        vft.SetUint(val)
                    }
                case reflect.Float32, reflect.Float64:
                    val, err := strconv.ParseFloat(values, 64)
                    if err == nil {
                        vft.SetFloat(val)
                    }
                case reflect.Bool:
                    val, err := strconv.ParseBool(values)
                    if err == nil {
                        vft.SetBool(val)
                    }

                }
            }

        }
    }

    func (this *Mysql) SelectOne(obj interface{}) error {
        query, param, err := selectOne(obj)
        if err != nil {
            return err
        }
        _, err = this.selectOne(obj, string(query), param...)

        if err != nil {
            return err
        }
        return nil
    }

晚些时间会把代码上传到github 希望大家指出不足之处 和大家共同进步

