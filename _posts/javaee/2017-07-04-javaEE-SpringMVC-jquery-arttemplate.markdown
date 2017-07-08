---
layout: post
title: "Spring MVC注解控制器同步跳转和ajax的异步请求的协作"
date: 2017-07-04 15:35:55
categories: javaEE
tags: [javaEE, codeing，JQuery, AJAX]
---

就着现在接触的项目优茶联项目中，谈谈在系统的个人中心里的我的收藏中，对收藏的商品进行取消收藏的流程

#### 流程

其大致流程如下

1. 首先，浏览器访问list.jsp这个页面，对应list.jsp这个页面的控制器直接返回一个页面路径(也就是直接返回一个页面)

<!-- more -->

2. 在list.jsp页面中点击取消收藏这个按钮，处理这个点击事件的js方法是favorite_list.js中的cancelFavorite()方法，方法获取商品id后进行jquery ajax的异步请求数据加载，其中发送请求的data数据中就包含商品id，并设置了请求方式和路径

3. 发送请求后，服务器根据再次根据异步请求的路径找到与路径相对应的控制器方法，把异步请求需要的数据处理交给这个控制器方法，等待控制器方法返回数据

4. 将放回的结果数据当做参数传入JQuery AJAX的异步请求方法中的success函数，根据结果情况做出相对应的响应

#### 关于success函数

jquery ajax的异步请求方法中，定义了一个success函数机制，如下面的实现代码

`success:function(result){}`

这样的

我也不知道这个success功能该怎么叫，它的工作原理我也不大清楚，说是success函数机制，这也只是我按自己的理解命名的
控制器方法在执行完数据的处理后，return操作后的结果，这就当是服务器的返回，原先缓存在浏览器中的favorite_list.js中的jquery ajax还在运行等待服务器的返回，
当其检测到服务器发送回来的数据后，将数据当做参数传到success函数机制中

就如下面代码

`success:function(返回数据){}`

后面success函数机制在根据返回的数据进行操作判断之前请求的数据是否处理成功
并做出对应的相应

这是我现在的猜测，
另外
根据根据控制器方法的不同返回结果，可分为不同的跳转方式

#### 同步跳转

控制器方法内部有数据的处理及最后返回一个页面地址，这样的属于同步跳转
控制器方法内部要是没有数据的处理，最后确返回一个页面地址，这样也数据同步跳转

#### 异步跳转

控制器方法内部只有数据的处理，最后也只返回数据，这样的数据异步跳转

还有就是，每个ajax的请求都对应一个控制器，每次请求都会对应实例化一个新的Request对象，后面事件触发的ajax数据请求与初次访问页面请求的Request对象是不同的

下面贴出相关文件的代码

#### list.jsp

