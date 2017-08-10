---
layout:    post
title:     "打造简单的个人博客"
subtitle:   " \"简单，笨笨的我\""
date:       2017/08/06
author:     "CaoFan"
header-img: "img/"
catalog: true
tags:
  - 其他

---

>"It's begining!"


## 前言


Fan Blog就这么样我拖拖拉拉的开始了，简直要被自己蠢死了！！

作为这个blog主人的原型说挂载在别人的地方太没意思了，然而我现在的水平也只能膜拜您了。嘻嘻！哎，不得不实话实说，就这个也搞了我几天，买域名，之后的我手足无措，原谅我是个电脑白痴，一步一步的完成了这个简单的blog，总之现在的我可以专心写我自己的东西了！


<p id="build"></p>
---

## 正文

**1.买域名**：① GoDaddy(本人就是在这里买的)后期听说有优惠码，大家可以试着去找找，应该可以便宜一点。主要是还支持支付宝。
         ② 阿里云  ③ name.com(看到知乎上面好多人推荐这个，大家也可以试试，毕竟我也是第一次买，没有经验) 
         ④ namecheap(也是知乎上推荐的，据说价格不便宜，但是速度之类的比较好，如果你是单纯的个人应用的话，价格差不多就可以了，不需要太贵，也不能太便宜)
* 还有一些注意：尽量买.com，接下来是.cn，.net。
               对于域名的选取，一般起简单易记的，大部分人会使用自己的中文或者英文名字。

这里是我购买的域名界面

![img](/img/in-post/first-article.jpg)

**2.域名解析**: 我使用的是DNSpod(https://www.dnspod.cn)免费解析域名。

这里是界面，添加你的域名

![img](/img/in-post/first-article-dnspod.jpg)

添加域名之后点击进去会出现

![img](/img/in-post/first-article-NS.jpg)

将里面的记录值填写到GoDaddy里面的Domain Manager更加服务器记录值

![img](/img/in-post/first-article-Nameservers.jpg)

这时我们要在DNSpod里面添加记录

![img](/img/in-post/first-article-add.jpg)

**解释一下：**
  1. 主机记录：表示的是域名的前缀。
    www: 表示解析www.webcaofan.com
    @: 表示解析webcaofan.com
  2. 记录类型
    A：表示域名指向一个ip地址。
    NS：表示域名服务器记录，DNSpod给我们提供了。
    CNAM: 表示该域名指向的另一个域名的ip地址。
这里我们是需要利用github page, 所以我们A记录的ip地址指向的是你github的地址，CNAM填写的是xxx.github.io.,这里的xxx表示是你的github用户名。

这样的话，等待一段时间大概十几分钟到半个小时，你的域名就解析完成了，接下来我们就可以搭建你的blog了。


**3.利用jekyll+github page搭建blog**
  * 你可以在jekyll模板里选自己喜欢的模板         (http://jekyllthemes.org/),然后fork到你的github上或者可以下载到本地，自己新建一个pro。
  * 不论是fork还是自己pro，都需要改名字，名字必须改为xxx.github.io,其中这个xxx随便你自己取。
  * 你可以参照README.md来修改为自己的内容。
  * 使用自己的域名，创建一个CNAME的文件夹，里面写你的域名，如webcaofan.com，即可。

## 总结

至此你这个简单的blog应该搭建完成了，其实就是简单的git应用，学好git是多么的重要，还有学好英语是多么的重要，那些theme都是英文的介绍，头大的我真的是果断选择中文的theme，嘻嘻。。


