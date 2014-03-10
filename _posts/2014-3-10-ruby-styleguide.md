---
layout: post
title: 【译】Ruby编码规范
categories: [编程,翻译,Ruby]
tags: [ruby,code style]
fullview: true
---
#【译】Ruby编码规范
---

本文档翻译与[Github ruby-style-guide](https://github.com/styleguide/ruby)    
本文档是GitHub内部使用的Ruby规范编码规范。原文档参考了[ruby-style-guid](https://github.com/bbatsov/ruby-style-guide)

##编码风格
---

*  使用空格符进行代码的缩进，而不是使用Tab

*  每行代码少于80个字符

*  不要再一行的后面留有空格

*  每个文件留一空行

*  操作符之后留一个空格，在逗号、冒号、大括号之后留空格 

		sum = 1 + 2   
   		a, b = 1, 2    
    	1 > 2 ? true : false; puts "Hi"    
    	[1, 2, 3].each { |e| puts e } 

*  不要在`(, [ 和 ], )`符号之前留空格

		some(arg).other
		[1, 2, 3].length

*  在` ! `空格前不要留空格

		!array.include?(element)
	
*  缩进 `case...when...`

		case
		when song.name == "Misty"
		  puts "Not again!"
		when song.duration > 120
		  puts "Too long!"
		when Time.now.hour > 21
		  puts "It's too late"
		else
		  song.play
		end
		
		kind = case year
		       when 1850..1889 then "Blues"
		       when 1890..1909 then "Ragtime"
		       when 1910..1929 then "New Orleans Jazz"
		       when 1930..1939 then "Swing"
		       when 1940..1950 then "Bebop"
		       else "Jazz"
		       end

*  在两个方法之间插入一空行，每个方法内通过空行来归约逻辑关系
	
		  def some_method
		    data = initialize(options)
		
		    data.manipulate!
		
		    data.result
		  end
		
		  def some_method
		    result
		  end
		  
##文档
---
使用[TomDoc](http://tomdoc.org/)规范注释文档格式，其格式大致如下：

		# Public: Duplicate some text an arbitrary number of times.
		#
		# text  - The String to be duplicated.
		# count - The Integer number of times to duplicate the text.
		#
		# Examples
		#
		#   multiplex("Tom", 4)
		#   # => "TomTomTomTom"
		#
		# Returns the duplicated String.
		def multiplex(text, count)
		  text * count
		end

##代码排版约定
---

*  定义方法时，无参数时省略圆括号，当有参数时需要加圆括号，如：

		   def some_method
		     # body omitted
		   end
		
		   def some_method_with_arguments(arg1, arg2)
		     # body omitted
		   end
	
*  不要使用for语句，除非你确定for语句优于迭代方案。请使用each,each方法中会使用for.注意：for语句并不会重新定义一个作用域，在其中定义的变量，在其外部依然可见。

		  arr = [1, 2, 3]

		  # bad
		  for elem in arr do
		    puts elem
		  end
		
		  # good
		  arr.each { |elem| puts elem }
		 
*  对于多行`if/uless`不要使用`then`

        # bad
		if some_condition then
		  # body omitted
		end
		
		# good
		if some_condition
		  # body omitted
		end
		
*  除非其他表达式太过于繁琐，尽量避免使用三元表达式（`?:`）。当时用（if/then/else/end）这种结构，并在单行出现时，请使用（`?:`）

		  # bad
		  result = if some_condition then something else something_else end
		
		  # good
		  result = some_condition ? something : something_else
		  
*  使用三元表达式时，请勿嵌套使用

		  # bad
		  some_condition ? (nested_condition ? nested_something : nested_something_else) : something_else
		
		  # good
		  if some_condition
		    nested_condition ? nested_something : nested_something_else
		  else
		    something_else
		  end
	
*  不提倡使用ruby中`and` 和 `or` 关键字，永远使用`&&`和`||`

*  在单行中有条件判断语句时，请使用`if/unless`

		  # bad
		  if some_condition
		    do_something
		  end
		
		  # good
		  do_something if some_condition
		  
*  **永远不要**出现`unless`和`else` 共同使用
		# bad
		unless success?
		  puts "failure"
		else
		  puts "success"
		end
		
		# good
		if success?
		  puts "success"
		else
		  puts "failure"
        end
     
*  在 `if/unless/while`的条件判断语句之后，判断表达式不要使用圆括号

		# bad
		if (x > 10)
		  # body omitted
		end
		
		# good
		if x > 10
		  # body omitted
		end
		
*  在单行块中使用`{...}`。**永远不要**在多行中使用`{...}`，而是使用`do...end`。在调用链中不要使用`do...end`
		
		names = ["Bozhidar", "Steve", "Sarah"]
		
		# good
		names.each { |name| puts name }
		
		# bad
		names.each do |name|
		  puts name
		end
		
		# good
		names.select { |name| name.start_with?("S") }.map { |name| name.upcase }
		
		# bad
		names.select do |name|
		  name.start_with?("S")
		end.map { |name| name.upcase }
		 
*  避免使用不必要的`return`

		# bad
		def some_method(some_arr)
		  return some_arr.size
		end
		
		# good
		def some_method(some_arr)
		  some_arr.size
		end
		
*  当为方法的参数赋默认值时，在`=`号前后添加空格

		# bad
		def some_method(arg1=:default, arg2=nil, arg3=[])
		  # do something...
		end
		
		# good
		def some_method(arg1 = :default, arg2 = nil, arg3 = [])
		  # do something...
		end
	
*  可以对使用返回值进行赋值

		# bad
		if (v = array.grep(/foo/)) ...
		
		# good
		if v = array.grep(/foo/) ...
		
		# also good - has correct precedence.
		if (v = next_value) == "hello" ...
			
*  在可以的时候尽量使用`||=`进行变量的初始化

		# set name to Bozhidar, only if it's nil or false
		name ||= "Bozhidar"
		
*  **永远不要**使用`||=`初始赋值布尔类型变量

		# bad - would set enabled to true even if it was false
		enabled ||= true
		
		# good
		enabled = true if enabled.nil?

*  避免使用Perl风格的变量命名（如$0-9,$,etc）.这种命名方式过于简单而无意义。请使用有意义的命名发如（$PROGRAM_NAME）。

*  **永远不要**再方法名之后添加空格
		
		# bad
		f (3 + 2) + 1
		
		# good
		f(3 + 2) + 1
		
*  如果方法名称之后是一个圆括号，则在之后都是用括号(如f((3 + 2) + 1))

*  使用`_`作为块中未使用的参数

		# bad
		result = hash.map { |k, v| v + 1 }
		
		# good
		result = hash.map { |_, v| v + 1 }
		
*  **永远不要**使用`===`判断对象的类型，`===`只是ruby作为实现如case语句时内部使用的特性方法，如果要判断一个变量的类型，请使用方法`is_a?`或者`kind_of?`

##命名
---

*  变量和方法名称使用 [snake_case](http://blog.lmorchard.com/2013/01/23/naming-conventions) 蛇底式命名规则

*  类和模块名称使用 [CamelCase](http://searchsoa.techtarget.com/definition/CamelCase)驼峰式命名规则

*  常量名称使用 [SCREAMING_SNAKE_CASE](#)大写的蛇底式命名规则

*  以`?`作为询问式方法的结尾(如 `Array#empty?`)

*  一个危险方法（直接改变`self`或者参数值的方法），不要在方法名之后加`!`

##类
---

*  避免使用类变量(@@),因为在类继承链中对类变量的修改会影响他的值

		class Parent
		  @@class_var = "parent"
		
		  def self.print_class_var
		    puts @@class_var
		  end
		end
		
		class Child < Parent
		  @@class_var = "child"
		end
		
		Parent.print_class_var # => will print "child"
		
*  使用`def self.method`定义类方法

		class TestClass
		  # bad
		  def TestClass.some_method
		    # body omitted
		  end
		
		  # good
		  def self.some_other_method
		    # body omitted
		  end
		  
*  当必要时使用 `class << self`,如 类属性或别名

		class TestClass
		  # bad
		  class << self
		    def first_method
		      # body omitted
		    end
		
		    def second_method_etc
		      # body omitted
		    end
		  end
		
		  # good
		  class << self
		    attr_accessor :per_page
		    alias_method :nwo, :find_by_name_with_owner
		  end
		
		  def self.first_method
		    # body omitted
		  end
		
		  def self.second_method_etc
		    # body omitted
		  end
		end

*  缩进`public`,`protected`,`private`方法，为他们之前的间隔留一空行

		class SomeClass
		  def public_method
		    # ...
		  end
		
		  private
		  def private_method
		    # ...
		  end
		end
		
*  除非必须使用`self`表示内部类或对象消息，其他情况不要使用self

		class SomeClass
		  attr_accessor :message
		
		  def greeting(name)
		    message = "Hi #{name}" # local variable in Ruby, not attribute writer
		    self.message = message
		  end
		end
		
##异常
---

*  不要使用异常作为程序流程控制

		class SomeClass
		  attr_accessor :message
		
		  def greeting(name)
		    message = "Hi #{name}" # local variable in Ruby, not attribute writer
		    self.message = message
		  end
		end
		
*  让异常抛出，而不是处理异常

		# bad
		begin
		  # an exception occurs here
		rescue
		  # exception handling
		end
		
		# still bad
		begin
		  # an exception occurs here
		rescue Exception
		  # exception handling
		end

##集合
---

*  使用 `%w` 创建字符串数组

		# bad
		STATES = ["draft", "open", "closed"]
		
		# good
		STATES = %w(draft open closed)

*  当数组中元素不可重复时，使用`Set`代替`Array`,`Set`实现了一个戊戌无重复元素的集合，是数组和哈希的结合体

*  使用`symbols`标识符代替字符串作为哈希的键

		# bad
		hash = { "one" => 1, "two" => 2, "three" => 3 }
		
		# good
		hash = { :one => 1, :two => 2, :three => 3 }
		
##字符串
---

*  使用字符串插入，而不用字符串连接

		# bad
		email_with_name = user.name + " <" + user.email + ">"
		
		# good
		email_with_name = "#{user.name} <#{user.email}>"

*  使用双引号`"`构造字符串对象。字符串插值或转义字符将会在双引号下有效，单引号`'`往往多在字符串字面量中使用

		# bad
		name = 'Bozhidar'
		
		# good
		name = "Bozhidar"
		
*  当需要多次构造长串的字符串时，使用`String#<<`而不是使用`String#+`。

		# good and also fast
		html = ""
		html << "<h1>Page title</h1>"
		
		paragraphs.each do |paragraph|
		  html << "<p>#{paragraph}</p>"
		end
		
##正则表达式
---

*  避免使用$1-9,而是使用有意义的命名

		# bad
		/(regexp)/ =~ string
		...
		process $1
		
		# good
		/(?<meaningful_var>regexp)/ =~ string
		...
		process meaningful_var
		
*  注意`^`/`$`表示了一行的开始和结尾，而不是表示整个字符串的结尾。当要匹配整个字符串时，使用`\A`和`\z`

		string = "some injection\nusername"
		string[/^username$/]   # matches
		string[/\Ausername\z/] # don't match

*  使用x来构造复杂的表达式

		regexp = %r{
		  start         # some text
		  \s            # white space char
		  (group)       # first group
		  (?:alt1|alt2) # some alternation
		  end
		}x
		
##百分号字面量
---

*  尽量使用%w

		STATES = %w(draft open closed)
		
*  使用在单行字符串中需要使用插值和内嵌的双引号时，使用`%()`

		# bad (no interpolation needed)
		%(<div class="text">Some text</div>)
		# should be "<div class=\"text\">Some text</div>"
		
		# bad (no double-quotes)
		%(This is #{quality} style)
		# should be "This is #{quality} style"
		
		# bad (multiple lines)
		%(<div>\n<span class="big">#{exclamation}</span>\n</div>)
		# should be a heredoc.
		
		# good (requires interpolation, has quotes, single line)
		%(<tr><td class="name">#{name}</td>)
		
*  当`\`匹配不止一个时 ，使用`%r`

		# bad
		%r(\s+)
		
		# still bad
		%r(^/(.*)$)
		# should be /^\/(.*)$/
		
		# good
		%r(^/blog/2011/(.*)$)
		
##  哈希
---

*  使用`=>`作为哈希的键值表示方式

		# bad
		user = {
		  login: "defunkt",
		  name: "Chris Wanstrath"
		}
		
		# bad
		user = {
		  login: "defunkt",
		  name: "Chris Wanstrath",
		  "followers-count" => 52390235
		}
		
		# good
		user = {
		  :login => "defunkt",
		  :name => "Chris Wanstrath",
		  "followers-count" => 52390235
		}



















