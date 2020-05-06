---
title: SpringBoot接入支付宝支付
date: 2020-3-20 22:36
urlname: 2020032001
categories: Java
tags:
  - Java
author: foochane
toc: true
mathjax: true
top: false
top_img: /images/banner/0.jpg
cover: /images/cover/11.jpg
---

## 1 开发前的准备

### 1.1 密钥的的生成

先去官网下载支付宝平台助手，下载地址：https://opendocs.alipay.com/open/291/105971 ，下载并安装平台助手，使用平台助手生成`应用公钥`和`应用私钥`。

### 1.2 创建应用配置账号信息

沙箱账号申请地址：https://openhome.alipay.com/platform/appDaily.htm

正式账号申请地址：https://open.alipay.com/platform/manageHome.htm

为了保证交易双方（商户和支付宝）的身份和数据安全，开发者在调用接口前，需要配置双方密钥，对交易数据进行双方校验。密钥包含应用私钥`（APP_PRIVATE_KEY）`和应用公钥`（APP_PUBLIC_KEY）`。生成密钥后，开发者需要在开放平台开发者中心进行密钥配置，配置完成后可以获取支付宝公钥`（ALIPAY_PUBLIC_KEY）`应用在代码中，对请求内容进行签名。

> 注意：当设置了应用公钥后，才可以查看支付宝公钥

### 1.3 开发需要的参数

- `appid`： 应用ID
- `private_key`：由支付宝开发平台助手生成的应用私钥
- `alipay_public_key`：是支付宝公钥，支付宝开发平台助手生成的应用公钥

 **这里特别需要注意的是`alipay_public_key`是蚂蚁金服开放平台获取的支付宝公钥，不是本地平台助手申请的`应用公钥`，不要弄错了，否则会签名失败** 

## 2 支付介绍

### 2.1 支付逻辑

1. 用户在商户系统下单

2. 商户系统调用支付宝的支付接口，发起支付请求
3. 用户登录，选择支付渠道，输入支付密码
4. 支付宝get请求`returnUrl`， 返回同步通知参数 
5. 支付宝post请求`notifyUrl`，返回异步通知参数
6. 商户可以通过 交易查询接口查询交易状态
7. 如果用户或者商户想退款，可以发起退款请求
8. 如果退款可以调用退款查询接口，查询退款信息
9. 下载对账单

### 2.2 关于支付宝中同步通知和异步通知的理解

在调用支付宝接口的时候，确认支付以后我们需要请求两个通知接口：同步通知（`returnUrl`）和异步通知（`notifyUrl`）。刚开始看文档的时候也不是很清楚这两个接口有什么区别，从流程图里面描述到的是，同步通知（`returnUrl`）是GET请求，“支付是否成功以异步通知为准”。异步通知（`notifyUrl`）是POST请求，但是图里面还是说了一句“支付是否成功以查询接口返回为准”。所以最终支付是否成功还是要看查询接口的返回结果，那同步通知和异步通知有什么用呢？

个人的理解是：只要同步通知和异步通知返回成功一般情况下其实已经可以认为支付成功了，我们就已经可以进行下一步的操作了。同步通知用于前端界面及时反馈给用户，告诉用户支付操作已经完成，然后让前端页面跳转的需要跳转的页面；异步通知用于后台业务逻辑的处理。当然为了保险，后面最好还是调用查询接口，对支付账单进行核对。

