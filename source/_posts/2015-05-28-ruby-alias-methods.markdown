---
layout: post
title: "ruby_alias_methods"
date: 2015-05-28 22:03:17 +0800
comments: true
categories: 
---
##### 首先看一下alias的用法，上一个实例是明了

    class Dog
      def say
  	    puts  "say hello"
      end
     alias :wang :say
    end
    
    dog = Dog.new
    dog.wang
   
    => say hello
    
在类中可以将一个实例方法重新命名,注意的是新方法在前面，用空格格开

##### alias_method的用法，

和alias一样，只是他是module的一个私有方法，而且它的方法名可以是字符中，而alias不行，看代码

    module Dog
      def say
  	    puts  "say hello"
    end

    alias_method "wang", "say"
    #alias_method :wang, :say
    end

    class Animal
	  include Dog
    end
    
    a = Anima.new
    a.wang
    
    =>say hello
    

##### alias_method_chain 是ActiveSupport的一个公有实例方法，用法和alias_method是一样的，这里不举例，最后附上alias_method_chain的源码
    
    def alias_method_chain(target, feature)    
      # Strip out punctuation on predicates or bang methods since    
      # e.g. target?_without_feature is not a valid method name.    
      aliased_target, punctuation = target.to_s.sub(/([?!=])$/, ''), $1    
      yield(aliased_target, punctuation) if block_given?         
      with_method, without_method = "#{aliased_target}_with_#{feature}#{punctuation}", "#{aliased_target}_without_#{feature}#{punctuation}"    
      alias_method without_method, target    
      alias_method target, with_method          
      case    
        when public_method_defined?(without_method)    
          public target    
        when protected_method_defined?(without_method)    
          protected target    
        when private_method_defined?(without_method)    
        private target    
      end    
    end    