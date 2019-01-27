---
layout: post
title:  "Ruby: Hiding the Structure of Input Data with the `Struct` Class (One of Many Cool Things in Sandi Metz's Book)"
date:   2019-01-24 12:00:00 -0400
categories: coding
---

When I first started coding, someone recommended that I read Sandi Metz's book, *Practical Object-Oriented Design in Ruby*. This advice stuck with me, and over the past few weeks, I've been trying to set aside time to read 10 pages a day. I'm most of the way through the book, and I've learned enough cool things to fill probably 20 blog posts. For now, I decided to focus on just one of these cool things, which is something that I hadn't come across anywhere else to date: Ruby's `Struct` class.   

Here's my understanding of the problem that Sandi identifies in the context of her `Struct` discussion (I'm going to try to keep it very simple).  Say you want to create a Ruby class and instances of that class are initialized with a particular data structure. For example, you want to keep track of things in your refrigerator, and you have a list of those items in an array.

```ruby
["ketchup", "milk", "questionable cheese"]
```

So you write a Refrigerator class, and Refrigerator objects accept the array when you initialize them.  You also have a few methods that rely on this data.  For example, `#print_refrigerator_items` iterates over the array to print each item so you can see it, while `#print_number_of_unique_things_in_refrigerator` prints the length of the array.

```ruby
class Refrigerator

  attr_reader :food_items

    # data is an array: ["ketchup", "milk", "questionable cheese"]
  def initialize(data)
      @food_items = data
  end

  def print_refrigerator_items
      food_items.each do | item |
        puts item
      end
  end

  def print_number_of_unique_things_in_refrigerator
    puts food_items.length
  end

end
```

One of Sandi's main points throughout the book is that an app should have reusable and flexible code that requires as few changes as possible when a new requirement for the app comes along.  Something that helps in this quest, she says, is "hiding data structures" from your classes.  In other words, can you make each class as agnostic as possible toward the structure of the data it works with?  The importance of this is probably best demonstrated by identifying a potential future problem with the code above.  Let's say your friend wants to use your Refrigerator class to print the names and count of unique items in her refrigerator.  Unfortunately, she wants to initialize a Refrigerator object with a hash that provides each unique item's quantity:

```ruby
{ :mayonnaise => 3, :pickles => 4, :beer => 1 }
```

Now for your code to work, you have to rework each of your important methods. This is a simple example that can be fixed in a minute or two, of course, but imagine if your class had tons of methods. All of them would have to change. Sandi's recommendation to "hide data structures" is trying to help avoid this problem. And the way to implement this recommendation, she advises, is basically to use a gatekeeper method that works in conjunction with `#initialize` and takes advantage of Ruby's `Struct` class.  This gatekeeper method digests and organizes the data first, so the rest of your methods don't have to.  Then, if the structure of the input data changes, the only thing that has to be updated is this gatekeeper method.  

But what is Ruby's `Struct` class? From Sandi's examples and the [Ruby documentation](https://ruby-doc.org/core-2.6/Struct.html), it strikes me as a tool to use as you say, "Hey, I want to transform each element in my input data structure into a lightweight object.  Then, my heavyweight methods can all universally work off of lightweight objects, rather than be dependent on the particular form of the input data."  `Struct` is the transformative tool here.  You call `Struct.new` with the methods to which you want you lightweight objects to respond.  It works something like this (using a refactoring of the code above, assuming the data is still an array):

```ruby
class Refrigerator

  attr_reader :food_items

    # `data` is still an array: ["ketchup", "milk", "questionable cheese"]
    # To populate @food_items, we're now referring to a gatekeeper method,
    # `#get_names`, that utilizes `Struct` below.
  def initialize(data)
      @food_items = get_names(data)
  end

  def print_refrigerator_items
      food_items.each do | item |
        puts item.name
      end
  end

  def print_number_of_unique_things_in_refrigerator
    puts food_items.length
  end

    # Use `Struct.new` to define a lightweight mini-class, FoodItem.  Instances of that
    # FoodItem class will respond to a #name method.
  FoodItem = Struct.new(:name)

  def get_names(data)
      # When `#initialize` is called, we populate @food_items with a new array...
    data.collect { |thing_name|
      FoodItem.new(thing_name)
    }
      # ...filled with lightweight FoodItem objects that respond to `#name', whose
      # return value we supplied with our input data array.
  end

end

```

Now when your friend comes along and wants to use your Refrigerator class with a completely new data structure - a hash - all you have to do is update your `Struct` call and `#get_names`.  Every other method stays the same:

```ruby
class Refrigerator

  attr_reader :food_items

    # `data` is now a hash: { :mayonnaise => 3, :pickles => 4, :beer => 1 }
    # But see below: FoodItems and `#get_names` are the only things we
    # have to update, which is a very quick fix!
  def initialize(data)
      @food_items = get_names(data)
  end

  def print_refrigerator_items
      food_items.each do | item |
        puts item.name
      end
  end

  def print_number_of_unique_things_in_refrigerator
    puts food_items.length
  end

    # We add a new method, `#quantity` to our lightweight FoodItem objects...
  FoodItem = Struct.new(:name, :quantity)

  def get_names(data)
      #...and now the values in our hash provide the return value for the `#quantity`
      # method call on a FoodItem object.
    data.collect { |thing_name, quantity|
      FoodItem.new(thing_name, quantity)
    }
  end

end

```

Hopefully the example above illustrates the value of hiding input data structures from your classes and give a taste of the value baked into Sandi Metz's book, *Practical Object-Oriented Design in Ruby*. Sandi writes so well that much her advice comes across as poetry. It's worth a read by anyone interested in object oriented design.
