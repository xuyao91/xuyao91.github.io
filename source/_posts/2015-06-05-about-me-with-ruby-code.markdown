---
layout: post
title: "about me"
date: 2015-06-05 10:49:54 +0800
comments: true
categories: 
---
```ruby
    class Person
      attr_accessor :name, :birthday, :gender
      
      def initialize(name,birthday,gender)
        @name = name
        @birthday = birthday
        @gender = gender
      end
      
      def company= name
        self.company = name
      end
      
      def position= position
        self.position = position
      end
      
      def hobby *arg
        "I like #{arg.split(',')}"
      end      
    end
    
    me = Person.new("徐耀","1991-02-03",1)
    me.nickname = %W(老徐 徐逗逗)
    me.company= "上海微客来软件技术有限公司"
    #TODO update at 2015-7-15
    me.company= "苏州医云健康有限公司"
    me.position = "Ruby Engineer"
    me.hobby "coding","reading","music","traveling","grils"
    me.weibo = "http://weibo.com/1676361452"
```