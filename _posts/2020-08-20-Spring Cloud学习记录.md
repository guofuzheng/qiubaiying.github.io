---
layout:     post                    # 使用的布局（不需要改）
title:      Spring Cloud学习记录               # 标题 
subtitle:   Spring Cloud学习记录 #副标题
date:       2020-08-20              # 时间
author:     GFZ                     # 作者
header-img: img/post-bg-miui-ux.jpg    #这篇文章标题背景图片
catalog: true                       # 是否归档
tags:                               #标签
    - Spring Cloud

---   
### 1、OpenFeign的调用格式
OpenFeign主要是面向接口的服务调用，就是说在服务提供方的接口是啥，他就在service定义一个接口，添加上FeignClient注解，在Controller里就可以调用了，但是有几点需要注意：1. 定义OpenFeign接口时需要加上服务方的调用接口，同时有参数的需要PathVariable。2. FeignClient中的就是服务方的服务名。3. 其实调用方的service里调用的其实是服务方的Controller里的方法。  
调用方的service书写方式如下：
```java
package org.example.springcloud.service;

import org.example.springcloud.entity.CommonResult;
import org.example.springcloud.entity.PaymentEntity;
import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.stereotype.Service;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

/**
 * @author guofuzheng
 */
@Service
@FeignClient(value = "cloud-payment-service")
public interface PaymentFeignService {
    /**
     * 依据主键id获取payment实体对象
     * @param paymentId payment主键
     * @return PaymentEntity
     */
    @GetMapping(value = "/payment/get/{id}")
    CommonResult<PaymentEntity> getPaymentById(@PathVariable("id") Long paymentId);
}
```   
调用方的Controller书写方式如下：
```java
package org.example.springcloud.controller;

import lombok.extern.slf4j.Slf4j;
import org.example.springcloud.entity.CommonResult;
import org.example.springcloud.entity.PaymentEntity;
import org.example.springcloud.service.PaymentFeignService;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

/**
 * @ClassName : OrderFeignController  //类名
 * @Description :        //描述
 * @Author : guofuzheng  //作者
 * @Date: 2020-08-20 14:13  //时间
 */
@RestController
@Slf4j
public class OrderFeignController {
    @Resource
    private PaymentFeignService paymentFeignService;
    @GetMapping(value = "/consumer/payment/get/{id}")
    public CommonResult<PaymentEntity> getPaymentById(@PathVariable("id") Long paymentId){
        return paymentFeignService.getPaymentById(paymentId);
    }
}
```
### 2、OpenFeign的超时控制
OpenFeign默认等待服务1秒，如果服务提供方的返回结果所用的时间超过1秒，那么OpenFeign就会报错，超时控制是在调用方的application.yml中配置，但是我配置的时候没有找到，网上的是:
```yml
ribbon.ReadTimeout=5000 //处理请求的超时时间，为5秒
ribbon.ConnectTimeout=5000 //连接建立的超时时长，为5秒
```
