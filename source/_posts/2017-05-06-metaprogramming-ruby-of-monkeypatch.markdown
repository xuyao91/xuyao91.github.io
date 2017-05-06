---
layout: post
title: "ruby元编程之猴子补丁"
date: 2017-05-06 16:48:53 +0800
comments: true
categories: 
---

####打开类
先看一个例子，
```ruby
3.times do 
  class Dog
    puts "wang..."
  end
end
=> wang...
   wang...
   wang...
```
<!-- more -->
上面的代码并非定义三个同名的类，而是第一次定义一个类，其它两次是重新打开这个类。
```ruby
class Dog
  def say
    puts "wang wang..."
  end
end

class Dog
  def eat
    puts "bone"
  end
end

dog = Dog.new
dog.say    #=> wang wang...
day.eat     #= bone
```
 从上面的例子可以看出，当Ruby开始着手定义一个类，并定义say()方法，第二次再提及Dog类时，如果它已经存在，Ruby就不用再定义了，只需要重新打开这个已经存在的类，并定义eat()方法。  

&emsp;像这种你总是可以重新打开一个已经存在的类并对它进行动态修改的技术，可以称之为***打开类(open class)***

####猴子打补丁
&emsp;你写了一个substitute方法，功能是在一个数组中，把一个指定的元素替换成另一个元素的,代码如下 
```ruby
  def substitute(array, from, to)
    array.each_with_index do |v, i|
      array[i] = to if v == from
    end 
  end    

#=> 
a = ["zh", "usa", "ja", "ck"]
substitute(a, "ja", "kr")
["zh", "usa", "kr", "ck"] 
```
&emsp;果然是个好方法，用起来很方便，但是很快你发现其实这个方法可以再重构一下，把它定义成Array中的一个实例方法岂不是更好，于是你又改了一下，代码如下
```ruby
class Array
  def substitute(from, to)
    self.each_with_index do |v, i|
      self[i] = to if v == from
    end 
  end
end

#=> 
a = ["zh", "usa", "ja", "ck"]
a.substitute("ja", "kr")
["zh", "usa", "kr", "ck"] 
```
&emsp;上面的代码是打开Array类，并再类中添加substitute()方法，可以看到，你并没有修改原始的Array类库，而仅仅是重新打开了它，再往里面增加方法，
简直是太完美了，Array类中竟然缺少这么一个好用的方法，还好你把它加上了。  

&emsp;像这种在不改变源码的情况下，对功能进行动态追加、修改的技术叫做*__猴子补丁(Monkey patch)__*

####猴子补丁引起的问题
&emsp;如上所说，在Ruby中，可以很轻松地打开一个已经定义的类，并往类中塞方法（包括String,Array类）
 这时你突然发现substitute这个单词太长，影响使用，你已经想到了一个更好的方法名replace,于是你把代码又改了一次，如下： 
```ruby
  class Array
    def replace(from, to)
      self.each_with_index do |v, i|
        self[i] = to if v == from
      end 
    end
  end 
```
&emsp;简直太完美了，你轻轻松松就在Ruby自己的类库中添加了方法，这时你旁边的同事一头雾水的在找bug,说自己明明没改过这块代码，怎么现在代码却报错了，你从容地凑过去看，帮他找bug, 代码如下。
```ruby
a = [ "a", "b", "c", "d", "e" ]
a.replace([ "x", "y", "z" ]) 
#=>[ "x", "y", "z" ]  #TODO 你同事预期出来的结果
```
&emsp;天啦噜，你同事怎么这么快就用了你刚定义的replace()方法，还SB一样后面只带一个参数，我方法里面明明有两个参数。。。
你同事说，我明明记得Array类中有个replace()方法，是把当前数组内容替换，怎么会报错呢？
这时你好像明白了什么，Array类中本来就有replace()方法，我刚才无意中把这个replace()给覆写了，为了证明你的说法，你把刚才定义的方法去掉，并在Array中的方法找了一遍，看下是否有已经定义过的replace()方法
```ruby
[].methods.grep(/replace/)
#=> [:replace] 
```

&emsp;好了，你的问题找到了，是刚才你意想天开地在Array类中加了一个已经存在的方法(覆写了方法),你悄悄地把代码改回去，并让你同事pull一下代码，再试下，应该不会错，你同事pull了代码，运行一下果然没错，这时更一头雾水，你说没问题就行了，去吃饭。。。
经过上面的事情，你总结出了几条经验
-   打开一个已定义的类，并往里面添加方法是很危险的，因为你并不知道类中是否已经存在这个方法，
-   猴子补丁是全局性的，一旦你修改了Array中的replace()方法，则系统中的所有数组都会加载这个方法
-   猴子补丁是不可见的，如果重定义了Array#replace()方法，则很难发现这个方法被修改了，由于是全局性的，你很难发现问题所在，也很难找出在哪个地方定义了这个方法。