list.jsp页面的代码

    <!DOCTYPE html>
    <%@ page contentType="text/html;charset=UTF-8" language="java" %>
    <html>
    <head>
        <title>我的收藏</title>
      <jsp:include page="/WEB-INF/jsp/www/common/common_css.jsp"></jsp:include>
      <jsp:include page="/WEB-INF/jsp/www/common/common_js.jsp"></jsp:include>
      <link rel="stylesheet" type="text/css" href="/static/css/person.css">
      <link rel="stylesheet" href="/static/css/person_center_nav_position.css">
      <link rel="stylesheet" href="/static/js/lib/kkpager/css/kkpager.css">
      <link rel="stylesheet" type="text/css" href="/static/css/collect.css">
    </head>
    <body>

    <%--头部 starting--%>
    <jsp:include page="/WEB-INF/jsp/www/include/header.jsp"></jsp:include>
    <%--头部 ending--%>

    <%--查询 starting--%>
    <jsp:include page="/WEB-INF/jsp/www/include/personal_header.jsp"></jsp:include>
    <%--查询 ending--%>

    <%--左侧工具栏 starting--%>
    <jsp:include page="/WEB-INF/jsp/www/include/nav_left_bar.jsp"></jsp:include>
    <%--左侧工具栏 ending--%>

    <!-- 主内容 -->
    <div class="person-wrap">
      <div class="person-cont">
        <div class="content">
          <%--右侧导航栏 starting--%>
          <jsp:include page="/WEB-INF/jsp/www/include/personal_menu.jsp"></jsp:include>
          <%--右侧导航栏 ending--%>

          <!-- 个人中心主内容 -->
          <div class="right-cont-wrap">
            <div>
              <ul id="top-nav" class="nav-menu clearfix">
                <li class="active"><a>收藏的商品</a></li>
                <%--<li><a>收藏的店铺</a></li>--%>
              </ul>
              <div class="top-right-box clearfix">
                <!-- 添加check出现批量操作面板 -->
                <div id="title-batch-box" class="option-wrap pull-left">
                  <div class="nomal-box">
                    <a id="batch-btn" class="batch-btn">批量处理</a>
                  </div>
                  <div class="check-box">
                    <div class="pull-left">
                      <%--<span class="check-icon check-tagt"></span>--%>
                      <lalel>
                        <input type="checkbox" class="check-all">
                        全选
                      </lalel>
                    </div>
                    <%--<a class="option-btn cart">加入购物车</a>--%>
                    <a class="option-btn collect" onclick="Collect.cancelFavorite()">取消收藏</a>
                    <a id="complete-btn" class="batch-btn">完成</a>
                  </div>
                </div>
                <div class="page-box pull-right">
                  <span class="pull-left"><span id="current-page" class="orange">1</span>/<span id="total-page">1</span></span>
                  <a id="page-pre" href="javascript:void(0);" class="page-btn pre disable" data-page-no="0"></a>
                  <a id="page-next" href="javascript:void(0);" class="page-btn next disable" data-page-no=""></a>
                </div>
              </div>
            </div>
            <!-- 添加check出现操作样式 -->
            <div id="collect-list-box" class="collect-list-wrap clearfix">
            
            </div>
            <div id="kkpager"></div>
          </div>
        </div>
          <!-- END个人中心主内容 -->
      </div>
    </div>
    <!-- END主内容 -->
    <%--尾部 starting--%>
    <jsp:include page="/WEB-INF/jsp/www/include/footer.jsp"></jsp:include>
    <%--尾部 ending--%>
    </body>
    </html>
    <script src="/static/js/user/person.common.js"></script>
    <script src="/static/js/lib/kkpager/kkpager.js"></script>
    <script src="/static/js/favorite/favorite_list.js"></script>

    <script id="tpl-list-item" type="text/html">

      <div class="list-item" data-id="{{id}}">
        <div class="item-inner">
          <a class="img-cover" href="/purchaseGoods/detail?goodsId={{good.id}}">
            <img width="160" height="160" src="{{good.image.url}}">
          </a>
          <a class="p-name" href="/purchaseGoods/detail?goodsId={{good.id}}">{{good.name}}</a>
          <div class="p-price red">¥<strong>{{good.price}}</strong></div>
          <div class="p-operate clearfix">
            <div class="op-btn">
              <a class="btn-compare" data-id="{{id}}" onclick="Collect.addToCartFromFavorite(this)">加入购物车</a>
            </div>
            <div class="op-btn">
              <a class="btn-cut" data-id="{{id}}" onclick="Collect.cancelFavorite(this)">取消收藏</a>
            </div>
          </div>
        </div>
        <div class="item-check">
          <i class="i-check"></i>
        </div>
      </div>

    </script>


这个页面中对应的控制器方法使用了同步加载的方法，其返回值是一个list.jsp页面路径
在这个页面中，关键数据使用了arttemplate模板，如<script id="tpl-list-item" type="text/html"></script>标签块中的代码实现，配合ajax实现list.jsp页面数据的异步加载并渲染

#### FavoriteController.java

