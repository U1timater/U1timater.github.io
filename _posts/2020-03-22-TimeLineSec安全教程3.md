---
layout:     post
title:      TimeLineSec安全教程（三）————文件上传、下载、包含漏洞
subtitle:   记录实验靶机过程中的积累
date:       2020-03-22
author:     Ultimater
header-img: img/post-bg-timelinesec.jpg
catalog: true
tags:
    - 网络安全
    - TimeLine Sec
---

>**感谢 TimeLine Sec 组织的此次基础教程实验，帮助众多安全人员巩固基础和提升技能，尤其对我这样的入门者，更是一个从理论向实践发展的好的契机！同时，MS08067安全实验室出版的《Web安全攻防——渗透测试实验指南》作为我实验过程中用来随时随地加深理论认知的书籍，内容详实清晰，循序渐进，起到了巨大的作用，特此感谢！**

# 【文件上传漏洞】

### 一、原理简介

网站web应用都有一些文件上传功能，比如文档、图片、头像、视频上传，当上传功能的实现代码没有严格校验上传文件的后缀和文件类型，此时攻击者就可以上传一个webshell到一个web可访问的目录上，并将恶意文件传递给PHP解释器去执行，之后就可以在服务器上执行恶意代码，进行数据库执行、服务器文件管理，服务器命令执行等恶意操作。

根据网站使用及可解析的程序脚本不同，此处可以上传的恶意脚本可以是PHP、ASP、JSP文件，也可以是篡改后缀后的这几类脚本。