####如何避免猴子补丁引起的问题
我们先来看个例子，
```ruby
module M
  def my_method
    "M#my_method()"
  end
end

class C
  include M
end

class D < C; end

D.new.my_method()  #=>"M#my_method()"
#TODO 查看类D的所的父类
D.ancestors   #=> [D, C, M, Object, Kernel, BasicObject]
```
&emsp;从上面列子可以看到，当我们在一个类中包含(include)一个模块时，Ruby创建了一个封装该模块的匿名类，并把这个匿名类插入到祖先链中，其在链中的位置正好包含在它的类的上方，这些封装的类就叫做***包含类(include class)***,或者叫***代理类(proxy class)***。

接下来我们重新打开ruby的irb,试着查看一下Array的祖先链
```ruby
Array.ancestors
=> [Array, Enumerable, Object, Kernel, BasicObject] 
#TODO可以看到Ruby初始Array类中的祖先一共有5个
```
&emsp;然后我们再打开rails环境下的console(命令rails console),其环境加载了包括rails的各种gems源码和你自己的代码，同样的查看Array中的祖先链
```ruby
Array.ancestors
=> [Array, RQRCode::CoreExtensions::Array::Behavior, V8::Conversion::Array, JSON::Ext::Generator::GeneratorMethods::Array, Enumerable, Object, PP::ObjectMixin, ActiveSupport::Dependencies::Loadable, V8::Conversion::Object, JSON::Ext::Generator::GeneratorMethods::Object, Kernel, BasicObject] 
```
&emsp;天呐，Array类怎么多出来这么多父类，为了找出原因，你随便找了个父类ActiveSupport::Dependencies::Loadable研究下，可以看到源码[点这里](https://github.com/rails/rails/blob/f243e7f0d0b2cef350127fd518532fedb65f0bd0/activesupport/lib/active_support/dependencies.rb),其中有一个hook!方法
```ruby
def hook!
  Object.class_eval { include Loadable }
  Module.class_eval { include ModuleConstMissing }
  Exception.class_eval { include Blamable }
end
```
&emsp;它的作用就是将所需的各种Meta Programming挂到Object和Module下，而其实这里ActiveSupport::Dependencies::Loadable并不是一个真正的类，看源码我们就可以发现，它其它是一个模块，被include到了Object类里，也就是我们上面讲到的，ActiveSupport::Dependencies::Loadable其实是一个代理类，那为什么要这样做呢，因为使用模块(带了各种namespace的)可以让猴子补丁更明显一些，想对而言比较容易追踪到他们，因为这种方式至少可以在祖先类中看到这个模块。

所以使用***命名空间(namespace)***可以有效地解决类名冲突，从而避免猴子补丁引起的问题

####Rails中如何防止猴子补丁引起的问题
&emsp;rails在ActviteRecord中，有个instance_method_already_implemented?()方法，点击[查看源码](https://github.com/rails/rails/blob/master/activerecord/lib/active_record/attribute_methods.rb),它会在你定义一个动态方法前，首先会检查有没有同名方法存在，如果有的话，方法会抛出一个异常，否则返回false,说明可以定义方法，举个栗子
```ruby
class Patient < ActiveRecord::Base
  def save
    'already defined by Active Record'
  end
end

Patient.instance_method_already_implemented?(:save)
#=>  ActiveRecord::DangerousAttributeError: save is defined by Active Record. Check to make sure that you don't have an attribute or method with the same name.
Patient.instance_method_already_implemented?(:aaaaaa)
 #=> false 
```
####Rake中如何防止猴子补丁引起的问题
&emsp;在rake 中有一个名为Module#rake_extension()的方法，这里看源码[点击查看](https://github.com/ruby/rake/blob/93e55a4ef1dbaee42f0f355f86d837c4e2551fc1/lib/rake/ext/core.rb)
```ruby
  def rake_extension(method) # :nodoc:
    if method_defined?(method)
      $stderr.puts "WARNING: Possible conflict with Rake extension: " +
        "#{self}##{method} already exists"
    else
      yield
    end
  end
```
&emsp;Rake在它想打开类来添加方法时，会使用rake_extension()方法检查它是否已经存在，如果错误地重定义一个已经存在的方法，那么rake_extension()会抛出一个警告信息
```ruby
require 'rake'
class String
  rake_extension("xyz") do
    def xyz
      "xyz"
    end
  end
end
#=> :xyz

class String
  rake_extension("to_s") do
    def to_s
      "to_s"
    end
  end
end

#=> WARNING: Possible conflict with Rake extension: String#to_s already exists
```

&emsp;最后，在使用猴子补丁的时候，千万不要覆写、修改了原类中的方法，不然你会走上一条不归路的。
######参考资料
《Ruby元编程》  
https://en.wikipedia.org/wiki/Monkey_patch
https://web.archive.org/web/20120730014107/http://wiki.zope.org/zope2/MonkeyPatch
https://github.com/rails/rails
http://thekaiway.com/2013/07/04/code-loading-of-rails/
https://github.com/ruby/rake/blob/master/lib/rake/ext/core.rb
