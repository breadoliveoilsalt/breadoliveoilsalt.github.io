---
layout: post
title:  "Rails: Thoughts on Creating a Custom Validation for an ActiveRecord `date` Datatype"
date:   2018-12-22 12:00:00 -0400
categories: coding
---

This post describes a solution I came up with to deal with the fact that ActiveRecord's `validates` macro does not play nicely with a model attribute whose datatype is "date".  Recently I've been upgrading [Garden Tracker](https://www.breadoliveoilsalt.com/projects/#garden-tracker), including adding a feature where a user can specify the specific date she planted something in order to get an estimate of when it can be harvested. The upgrading involved creating a new migration for a "Planting" model.  The migration added among other things, a `date_planted` attribute with an ActiveRecord "date" datatype.  For example:

```ruby
class UpdatePlantings < ActiveRecord::Migration[5.1]
  def change
    add_column :plantings, :date_planted, :date
  end
end
```

When a user submitted a form to create a Planting instance, I wanted the Planting model to validate the input for `date_planted` to ensure no invalid data was saved.  I quickly learned, however, that the classic ActiveRecord validation macro would not work as intended, especially when I was interested in displaying input errors to the user.

```ruby
  # In other words, something like this is not helpful:
class Planting < ActiveRecord::Base
  validates :date_planted, presence: true
end
```

The problem seemed to be the following:

1) Assume a Planting object has several attributes in addition to `date_planted`, such as `species`, `expected_maturity_date`, etc.  Also assume that when a user submitted a form to create a new Planting object, all the attributes were correct, except the input for `date_planted` was a non-existent date ("2/30/2018").

2) When the form to create this Planting object is submitted, the `date_planted` input appears to be filtered through the migration's *datatype* first, before hitting the Planting model's ActiveRecord validator.  The datatype recognizes that the input could not be used to create a valid Date object, so the datatype automatically sets `date_planted` to `nil`.

3) After that, `date_planted` is tested by the Planting model validator. Upon seeing that `date_planted` is `nil`, the validator will not save the object, and pushes an error to the unsaved object stating that the user did not input anything at all for `date_planted`.  This is clearly not the case, and if we return the user to the Planting form with this error on display, it will no doubt be confusing and unclear.

When trying to figure out a potential solution, I came across [this helpful discussion](https://github.com/rails/rails/issues/29272).  It introduced me to the `before_type_cast` accessor. Here's the way I simplify this tool in my mind (with a little help from the [documentation](https://api.rubyonrails.org/classes/ActiveRecord/Base.html#class-ActiveRecord::Base-label-Accessing+attributes+before+they+have+been+typecasted)): using `before_type_cast` allows a Rails programmer to access the raw data that a user submitted before the data hits any of the ActiveRecord checks, including the migration datatype.  So, when a user tries to create a Planting and a Planting object is instantiated, the raw user input can be examined on the back end with:

```ruby
self.date_planted_before_type_cast
# "self" here is the Planting object that was just instantiated
```

Now that I had a way to effectively intercept the user's raw input (regardless of whether the migration's datatype changed it), I knew I would need a [custom validator](https://guides.rubyonrails.org/active_record_validations.html#performing-custom-validations) to check the date that the user provided. Since the raw input would be a string, my mind drifted toward `Date.parse` as a way to check whether the string represented a valid date.  But `Date.parse` throws an error when the string is not a valid date.  So how does one translate a Ruby error into a Rails error message?

The `valid_date_planted?` method below is what I came up with.  Basically, it uses a "[begin/rescue](https://www.vikingcodeschool.com/falling-in-love-with-ruby/throwing-and-handling-errors)" error handling pattern to test the user's date input by actually testing whether `Date.parse` throws an error when it processes that input:

```ruby
class Planting < ActiveRecord::Base

    # Use macro to call custom validator, #valid_date_planted?, before saving a
    # Planting object:
  validate :valid_date_planted?

  def valid_date_planted?
    # Use classic Ruby error handling to see if Date.parse throws an error
    # because the user's original input for `date_planted` is an invalid date...
    begin
      Date.parse(self.date_planted_before_type_cast)
    # ...and if Date.parse does throw an error, then add an error message to
    # `self` (i.e., the Planting instance we are validating).
    rescue
      self.errors.add(:date_planted, "for #{self.name} is not a valid date")
    end
end
```

Assuming our Planting Controller then re-renders the Planting form and passes along the invalid Planting instance, we can then display the error with some simple ERB, something like this:

```
<div>
  <h3> Sorry, there were some problems with your input: </h3>
    <ul>
      <%  @planting.full_messages.each do |msg| %>
        <li style="list-style:none;"> <%= msg %></li>
      <% end %>
    </ul>
</div>
```

But what if we don't want to force the user to provide a date when filling out a form to create a Planting? If the user left the `date_planted` field blank, it would be equal to a blank string ("") when tested by the migration datatype or the custom validator.  To solve this in another context, I simply added a conditional statement and a "short circuit".  Here's what it would have looked like if applied to the custom validator above:

```ruby
class Planting < ActiveRecord::Base

    # Use macro to call custom validator, #valid_date_planted?, before saving a
    # Planting object:
  validate :valid_date_planted?

  def valid_date_planted?
        # Adding a conditional statement:
    if self.date_planted_before_type_cast

        # Short circuit the validator and let it pass if date_planted equals "",
        # meaning date was not entered:
      return true if self.date_planted_before_type_cast == ""
        # Otherwise, same as above
      begin
        Date.parse(self.date_planted_before_type_cast)
      rescue
        self.errors.add(:date_planted, "for #{self.name} is not a valid date")
      end
    end
end
```

If you are struggling with how to validate a user's date input with Rails/ActiveRecord, you're not crazy.  A quick Google search will reveal many others have faced the same challenge.  I hope this discussion provides some assistance and ideas if you find your in this boat.  Best of luck!
