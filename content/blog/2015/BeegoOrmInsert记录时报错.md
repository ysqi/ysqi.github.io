---
date: 2015-09-18T08:22:13
title: BeegoOrmInsert记录时报错
slug: beego-orm-insert-error
topics:
- 开发
tags: 
  - Go
  - sql
  - beego
  - 问题
disqus_identifier: 100018
---

前几天在使用Beego的ORM往Mysql中写入数据时失败，这里记录下原因。

<!-- more -->

当时代码是这样写的
```Go
o,err := orm.NewOrm()
if err!=nil{
    return err
}

record := &models.User{}
err=o.Insert(&record)
if err!=nil {
    return err
}
```
在执行此代码报错，错误信息：
```Shell
<Ormer> table: `.` not found, maybe not RegisterModel
```

上面代码的错误性是否非常明显的，是因为本身`record`是指针对性，而在Insert时又传入的是指针的指针`&record`

#### 来看下Beego ORM 内部是如何获取表名的

源代码如下：
```Go
// get model info and model reflect value
func (o *orm) getMiInd(md interface{}, needPtr bool) (mi *modelInfo, ind reflect.Value) {
	val := reflect.ValueOf(md)
	ind = reflect.Indirect(val)
	typ := ind.Type()
	if needPtr && val.Kind() != reflect.Ptr {
		panic(fmt.Errorf("<Ormer> cannot use non-ptr model struct `%s`", getFullName(typ)))
	}
	name := getFullName(typ)
	if mi, ok := modelCache.getByFN(name); ok {
		return mi, ind
	}
	panic(fmt.Errorf("<Ormer> table: `%s` not found, maybe not RegisterModel", name))
}

// insert model data to database
func (o *orm) Insert(md interface{}) (int64, error) {
	mi, ind := o.getMiInd(md, true)
	id, err := o.alias.DbBaser.Insert(o.db, mi, ind, o.alias.TZ)
	if err != nil {
		return id, err
	}

	o.setPk(mi, ind, id)

	return id, nil
}
```

通过反射查找record类型，并获取该类型的名称,但因为传入的并非对性的指针，导致获取相应类型的名称是出差。

如下测试：
<iframe style="border:1px solid" src="https://wide.b3log.org/playground/d2fcf822a9fd31da343f482a7969ac8b.go?embed=true" width="100%" height="600"></iframe>


综合上述，在传入对象时，只能转入对象的指针，这样才能保证通过反射获取相关数据是正确合理的。

上面就是解决在使用beego的ORM框架时报错的分析经过。
