---
title: 从零开始搭建一个免费的图床
description: 
date: 2025-10-24
slug: build-a-free-image-hosting
image: https://ryan1998.dpdns.org/cover/post/build/build-a-free-image-hosting-2.png
categories:
    - Cloudflare
    - DigitalPlat
    - Github
tags:
    - Cloudflare R2
    - DigitalPlat
    - FreeDomain
---

## 资源使用说明

本教程基于 **Cloudflare R2** 和 **DigitalPlat FreeDomain** 实现, 需要在这两个平台上注册相应的账号, 根据注册账号引导按步骤完成注册即可.

- [**Cloudflare R2**](https://www.cloudflare.com/) :  提供免费的对象存储服务, 是本教程的核心资源
- [**DigitalPlat FreeDomain**](https://domain.digitalplat.org/) :  提供免费的域名, 方便直接访问 Cloudflare R2 上的资源

## 在 Cloudflare 平台上创建 R2 对象存储的存储桶

1. 登录 Cloudflare 平台后, 依次点击 存储和数据库=>R2对象存储=>概述

    ![bind-payment](https://ryan1998.dpdns.org/illus/build-image-hosting/1.01-bind-payment.png)

    > 如果是新账号会提示绑定支付方式(信用卡/Paypal)
    > 
    > Cloudflare R2 提供 10G/月 的免费额度, 用于存放博客里的图片(封面,插图)是完全足够的.

2. 完成绑定支付方式之后, 即可点击右上角创建存储桶.
   ![create-bucket-1](https://ryan1998.dpdns.org/illus/build-image-hosting/1.02-create-bucket-1.png)

3. 在创建存储桶页面设置存储桶名称及位置(国内建议选择:亚太地区), 然后点击创建存储桶.
   ![create-bucket-2](https://ryan1998.dpdns.org/illus/build-image-hosting/1.03-create-bucket-2.png)

   稍等一会儿后会提示"您的存储桶已准备就绪。添加文件即可开始使用", 预示着存储桶创建完成!
   ![create-bucket-complete](https://ryan1998.dpdns.org/illus/build-image-hosting/1.04-create-bucket-complete.png)


## 在 DigitalPlat 平台上创建 FreeDomain 免费域名

0. 访问 [注册账号](https://ryan1998.dpdns.org/illus/build-image-hosting/https://dash.domain.digitalplat.org/auth/register) 页面

    按提示输入信息, 信息确认无误后, 点击 Register 按钮
    ![create-account](https://ryan1998.dpdns.org/illus/build-image-hosting/2.01-create-account.png)

1. 然后会提示 注册账号的验证邮件已经发送到邮箱里,需要去邮箱里验证一下
    ![verification-email](https://ryan1998.dpdns.org/illus/build-image-hosting/2.02-verification-email.png)

2. 登录到邮箱, 如何收件箱没有, 则可能在垃圾邮件里. (Gmail收不到邮件,建议使用其他邮箱)
    ![verification-email](https://ryan1998.dpdns.org/illus/build-image-hosting/2.03-verification-email-2.png)

    复制验证邮件里的验证链接 http://dash.domain.digitalplat.org/auth/register/* 到浏览器了打开即可完成验证
    ![registration-successful](https://ryan1998.dpdns.org/illus/build-image-hosting/2.04-registration-successful.png)


3. 回到 [账号登录](https://ryan1998.dpdns.org/illus/build-image-hosting/https://dash.domain.digitalplat.org/auth/login) 页面, 使用刚才注册好的账号进行登录
   ![login](https://ryan1998.dpdns.org/illus/build-image-hosting/2.05-login.png)

   登录后选择 KYC 验证方式(仅支持 Github OAuth 验证)
   ![KYC-verification](https://ryan1998.dpdns.org/illus/build-image-hosting/2.06-KYC-verification.png)
   使用 Github 账号登录验证完成之后进入 DigitalPlat FreeDomain 首页

4. 登录到 DigitalPlat FreDomain 平台, 按提示收藏 Github 上的 FreeDomain 项目即可获得1个额外的域名
    
    (DigitalPlat FreeDomain 平台默认仅赠送一个免费域名)

    登录 DigitalPlat FreeDomain 平台, 按首页提示打开 Github 上的 FreeDomain 项目, 
    
    在 Github 上给 FreeDomain 项目点击右上角 ⭐️Star 
    
    回到 DigitalPlat FreeDomain 平台, 验证 Github 账号即可<u>获得1个额外的免费域名</u>
    ![get-a-extra-domain](https://ryan1998.dpdns.org/illus/build-image-hosting/2.07-get-a-extra-domain.png)


5. 在 DigitalPlat FreeDomain 平台点击左侧菜单栏中的 Domain Registration
    
    在 Domian name 输入框输入想要注册的域名

    选择 dpdns.org 作为主域名

    勾选 同意条款

    点击 Check Availability 检查域名是否可用
    ![Check Availability](https://ryan1998.dpdns.org/illus/build-image-hosting/2.08-Check-Availability.png)

    如果可用则已经注册成功!

## 将刚注册好的域名托管到 Cloudflare 平台上

1. 在 Cloudflare 上点击添加域, 并输入刚才注册好的域名, 然后点击继续
    ![alt text](https://ryan1998.dpdns.org/illus/build-image-hosting/3.01-add-domain.png) 

2. 选择免费计划
   ![alt text](https://ryan1998.dpdns.org/illus/build-image-hosting/3.02-select-free.png) 

3. 将 Cloudflare 生成的域名服务器替换到 DigitalPlat 自动生成的域名服务器, 并保持配置
    ![alt text](https://ryan1998.dpdns.org/illus/build-image-hosting/3.03-config-ns-server.png) 
    ![alt text](https://ryan1998.dpdns.org/illus/build-image-hosting/3.04-config-ns-server-2.png) 

4. 在 Cloudflare 上点击下方的 立即检查名称服务器
    ![alt text](https://ryan1998.dpdns.org/illus/build-image-hosting/3.05-check-ns-server.png) 

5. 稍等一会儿, 域名状态将会变成 活动
    ![alt text](https://ryan1998.dpdns.org/illus/build-image-hosting/3.06-change-status.png) 


## 将托管在 Cloudflare 上的域名绑定到 R2存储桶 的自定义域中

1. 将域名绑定到R2存储桶中
    ![alt text](https://ryan1998.dpdns.org/illus/build-image-hosting/4.01-bind-domain.png) 

2. 通过域名访问R2存储桶中的资源
    ![alt text](https://ryan1998.dpdns.org/illus/build-image-hosting/4.02-access-resources.png) 
    现在就可以通过域名访问R2存储桶中的资源了.
    ![alt text](https://ryan1998.dpdns.org/illus/build-image-hosting/4.03-access-resources2.png) 


## 总结
1. 在Cloudflare上创建R2存储桶
2. 在DigitalPlat上注册域名
3. 将新注册的域名托管到Cloudflare上
4. 将托管好的域名绑定到R2存储桶自定义域中