其控制器类FavoriteController的部分代码如下

    @Controller
    @RequestMapping("/favorite")
    public class FavoriteController {

        @Autowired
        private FavoriteB2bService favoriteB2bService;

        @RequestMapping("/list")
        public String list() {
            return "www/favorite/list";
        }
        
        @RequestMapping("/cancleFavorite")
        @ResponseBody
        public ResultData cancleFavorite(HttpServletRequest request, String modelJsonStr) {
            if (StringUtils.isBlank(modelJsonStr)) {
                return ResultData.createFail("商品信息不存在");
            }
            List<String> ids = Lists.newArrayList();
            ids = JsonUtils.toObject(modelJsonStr, ListUtils.getCollectionType(List.class, String.class));
            String userId = AccountUtils.getCurrShopId(); //TODO 获取当前登录用户id
            favoriteB2bService.cancleFavorite(ids, userId, OrderBizType.purchase);
            return ResultData.createSuccess();
        }
        
        @RequestMapping("/getFavoriteList")
        @ResponseBody
        public ResultData getFavoriteList(HttpServletRequest request, String modelJsonStr) {
            if (StringUtils.isBlank(modelJsonStr)) {
                return ResultData.createFail("商品信息不存在");
            }
            FavoriteQuery favoriteQuery = JsonUtils.toObject(modelJsonStr, FavoriteQuery.class) != null ?
                    JsonUtils.toObject(modelJsonStr, FavoriteQuery.class) : new FavoriteQuery();
            String userId = AccountUtils.getCurrShopId();
            String storeLevelId = AccountUtils.getCurrStoreLevelId();
            favoriteQuery.setStoreLevelId(storeLevelId);
            favoriteQuery.setUserId(userId);
            favoriteQuery.setOrderBizType(OrderBizType.purchase);
            return favoriteB2bService.getFavoriteList(favoriteQuery);
        }

        ....

    }

观察上面的代码可以知道这是个基于注解实现的控制器，
使用@Controller来声明一个控制器类，
@RequestMapping来声明一个控制器方法，RequestMapping方法除了有value属性外，还有method属性

#### favorite_list.js

favorite_list.js中的部分代码

    $(function() {
        if (!$.checkLogin()) { //验证登录
            return;
        }
        Collect.init();
    });

    var Collect = {
        init : function() {
            Collect.showBatchBox();
            Collect.hideBatchBox();
            Collect.prePage();
            Collect.nextPage();
            Collect.initCheck();
            Collect.getFavoriteList(1);
            Collect.initFavoritePage();
        },

        cancelFavorite: function(target) {
            var queryData = [];
            if (!target) {
                $(".collect-list-wrap").find(".list-item.active").each(function(i, perFavorite) {
                    queryData.push($(perFavorite).attr("data-id"));
                });
            } else {
                queryData.push($(target).attr("data-id"));
            }
            if (queryData.length <= 0) {
                return;
            }
            $.ajax({
                type: "POST",
                data: {modelJsonStr : JSON.stringify(queryData)},
                url: "/favorite/cancleFavorite",
                success: function(result) {
                    if (result.success) {
                        layer.msg("取消成功！", function() {
                            Collect.initFavoritePage();
                            Collect.getFavoriteList(1);
                        });
                    } else {
                        layer.msg(result.msg);
                    }
                }
            });

        },
        
        getFavoriteList: function(pageNo) {
            var ii = layer.load();
            var queryData = {
                pageNum: pageNo
            };
            $.ajax({
                type: "POST",
                data: {modelJsonStr : JSON.stringify(queryData)},
                url: "/favorite/getFavoriteList",
                success: function(result) {
                    //此处用setTimeout演示ajax的回调
                    layer.close(ii);
                    if (result.success) {
                        $("#collect-list-box").empty();
                        $.each(result.data, function(i, perGoods) {
                            var tpl_list_item = template("tpl-list-item", perGoods);
                            $("#collect-list-box").append($(tpl_list_item));
                        });
                    } else {
                        layer.msg(result.msg);
                    }

                }
            });
        },

    }

结合上面的favorite_list.js中的代码，来进一步了解异步加载数据的实现

list.jsp页面中引用了favorite_list.js这个文件，在第一次访问list.jsp页面并完成文档加载后会自动调用favorite_list.js中的文档就绪函数$(function(){}),以此来初始化Collect对象，并且调用favorite_list.js中的getFavoriteList()方法，实话说，这个方法就是主要用来实现异步加载list.jsp页面所需要的数据的主要实现部分

getFavoriteList()方法中定义了jquery ajax的异步数据请求，设置了url,使得具体的数据获取操作交给控制器类FavoriteController中相对应的控制器方法,且使用了success函数机制等待控制器方法返回的结果
并在方法内将结果处理遍历出来，遍历的方法里面使用了arttemplate中的template()方法，将数据对象都渲染到jsp页面
遍历出来的数据div块都在类名为collect-list-box的div块里，所以猜测arttemplate模板只能放到页面最后
