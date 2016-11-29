---
layout:     post
title:      "Summary in the second week "
subtitle:   " \"work in QianMi\""
date:       2016-11-29 
author:     "Yang"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - 总结
---

> “Yeah It's on. ”

# 第二周开发总结

## 一:接口方面:

### 1）接口声明
   先进行接口相关的声明，然后在实现类中进行相关的实现，一般都是调用相关的service在      service类中实现相关的业务逻辑(包括在service接口中进行声明和在实现类中书写具体的实现逻辑) 。

### 2）返回值的类型: 
   如果是列表类型的可以声明一个BO型的List。如在sale中写过的DepositDetailBOList 类中的属性应该包括list长度例如声明totalCount便会  后期如分页时的使用，另外还应该包括一个dataList属性用来存储返回的BO类型的数据,例如List<?>dataList，这种情况下需要先调用一个方法查询满足条件记录总的数目，然后调用另一个方法查询具体的数据信息。在处理返回值数据时，我们新建一个VO型的数据，用来存储需要的返回数据，这里没有将返回值得数据直接作为sql的返回值类型，因为有的返回值数据不能直接。 通过查询获取，需要二次处理，这种情况下便需要新建一个数据存储类，如果可以所有字段可以直接查询出来，那么就不需要这一步了。  如果返回的是一个键值对类型的数据，例如这次做的查询统计中查询收入对应的金额，订单数，便可以通过一个map类型的变量来保存数据，例如：`Map<String,Object> map = new HashMap<~>();` 调用`map.put(key,object)`

### 3）CURD操作
  当然在service实现数据操作都是调用DAO中相应的接口如果需要做一个数据操作，首先应该在DAO接口中声明调用的方法，然后在DAO实现类中实现具体的调用逻辑，主要是包装相应的参数。如果是查询类方法，单个参数可以使用String类型的数据，如果是多个参数便可以使用map类型包装参数，例如`Map<String,Object> map = new HashMap();` 如果是插入或者更新操作则可以传入BO类型的数据，当然数据的更新和插入可以在一个方法里面封装: 例如: `f(applyBO.getApplyId()!= null && StringUtils.isNotEmpty(applyBO.getApplyId()))`通过判断现有BO中是否存在Id字段信息来判断是插入操作还是更新操作。在做插入数据操作时，可以使用seq_common.nextval来作为下一条记录对应的ID,插入时间可以直接使用sysdate，当然对应的为DATE类型的数据。 如果是删除操作，则直接传入相应的参数就可以了，当然这里面的删除操作不是真正意义上的删除而只是将记录的状态进行相应的更新。 在书写dao对应的xml映射文件时，需要注意的是，如果你的参数类型或者返回值类型是自定义的那么需要在文件中进行相应的引入，例如；
 `<typeAlias alias="DepositStatics" type="com.ofpay.ofsaleapi.pojo.cateapply.DepositStatics" />`
 在进行具体sql语句书写时，需要对传入的查询条件进行判断，例如:
   `<isNotEmpty property="detailId" prepend=" AND ">
			DETAIL_ID = #detailId#
   </isNotEmpty>`
其中property对应的字段名便是你在之前传入map中对应的key值，或者是参数类型中对应的属性名，mabatis会帮你创建相应的映射关系，而DETAIL_ID便是表中对应的字段名称。当然如果你进行比较的日期类型的数据，需要注意的是，如果你传入的是string类型的参数，那么由于数据库中对应的日期类型为DATE型，所以在进行比较之前需要进行相应的类型转换，
例如:
    `<isNotEmpty property="startTime" prepend="AND">
		<![CDATA[ ADD_TIME >=to_date(#startTime#,'yyyy-mm-dd hh24:mi:ss') ]]>
	</isNotEmpty>`

### 4)分页实现:
   `<![CDATA[
        select * from (
          select t.*,rownum rownum_ from (
        ]]>
            SELECT  TYPE type,rownum rn
            FROM  open_deposit_detail
            WHERE 1 = 1
            ORDER BY ADD_TIME DESC
            <![CDATA[
          ) t where rownum <= #end#
        )
        where rownum_ > #start#
    ]]>`	需要先进行一次查询将所有的rownum查询出来，然后进行比较，在外层查询比较rownum和起始的值	

### 5)查询日期转换:
   如果查询的日期字段为DATE类型，展示时需要的是String类型的数据，那么可以在查询时便进行相应的展示，例如:
   `SELECT  to_char(ADD_TIME,'yyyy-MM-dd HH24:mi:ss')` addTime 查询出的数据变为“2016-11-25 16:52:51”这种格式
   
   
### 6)多条件查询:
   `select user_code userCode,
		  count(case when type IN(0,1) then 1 end) totalIncomecount,
		  count(case when type = 1 then 1 end) totalExpensecount`
   当需要在同一条sql中按照多种条件来查询数据时，可以使用case when then 来进行相应的查询处理
   
## 二:web端开发步骤:
* 1:获取jsp页面返回,在controller类中调用http请求返回对应的jsp页面
      时间搜索条件，统一使用框架的控件，例如本次使用的是:<span id="queryDate"></span>      
       
* 2:通过ajax方法异步实现搜索逻辑
      通过Form类获取相应的前台页面参数，其中的参数对应的name属性需要和Form类中的成员属性名存在对应关系
      form类默认需要继承BaseTableForm类      
      string类型的数据进行比较时，需要使用equals方法。      
      返回给前台页面list列表时，可以调用通用方法:例如:
      return dataTableJson(depositDetailBOList.getTotalcount(), depositDetailBOList.getDatalist());
      前台:调用通用方法展示数据，其中列名需要和返回的数据类型中的字段名进行相应的对应          
       
* 3:通过ajax方法异步实现数据统计功能
      如果返回值类型为string类型或者Map类型的键值对，可以调用return getSuccessResult(countDepositDetail);返回数据给前台
    
## 三:代码更新步骤:
  1. 将代码push到git上
  2. 到jenkins进行打包,要进行配置为开发的分支，然后进行打包下一个版本
  3. 然后到布加迪进行操作，将自己打好的最新的包安装一下，然后启动，

    注意点:
    从git上pull完代码后，一般拉下来
    的都是主分支，需要在idea上切换为开发分支，然后再进行其他操作

## 四:开发出现的问题及解决办法:
   
 1. 查询出错
     原因:查询不到结果，报空指针错误。没有考虑到查询数据不存在的情况，当查询数据为空时，也就是字段信息为null时，对null进行具体操作便会
         报空指针错误
     预防措施:
         对每个查询结果都做非空校验，或进行非空处理。例如: 
         `String amount = "null".equals(String.valueOf(detailBO.getAmount()))? "0" :detailBO.getAmount().toString();`
         便是对amount字段进行非空校验或处理。
 2. 数据展示格式不正确，需求上要满足两位小数
    原因:对需求理解不清晰
    预防措施:
       详细阅读需求和开发原型图，并在数据处理时满足相应的格式。例如:
       `String amount = "null".equals(String.valueOf(detailBO.getAmount()))? " ":String.format("%.2f", detailBO.getAmount());`
       将查询到的数据保留两位小数进行处理。
       
 3. 展示统计数据信息不正确 
     原因:统计数据按分组查询，只展示了一组查询数据
     预防措施:
         在做查询操作时，做好具体的分析，如果是多组数据的话，做好数据的合并然后再返回展示

 4. 查询信息不准确 
     原因:查出整张表里面的数据，并没有做条件限制，忘记传入用户身份信息，对业务知识不清晰
     预防措施:
         对业务知识做好充分了解，又不会或者不清晰的地方积极主动向同事请教和商量
