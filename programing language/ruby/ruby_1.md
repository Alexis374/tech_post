####basic ruby
----
+ install gems：
	+ install remote: `gem install <gem-name>`
	+ install local: download the .gem file from [rubygems.org](rubygems.org), then run `gem install -l <gem path>`
	+ add .gem/bim to the path: add `export PATH="~/.gem/ruby/2.2.0/bin:$PATH"` to `.bashrc`
+ use pry instead of irb ,in a rb file,insert `binding.pry`,then `ruby thatfile`,the progrom will pause on that statement for debugging.
+ ![ruby_doc](../../img/pl/ruby_doc.jpg)
+ formatting string: `a=4.0; "My favorite number is #{a}!"`
+ symbol: a colon (:) before a word. reference something like a string but don't ever intend to print it to the screen or change it
+ nil: ` "Hello, World".nil?` returns false
+ `'4'.to_i`,`'4'.to_f`,`4.to_s` `'hi there 4'.to_i` => 0 `'4 hi there'.to_f` => 4.0
+ array: `a=[1,2];a<<3` => a=[1,2,3]
+ hash: {:dog => 'barks', :cat => 'meows', :pig => 'oinks'}
+ An expression in Ruby always returns something, even if that's an error message or nil
+ variables point to values in memory, and are not deeply linked to each other. `a=4;b=a;a=7;b==4`=>true
+ get user input: `gets`,return a string ends with`\n`,`gets.chomp`remove the last `\n`
+ block({} or do/end) define a variable scope; Inner scope can access variables initialized in an outer scope, but not vice versa
+ not all {} or do/end create a block,the key thing is that whether they follow a method invocation
```ruby
arr  = [1,2,3]
for i in arr do # not a block because it does not follow a method invocation
	a=5
end
puts a # can access a,but if a is initialized in a block, a can't be accessed.

3.times do |i|  # do/end follows the times methods invocation
	b=4
end
puts b # can't access b
+ methods create their own scope.
+ types of variable: constant,global($),class(@@),instance(@),local
+ method: `def(params='default')`. some method will change the argument,such as `array.pop`
+ methods ALWAYS return the evaluated result of the last line of the expression unless an explicit return comes before it
+ `p` return the value of arguments `puts` `print` return nil,`puts`,`p` results in a new line, while `print` doesn't.
+ `if elsif else end`,`if x == 3 then puts "x is 3" end`,`puts "x is 3" if x == 3`,`puts "x is NOT 3" unless x == 3`
+ ```ruby

a = 5

case a
when 5
  puts "a is 5"
when 6
  puts "a is 6"
else
  puts "a is neither 5, nor 6"
end

a = 5

answer = case a
  when 5
    "a is 5"
  when 6
    "a is 6"
  else
    "a is neither 5, nor 6"
  end

puts answer

a = 5

answer = case
  when a == 5
    "a is 5"
  when a == 6
    "a is 6"
  else
    "a is neither 5, nor 6"
  end

case num
  when 0..50
    puts "#{num} is between 0 and 50"
  when 51..100
    puts "#{num} is between 51 and 100"
  else
    if num < 0
      puts "You can't enter a negative num!"
    else
      puts "#{num} is above 100"
    end
  end
```