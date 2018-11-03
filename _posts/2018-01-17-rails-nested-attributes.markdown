---
layout: post
title:  "Rails: Generating Error Messages when Updating a Nested Form Without `accepts_nested_attributes_for`"
date:   2018-01-17 12:00:00 -0400
categories: coding
---

I've been working on creating a Rails app called Garden Tracker. It allows a user to keep track of basic information about the user's gardens, the species within those gardens, and the particular planting quantities in each garden. A user can create a password protected account (via `has_secure_password` and bcrypt) or can create an account by logging in with his or her GitHub account (via OmniAuth).

I came across an odd problem while working on the app, based on somewhat unconventional circumstances. My models included [Active Record validations](https://guides.rubyonrails.org/active_record_validations.html) to ensure that the user was entering correct data into forms. The challenge that arose was this: when a user submitted a nested form to edit a parent and child object at the same time, Rails would generate Active Record validation errors for the instance of the parent object, but not for the instance of the nested child object. This was not due to Rails' fault, but due to a peculiarity of the app. Below are more details and how I fixed the problem. I imagine there are other ways to solve the problem, but I figured I would describe what I did here, in case helpful to someone else.    

As part of the app's basic architecture, the app includes a Garden model and a Planting model. A Garden object `has_many` Planting objects, while each Planting object `belongs_to` a Garden object.  When a user  edited an instance of a Garden object via a form, I also wanted the user to be able to create or edit any Planting object belonging to that Garden object. Thus, the form to edit a Garden would require a nested form for each Planting.

Typically, the Rails class method [`accepts_nested_attributes_for`](https://api.rubyonrails.org/classes/ActiveRecord/NestedAttributes/ClassMethods.html) would handle this just fine. A catch of creating this app, however, is that the Flatiron School (where I'm currently enrolled) will be assessing the app and requires that the nested child must be instantiated by a custom writer method. This partially overrode the functionality of `accepts_nested_attributes_for`, making it slightly unreliable.  In particular, as I worked my way through the app, a side effect of this catch emerged and created the problem described above: if a user submitted invalid data via a nested form to edit a Garden and a Planting, Rails would generate Active Record error messages for the Garden object, but not the Planting object.  

I turned to the [Rails guides](https://guides.rubyonrails.org/active_record_validations.html) for ideas on how to solve this.  The Rails guide on Active Record validations describes how one can add custom error messages for a particular attribute of an object instance.  To borrow from the Rails guide, for example:

```ruby

class Person < ApplicationRecord
  def a_method_used_for_validation_purposes
    errors.add(:name, :invalid_characters, not_allowed: "!@#%*()_-+=")
  end
end

```
The guide also describes how, if a programmer wants to add a general error message for an object instance, rather than for any particular attribute of the instance, the programmer can push the error message to a [:base] attribute, like so:

```ruby
class Person < ApplicationRecord
  def a_method_used_for_validation_purposes
    errors[:base] << "This person is invalid because ..."
  end
end

```

The solution I eventually came up with relied on this latter example. I speculated that if I always had access to the errors of a Garden instance, I could push any validations errors about a Planting instance to the `[:base]` of the associated Garden instance. To do this, I couldn't rely on Active Record's #update method, however; I'd need a custom updating method whenever the user submitted information about a child Planting instance. And this custom updater would have to use `self` properly to refer to the Garden instance when dealing with an associated Planting instance.

Here's what the code looked like, with comments describing the sequence of how things are intended to function.  This ended up working and allowed me to display any validation errors when a user submitted a Garden form with nested form for one or more Plantings.  Hope it helps if you find yourself in similar circumstances.

```ruby
# GARDEN CONTROLLER:

    # The app's router sends the data to GardenController#update when user submits a form
    # for editing a Garden object with a nested form for editing a
    # Planting object.
def update
    # If there is no data about the Planting object in the params, simply
    # update the Garden object using Active Record's #update...
  if !garden_params[:plantings_attributes]
    @garden.update(garden_params)
    # ...and then see if there are any errors via a private method (further below):
    test_update_and_redirect
  else
    # If, however, the user has submitted information to update
    # a Planting object via a nested form, head to the custom updater
    # method in the Garden model:
    @garden.custom_updater_for_nested_params(garden_params)
    # Then, as above, see if there are any errors:
    test_update_and_redirect
  end
end

# GARDEN MODEL CUSTOM UPDATER METHOD:
  # Note: Garden Model has Active Record validations, for the
  # attributes/parameters below, not shown here.

def custom_updater_for_nested_params(garden_params) # garden_params includes Planting params

      # Update model with user's Garden information, such as the Garden's
      # name, description, and square feet:
    self.name = garden_params[:name]
    self.description = garden_params[:description]
    self.square_feet = garden_params[:square_feet]
      # If the Garden can be saved (meaning the Garden data submitted has
      # passed the Active Record validations in the Garden model), update
      # the associated Planting objects with the planting's species and quantity:
    if self.save
      garden_params[:plantings_attributes].values.each do | planting_attributes |
        planting = self.plantings.find_by(id: planting_attributes[:id])
        planting.species_id = planting_attributes[:species]
        planting.quantity = planting_attributes[:quantity]
        planting.save
          # If after trying save the Planting object, the information about the
          # Planting is invalid and there are validation errors, take the error
          # message about the Planting and push those errors to the Garden
          # object errors (i.e., self.errors, with 'self' referring to the Garden object).
        if planting.errors.messages != {}
          self.errors[:base] << "Errors with the #{planting.name} planting: #{planting.errors.full_messages.join(". ")}."
        end
      end
    end
  end


# GARDEN CONTROLLER PRIVATE METHOD:

def test_update_and_redirect
    # Thanks to #custom_updater_for_nested_params, any errors for
    # the Garden or the Planting object will appear under
    # @garden.errors.messages. If there are any such errors,
    # redirect user back to the edit page with the nested form. The edit
    # page then renders the errors, prompting the user to re-submit the form.
  if @garden.errors.messages != {}
    render :edit
  else
    # If there are no errors, redirect user to the Garden show page and
    # let user know everything is ok.
    flash[:message] = "#{@garden.name} was updated."
    redirect_to garden_path(@garden.id)
  end
end
```
