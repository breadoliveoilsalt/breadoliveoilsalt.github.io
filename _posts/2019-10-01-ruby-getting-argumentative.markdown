---
layout: post
title: "Ruby: We're Getting Argumenative (Aka: Notes on Different Types of Ruby Arguments)"
date:   2019-10-01 12:00:00 -0400
categories: coding
---

I've been returning to Ruby recently, and it seemed like an appropriate time to demystify for myself some of the symbols and patterns that appear when arguments are being passed to method calls.  For example, if you're using Ruby on Rails, you might see a `#render` method call like this:

```ruby
render :new
```

Or you might see a more complicated `#before_action` method call like this:

```ruby
before_action :check_if_signed_in, only: [:edit, :update, :destroy]
```

Even common macros like `attr_accessor` seem to follow a similar pattern:

```ruby
attr_accessor :open, :close, :edit
```

So what's going on here?  Where do the arguments begin and end?  Why are there colons?  Why are the colons in different places (before an argument vs. afterward)?  These and other questions have been on my mind. 

For better or worse, I've come to realize that seeing **method calls** like the ones above may allow me to make certain assumptions about the arguments that the methods will take.  However, to really understand what's going on under the hood, one really needs to inspect the method signature (that is, the method name and list of arguments that follow).

That being said, here are some notes and reminders to myself about why arguments like the ones above are possible. Hope they help in case you have similar questions.

----------
<p/>

* **Ruby arguments do not need to be surrounded by parentheses.  So if you see a method call followed by a space and a list, start assuming the list is a bunch of arguments.**

In other words, this...

```ruby
before_action :check_if_signed_in, only: [:edit, :update, :destroy]
```
...is the same as this:

```ruby
before_action(:check_if_signed_in, only: [:edit, :update, :destroy])
```
<p/>
----------
<p/>

* **Ruby method signatures allow different *types* of aguments. These types include:** 
    * **required arguments** 
    * **optional arguments (that is, arguments with default values)** 
    * **variable arguments (arguments that can vary in amount)** 
    * **keyword arguments (a hash-like way of passing arguments to a method call)**

A method signature can mix and match these different types. But if you do this, there should be a particular order for using the different types of agruments in a method signature.

