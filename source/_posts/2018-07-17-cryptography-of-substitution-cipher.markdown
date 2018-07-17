---
layout: post
title: "密码学(二)之代换密码"
date: 2018-07-17 17:28:46 +0800
comments: true
categories: 
---

###（一）代换密码（替换密码）

上一讲中，我们讲移位密码其实是将字母表中的字母一一对应到各数字，然后通过数字平移来进行加密，古典密码学中还有一种比较有名的加密方法，就是将明文中的字母表对应到一套密文的字母表，这种加密方法我们叫**代换密码（substiution cipher）**或叫**替换密码**，下图就是一个简单的代换密码对应表
<!-- more --> 

![image](http://upload-images.jianshu.io/upload_images/1796624-db5bd0e79f37103e?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###（二）代换密码的加密

上面所说，代换密码其实就是将明文里的字母按照字母表替换成密文里的字母，还是举一个例子，假设现在有一个字符“welcome to china”，根据上面的密码替换表，将明文里面的每个字母依次换成对应的密文，如下：

```ruby
w->C
e->X
l->G
c->H
o->B
m->E
e->X
t->Q
o->B
c->H
h->T
i->J
n->N
a->W
```

这样就可以得到密文CXGHBEXQBHTJNW

###（三）代换密码的解密

代换密码的解密非常简单，只要将加密的替换表进行反向操作，这里不再操作

这里可以发现，代换密码主要是要建立起一套明文与密文之间的加密对应的替换关系，只要有这套密码替换表，加、解密就变得很容易

###（四）代换密码的破解

上一讲我们知道，移位密码其实是很好破解的，因为密钥总量一共就26位，只要我们试26次，就一定能试出一个正解的，那么代换密码是也可以通过穷举的方式来破解呢？

我们知道代换密码是把明文的26个字母随机对应密文的26个字母上，也就意味着明文中第一个字母a可以对应到密文中A,B,C,D…Z 26个字母中的任一个，以此类推，我们就可以计算出代换密码的密钥总数为：
```
26 x 25 x 24 x 23 x22 x 21 x … x 1 = 403291461126605635584000000
```
像这种一种密码能够使用的所有密钥的集合，叫做**密钥空间（keyspace）**

上面的密钥的量非常大，用穷举法来破译几乎是不可能的。

使用穷举法不能破译，但并不能说明就是安全的，我们可以使用[频率分析](https://baike.baidu.com/item/%E5%AD%97%E6%AF%8D%E9%A2%91%E7%8E%87/9669044?fr=aladdin)来破译代换密码，频率分析就是利用明文中的字母出现频率与密文中的字母的出现频率一致这一特性，

下面是【密码学原理与实践】书中的一个例子，可以参考一下

现假设有一段密文如下，现需将其解密出明文
```
YIFQFMZRWQFYVECFMDZPCVMRZWNMDZVEJBTXCDDUMJ

NDIFEFMDZCDMQZKCEYFCJMYRNCWJCSZREXCHZUNMXZ

NXUCDRJXYYSMRTMEYIFZWDYVZVYFZUMRZCRWNZDZJJ

XZWGCHSMRNMDHNCMFQCHZJMXJXWIFJYUCFWDJNZDIR
```
这种密文的频率分析如下图：

![image](http://upload-images.jianshu.io/upload_images/1796624-3e73907eab25c8f3?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

根据[英文字母出现的频率](https://zh.wikipedia.org/wiki/%E5%AD%97%E6%AF%8D%E9%A2%91%E7%8E%87)排序统计，一般的排序是这样的e,t,a,o,I,n,s,h,r,d,l,u,c,m,f,w,y,p,v,b,g,k,j,q,x,z 而且一般英语文章中出现频率最高的的字母是e,这一点基本不会错的。

根据上图所示，字母Z出现的次数是20，远远高于其它密文字母，所以我们可以假设Z->e。其它出现至少10余次的官方字母是C,D,F,J,M,R,Y，我们希望这些字母对应的是

e t a o l n s h r中的一个子集，

我们现在假设了Z->e，现注意一下形如-Z, Z-的两字母组，我们发现出现这种类型的最一般的两字母组是DZ和ZW,各都出现了4次；NZ和ZU出现3次，RZ HZ XZ FZ ZR ZV ZC ZD ZJ各出现2次；又因ZW出现4次，而WZ一次也未出现，同时W比许多其它字母出现的次娄少，所以我们可以假设W->d，又因为DZ出现4次而ZD出现2次，故可猜测D是{r,s,t}中的任一个，具体是哪个还不清楚。

如上面猜测， Z->, D->d，再看看密文并注意到ZRW出现在密文的开始部分，RW后面也出现过，因为R在密文频繁地出现，而nd是一个常见的两字母组，所以我们可以假设R->n作为可能的情况，这样我们便有了如下的形式
```ruby
------end---------e-----ned--e------------
YIFQFMZRWQFYVECFMDZPCVMRZWNMDZVEJBTXCDDUMJ
--------e----e---------n-d----en----e----e
NDIFEFMDZCDMQZKCEYFCJMYRNCWJCSZREXCHZUNMXZ
-e---n------n------ed-----e-e--ne-nd-e-e--
NZUCDRJXYYSMRTMEYIFZWDYVZVYFZUMRZCRWNZDZJJ
-ed-----n-----------e----ed-------d---e--n
XZWGCHSMRNMDHNCMFQCHZJMXJZWIFJYUCFWDJNZDIR
```
接下来我们可以试试N->h，因为NZ是一个常见的两字组而ZN不是一个常见的两字母组，如果这个猜测是正确的，则明文ne-ndhe很可能说明C->a，结合这些收市，我们又进一步有:
```
------end-----a---e-a--nedh--e------a-----
YIFQFMZRWQFYVECFMDZPCVMRZWNMDZVEJBTXCDDUMJ
h-------ea---e-a---a----ehad-a-en-a-e-h--e
NDIFEFMDZCDMQZKCEYFCJMYRNCWJCSZREXCHZUNMXZ
he-a-n------n------ed---e---e--neandhe-e--
NZUCDRJXYYSMRTMEYIFZWDYVZVYFZUMRZCRWNZDZJJ
-ed-a---nh---ha---a-e----ed-----a-d--he--n
XZWGCHSMRNMDHNCMFQCHZJMXJZWIFJYUCFWDJNZDIR
```
现在考虑出现次数高的密文字母M,由前面分析，密文段RNM密钥成nh-，这说明h-是一个词的开头，所以M很可能是一个元音，因为已经使用了a和e,所以猜测M->{i或o}，因为ai是一个比ao出现次数更高的明文组，所以首先猜测M->I，这样就有：
```
-----iend-----a-i-e-a-inedh--e------a---i-
YIFQFMZRWQFYVECFMDZPCVMRZWNMDZVEJBTXCDDUMJ
h-----i-ea-i-e-a---a-i-nhad-a-en--a-e-hi-e
NDIFEFMDZCDMQZKCEYFCJMYRNCWJCSZREXCHZUNMXZ
he-a-n-----in-i----ed---e---e-ineandhe--e-
NZUCDRJXYYSMRTMEYIFZWDYVZVYFZUMRZCRWNZDZJJ
-ed-a--inhi--hai--a-e-i---ed----a--d-he--n
XZWGCHSMRNMDHNCMFQCHZJMXJZWIFJYUCFWDJNZDIR
```
下面需要确定明文o对应的密文，因为为是一个经常出现的字母，所以我们猜测相应的密文字母是D F J Y 中的一个，Y似乎最有可能，否则将得到长元音，即从CFM或CJM中得到aoi,因此假设Y->o。

剩下密文字母中三个最高频率的字母是D F J,我们猜测他们以某种次序解密成r s t， 三字母NMD两次出现说明很可能D->s,对应的明文三字母组为his，HNCMF可能是chair的加密，说明F->r,H->c，同时排除J->t，于是我们有了：
```
o-r-riend-ro--arise-a-inedhise--t---ass-it
YIFQFMZRWQFYVECFMDZPCVMRZWNMDZVEJBTXCDDUMJ
hs-r-riseasi-e-a-orationhadta-en--ace-hi-e
NDIFEFMDZCDMQZKCEYFCJMYRNCWJCSZREXCHZUNMXZ
he-asnt-oo-in-i-o-redso-e-ore-ineandhesett
NZUCDRJXYYSMRTMEYIFZWDYVZVYFZUMRZCRWNZDZJJ
-ed-ac-inhischair-acet-i-ted-to-ardsthes-n
XZWGCHSMRNMDHNCMFQCHZJMXJZWIFJYUCFWDJNZDIR
```
有了上面的提示，就很容易确定出明文，解密明文如下
```
Our friend from paris examined his empty glass with surprise, as ifevaporation has taken place while he wasn’t looking, I poured some more wine and hesettled back in his chair,face tilted up towards the sun.
```
###参考资料

【密码学原理与实践（第三版）】

【图解密码技术】