![001](https://foochane.cn/images/2020/001.jpg)

需要注意的问题：

1. `returnUrl`和`returnUrl`这两个的接口地址必须放在公网上，需要外网能访问到，否则支付宝是调不到这两个接口的。本地测试可以使用 natapp 做内外网穿透。
2. 经过测试异步通知和同步通知没有很明确的先后顺序，同步通知并不一定在异步通知之前被调用。



### 2.3 重点看官方的Demo

其实官方的Demo已经写的很详细了，每个接口的调用已经给出了详细的演示。所以后面的代码基本也是照着官方的Demo来的。

电脑网站支付Demo下载：https://opendocs.alipay.com/open/270/106291



## 3 支付接入代码

### 3.1 添加依赖

这里使用的是`SpringBoot`，先在maven里引入官方的依赖：

```xml
<!-- 支付宝 alipay sdk -->
<!-- https://mvnrepository.com/artifact/com.alipay.sdk/alipay-sdk-java -->
<dependency>
    <groupId>com.alipay.sdk</groupId>
    <artifactId>alipay-sdk-java</artifactId>
    <version>4.9.124.ALL</version>
</dependency>
```



### 3.2 添加配置类

接着添加已经配置类：

```java
package com.foochane.awpay.test.config;

/**
 * Created by foochane on 2020/5/2.
 */
public class MyAliPayConfig {
    // natapp内外网穿透地址
    public static final String natUrl = "http://znznca.natappfree.cc";

    // 应用ID,您的APPID，收款账号既是您的APPID对应支付宝账号
    public static String app_id = "2016101800717168";//在后台获取（必须配置）

    // 商户私钥，您的PKCS8格式RSA2私钥 ,使用支付宝平台助手生成
    public static String merchant_private_key = "xxxxxxxxxxxxxx";

    // 支付宝公钥,查看地址：https://openhome.alipay.com/platform/keyManage.html 对应APPID下的支付宝公钥。
    public static String alipay_public_key = "xxxxxxxxx";

    // 服务器异步通知页面路径  需http://格式的完整路径，不能加?id=123这类自定义参数，必须外网可以正常访问
    public static String notify_url = natUrl + "/ali/pay/notify";

    // 页面跳转同步通知页面路径 需http://格式的完整路径，不能加?id=123这类自定义参数，必须外网可以正常访问
    public static String return_url = natUrl + "/ali/pay/return";

    // 签名方式
    public static String sign_type = "RSA2";

    // 字符编码格式
    public static String charset = "utf-8";

    // 支付宝网关
    //注意：沙箱测试环境，正式环境为：https://openapi.alipay.com/gateway.do
    public static String gatewayUrl = "https://openapi.alipaydev.com/gateway.do";

}
```



### 3.3 编写支付代码

然后就可以在Controller层编写支付代码了，这里只是为了演示，所以就之编写controller层。

这里引入的是电脑网站支付，代码流程如下：

1. 创建`AlipayClient` 类
2. 设置请求参数（这里参数是写死的）
3. 发起支付请求，获取支付表单
4. 直接将支付表单输出到页面

具体代码如下：

```java
package com.foochane.awpay.test.controller;


import com.alipay.api.AlipayApiException;
import com.alipay.api.AlipayClient;
import com.alipay.api.DefaultAlipayClient;
import com.alipay.api.internal.util.AlipaySignature;
import com.alipay.api.request.*;
import com.foochane.awpay.test.config.MyAliPayConfig;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.ResponseBody;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;


/**
 * Created by foochane on 2020/5/2.
 */

@RestController
public class AlipayController {


    @ResponseBody
    @RequestMapping(value = "ali/pay/order/create",method = RequestMethod.GET)
    public void pay (HttpServletRequest httpRequest,
                     HttpServletResponse httpResponse) throws IOException {
        // 1 创建AlipayClient
        AlipayClient alipayClient =  new DefaultAlipayClient(  MyAliPayConfig.gatewayUrl,
                MyAliPayConfig.app_id,
                MyAliPayConfig.merchant_private_key,
                "json",
                MyAliPayConfig.charset,
                MyAliPayConfig.alipay_public_key,
                MyAliPayConfig.sign_type);


        // 2 设置请求参数
        //商户订单号，商户网站订单系统中唯一订单号，必填
        String out_trade_no = "P"+System.currentTimeMillis();
        //付款金额，必填
        String total_amount = "80";
        //订单名称，必填
        String subject = "商品购买";
        //商品描述，可空
        String body = "购买商品的描述信息";
        // 该笔订单允许的最晚付款时间，逾期将关闭交易。取值范围：1m～15d。m-分钟，h-小时，d-天，1c-当天（1c-当天的情况下，无论交易何时创建，都在0点关闭）。 该参数数值不接受小数点， 如 1.5h，可转换为 90m。
        String timeout_express = "10m";
        AlipayTradePagePayRequest alipayRequest =  new  AlipayTradePagePayRequest(); //创建API对应的request
        alipayRequest.setReturnUrl(MyAliPayConfig.return_url);
        alipayRequest.setNotifyUrl(MyAliPayConfig.notify_url); //在公共参数中设置回跳和通知地址
        alipayRequest.setBizContent("{\"out_trade_no\":\"" + out_trade_no + "\","
                + "\"total_amount\":\"" + total_amount + "\","
                + "\"subject\":\"" + subject + "\","
                + "\"body\":\"" + body + "\","
                + "\"timeout_express\":\"" + timeout_express + "\","
                + "\"product_code\":\"FAST_INSTANT_TRADE_PAY\"}"); //填充业务参数

        // 3 发起支付请求
        String form = "";
        try  {
            form = alipayClient.pageExecute(alipayRequest).getBody();  //调用SDK生成表单
        }  catch  (AlipayApiException e) {
            e.printStackTrace();
        }

        System.out.println("*************************** 支付请求成功 *******************************");
        System.out.println("返回表单：");
        System.out.println(form);
        System.out.println("*************************** 支付请求成功 *******************************");

        httpResponse.setContentType( "text/html;charset="  + MyAliPayConfig.charset);
        httpResponse.getWriter().write(form); //直接将完整的表单html输出到页面
        httpResponse.getWriter().flush();
        httpResponse.getWriter().close();

    }
}
```



## 4 同步通知代码



```java
   /**
     * 支付宝同步通知页面
     * return_url必须放在公网上
     */
    @ResponseBody
    @RequestMapping("ali/pay/return")
    public String returnUrl(HttpServletRequest httpRequest,
                            HttpServletResponse httpResponse) throws Exception {

        System.out.println("支付成功, 进入同步通知接口...");

        // 获取支付宝GET过来反馈信息
        Map<String, String> params = new HashMap<String, String>();
        Map<String, String[]> requestParams = httpRequest.getParameterMap();
        for (Iterator<String> iter = requestParams.keySet().iterator(); iter.hasNext(); ) {
            String name = (String) iter.next();
            String[] values = (String[]) requestParams.get(name);
            String valueStr = "";
            for (int i = 0; i < values.length; i++) {
                valueStr = (i == values.length - 1) ? valueStr + values[i]
                        : valueStr + values[i] + ",";
            }

            // 乱码解决，这段代码在出现乱码时使用
            // valueStr = new String(valueStr.getBytes("ISO-8859-1"), "utf-8");

            params.put(name, valueStr);
        }

        // 调用SDK验证签名
        boolean signVerified = AlipaySignature.rsaCheckV1(params,
                                                          MyAliPayConfig.alipay_public_key,
                                                          MyAliPayConfig.charset,
                                                          MyAliPayConfig.sign_type);


        // 验签成功
        if (signVerified) {

            // 同步通知返回的参数（部分说明）
            // out_trade_no :	商户订单号
            // trade_no : 支付宝交易号
            // total_amount ： 交易金额
            // auth_app_id/app_id : 商户APPID
            // seller_id ：收款支付宝账号对应的支付宝唯一用户号(商户UID )
            System.out.println("****************** 支付宝同步通知成功   ******************");
            System.out.println("同步通知返回参数：" + params.toString());
            System.out.println("****************** 支付宝同步通知成功   ******************");


        } else {
            System.out.println("支付, 验签失败...");
        }

        // 返回支付操作完成后需要跳转的页面,这里把返回的参数直接传给页面
        return params.toString();
    }

```



## 5 异步通知代码

```java
    /**
     * 支付宝服务器异步通知
     * notify_url必须放入公网
     */
    @RequestMapping(value = "/ali/pay/notify")
    @ResponseBody
    public String notify(HttpServletRequest request, HttpServletRequest response) throws Exception {

        System.out.println("支付成功, 进入异步通知接口...");

        //获取支付宝POST过来反馈信息
        Map<String, String> params = new HashMap<String, String>();
        Map<String, String[]> requestParams = request.getParameterMap();
        for (Iterator<String> iter = requestParams.keySet().iterator(); iter.hasNext(); ) {
            String name = (String) iter.next();
            String[] values = (String[]) requestParams.get(name);
            String valueStr = "";
            for (int i = 0; i < values.length; i++) {
                valueStr = (i == values.length - 1) ? valueStr + values[i]
                        : valueStr + values[i] + ",";
            }

            //乱码解决，这段代码在出现乱码时使用
			// valueStr = new String(valueStr.getBytes("ISO-8859-1"), "utf-8");

            params.put(name, valueStr);
        }

        boolean signVerified = AlipaySignature.rsaCheckV1(params,
                                                          MyAliPayConfig.alipay_public_key,
                                                          MyAliPayConfig.charset,
                                                          MyAliPayConfig.sign_type); //调用SDK验证签名

        if (signVerified) {

            // 异步通知返回的参数（部分说明）
            // out_trade_no ：商户订单号，商户网站订单系统中唯一订单号
            // trade_no ：支付宝交易号，支付宝交易凭证号
            // app_id ：开发者的app_id
            // out_biz_no ：商户业务号
            // buyer_id ：买家支付宝用户号，买家支付宝账号对应的支付宝唯一用户号，以 2088 开头的纯 16 位数字
            // seller_id ：卖家支付宝用户号 卖家支付宝用户号
            // trade_status ：交易状态
            // total_amount ：订单金额
            // receipt_amount ：实收金额
            // invoice_amount ：开票金额
            // buyer_pay_amount ：付款金额
            // point_amount ：集分宝金额 使用集分宝支付的金额，单位为元，精确到小数点后2位
            // subject ：订单标题
            // body ：商品描述
            // fund_bill_list ：支付金额信息
            // passback_params  回传参数
            System.out.println("********************** 支付宝异步通知成功   **********************");
            System.out.println("异步通知返回参数：" + params.toString());
            System.out.println("********************** 支付宝异步通知成功   **********************");


            if (params.get("trade_status").equals("TRADE_FINISHED")){

                // 注意： 注意这里用于退款功能的实现
                System.out.println("执行退款相关业务");
            }else if (params.get("trade_status").equals("TRADE_SUCCESS")) {

                // 1. 根据out_trade_no 查询订单
                // 2. 判断total_amount 是否正确，即是否为商户订单创建时的金额
                // 3. 校验通知中的seller_id（或者seller_email) 是否为out_trade_no这笔单据的对应的操作方（有的时候，一个商户可能有多个seller_id/seller_email）
                // 4. 验证app_id是否为该商户本身
                // 5. 判断该笔订单是否在商户网站中已经做过处理
                // 6. 如果没有做过处理，根据订单号（out_trade_no）在商户网站的订单系统中查到该笔订单的详细，并执行商户的业务程序，修改订单状态
                // 7. 如果有做过处理，不执行商户的业务程序
                System.out.println("执行相关业务");
            }

        } else {
            System.out.println("支付, 验签失败...");
        }

        return "success";
    }

```



## 6 订单查询代码

```java
   /**
     * 订单查询
     * @param out_trade_no 商户订单号，商户网站订单系统中唯一订单号
     * @param trade_no 支付宝交易号 注意： out_trade_no 和  trade_no 二选一即可
     * @return result
     *
     */
    @ResponseBody
    @RequestMapping(value = "/ali/pay/order/query")
    public String query(String out_trade_no,String trade_no){

        System.out.println("订单查询...");

        //获得初始化的AlipayClient
        AlipayClient alipayClient = new DefaultAlipayClient(MyAliPayConfig.gatewayUrl, MyAliPayConfig.app_id, MyAliPayConfig.merchant_private_key, "json", MyAliPayConfig.charset, MyAliPayConfig.alipay_public_key, MyAliPayConfig.sign_type);

        //设置请求参数
        AlipayTradeQueryRequest alipayRequest = new AlipayTradeQueryRequest();

        // out_trade_no 和 trade_no 设置二选一即可
        alipayRequest.setBizContent("{\"out_trade_no\":\""+ out_trade_no +"\","+"\"trade_no\":\""+ trade_no +"\"}");

        //请求
        String result = null;
        try {
            result = alipayClient.execute(alipayRequest).getBody();
        } catch (AlipayApiException e) {
            e.printStackTrace();
        }

        //输出
        System.out.println("订单查询结果：" +result);

        return  result;
    }

```



## 8 退款代码代码

```java
    /**
     * 退款
     * @param out_trade_no 商户订单号，商户网站订单系统中唯一订单号
     * @param trade_no 支付宝交易号 注意： out_trade_no 和  trade_no 二选一即可
     * @param refund_amount 需要退款的金额，该金额不能大于订单金额，必填
     * @param refund_reason 退款的原因说明
     * @param out_request_no 标识一次退款请求，同一笔交易多次退款需要保证唯一，如需部分退款，则此参数必传
     * @return result
     *
     */
    @ResponseBody
    @RequestMapping(value = "/ali/refund/order/create")
    public String refund(String out_trade_no,String trade_no,String refund_amount, String refund_reason, String out_request_no){

        System.out.println("退款...");

        //获得初始化的AlipayClient
        AlipayClient alipayClient = new DefaultAlipayClient(MyAliPayConfig.gatewayUrl, MyAliPayConfig.app_id, MyAliPayConfig.merchant_private_key, "json", MyAliPayConfig.charset, MyAliPayConfig.alipay_public_key, MyAliPayConfig.sign_type);

        //设置请求参数
        AlipayTradeRefundRequest alipayRequest = new AlipayTradeRefundRequest();


        alipayRequest.setBizContent("{\"out_trade_no\":\""+ out_trade_no +"\","
                + "\"trade_no\":\""+ trade_no +"\","
                + "\"refund_amount\":\""+ refund_amount +"\","
                + "\"refund_reason\":\""+ refund_reason +"\","
                + "\"out_request_no\":\""+ out_request_no +"\"}");

        //请求
        String result = null;
        try {
            result = alipayClient.execute(alipayRequest).getBody();
        } catch (AlipayApiException e) {
            e.printStackTrace();
        }

        //输出
        System.out.println("退款结果:" + result);

        return result;
    }
```



## 9 退款查询代码

```java
   /**
     * 退款查询
     * @param out_trade_no  商户订单号，商户网站订单系统中唯一订单号
     * @param trade_no 支付宝交易号， 注意： out_trade_no 和  trade_no 二选一即可
     * @param out_request_no 请求退款接口时，传入的退款请求号，如果在退款请求时未传入，则该值为创建交易时的外部交易号，必填
     * @return result
     */
    @ResponseBody
    @RequestMapping(value = "/ali/refund/order/query")
    public String refundQuery(String out_trade_no, String trade_no, String out_request_no){
        System.out.println("退款查询......");
        //获得初始化的AlipayClient
        AlipayClient alipayClient = new DefaultAlipayClient(MyAliPayConfig.gatewayUrl, MyAliPayConfig.app_id, MyAliPayConfig.merchant_private_key, "json", MyAliPayConfig.charset, MyAliPayConfig.alipay_public_key, MyAliPayConfig.sign_type);

        //设置请求参数
        AlipayTradeFastpayRefundQueryRequest alipayRequest = new AlipayTradeFastpayRefundQueryRequest();
        alipayRequest.setBizContent("{\"out_trade_no\":\""+ out_trade_no +"\","
                +"\"trade_no\":\""+ trade_no +"\","
                +"\"out_request_no\":\""+ out_request_no +"\"}");

        //请求
        String result = null;
        try {
            result = alipayClient.execute(alipayRequest).getBody();
        } catch (AlipayApiException e) {
            e.printStackTrace();
        }

        //输出
        System.out.println("退款查询结果："+result);;

        return result;
    }

```



## 10 订单关闭代码

```java
   /**
     * 交易关闭
     * @param out_trade_no  商户订单号，商户网站订单系统中唯一订单号
     * @param trade_no 支付宝交易号， 注意： out_trade_no 和  trade_no 二选一即可
     * @return result
     */
    @ResponseBody
    @RequestMapping(value = "/ali/order/close")
    public String close(String out_trade_no, String trade_no){
        System.out.println("交易关闭.....");

        //获得初始化的AlipayClient
        AlipayClient alipayClient = new DefaultAlipayClient(MyAliPayConfig.gatewayUrl, MyAliPayConfig.app_id, MyAliPayConfig.merchant_private_key, "json", MyAliPayConfig.charset, MyAliPayConfig.alipay_public_key, MyAliPayConfig.sign_type);


        //设置请求参数
        AlipayTradeCloseRequest alipayRequest = new AlipayTradeCloseRequest();
        alipayRequest.setBizContent("{\"out_trade_no\":\""+ out_trade_no +"\"," +"\"trade_no\":\""+ trade_no +"\"}");

        //请求
        String result = null;
        try {
            result = alipayClient.execute(alipayRequest).getBody();
        } catch (AlipayApiException e) {
            e.printStackTrace();
        }

        //输出
        System.out.println("交易关闭结果："+result);

        return result;
    }
```





## 11 完整的代码地址

代码地址：[https://github.com/foochane/aw-pay](https://github.com/foochane/aw-pay)

这个代码只是简单的写controller层的代码，只是为了快速体验一下支付宝的接口调用，后期将进行具体的完善。

