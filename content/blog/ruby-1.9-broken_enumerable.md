+++
date = "2012-09-14"
Categories = ["Development","GoLang"]
Description = ""
Tags = ["Development","Ruby", "Functional"]
title = "Ruby 1.9 Broken Enumerable"
+++

I was working with the `each_with_index` Enumerator on Ruby 1.9.3 and I stumbled on a strange behavior.
Let's take a look at 2 examples:

```ruby
>> %w{Bob Roger Joe}.each_with_index.select{|*args| puts args.inspect}
[["Bob", 0]]
[["Roger", 1]]
[["Joe", 2]]
```

```ruby
>> %w{Bob Roger Joe}.each_with_index.map{|*args| puts args.inspect}
["Bob", 0]
["Roger", 1]
["Joe", 2]
```

The _select_ block passes one argument which is an array containing the object and the index. However, _map_ passes two arguments: the object being interated on and the index. 

If we look at the [Enumerable#map documentation](http://ruby-doc.org/core-1.9.3/Enumerable.html#method-i-map), we read that the block needs one argument: obj. So we can intuitively think that in the case of `each_with_index`, the obj would be an array containing the object iterated on and the index. But it's clearly not the case.

Since _map_ and _select_ methods are implemented in the Enumerable module using the _each_ method, let's get a look a the _Eumerator#each_ 

```ruby
>> %w{Bob Roger Joe}.each_with_index.each { |*args| puts args.inspect }
["Bob", 0]
["Roger", 1]
["Joe", 2]
```

As we can see, the _each_ method that the Enumerable module uses can have any number of block arguments. I imagine that under the hood _map_ uses the splat operator.

Let's create our own Enumerable class:
```ruby
class MyEnumerable
  include Enumerable
  
  def each
    3.times { yield 1, 2, 3, 4 }
  end
end

enum = MyEnumerable.new
enum.map { |*args| puts args.inspect }
[1, 2, 3, 4]
[1, 2, 3, 4]
[1, 2, 3, 4]
[1, 2, 3, 4]
```

The big question is: Which Enumerable methods behave like the _map_ method? Lets calculate the number of arguments of some Enumerable methods that are using a _each_ method that uses 1 and 2 block arguments.

```ruby
enum_methods = %w{inject map select count find group_by flat_map drop_while each_entry detect find_index max_by min_by minmax none? one? partition reject reverse_each sort_by}

nb_args = enum_methods.inject({}) do |memo, method|
  memo[method] ||= {}
  proc1 = Proc.new { |*args| memo[method][:each_with_1_arg] = args.length }
  [1,2].send(method, &proc1) rescue nil
  proc2 = Proc.new{ |*args| memo[method][:each_with_2_arg] = args.length }
  [1,2].each_with_index.send(method, &proc2) rescue nil
  memo
end

puts nb_args.to_yaml
---
inject:
  :each_with_1_arg: 2
  :each_with_2_arg: 2
map:
  :each_with_1_arg: 1
  :each_with_2_arg: 2
select:
  :each_with_1_arg: 1
  :each_with_2_arg: 1
count:
  :each_with_1_arg: 1
  :each_with_2_arg: 2
find:
  :each_with_1_arg: 1
  :each_with_2_arg: 1
group_by:
  :each_with_1_arg: 1
  :each_with_2_arg: 1
flat_map:
  :each_with_1_arg: 1
  :each_with_2_arg: 2
drop_while:
  :each_with_1_arg: 1
  :each_with_2_arg: 1
each_entry:
  :each_with_1_arg: 1
  :each_with_2_arg: 1
detect:
  :each_with_1_arg: 1
  :each_with_2_arg: 1
find_index:
  :each_with_1_arg: 1
  :each_with_2_arg: 2
max_by:
  :each_with_1_arg: 1
  :each_with_2_arg: 1
min_by:
  :each_with_1_arg: 1
  :each_with_2_arg: 1
minmax:
  :each_with_1_arg: 2
  :each_with_2_arg: 2
none?:
  :each_with_1_arg: 1
  :each_with_2_arg: 2
one?:
  :each_with_1_arg: 1
  :each_with_2_arg: 2
partition:
  :each_with_1_arg: 1
  :each_with_2_arg: 1
reject:
  :each_with_1_arg: 1
  :each_with_2_arg: 1
reverse_each:
  :each_with_1_arg: 1
  :each_with_2_arg: 1
sort_by:
  :each_with_1_arg: 1
  :each_with_2_arg: 1
```

The methods that behave differently with a `each` block with 2 arguments than with 1 argument are: `map`, `count`, `flat_map`, `find_index`, `none?`, `one?`. Which is 7 methods out of 20. How can we remember those?

If you run the last script on Ruby 1.8.7 you will see that the map `method` and its friends are not splatting the arguments.

I am trying to find the logic in those changes in Enumerable but I don't find any. So do you think like me that Enumerable is broken? 
