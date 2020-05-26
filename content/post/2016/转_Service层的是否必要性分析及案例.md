
---
date: 2016-12-31T11:33:22+08:00
title: "Service层的是否必要性分析及案例"
description: ""
disqus_identifier: 1485833602918246203
slug: "Serviceceng-de-shi-fou-bi-yao-xing-fen-xi-ji-an-li"
source: "https://segmentfault.com/a/1190000005928036"
tags:
- golang
- rust
- asp.net
- c++
- php
categories:
- 编程语言与开发
---

序言
----

此前，我看过这样的一个提问“Yii2框架中，有必要再分离service层么？”，从别人的回答中，自己也收获了答案，但我觉得还需要有个活生生的粟子，才具有更加清晰明了和强有力的说服力。如对我的实战经历感兴趣的继续往下看，喜欢的还可以点击推荐和收藏。在举粟子前，我先讲讲service是什么？有什么作用吧？免得还有人糊涂。\
1、service是什么？\
在面向OO的系统里，service就是biz
manager，在面向过程的系统里service就是TS脚本。\
2、service有什么作用？\
service层的作用就是把这些需要多个model参与的复杂业务逻辑单独封装出来，这些model之间不再发生直接的依赖，而是在service层内协同完成逻辑。service层的第一个目的其实就是对model层进行解耦。

需求分析
--------

1、在Yii2框架中建立service层，专门处理公共且复杂的业务逻辑。

效果图
------

1、在common下建立个service层。\

2、部分公共数据处理逻辑（主要的数据处理都写在这里）。\

代码分析
--------

1、在commonservice下写个CluesBranchService.php文件，CluesBranchService类继承本模块主要的models类Chance。凡是关于Chance的公共业务逻辑都往这个文件里写。

    namespace common\service;

    use Yii;
    use api\modules\v1\models\Sales;
    use api\modules\v1\chance\models\Chance;

    /**
     * //下属的线索公共数据处理逻辑
     */
    class CluesBranchService extends Chance
    {
        //下属的线索列表
        public static function getIndex()
        {
            $SalesModel = new Sales();
            $uids = $SalesModel->sevenChild(Yii::$app->user->id);
            if(count($uids)){
                $query = Chance::find()->where(['in','owner_id',$uids]);
            }else{
                $query = Chance::find()->where(['owner_id'=>'-1']);
            }
           return $query;
        }
    }

2、Controllers里调用。

    use common\service\CluesBranchService;

    $query = CluesBranchService::getIndex();

注释：这里返回的是\$query，而不是查询的结果，用过Yii2的都知道列表实现分页用的是ActiveDataProvider，不需要查出结果，为了统一起来所以这里直接返回\$query。如有特殊需要加where、andWhere或者获取数据结果的可以这样\$query-&gt;where(\['条件'\]);\$query-&gt;all()。

分析总结
--------

以上是一个业务逻辑比较简单的service层的实现方式，看到这里可能还有人疑惑，到底应不应该分离service层？\
简单粗暴的总结来说，如果你的某个业务逻辑，需要用到多个model，就放到service层里面去，如果只是这个model自己的事，跟其它的model没有任何关系，放到model里面就好。\
如果你的系统本来就很小，业务逻辑也超级简单，也不存在长期演进迭代的需求，随你怎么高兴怎么写都行。

相关资料
--------

Yii2框架中，有必要再分离service层么？：<https://segmentfault.com/q/1010000003849810>

