---
layout: post
title: "密码学(一)之移位密码(恺撒密码)"
date: 2018-07-13 14:03:33 +0800
comments: true
categories: 
---

###前言
   密码学的基本目的是使得两个在不安全的信道上进行安全的通信，在计算机网络中，现假设有两个人Alice和Bob，Alice想发送消息给Bob，告诉他明天凌晨2点毒贩将在4号码头进行交易，请Bob配合缉毒。这里Alice有一个麻烦就是他怕在信息传输中可能会被敌人Oscar监听，甚至篡改消息内容，导致整个计划失败，Bob也对Alice发过来的消息有疑虑，不知道消息是否为Alice本人所发，消息内容是否真实等等。  
   <!-- more --> 

 这里密码学的重要性就体现出来了，假设Alice事先和Bob商量好一个密钥（key）,Alice通过密钥将明文（plaintext）加密成密文，在网络中传输，Bob收到Alice传过来的密文（ciphertext），用事先商量好的密钥进行解密，得到明文，而敌人即使监听到Alice发送的消息，也是加密过的密文，由于不知道密钥，所以无法知道真实的明文，整个过程如下。
![image.png](https://upload-images.jianshu.io/upload_images/1796624-95050e2fc8412a32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
###移位密码（Shift Cipher）、恺撒密码(Caesar Cipher)
移位密码是古典密码学中最早，最简单的一种加、解密码方法，最早可追溯至古罗马时代，尤利乌斯:恺撒曾经使用过此密码。 
  
移位密码是通过将明文中所使用的字母按照一定的字数进行“平移”来加密，为了简化内容，在这里我们只使用英文字母作为示例，我们用小写字母(a,b,c,d…)来表示明文，用大写字母（A,B,C,D…）来表示密文。
  
最早期时，一般将字母平移3位，也就是a->D，b->E，c->F,
这种最早时平移3位是叫**恺撒密码(Caesar Cipher)**，后来经过推广，平移位数也不一定是3位，可以是其它任何整数位，这种又叫**移位密码（Shift Cipher）**，可以知道，恺撒密码是移位密码的一个特例（key=3时），下面是恺撒密码平移的工作方式  
![image.png](https://upload-images.jianshu.io/upload_images/1796624-e1fa042fd62a2c56.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
###移位密码的加密
 使用移位密码可以用来加密普通的英文句子，但是我们要建立英文字母和模26剩余之间一一对应关系，如A->0，B->1,…Z->25。其列表如下：  
![image.png](https://upload-images.jianshu.io/upload_images/1796624-0dbef7eedef6d7dc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)  
现假设我们有明文P=china，密钥K=3, 现要将其加密，根据上表，将字符P中的每个字母平移3位，得到如下情况:  
```ruby
c->F
h->K
i->L
n->Q
a->D
```
至此，明文china就被转换成了密文FKLQD，  
具体用程序算法可归纳如下:
```ruby
a，首先将明文中的字母按照上表对应成相应的数字：
   2 7 8 13 0
b，再将上面的数字与密钥K=3相加：
   5 10 11 16 3
c，再对各个数字取模26运算，可得：
   5 10 11 16 3
d，最后将各数字转化为字母即可得密文
   F  K  L  Q  D
```
具体代码可参考这里
[https://github.com/xuyao91/cryptography/blob/master/caesar/caesar.rb#L10](https://github.com/xuyao91/cryptography/blob/master/caesar/caesar.rb#L10)
###移位密码的解密

移位密码的解密也非常简单，只要使用加密时用的密钥进行反向平移操作，刚来的例子只要将密文反向平移3位就行，可得到如下：
```ruby
F->c
K->h
L->i
Q->n
D->a
```
具体程序的算法其实就是加密的反向操作，如下：
```ruby
a， 将密文转化成对应的字母
    5 10 11 16 3
b， 将各数字减去密钥K=3，得到如下数字
    2 7 8 13 0
c， 再对各个数字取模26运算，得：
    2 7 8 13 0
d， 最后将数字转化成字母得
    c h i  n a
```
具体代码可参考这里：
[https://github.com/xuyao91/cryptography/blob/master/caesar/caesar.rb#L18](https://github.com/xuyao91/cryptography/blob/master/caesar/caesar.rb#L18)

###移位密码的暴力破解

通过上面的例子可以知道，我们只要拿到的密钥K，就可以密文解密，那么有没有不用密钥就可以解密的呢，在移位密码中，密钥就是字母平移的位数，因为因为字母表里只有26个字母（0-25），所以加密的密钥一共就是0-25之间的26个数字，我们可以把26个数字全部当作密钥试一次，解密出来其中有一个明文肯定是对的。  

像这种将所有可能的密钥全部尝试一遍的，我们叫**暴力破解(brute-force attack)**，这种方法本质是在所有的密钥中找出正确的那一个，因此又称为**穷举搜索（exhaustive search）**。

可以将上面的例子通过暴力破解试一下，密钥从0开始，一直试到25，得到如下效果：
```ruby
FKLQD -> 第0次破解 -> fklqd
FKLQD -> 第1次破解 -> ejkpc
FKLQD -> 第2次破解 -> dijob
FKLQD -> 第3次破解 -> china
FKLQD -> 第4次破解 -> bghmz
FKLQD -> 第5次破解 -> afgly
FKLQD -> 第6次破解 -> zefkx
FKLQD -> 第7次破解 -> ydejw
FKLQD -> 第8次破解 -> xcdiv
FKLQD -> 第9次破解 -> wbchu
FKLQD -> 第10次破解 -> vabgt
FKLQD -> 第11次破解 -> uzafs
FKLQD -> 第12次破解 -> tyzer
FKLQD -> 第13次破解 -> sxydq
FKLQD -> 第14次破解 -> rwxcp
FKLQD -> 第15次破解 -> qvwbo
FKLQD -> 第16次破解 -> puvan
FKLQD -> 第17次破解 -> otuzm
FKLQD -> 第18次破解 -> nstyl
FKLQD -> 第19次破解 -> mrsxk
FKLQD -> 第20次破解 -> lqrwj
FKLQD -> 第21次破解 -> kpqvi
FKLQD -> 第22次破解 -> jopuh
FKLQD -> 第23次破解 -> inotg
FKLQD -> 第24次破解 -> hmnsf
FKLQD -> 第25次破解 -> glmre
```
可以知道，其实在试到第3次的时候，明文就被试出来了
代码可以参考这里
[https://github.com/xuyao91/cryptography/blob/master/caesar/caesar.rb#L26](https://github.com/xuyao91/cryptography/blob/master/caesar/caesar.rb#L26)

###脚本示例

  我们用脚本试一下上面所说的各种方法，脚本代码在这里

[https://github.com/xuyao91/cryptography/blob/master/caesar/caesar.rb](https://github.com/xuyao91/cryptography/blob/master/caesar/caesar.rb)

```ruby
#明文是 I am peter, I love china， 密钥是6
message = "iampeterxuilovechina"
caesar = Caesar.new 6

#调用encoder方法加密出密文
cipher = caesar.encoder(message) => OGSVKZKXDAORUBKINOTG

#调用decoder方法解密出明文
caesar.decoder(cipher) => iampeterxuilovechina

#暴力破解
OGSVKZKXDAORUBKINOTG -> 第0次破解 -> ogsvkzkxdaorubkinotg
OGSVKZKXDAORUBKINOTG -> 第1次破解 -> nfrujyjwcznqtajhmnsf
OGSVKZKXDAORUBKINOTG -> 第2次破解 -> meqtixivbympsziglmre
OGSVKZKXDAORUBKINOTG -> 第3次破解 -> ldpshwhuaxloryhfklqd
OGSVKZKXDAORUBKINOTG -> 第4次破解 -> kcorgvgtzwknqxgejkpc
OGSVKZKXDAORUBKINOTG -> 第5次破解 -> jbnqfufsyvjmpwfdijob
OGSVKZKXDAORUBKINOTG -> 第6次破解 -> iampeterxuilovechina
OGSVKZKXDAORUBKINOTG -> 第7次破解 -> hzlodsdqwthknudbghmz
OGSVKZKXDAORUBKINOTG -> 第8次破解 -> gykncrcpvsgjmtcafgly
OGSVKZKXDAORUBKINOTG -> 第9次破解 -> fxjmbqbourfilsbzefkx
OGSVKZKXDAORUBKINOTG -> 第10次破解 -> ewilapantqehkraydejw
OGSVKZKXDAORUBKINOTG -> 第11次破解 -> dvhkzozmspdgjqzxcdiv
OGSVKZKXDAORUBKINOTG -> 第12次破解 -> cugjynylrocfipywbchu
OGSVKZKXDAORUBKINOTG -> 第13次破解 -> btfixmxkqnbehoxvabgt
OGSVKZKXDAORUBKINOTG -> 第14次破解 -> asehwlwjpmadgnwuzafs
OGSVKZKXDAORUBKINOTG -> 第15次破解 -> zrdgvkviolzcfmvtyzer
OGSVKZKXDAORUBKINOTG -> 第16次破解 -> yqcfujuhnkybelusxydq
OGSVKZKXDAORUBKINOTG -> 第17次破解 -> xpbetitgmjxadktrwxcp
OGSVKZKXDAORUBKINOTG -> 第18次破解 -> woadshsfliwzcjsqvwbo
OGSVKZKXDAORUBKINOTG -> 第19次破解 -> vnzcrgrekhvybirpuvan
OGSVKZKXDAORUBKINOTG -> 第20次破解 -> umybqfqdjguxahqotuzm
OGSVKZKXDAORUBKINOTG -> 第21次破解 -> tlxapepciftwzgpnstyl
OGSVKZKXDAORUBKINOTG -> 第22次破解 -> skwzodobhesvyfomrsxk
OGSVKZKXDAORUBKINOTG -> 第23次破解 -> rjvyncnagdruxenlqrwj
OGSVKZKXDAORUBKINOTG -> 第24次破解 -> qiuxmbmzfcqtwdmkpqvi
OGSVKZKXDAORUBKINOTG -> 第25次破解 -> phtwlalyebpsvcljopuh
```

可以知道，移位密码其实是很弱的，我们可以很容易将他破解出来，但是在古代那会还是挺有用的。

###参考资料

【密码学原理与实践（第三版）】

【图解密码技术】



