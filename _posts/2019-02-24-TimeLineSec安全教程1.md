---
layout:     post
title:      TimeLineSec安全教程（一）————SQL注入
subtitle:   记录做靶机的过程和过程中的积累
date:       2020-02-24
author:     Ultimater
header-img: img/post-bg-20191109.jpg
catalog: true
tags:
    - 网络安全
    - TimeLine Sec
---

>感谢 TimeLine Sec 组织的此次基础教程实验，帮助众多安全人员巩固基础和提升技能，尤其对我这样的入门者，更是一个从理论向实践发展的好的契机！同时，MS08067安全实验室出版的《Web安全攻防——渗透测试实验指南》作为我实验过程中用来随时随地加深理论认知的书籍，内容详实清晰，循序渐进，起到了巨大的作用，特此感谢！

### 一、前言

SQL注入就是指Web应用程序对用户输入数据的合法性没有判断，前端传入后端的参数是攻击者可控的，并且参数带入数据库查询，攻击者可以通过构造不同的SQL语句来实现对数据库的任意操作。

SQL注入按照不同的分类方法可以分为很多种，如报错注入、盲注、Union联合注入等。

本次 TimeLine Sec 的SQL注入实验靶机为 pikachu ，感兴趣的可以自行在本地搭建练习，详情见[搭建教程](https://mp.weixin.qq.com/s/z87ddIq79BlwSXNt0cMF2w)

### 二、数字型注入