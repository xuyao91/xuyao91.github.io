---
layout: post
title: "how to understand rails virtual attributes"
date: 2015-06-04 16:37:50 +0800
comments: true
categories: 
---

#### 在网上看到关于rails  virtual attributes 的例子,得觉挺好玩的,自己动手试一下,好叼

有时候在一张表里面有两个字段,但是实际表单里其实只有一个文本框输入,比如老外的名字分first_name last_name,
在表里面是分开保存的,但其实是在一个文本框中输入的,这里使用virtal attriubtes最恰当不过了,看表结构

<!-- more -->
表里有first_name和full_name两个属性
```ruby
    class CreateTeachers < ActiveRecord::Migration
      def change
        create_table :teachers do |t|
          t.string :first_name
          t.string :last_name
          t.integer :age
          t.integer :gender
          t.string :mobile
    
          t.timestamps null: false
        end
      end
    end

```


 但是有时候在页面上不需要用两个文本框,只要一个full_name就ok
```ruby
    <%= form_for(@teacher) do |f| %>

      #只在有一个full_name 的文本框就可以
      <div class="field">
        <%= f.label :full_name %><br>
        <%= f.text_field :full_name %>
      </div>
      <div class="field">
        <%= f.label :age %><br>
        <%= f.number_field :age %>
      </div>
      <div class="field">
        <%= f.label :gender %><br>
        <%= f.number_field :gender %>
      </div>
      <div class="field">
        <%= f.label :mobile %><br>
        <%= f.text_field :mobile %>
      </div>
      <div class="actions">
        <%= f.submit %>
      </div>
    <% end %>
```          

model层可以这样写

```ruby
    class Teacher < ActiveRecord::Base
    
      def full_name
          [first_name, last_name].join(' ')
      end	
    
      def full_name= (full_name)
      	split = full_name.split(' ', 2)
      	self.first_name = split[0]
      	self.last_name = split[1]
      end	
    end
```
这里full_name就是一个虚拟属性(virtual attributes)
上面两个方法其实就是getter和setter(java中有这种概念)

