---
layout:     post
title:      TimeLineSec安全教程（二）————XSS
subtitle:   记录实验靶机过程中的积累
date:       2020-03-06
author:     Ultimater
header-img: img/post-bg-timelinesec.jpg
catalog: true
tags:
    - 网络安全
    - TimeLine Sec
---

>**感谢 TimeLine Sec 组织的此次基础教程实验，帮助众多安全人员巩固基础和提升技能，尤其对我这样的入门者，更是一个从理论向实践发展的好的契机！同时，MS08067安全实验室出版的《Web安全攻防——渗透测试实验指南》作为我实验过程中用来随时随地加深理论认知的书籍，内容详实清晰，循序渐进，起到了巨大的作用，特此感谢！**

### 一、前言

跨站脚本（XSS）是一种针对网站应用程序的安全漏洞攻击技术，是代码注入的一种。它允许恶意用户将代码注入网页，其他用户在浏览网页时就会受到影响。恶意用户利用XSS代码攻击成功后，可能得到很高的权限（如执行一些操作）、私密网页内容、会话和cookie等各种内容。

XSS攻击可以分为三种：反射型、存储性和DOM型。

危害：存储型 > 反射型 > DOM型

    反射型：交互的数据一般不会被存在数据库里面，一次性，所见即所得，一般出现在查询页面等；

    存储型：交互的数据会被存在数据库里面，永久性存储，一般出现在留言板，注册等页面；

    DOM型：不与后台服务器产生数据交互，是一种通过DOM操作前端代码输出的时候产生的问题，一次性，也属于反射型。

XSS漏洞测试流程：

    1.在目标上找输入点，比如查询接口、留言板；

    2.输入一组 “特殊字符（>，'，"等）+唯一识别字符” ，点击提交后，查看返回源码，看后端返回的数据是否有处理；

    3.通过搜索定位到唯一字符，结合唯一字符前后语法确定是否可以构造执行js的条件（构造闭合）；

    4.提交构造的脚本代码（以及各种绕过姿势），看是否可以成功执行，如果成功执行则说明存在XSS漏洞。

本次课程使用的靶场是xss.haozi.me和Pikachu。

### 二、xss.haozi.me——0x01

![svhvdbjshvdkskjvd](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/svhvdbjshvdkskjvd)

看到注入点是在`<textarea> </textarea>`标签中，我们只要在构造的payload中闭合掉该标签就行。

payload——`</textarea><script>alert(1)</script><textarea>`

![QQ图片20200306165848.png](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/QQ%E5%9B%BE%E7%89%8720200306165848.png)

### 三、xss.haozi.me——0x03

![ajlhvjsahvdjsdsifavh](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/ajlhvjsahvdjsdsifavh)

由服务端代码可以看到，将圆括号和方括号都进行转义处理，所以此时我们用html实体转义对输入的括号进行转义，构造payload:`<img src=1 onerror=alert&#40;1&#41;>`，其中`%#40；`是`（ `的html实体转义，`%#41；`是` ）`的html实体转义。原理：我们的内容输入到后台之后避开了括号的检测，之后服务器将payload返回到客户端时，浏览器进行html解析的过程中就会将两个html实体解析为我们要的括号。

![khvskaskjvkjsvkjskvdsv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/khvskaskjvkjsvkjskvdsv)

### 四、xss.haozi.me——0x05

![dfsjvjsdvksdbkdvbshdbv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/dfsjvjsdvksdbkdvbshdbv)

通过服务端代码可以看出这里将输入中的`-->`会替换为笑脸，来防止我们输入恶意代码对前面的注释符`<!--`进行闭合。

![wgjskjbjshdkvhskvd](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/wgjskjbjshdkvhskvd)

此时考察的是对注释符的理解，注释符一般有两种，一种是`<!-- -->`，而另一种是`<!-- --!>`。知道这个知识点之后，我们就可以通过第二种注释符的形式进行绕过，构造payload：`--!><script>alert(1)</script>`

![dshvkshdkvjhskjdvhksv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/dshvkshdkvjhskjdvhksv)

### 五、xss.haozi.me——0x06

![sjvjsdjvsjdvkjshdkjvhskjdv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/sjvjsdjvsjdvkjshdkjvhskjdv)

可见服务端用正则表达式过滤了`auto`，`on.*=`,` > `并将他们转为` _ `，但是此处没有过滤换行符，所以可以通过换行来绕过匹配。

```
type="image" src="" onerror
=alert(1)
```

![sjvjsdvkjskjdvssbs](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/sjvjsdvkjskjdvssbs)

### 六、xss.haozi.me——0x08

![sdvdsvksjdvkjsdjvskjdv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/sdvdsvksjdvkjsdjvskjdv)

正则过滤了`</style>`，导致我们无法闭合前面的`<style>`标签，此时通过加空格的方式来造成正则逃逸。构造payload：`</style ><script>alert(1)</script>`

![adbbsbdbdbfdfbdndngdbgd](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/adbbsbdbdbfdfbdndngdbgd)

### 七、XSS之盲打

![svhdvjsdvjsvdahgcsjagjc](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/svhdvjsdvjsvdahgcsjagjc)

xss盲打不是一种攻击方式，而是一种应用场景.

这里输入什么都只会收到“ 谢谢参与，阁下的看法我们已经收到！”。因此猜测输入的内容都会在后台显示，我们现在模拟管理员登陆后台。

![kskbjvskjdvkjsvdkjdsv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/kskbjvskjdvkjsvdkjdsv)

