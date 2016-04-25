---
layout: post
title: "ruby中include和extend的比较"
date: 2015-06-29 12:21:35 +0800
comments: true
categories: 
---
本文翻译,改编自[Include vs Extend in Ruby](http://www.railstips.org/blog/archives/2009/05/15/include-vs-extend-in-ruby/)  

在类中引用一个模块,有两种方式,一种是include,另一种是extend,两种的大致区别就是:  
**_include是给class增加一个实例方法,而extend是增加一个类方法_  **  
我们看一下小例子.
<!-- more -->
```ruby
module Foo
  def foo
    puts 'heyyyyoooo!'
  end
end

class Bar
  include Foo
end

Bar.new.foo # heyyyyoooo!
Bar.foo # NoMethodError: undefined method ‘foo’ for Bar:Class

class Baz
  extend Foo
end

Baz.foo # heyyyyoooo!
Baz.new.foo # NoMethodError: undefined method ‘foo’ for #<Baz:0x1e708>

```
如你所见,include生成了一个类的实例foo方法,而extend生成了一个类本身的类方法.



#####Include 例子  
如果你想了解更多module之间使用include共享方法,你可以阅读我另一则文章在[how I added simple permissions](http://railstips.org/2009/4/20/how-to-add-simple-permissions-into-your-simple-app-also-thoughtbot-rules),那篇文章中permissions模块被好几个models引入,所以可以共享方法,这就是我要说的,所以你想了解更多可以查看那篇文章  


#####一个普遍的共识
虽然include是增加一个实例方法,但是有一个普遍的共识,你以后会明白在ruby中使用include即可以增加类方法也可以增加实例方法,原因就是include有一个self.included的钩子方法,你可以用它来修改类中对于模块的引入,据我所知,extend没有钩子方法(这是作者错误的想法,见下面说明),不过这是有争议的,但是因为经常使用所以我想我会提到它,让我们看个小例子
```ruby
module Foo
  def self.included(base)
    base.extend(ClassMethods)
  end
  
  module ClassMethods
    def bar
      puts 'class method'
    end
  end
  
  def foo
    puts 'instance method'
  end
end

class Baz
  include Foo
end

Baz.bar # class method
Baz.new.foo # instance method
Baz.foo # NoMethodError: undefined method ‘foo’ for Baz:Class
Baz.new.bar # NoMethodError: undefined method ‘bar’ for #<Baz:0x1e3d4>
```
但是rails现在引入了[ActiveSupport::Concern](http://api.rubyonrails.org/classes/ActiveSupport/Concern.html),上面的方法就可以写成这样  

```ruby
module Foo
  extend ActiveSupport::Concern
  
  module ClassMethods
    def bar
      puts 'class method'
    end
  end
  
  def foo
    puts 'instance method'
  end
end

class Baz
  include Foo
end

Baz.bar # class method
Baz.new.foo # instance method
Baz.foo # NoMethodError: undefined method ‘foo’ for Baz:Class
Baz.new.bar # NoMethodError: undefined method ‘bar’ for #<Baz:0x1e3d4>
```


#####说明
上面原作者说extend没有hook是错误的,extend其实有一个叫self.extended的方法,作用和include中的self.included是差不多的,可看下面的例子  
```ruby
module Foo
  def self.extended(klass)
    puts "extended by class #{klass}"
  end
end

class Baz
  extend Foo
end

=> extended by class Baz
```
