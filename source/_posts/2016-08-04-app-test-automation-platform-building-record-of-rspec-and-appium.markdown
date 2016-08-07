---
layout: post
title: "App自动化测试平台搭建记录之rspec与appium的完美组合"
date: 2016-08-04 18:31:04 +0800
comments: true
categories: 
---
我们的预期目标是每天定时跑测试代码，并生成相应的测试报告，所以这里需要一个比较完善的测试框架来做这些事情，这里我选择的是rspec，
####rspec与appium组合
*   将app(ios和android)端每个页面封装成自己的类，页面中的每个控件实例成方法，使用到哪个页面只需将对应的类初始化一下。
*   写测试用例的人员只需写对应的测试用例，而不需要关注每个页面的控件封闭，而写控件封装的人，则只需写类的封装，也不必关注测试用例，这样极大的提高了开发效率
*   考虑到ios与android中大部分页面的控件以及逻辑都是一样的，所以针对ios和android测试，spec代码只需写一份，这样做的好处是将测试用例与逻辑代码分离，便于以后维护，同时测试用例的扩展性和重用性也大大得到的提高

####（页面）类的封装
测试开发流程具体如下，以app上的添加亲友为例
![img](http://7xjibn.com1.z0.glb.clouddn.com/Screenshot_2016-08-06-17-58-40_com.yydys.png)
可以看到添加亲友页面上有4个控件，那么可以将这个页面封装成一个类，这些控件就是对应的实例方法
```ruby
#android对应的类
module Simulator
  module Android
    module Pages
      module Patient
        class AddRelative < Simulator::Android::Pages::Patient::Base

          def my_relatives_link
            text("我的亲友")
          end

          def add_relative_button
            find(id: "com.yydys:id/add_folks")
          end

          def relative_select
            #text("是你的")
            find(id: "com.yydys:id/role_name")
          end

          def name_option_selected name="爸爸"
            text(name)
          end

          def mobile_num_text_field
            find(id: "com.yydys:id/mobile")
          end

          def glucose_sms_reminder
            find(id: "com.yydys:id/glucose_exception_togglebtn")
          end

          def medicine_sms_reminder
            find(id: "com.yydys:id/forget_medication_togglebtn")
          end

          def save_button
            find(id: "com.yydys:id/right_btn")
          end
        end
      end
    end
  end
end


#ios对应的类
module Simulator
  module Ios
    module Pages
      module Patient
        class AddRelative < Simulator::Ios::Pages::Patient::Base
          def my_relatives_link
            text("我的亲友")
          end

          def add_relative_button
            button_exact("添加亲友")
          end

          def relative_select
            text("是您的")
          end

          def name_option_selected name="爸爸"
            text(name)
          end

          def mobile_num_text_field
            textfield_exact("请输入手机号")
          end

          def glucose_sms_reminder
            find(xpath: "//UIAApplication[1]/UIAWindow[1]/UIATableView[1]/UIATableCell[3]/UIASwitch[1]")
          end

          def medicine_sms_reminder
            find(xpath: "//UIAApplication[1]/UIAWindow[1]/UIATableView[1]/UIATableCell[3]/UIASwitch[2]")
          end

          def save_button
            button_exact("保存")
            #find(xpath: "//UIAApplication[1]/UIAWindow[1]/UIATableView[1]/UIATableCell[4]/UIASwitch[1]")
          end

          def return_button
            button_exact("返回")
          end
        end
      end
    end
  end
end
```
可以看到对应页面上的每个控件，ios和android都会有自己的封闭方法，这样做提高了代码的重用性，与测试代码分离，便于以后维护
####测试用例
上面说道一般ios和android页面上的控件是一样的，所以正常情况下，我们只需写一份测试代码就可以，但是ios和android的测试代码还是要分开跑的，所以这里我引用了rpsec的[share_example](http://rspec.info/documentation/3.5/rspec-core/#Shared_Examples_and_Contexts)来做
```ruby
#android的测试用例
require_relative '../../../shared/patient/v1/add_relative_spec'
RSpec.describe "AddRelative	" do
  include_examples "share_add_relative"	, "android"
end	

#ios的测试用例
require_relative '../../../shared/patient/v1/add_relative_spec'
RSpec.describe "AddRelative" do
  include_examples "share_add_relative"	, "ios"
end	


#shared部分
require_relative '../../../spec_helper'
RSpec.shared_examples "share_add_relative" do |device|
  before(:all) do
    login device
  end

	#Appium.promote_appium_methods Object
	it "add relative" do
		begin
  			sleep(8)
			wait {add_relative_page.user_center_bar.click}
			wait {add_relative_page.my_relatives_link.click}
			wait {add_relative_page.add_relative_button.click}
			if @device.appium_device ==  :android
				wait {add_relative_page.relative_select.click}
				wait {add_relative_page.name_option_selected("爸爸").click}
			else
				wait {add_relative_page.relative_select.click}
				wait {add_relative_page.return_button.click}
			end

			wait {add_relative_page.mobile_num_text_field.type("13524836959")}
			if @device.appium_device ==  :android
				#screen = wait {add_relative_page.boot_screen}
				#size = screen.window_size
				#start_x, end_x, start_y, end_y = size.width - 5, size.width/100, size.height/2, size.height/2
				#wait {exists{swipe(start_x:start_x,start_y:start_y, end_x:end_x,end_y:end_y, duration:1000)}}
			else
				wait {add_relative_page.glucose_sms_reminder.click}
				#wait {add_relative_page.medicine_sms_reminder.click}
			end
			#wait {add_relative_page.glucose_sms_reminder.}
			#wait {add_relative_page.mobile_num_text_field.type("11000009123")}
			wait {add_relative_page.save_button.click}
			expect(exists{add_relative_page.add_relative_button}).to be_truthy
	rescue => e
			puts e.message
			@driver.quit if @driver
			raise e
	end

	end

	after do
	  @driver.quit if @driver
	end

end

```
可以看到写测试用例的人员，不必关注底层控件的封装，只需按正常逻辑写测试用例就可
####测试报告
![img](http://7xjibn.com1.z0.glb.clouddn.com/C8EA5B20-316A-45CB-A950-5F393B4AEC85.png)

####未完成的工作
*   配置jenkins，使其每天定时打包最新的代码
*   动态数据的载入
*   不同app版本的测试
*   混合，h5页面的测试