WebShell 就是以 asp、php、jsp 或者 cgi 等网页文件形式存在的一种命令执行环境，也可以将其称之为 一种网页后门。攻击者在入侵了一个网站后，通常会将这些 asp 或 php 后门文件与网站服务器 web 目录 下正常的网页文件混在一起，然后使用浏览器来访问这些后门，得到一个命令执行环境，以达到控制网站服务器的目的(可以上传下载或者修改文件，操作数据库，执行任意命令等）。

本篇教程用到的靶机为DoraBox，有兴趣的同学可以自行搭建靶场并完成实验。

### 二、JS限制文件上传

客户端检测一般只是在JavaScript代码中加入了对扩展名的黑白名单检查，这种方式只能防止一些普通用户上传错误，只要用burpsuite在文件上传时进行截断改文件后缀名就可绕过。

在上传点上传后缀名为php类型的文件时发现被前端JS限制上传并弹框

![akdbvkabkv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/akdbvkabkv)

此处我们将文件后缀名改为.jpg之后进行上传，绕过浏览器的前端检测之后打开burp抓取我们的上传包

![sauvadvadbv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/sauvadvadbv)

在已经绕过前端、即将发送出去的数据包中将文件后缀又改回php格式，之后放行数据包

![sdvbakdbvabvdhabdv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/sdvbakdbvabvdhabdv)

成功上传文件

![dsdvkbsvhbshbvd](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/dsdvkbsvhbshbvd)

### 三、MIME限制文件上传

MIME类型的检查就是检查数据包中Content-Type的值，MIME类型决定了某种扩展名用什么应用程序打开。

我们可以先上传jpg格式文件，并抓包

![jhksdvbkhabdvk](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/jhksdvbkhabdvk)

发现这里的Content-Type参数变为 image/jpeg，我们只需要将文件再改为php后缀就可以绕过Content-Type限制并让服务端正常解析文件了

![dsjdbvkbv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/dsjdbvkbv)

改完之后数据包放行，发现上传成功

![akdvkavdbkabdv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/akdvkavdbkabdv)

### 四、扩展名限制文件上传

扩展名限制文件上传与前端js后缀名检测类似，只不过是在后端进行检查，有时候还可以配合解析漏洞结合目录路径攻击。

在前期上传绕过服务端检测时，发现在我们靶机为phpstudy和wamp的环境下，可以通过php3、phtl、pht等格式绕过服务端检测进行上传，但是并不能解析；之后又利用了Apache的后缀名从右向左解析的漏洞（Apache的解析漏洞主要特性为Apache是从后面开始检查后缀，按最后一个合法后缀执行，整个漏洞的关键就是Apache的合法后缀到底是哪些，不是合法后缀的都可以被利），也是能上传但是不能被解析；也尝试了%00截断方法进行绕过，依旧不能被解析。

如图所示，演示利用Apache解析漏洞上传成功但无法解析的情况

![adhvkjvkjakvj](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/adhvkjvkjakvj)

![avkaksca](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/avkaksca)

由于靶机是被部署在windows服务器下的，经过多次尝试，突然想到可以利用Windows会自动的忽略掉文件最后面的` . `的特性，在上传抓包的过程中将` webshell.php `修改为` webshell.php. `，从而进行绕过限制，并且也可以保证上传的文件能够被phpstudy当作php文件解析

![ahcbkahckah](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/ahcbkahckah)

可见成功绕过限制

![dvsddbsbsbsb](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/dvsddbsbsbsb)

上传成功后服务器Windows的机制会自动将最后的` . `抹去，此时我们再打开上传的文件

![sadvhkajvbkjabvk](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/sadvhkajvbkjabvk)

成功解析~

### 五、内容限制文件上传

这道题我们打开后台源码，看到内容限制的php源码中用到了`getimagesize()`函数，这个函数将测定任何 GIF，JPG，PNG，SWF，SWC，PSD，TIFF，BMP，IFF，JP2，JPX，JB2，JPC，XBM 或 WBMP 图像文件的大小并返回图像的尺寸以及文件类型和一个可以用于普通 HTML 文件中 IMG 标记中的 height/width 文本字符串。所以这里凡是上传不含图片的文件，都会因检测不到图像大小而显示上传失败。所以我们开始图片马的制作，图片马顾名思义就是以图片作为载体的木马文件，我们将恶意代码插入到图片文件中进行上传。

```
使用CMD制作一句话木马。
参数/b指定以二进制格式复制、合并文件; 用于图像类/声音类文件
参数/a指定以ASCII格式复制、合并文件。用于txt等文档类文件
copy 1.jpg/b+1.php 2.jpg 
//意思是将1.jpg以二进制与1.php合并成2.jpg
那么2.jpg就是图片木马了。
```

![slvjdkjdbvk](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/slvjdkjdbvk)

生成的图片马webshell.jpg已经自动在桌面生成

![ajvdbkabvka](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/ajvdbkabvka)

上传成功，要用菜刀连接的时候发现要利用的话仅仅图片马是不能被网页解析也不能被菜刀连接的，要让木马图成功的发挥作用必须是php、asp等后缀的文件才行

心生一计，既然后台只检验文件的内容而不检验文件的后缀名，干脆就将图片马的名称由webshell.jpg改为webshell.jpg.php，我们再次上传

![avhkadvkjakvd](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/avhkadvkjakvd)

上传成功，果然没有对后缀名做限制，此时该php也可以被解析

![ajdbvkabvkhaadnfndn](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/ajdbvkabvkhaadnfndn)

用菜刀连接也能成功连接了

![jvkjsdabvkjskv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/jvkjsdabvkjskv)

### 六、上传文件的分类

1.上传文件是PHP、JSP、ASP等脚本代码，服务器的Web容器解释并执行了用户上传的脚本，导致代码执行；

 
2.上传文件是crossdomain.xml，导致可以控制Flash在该域下的行为(其他通过类似方式控制策略文件的情况类似);

 
3.上传文件是病毒、木马文件，黑客用以诱骗用户或者管理员下载执行植入到pc中。

 
4.上传文件是钓鱼图片或为包含了脚本的图片，在某些版本的浏览器中会被作为脚本执行，被用于钓鱼和欺诈。

在大多数情况下，文件上传漏洞一般都是指“上传的Web脚本被服务器解析从而获取网站shell权限”，也就是webshell，要完成上传漏洞攻击需要满足以下几个条件:

1.上传的文件能够被Web容器解释执行，所以文件上传后所在的目录需要解析器可以执行目录下的文件，也就是说文件目录必须在web容器覆盖路径内才行。

 
2.用户可以直接通过浏览器进行访问这个shell文件，如果web容器不能解析这个文件，那么也不能算是漏洞。

 
3.最后，上传的shell文件如果被安全检查、格式化、图片压缩等功能改变了内容，则也可能导致攻击不成功。

### 七、修复与防御

1.设置文件上传的目录设置为不可执行;

 
2.使用白名单方式检查文件类型，处理图片可以使用压缩函数或者resize函数，在处理图片的同时破坏图片中可能包含的恶意代码;

 
3.对文件进行随机性且不可猜解的重命名;

 
4.不能有本地文件包含漏洞及解析漏洞。

# 【文件下载漏洞】

### 一、原理简介

文件下载功能在很多web系统上都会出现，一般我们当点击下载链接，便会向后台发送一个下载请求，一般这个请求会包含一个需要下载的文件名称，后台在收到请求后 会开始执行下载代码，将该文件名对应的文件response给浏览器，从而完成下载。 如果后台在收到请求的文件名后,将其直接拼进下载文件的路径中而不对其进行安全判断的话，则可能会引发不安全的文件下载漏洞。

此时如果 攻击者提交的不是一个程序预期的的文件名，而是一个精心构造的路径(比如`../../../etc/passwd`),则很有可能会直接将该指定的文件下载下来。 从而导致后台敏感信息(密码文件、源代码等)被下载。

所以，在设计文件下载功能时，如果下载的目标文件是由前端传进来的，则一定要对传进来的文件进行安全考虑。

### 二、Unsafe Filedownload

![skdvbksddbvbsdv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/skdvbksddbvbsdv)

点击球星下边的名字链接就可以下载对应的照片，我们抓包看看。

![skugvksvkskvdv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/skugvksvkskvdv)

看到服务器通过我们上传的`filename`参数下载文件，我们尝试切换路径

![sacakjvhakvhavav](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/sacakjvhakvhavav)

在后台目录看到我们下载的路径在download文件夹，我们尝试返回一级目录并尝试能否下载其中的`execdownload.php`文件

![alvdjjlvjdvj](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/alvdjjlvjdvj)

成功进行了文件下载。

因此我们尝试进行构造路径，获取`C:/Windows/win.ini`文件

![akvkabvkabkvhb](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/akvkabvkabkvhb)

成功获取。这里有个注意的点是`../`返回的层级一定要达到大于等于当前位置的文件深度，这样才能保证路径最终返回到C盘根目录下。还需要注意的是桌面文件夹是在C盘内的，桌面文件夹的路径到C盘的路径还有大概三层深度，在做任意文件下载漏洞的过程中不能只反向到桌面文件夹的位置就停止，这是我在实践的过程中遇到的一个小障碍，分享出来作为教训。

### 三、任意文件读取

打开靶机页面看到是一个下载文件界面

![jbvskvdbkabdvkabv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/jbvskvdbkabdvkabv)

由于有了之前的经验，这里我直接回撤了十几个目录（必顶到C盘），然后尝试读取了`win.ini`文件并成功,如图

![ajvdjabvkjabsvk](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/ajvdjabvkjabsvk)

看看网页效果，已经读取到了win.ini的内容

![adjlvjadvbja](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/adjlvjadvbja)

![adlvhuvdkadbvk](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/adlvhuvdkadbvk)

### 四、常见下载利用文件位置

1.Windows：

```
C:\boot.ini //查看系统版本

C:\Windows\System32\inetsrv\MetaBase.xml //IIS配置文件

C:\Windows\repair\sam //存储系统初次安装的密码

C:\Program Files\mysql\my.ini //Mysql配置

C:\Program Files\mysql\data\mysql\user.MYD //Mysql root

C:\Windows\php.ini //php配置信息

C:\Windows\my.ini //Mysql配置信息

C:\Windows\win.ini //Windows系统的一个基本系统配置文件
```

2.Linux:

```
/root/.ssh/authorized_keys

/root/.ssh/id_rsa

/root/.ssh/id_ras.keystore

/root/.ssh/known_hosts //记录每个访问计算机用户的公钥

/etc/passwd

/etc/shadow

/etc/my.cnf //mysql配置文件

/etc/httpd/conf/httpd.conf //apache配置文件

/root/.bash_history //用户历史命令记录文件

/root/.mysql_history //mysql历史命令记录文件

/proc/mounts //记录系统挂载设备

/porc/config.gz //内核配置文件

/var/lib/mlocate/mlocate.db //全文件路径

/porc/self/cmdline //当前进程的cmdline参数
```

# 【文件包含漏洞】

### 一、原理简介

程序开发人员一般会把重复使用的函数写到单个文件中，需要使用某个函数时直接调用此文件，而无需再次编写，这中文件调用的过程一般被称为文件包含。

程序开发人员一般希望代码更灵活，所以将被包含的文件设置为变量，用来进行动态调用，但正是由于这种灵活性，从而导致客户端可以调用一个恶意文件，造成文件包含漏洞。

几乎所有脚本语言都会提供文件包含的功能，但文件包含漏洞在PHP Web Application中居多,而在JSP、ASP、ASP.NET程序中却非常少，甚至没有，这是有些语言设计的弊端。

在PHP中经常出现包含漏洞，但这并不意味这其他语言不存在。

常见PHP包含函数的意思：

```
include()：执行到include时才包含文件，找不到被包含文件时只会产生警告，脚本将继续执行
```

```
require()：只要程序一运行就包含文件，找不到被包含的文件时会产生致命错误，并停止脚本
```

```
include_once()和require_once()：若文件中代码已被包含则不会再次包含
```

### 二、漏洞分类

1.本地文件包含漏洞——LFI

本地包含顾名思义，就是在网站服务器本身存在恶意文件，然后利用本地文件包含使用

2.远程文件包含漏洞——RFI

远程文件包含就是调用其他网站的恶意文件进行打开

### 三、漏洞验证

```
index.php?f=../../../../../../etc/passwd

index.php?f=../index.php

index.php?f=ﬁle:///etc/passwd
```

当参数f的参数值为php文件时，若是文件被解析则是文件包含漏洞，

若显示源码或提示下载则是文件查看与下载漏洞。

### 四、任意文件包含

![ljavdbjabdvja](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/ljavdbjabdvja)

先通过后台看看该页面同路径下的文件，尝试打开txt.txt，发现会被该页面解析并展示，可知存在文件包含漏洞

![advnjanjvbnajlvn](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/advnjanjvbnajlvn)

![jdvgkavkahvadv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/jdvgkavkahvadv)

因为是任意文件包含，我们不用回撤目录，直接路径跳转C:/Windows/win.ini就能成功打开

![avjdjabvdkjabdjv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/avjdjabvdkjabdjv)

### 五、目录限制文件包含

![ahdvbhabvhab](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/ahdvbhabvhab)

这里我们发现使用C:/Windows/win.ini是无法让页面定位到我们的文件的，服务端对目录路径做了限制，只允许包含指定文件下的文件，所以需要我们进行回跳，跳到根目录（这里指C盘）再打开根目录下的文件。

![sdvjbsjdvbk](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/sdvjbsjdvbk)

# 【参考文献】

[简单粗暴的文件上传漏洞](https://paper.seebug.org/560/#apache)

[某些环境下绕过php后缀黑名单上传webshell](https://blog.csdn.net/nzjdsds/article/details/81412125)

[文件解析漏洞总结-Apache](https://blog.csdn.net/wn314/article/details/77074477)

[利用最新Apache解析漏洞（CVE-2017-15715）绕过上传黑名单](https://www.leavesongs.com/PENETRATION/apache-cve-2017-15715-vulnerability.html)

[由Upload-labs的几关引发的思考](https://xz.aliyun.com/t/5791)

[浅谈木马图提权](https://www.dazhuanlan.com/2019/10/12/5da10038c65d4/)

[上传漏洞的原理与总结](https://www.kanxue.com/book-6-235.htm)

[任意文件下载漏洞学习](https://www.cnblogs.com/zhaijiahui/p/8459661.html)

[Web安全Day9 - 文件下载漏洞实战攻防](https://xz.aliyun.com/t/6590)

[文件上传、文件包含和目路遍历杂谈](https://www.cnblogs.com/lsdb/p/9645663.html)

[文件包含漏洞原理分析](https://zhuanlan.zhihu.com/p/25069779)



