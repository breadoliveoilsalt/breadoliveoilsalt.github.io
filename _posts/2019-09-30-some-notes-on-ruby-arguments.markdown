---
layout: post
title: "Some Notes on Passing Arguments to Ruby Methods"
date:   2019-09-30 12:00:00 -0400
categories: coding
---

I've been returning to Ruby recently, and it seemed like an appropriate time to demystify for myself some of the symbols and patterns that appear when methods are being called with arguments.  For example, if you're using Rails, you might see a `#render` method call like this:

```ruby
render :new
```

Or you might see a `#before_action` method call like this:

```ruby
before_action :check_if_signed_in, only: [:edit, :update, :destroy]
```