[This source](https://www.rubyguides.com/2018/06/rubys-method-arguments/) (which is very good) recommends adhering to this order when using different types of arguments in a method signature:

`required -> optional -> variable -> keyword`

We'll tackle each of these types of arguments in the dicussion below.

<p/>
----------
<p/>

* **Required Arguments: In the method signature, you can mandate that a *required argument* be passed for a method call in two ways.  Either use a plain old variable without a default, or use a "keyword argument" without a default value.** 

A plain old variable without a default looks like this:

```ruby
def method_requires_argument(plain_old_argument)
```

If you call `#method_requires_argument` without passing something for `plain_old_argument`, the ruby interpreter will throw an `ArgumentError`.

A keyword argument without a default is a varialbe followed by a colon, like this:
```ruby
def method_requires_keyword_argument(needed:)
```

Keyword arguments are a way to pass hash-like arguments to a method, without wrapping the arguments in curly brackets.  Keyword arguments provide two main advantages: (1) if your method signature has just keyword arguments, then arguments can be passed in any order and (2) it's easier to understand what is being passed in when a method is called.  

For example, a method signature can require two keyword arguments, like so:

```ruby
def method_requires_keyword_arguments(needed:, also_needed:)
```

And the method can be called by passing in hash-like arguments in any order, like so:

```ruby
method_requires_keyword_arguments(also_needed: "A good reason", needed: "To Do something")
```

<p/>
----------
<p/>
* **Optional Arguments: An optional argument means that the argument does not have to be passed to the method when the method is called.  The method can still *depend on* an argument, but the method signature provides a default value as a stand-in if the argument is not provided when the method is called.**

If your method signature relies on a plain old variable argument, the method signature can provide a default value (making the argument optional) using an `=` operator, like so:

```ruby
def method_with_optional_argument(your_argument = "The default value")
```

If your method signature relies on keyword arguments, the method signature can provide a default value (making the argument optional) by providing a default value after the keyword's colon, like so:

```ruby
def method_with_optional_keyword_argument(your_argument: "The default value")
```

<p/>
----------
<p/>

* **Variable Arguments: A method signature can allow for a variable amount of arguments to be passed when the method is called.**

If not using keyword arguments, a method signature can specify that it allows a variable amount of arguments using `*` like so:

```ruby
def method_with_variable_arguments(*args)
```

When multiple arguments are passed to `#method_with_variable_arguments` above, the list of arguments is converted into an array. For example:

```ruby

def method_with_multiple_arguments(*args)
    pp args
end

method_with_multiple_arguments("Sally", "Betty", "Tommy")
# => ["Sally", "Betty", "Tommy"]
```

If using keyword arguments, a method signature can specify that it allows a variable amount of keyword arguments using `**` like so:

```ruby
def method_with_variable_keyword_arguments(**args)
```

When multiple arguments are passed to `#method_with_variable_keyword_arguments` above, the list of arguments is converted into a hash. For example:

```ruby

def method_with_multiple_keyword_arguments(**args)
    pp args
end

method_with_multiple_keyword_arguments(first: "Sally", second: "Betty", third: "Tommy")
# => {:first => "Sally", :second => "Betty, :third => "Tommy}
```

<p/>
----------
<p/>

* **Keyword Arguments: Instead of passing a hash to a method, a method signature can allow for arguments to be passed in a "key: value" format without the curly brackets.  It's hash-like.**

For example, a method signature might look like this:

```ruby
def method_requires_keyword_arguments(needed:, also_needed:)
```

And the method call might look like this: 

```ruby
method_requires_keyword_arguments(also_needed: "A good reason", needed: "To Do something")
```

See above under Required Arguments for some more info.

<p/>
----------
<p/>

* **On Passing Symbols to Method Calls: With the keyword arguments above, the colon comes after the variable names.  Sometimes the colon comes before them.  What's going on there?**

Remember to distinguish a method signature from a method call.  This is a method signature:

```ruby
def method_requires_arguments(first_arg, second_arg)
```

While this could be a method call for that method:

```ruby
method_requires_arguments(:apple, :pear)
```

Because Ruby allows you to omit parentheses when calling a method, the same method call might look like this: 

```ruby
method_requires_arguments :apple, :pear
```

When the method above is being called, symbols are being passed to the method as values for the variables declared in the method signature.  Symbols are a special types of Ruby objects that are [often compared to strings and often used interchangably with strings](https://culttt.com/2015/04/22/what-are-symbols-in-ruby/).  Symbols are fundamentally different from strings, however, because [each symbol is immutable](https://learn.co/lessons/ruby-symbols) and thus unique.  This uniqueness allows for [some memory efficiency](https://medium.com/@yoelblum_45935/ruby-symbols-the-good-and-bad-parts-6ae8e10f57da) when symbols are used rather than strings, in come cases.

In any event, research seems to indicate that when symbols are passed as arguments, it's a good bet that the symbols are stand-ins for strings and are being interpolated somewhere in the method's body.  For example, the `attr_accessor` method takes symbols as arguments...

```ruby
attr_accessor :open, :close, :edit:
```

...and you can see how others have tried to recreate these methods [here](https://mikeyhogarth.wordpress.com/2011/12/01/creating-your-own-attr_accessor-in-ruby/) and [here](https://medium.com/@amliving/diving-into-rubys-attr-accessor-d34e2ccc5477) by interpolating the symbols into strings in the body of the method. 

But before you dive into those resources, here's a simple demonstration of how a symbol can be interpolated: 

```ruby
def print_word(msg)
    pp "Here's the word you wanted to print: #{msg}"
end

print_word :blamo!
# => "Here's the word you wanted to print: blamo!"
```

So when you see a symbol passed to a method as an argument, it's not off base if your first assumption is that the symbol is being used as a stand-in for a string. 
  
<p/>
----------
<p/>

I hope the above serves as a quick crash course or refresher on different types of Ruby arguments.  Again, the method signature in the source code is what will really inform you about what you can pass into the method.  But to tie all of the above together, when you see something like this...

```ruby
before_action :check_if_signed_in, only: [:edit, :update, :destroy]
```

...you might be at least able to make certain assumptions, like:

* `before_action` is a method that is being called; stylistically the developer is not wrapping the arguments in parentheses

* In the Rails context, a developer usually relies on `#before_action` to execute certain helper methods.  In this case, the helper method would be `#check_if_signed_in`.  But what might be happening under the hood of `#before_action` is that it is accepting a symbol argument that `#before_action` interpolates into a string. It might then use a metaprogramming method to ensure that the actual method `#check_if_signed_in` is executed when appropriate.  

* `#before_action` accepts an additional argument after `:check_if_signed_in`; that is, the second argument `only: [:edit, :update, :destroy]`.  The `only:` tells us that this is a keyword argument.  Because the second argument is not needed in the Rails context, the method signature might provide that its placeholder variable is an optional argument with a default value provided, or it might be a variable argument.  

Of course, without examining the code base and the actual method signature for `#before_action`, we can't be sure about the assumptions above.  But at least , but at least we might hav
is actually being passed to `#before_action` is a symbol that `before_action`gk f you pass `:check_if_signed_in` to `before_action`, may sound to method or something else you have created in your own code,  