编号为8的数据就是我们输入的payload，可以看到在管理员后台出现了弹窗。

在实际应用场景中我们可以在输入框插入能获取用户cookie的XSS-payload，就可以在XSS平台等待管理员登陆后台之后的cookie，并模拟管理员身份完成攻击操作了。

### 八、XSS之过滤

实际中的系统，或多或少都会做一些安全措施，但是这些安全措施也能方法、逻辑不严谨，可以被绕过。

转换的思路：

    前端限制绕过，直接抓包重放，或者修改html前端代码。比如反射型XSS(get)中限制输入20个字符；

    大小写，比如<SCRIPT>aLeRT(111)</sCRIpt>。后台可能用正则表达式匹配，如果正则里面只匹配小写，那就可能被绕过；

    双写（拼凑），<scri<script>pt>alert(111)</scri</script>pt>。后台可能把<script>标签去掉换，但可能只去掉一次；

    注释干扰，<scri<!--test-->pt>alert(111)</sc<!--test-->ript>。加上注释后可能可以绕过后台过滤机制。

编码的思路：

    后台过滤了特殊字符，比如<script>标签，但该标签可以被各种编码，后台不一定过滤；

    当浏览器对该编码进行识别时，会翻译成正常的标签，从而执行。

**编码应该在输出点被正常识别和翻译，不能随便使用编码。**

![skvksdvkjhskjdvhjsdv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/skvksdvkjhskjdvhjsdv)

![kjdsvkjsdvjavhakjkjva](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/kjdsvkjsdvjavhakjkjva)

![svljhskjdvksbdvkbsdv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/svljhskjdvksbdvkbsdv)

经过测试发现`<script`字符被过滤了，此处我们先尝试用大小写绕过。

![shvkjhskvjhskfvsvvd](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/shvkjhskvjhskfvsvvd)

![akjvkavhashvbhsdbvh](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/akjvkavhashvbhsdbvh)

发现大小写轻松绕过对`<script`的过滤。

### 九、XSS之htmlspecialchars

htmlspecialchars()是PHP里面把预定义的字符转换为HTML实体的函数。

**(1)预定义的字符是:**

`&` 成为 `&amp`

`"` 成为 `&quot`

`'` 成为 `&#039`

`<` 成为 `&lt`

`>` 成为 `&gt`

**(2)可用引号类型:**

`ENT_COMPAT`：默认，仅编码双引号；

`ENT_QUOTES`：编码双引号和单引号；

`ENT_NOQUOTES`：不编码任何引号。

我们先来看看后台过滤了哪几种字符串

![djnvjskjdvskjvdkjsdv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/djnvjskjdvskjvdkjsdv)

![wgjkvkjsdvkjsdvkjskdv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/wgjkvkjsdvkjsdvkjskdv)

可以看到`<`,`>`,`&`,`"`都被转换成了html实体，只有单引号`'`没有被转换。

根据源码 `<p class='notice'>你的输入已经被记录:</p><a href=''&lt;&gt;&amp;&quot;123'>'&lt;&gt;&amp;&quot;123</a> `,我们可以先用单引号闭合a标签，然后构造payload弹框————`'onclick='alert(1)'`。

提交之后点击返回页面的超链接就能看到弹框。

![hsdvkjhskjdhvjshdv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/hsdvkjhskjdhvjshdv)

### 十、XSS之href输出

这里我们随意输入一串数字

![dsbsdsdvsvsdvsdvs](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/dsbsdsdvsvsdvsdvs)

点击超链接并查看下源码

![dasvsvsdvsdvsdvsd](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/dasvsvsdvsdvsdvsd)

发现我们输入的字符显示在a标签的href属性中，而且通过查看源码可以知道会对百度以外的网址进行htmlspecialchars()转义。在这里我们可以用javascript协议来执行JS，构造payload——`javascript:alert(1)`，此时点击标签就会出现弹框了。

![sdvsdkvsdvksdvs](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/sdvsdkvsdvksdvs)

![sdgsgsdasafscacvascv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/sdgsgsdasafscacvascv)

**如果要对 href 做处理，一般有两个逻辑：**

    输入的时候只允许 http 或 https 开头的协议，才允许输出

    其次再进行htmlspecialchars处理

### 十一、XSS之js输出

查看网页源代码

![sdbssfvsgsvdsvsbvs](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/sdbssfvsgsvdsvsbvs)

我们发现该页面将我们的输入带入js中动态执行，此时我们要绕过的话，只需要闭合掉源码的<script>之后再插入我们的js标签代码就可以了。因此构造payload——`</script><script>alert(6666)</script><script>`

![agvasdvasfasfvasfv](https://raw.githubusercontent.com/U1timater/U1timater.github.io/master/img-in-issue/agvasdvasfasfvasfv)

************************************************************************************
### 部分参考网页

[XSS编码与绕过](https://www.cnblogs.com/flycat-2016/p/6507543.html)

[HTML特殊符号对照表](https://tool.chinaz.com/tools/htmlchar.aspx)

[XSS过滤绕过技巧](https://blog.csdn.net/Fly_hps/article/details/79623908)

[XSS 绕过常用语句](https://damit5.com/2017/10/15/XSS-%E7%BB%95%E8%BF%87%E5%B8%B8%E7%94%A8%E8%AF%AD%E5%8F%A5/)

[XSS过滤圆括号一些可执行的payload](http://zone.secevery.com/article/305)












































