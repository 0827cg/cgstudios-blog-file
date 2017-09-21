---
layout: post
title: "Spring中的jdbcTemplate和FreeMarker的初步接触"
date: 2017-07-11 21:35:01
categories: javaEE
tags: [javaEE, codeing]
---

这么晚还没下班，没毛病吧，人生就是这么刺激....
说说今天接触到的东西
项目中定义了一个PurchaseOrderService这个接口，继承自BaseCRUDService这个底层的数据库操作接口,BaseCRUDService是封装在base-daoTemplate这个jar包中,估计这个base-daoTemplate.jar这个jar包是公司自己写的

<!-- more -->

里头封装了一些基本的crud操作这是操作数据库的一种方式

### BaseCRUDService.java

BaseCRUDService.java代码如下

    public interface BaseCRUDService<T> {
        T createEntity();

        Long count(Map<String, Object> var1);

        Long count(List<Cond> var1);

        Long count(Conds var1);

        T getById(Serializable var1);

        T get(Map<String, Object> var1);

        T get(Conds var1);

        List<T> list(Map<String, Object> var1);

        List<T> list(List<Cond> var1);

        List<T> list(Conds var1);

        void deleteById(Serializable var1);

        void delete(Conds var1);

        T insert(T var1);

        T update(T var1);

        T update(Update var1);
    }


第二中方式使用spring框架的jdbcTemplate这个jar包来操作

    String sql = FreeMarkerHelper.getValueFromTpl("sql/purchaseOrder/PurchaseOrderListList.sql", data);
    List<PurchaseOrderListVo> orderListVoList = jdbcTemplate.query(sql, ParameterizedBeanPropertyRowMapper.newInstance(PurchaseOrderListVo.class));

这感觉有点搞头
例如一个方法代码如下

    private ListMultimap<String, PurchaseOrderListVo> getOrderListList(List<String> orderIdList) {
        Map<String, Object> data = Maps.newHashMap();
        String orderIds = StringUtils.join(orderIdList, ",");
        if (StringUtils.isBlank(orderIds)){
        	orderIds = "-1";
        }
        data.put("orderIds",  orderIds);
        String sql = FreeMarkerHelper.getValueFromTpl("sql/purchaseOrder/PurchaseOrderListList.sql", data);
        List<PurchaseOrderListVo> orderListVoList = jdbcTemplate.query(sql, ParameterizedBeanPropertyRowMapper.newInstance(PurchaseOrderListVo.class));
        ListMultimap<String, PurchaseOrderListVo> listMultimap = ArrayListMultimap.create();
        // 售后信息
        Map<String, Map<String, List<OrderItemAsFlowVo>>> flowMap = orderItemAsFlowQueryService.getListFlowList(orderIdList);
        for (PurchaseOrderListVo listVo : orderListVoList) {
            if (flowMap.get(listVo.getOrderId()) != null) {
                listVo.setFlowList(flowMap.get(listVo.getOrderId()).get(listVo.getPurGoodsSkuId()));
            }
            listMultimap.put(listVo.getOrderId(), listVo);
        }
        return listMultimap;
    }

感觉这个方法涉及到的工具代码真是牛逼,来解析下这个方法
只节解析对我有用的，其他的功能我不管
首先代码第一行
`Map<String, Object> data = Maps.newHashMap()`
声明并实例化了一个Map类型的data对象
之后
`data.put("orderIds",  orderIds);`
这一行代码将数据存入到data对象中
`key=orderIds,value=orderIds`
然后使用FreeMarkerHelper这个类中的getValueFormTpl()这个方法来解析一个sql文件，并实现将搜索需要的参数参合到其中,其代码的实现就是将data对象作为参数传入，并且将sql文件中的参数变量使用data中的key匹配并用value替换

### sql文件

PurchaseOrderListList.sql文件的代码如下

    SELECT
	    group_concat(id) as id,
	    order_id,
	    pur_goods_sku_id,
	    pur_goods_id,
	    status,
	    sum(number) as number,
	    sum(final_amount) as final_amount,
	    pur_goods_name,
	    pur_goods_img_url,
	    pur_goods_amount as pur_goods_amount,
	    pur_goods_type,
	    ROUND(supply_price / number, 2) as supply_price,
	    create_time,
	    is_sample,
	    sum(sum_amount) as sum_amount,
	    cargo_sku_id,
	    sum(original_amount) as original_amount,
	    sum(discount_amount) as discount_amount,
	    return_time,
	    last_updated,
	    gift_flag,
	    sum(return_amount) as return_amount
    FROM
	    purchase_order_list
    WHERE
	    order_id in (${orderIds})
	    and is_valid = 'T'
	    and del_flag = 'F'
    group by pur_goods_sku_id,order_id

观察PurchaseOrderListList.sql中的代码，其中有个
`${orderIds}`
这个就是用来和data对象中的key相匹配，并将data对象中key值orderIds的value传入到PurchaseOrderListList.sql文件中，FreeMarkerHelper再读取PurchaseOrderListList.sql中的内容，之后返回一个String类型的操作数据库的字符串语句
之后，调用jdbaTemplate中的query()方法来操作数据库，并返回一个list集合

### FreeMarkerHelper.java

其FreeMarkerHelper代码如下

    public class FreeMarkerHelper {

	    private static final Configuration config;
	
	    static{
		    config = new Configuration(Configuration.VERSION_2_3_22);
		    config.setDefaultEncoding("UTF-8");
		    config.setOutputEncoding("UTF-8");
		
		    try {
    //			System.out.println("===========" + new ClassPathResource("./classes").getFile().getAbsolutePath() + "===========");
    //			System.out.println("===========" + new ClassPathResource("./sql").getFile().getAbsolutePath() + "===========");
    //			System.out.println("===========" + new ClassPathResource("sql").getFile().getAbsolutePath() + "===========");
    //			System.out.println("===========" + new ClassPathResource("").getFile().getAbsolutePath() + "===========");
    //			System.out.println("===========" + new ClassPathResource("/").getFile().getAbsolutePath() + "===========");
    //			System.out.println("===========" + new ClassPathResource(".").getFile().getAbsolutePath() + "===========");
			    config.setDirectoryForTemplateLoading(new File(CfgConstants.getProperties().get("tplPath")));
		    } catch (IOException e) {
			    // TODO Auto-generated catch block
			    e.printStackTrace();
		    }
	    }
	
	    public static String getValueFromTpl(String tplName, Object dataModel) {
		    String value = null;
		    Template template = null;
		    try {
			    template = config.getTemplate(tplName);
			    value = FreeMarkerTemplateUtils.processTemplateIntoString(template, dataModel);
		    } catch (Exception e) {
			    e.printStackTrace();
		    }

		    return value;
	    }
	
    }


这个FreeMarkerHelper类真是厉害，这类中的代码我根本看不懂,只知道是调用了FreeMarker模板引擎
只是我没接触过FreeMarker，问了前辈，前辈说也不知道，表示这个项目中使用FreeMarker只是用来组装sql语句的,并让我不要去研究底层的东西，学会怎么用就行.....

ps:得找个机会总结下最近学到的东西了，不然就忘了





