---
layout: post
title: "JavaEE-Hibernate标签中的懒加载与Session事物"
date: 2017-06-29 19:02:55
categories: javaEE
tags: [javaEE, codeing]
---

最近开发的项目，谈开发算不上，只是在前辈们写的代码基础上添写代码，对系统增加功能。在阅读以前的代码过程中，遇到了懒加载，自己对懒加载的根本不了解，就向公司里的老司机请教。

这里就作为总结，谈自己现在对懒加载的认识

ps:用Hibernate还不如用Mybatis..

<!-- more -->

#### 懒加载的认识

例如下面的实体类

    @Entity
    @Table(name = "cruise_ship_project_classify")
    public class CruiseShipProjectClassify extends com.framework.hibernate.util.Entity {
        
        @ManyToOne
        @JoinColumn(name = "parentId")
        private CruiseShipProjectClassify cruiseShipProjectClassify;

        @OneToMany(mappedBy = "cruiseShipProjectClassify", fetch = FetchType.LAZY)
        private Set<CruiseShipProject> cruiseShipProject;
    }

看上面的代码可以得出：
    实体类CruiseShipProjectClassify和实体类CruiseShipProject是一对多的关系
    并对CruiseShipProject设置了懒加载(fetch = FetchType.LAZY)

拿上面的代码来解释其懒加载的定义:
    使用了懒加载时，系统控制器在获取CruiseShipProjectClassify对象数据后，在调试模式下会自动获取CruiseShipProject对象的数据，但在正常运行下时，只有当需要用到CruiseShipProject对象数据的时候，才会调用工具去数据库中自动获取CruiseShipProject对象数据。如果没定义使用懒加载，那么系统在每次运行时都会自动去数据库中获取CruiseShipProject对象的数据，无论当前控制器及页面是否用到CruiseShipProject对象数据。如果此时控制器及页面不需要用到这个对象的数据，这个对象又未定义懒加载，那么系统就消耗了本不该消耗的内存去加载搜索其数据，造成了内存的浪费，运行耗时且速度缓慢。
    使用了懒加载和未使用懒加载相对比起来，其作用效果就很明显了

然而，需要注意的是，jsp最好不要直接使用其定义了懒加载的对象数据
拿上面的代码来说，也就是当jsp页面需要CruiseShipProject对象的数据时候，最好不要调用cruiseShipProject这个对象，就是CruiseShipProject定义懒加载而声明的对象

其原因很简单，就是数据可能会取不到

#### Session认识

这就需要对Hibernate的Session进行了解了

需要知道

如果系统底层的crud操作是直接调用Spring中打包的HibernateTemplate来实现，那么Session事物的开启和关闭都将自动完成，而且我也不知道具体在什么时候完成开启和关闭的操作。只知道在需要对数据库进行操作和访问的时候，就会激活Session事物，所有的对数据库操作和访问的语句都将整合到一个Session事物中，这更能确定数据库信息的安全操作。这个Session事物中只要有一条sql语句执行出错，那么整个Session事物操作将会终止，数据库中的信息将会进行回滚，Session事物的连接也将会关闭，这确保了数据的正确性和安全保护

再回到懒加载，

在控制器中，定义了方法获取CruiseShipProjectClassify对象的数据，使用了懒加载的CruiseShipProject将会在遇到了需要对CruiseShipProject对象数据进行使用的时候，才会进行二次获取CruiseShipProject数据，如果没有遇到，那将永远不会其获取CruiseShipProject对象数据

而在进行二次获取时，此时因控制器方法而产生的Session事物已经关闭，那么获取到的CruiseShipProject对象值将会是Null

那么什么时候才是需要对CruiseShipProject对象数据进行使用的时候？

例如，一个jsp页面在需要数据的时候，就会调用该页面所对应的控制器去操作，控制器不知道页面具体需要什么数据，只会根据代码去调用相应的service...控制器方法中执行获取CruiseShipProjectClassify对象的数据并返回，其中本该一次性获取的cruiseShipProject，却因为使用了懒加载，使得控制器并不会去查询cruiseShipProject对象的数据，cruiseShipProject对象的数据就为Null.并返回

当jsp页面定义了${cruiseShipProject.xxx}字段时，就是为例等待控制器返回的数据并填充，当控制器发现这个${cruiseShipProject.xxx}字段的时候，就意味着需要使用cruiseShipProject对象数据，那么就会进行对cruisShipProject数据的二次获取

而此时，原先控制器激活的Session事物连接已经关闭，访问不到数据库，所以，最终还是获取不到定义了懒加载的CruiseShipProject实体类的对象cruiseShipProject对象数据

其实，同事所说的，这有个不确定因素，有的时候获取得到，有的时候获取不到

总之，jsp页面最好不要这样调用已经使用了懒加载的实体类对象

需要该实体类对象数据的时候，就直接在控制器中自己添加一条代码，以此来获取该实体类对象数据

另外，在debug调试模式下，是可以获取使用了懒加载对象的数据，不然怎么debug.....


