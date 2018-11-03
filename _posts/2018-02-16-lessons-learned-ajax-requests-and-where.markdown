---
layout: post
title:  "Rails: Some Lessons Learned on Making More Efficient Active Record Queries for `$.ajax()` Requests using `.where()`"
date:   2018-02-16 12:00:00 -0400
categories: coding
---

I just added some upgrades to my “Garden Tracker” app -- a Rails app that keeps track of one’s gardens and the species and plantings within those gardens. The goal was to incorporate some jQuery `$.ajax()` requests (“ajax requests”) to make the retrieval of certain information more seamless for a user. I learned many lessons in implementing these upgrades, but the strongest lesson has to do with what I learned about making more efficient Active Record Queries.  

As some background: the app has a garden "show" page that displays details about one of a user's gardens. I wanted to implement a "Next Garden" button on the show page.  The plan was for the button to (1) trigger an ajax request to the app's server for data about a garden that the user created subsequent to the currently displayed garden and (2) then result in the replacement of the currently displayed information with the newly retrieved data.  

The problem I encountered almost immediately in trying to implement the “Next Garden” button was that the ids of a user’s gardens were not necessarily consecutive.  For example, a user might have gardens with ids 1, 2, 5, 7, and 10.  So the “Next Garden” button would have to know how to navigate these ids without assuming the ids came one after the other.  

After chewing on this problem a bit, I began to dig myself into a rabbit hole and implement an unnecessarily complicated solution.  It worked, and it's described in the numbered steps below (but feel free to skip it, because things get a lot cleaner in the paragraphs that follow):

> 1. The show page for a garden loads with a hidden "Next Garden" button.  
> 2. Once the show page finishes loading, the JavaScript automatically makes an ajax request to retrieve an array of all of the garden ids of a user.  This is supplied by an instance method in the Garden model.
> 3. If the ajax request is successful and the array of ids is received by the browser, through a string of .then() chains, the array is stored in a JavaScript variable and the “Next Garden” button is made visible.  An event listener is attached to the “Next Garden” button, and if the user clicks it, a second ajax request is made to retrieve and display the data for user’s next garden (marching along the array of garden ids).  In addition, another JS function is called to keep track of whether the currently displayed garden is the last garden in the array, in which case, the clicking the “Next Garden” button causes the user’s first garden to be displayed.

This complicated solution made an ajax request whether or not the user clicked the "Next Garden" button, and then another ajax request every time the user clicked it again button. When a mentor reviewed it, he quickly pointed out that the first ajax requests could be tossed out, and the JavaScript made a lot simpler, with a better Active Record query.

For example, for the garden show page, let's make sure the user's id and the id of the currently displayed garden are attached to the Next Garden button (via HTML and Rails/ERB):

```erb
<div class="button_display">
  <%= button_tag "Next Garden", id: "next_garden_button", data: {garden_id: @garden.id, user_id: @garden.user_id} %></h1>
</div>

<div id=<%="garden_display"%>
  <%# HTML to display garden information (omitted) %>
</div>
```

Then, when the page loads, let's attach a listener that works as follows:

```javascript
function attachGardenListeners() {

  $("#next_garden_button").on("click", function(e) {
    e.preventDefault()

      // Grab the id of the user and the id of the currently displayed garden:
    const userId = e.target.dataset.userId
    const gardenId = e.target.dataset.gardenId

      // Make an ajax request that includes these ids in the params:
    $.ajax({
        url: `/users/${userId}/gardens/${gardenId}/next`,
        method: "GET"
      })
      // The ajax get request gets routed to #next in the Garden Controller
      // (below).  This method sends back information about the next garden
      // that the user created, or if the user's last garden was displayed,
      // sends back information about the user's first garden.
      .then(function(data) {
      // Some code is omitted here for simplicity sake.  But the data that
      // comes back is used to update the "New Garden" button with the new
      // garden's id, to be used by this procedure the next time the button
      // is clicked:
          $("#next_garden_button").attr("data-garden-id", gardenObject.id)

      // And then the returned data is used to generate an HTML "insert"
      // to display the information about the new garden
        const insert = renderGardenShow(data) // function omitted
        return insert
      })
        // Insert the "insert" into the DOM within the garden_display container.
      .then(function(insert) {
        //
        $(`#garden_display`).html(insert)
      })
  })
}


```
When the "Next Garden" button triggers the ajax request, the app's router directs the request to a #next method within the Garden Controller, which then behaves as follows:

```ruby
def next
      # Identify the user:
    user = User.find_by(id: params[:user_id])
      # Make an Active Record request using `.where()` to grab the first of the
      # user's gardens with an id greater than the current garden's id.
    garden = user.gardens.where("id > ?", params[:garden_id]).first
      # If the query returns a garden, then send the new garden's data
      # back to the browser, where the JavaScript above will handle its
      # display.
    if garden
      render json: garden
      # If no query is returned (for example, because the user's last garden is
      # the currently displayed garden, so there is no "next garden"), then
      # simply send back data on the user's first garden.
    else
      render json: user.gardens.first
    end
  end

```

With this improved way of doing things, an ajax request is made only when the "Next Garden" button is clicked, and the Gardens Controller makes sure that only the most relevant information is succinctly passed back to the browser.  Then the app's JavaScript app only has to plug it in and display. The controller does this through a simple use of an Active Record `.where()` relational query. And I think that's why this lesson sticks out in my head more than any other lesson from this refactoring: I had initially implemented an ajax/JavaScript solution that was way more complicated and more verbose, and all it took to improve the code's efficiency and optics was a mentor pointing out that I should use a simple `.where()` query in the relevant controller, as the query was meant to be used.

Taking a step back after I made these improvements, I realized I should know more about `.where()` queries than I currently do. So I did some digging. There are good resources out there which I'm sure will be a huge help in the future, but for now, here is my super-rudimentary cheat sheet.

* You can pass a string or a hash to the `.where` query.  For example:
  * `garden = Garden.where("id > ?", params[:garden_id])`
  * `garden = Garden.where(square_feet: 25)`
<p />

* If you use a string, do not use a pure string. Make sure to use the following format, where the arguments after the string are used to fill in one or more question marks.  This avoids being vulnerable to SQL injection hacks.
  * `garden = Garden.where("id > ?", params[:garden_id])`
<p />

* To make a query by eliminating certain values, you can follow `.where` with `.not()`.  For example, from the Rails Guide:
  * `client = Client.where.not(locked: true)`
<p />

* To get the amount of records returned from the query, the Rails Guide suggests appending `.count` (`.length` is not mentioned):
  * `garden_count_25_feet = Garden.where(square_feet: 25).length`
<p />

* If you want to see the SQL statement being generated by `.where()` to check its accuracy, append `.to_sql`
  * `Garden.where("id > ?", params[:garden_id]).to_sql`
<p />

* Or, for other feedback that might be helpful, try appending `.explain`
  * `Garden.where("id > ?", params[:garden_id]).explain`
<p />

* Some good resources (with my thanks to their authors):
  * [Rails Guide on Active Record Queries](https://guides.rubyonrails.org/active_record_querying.html)
  * [Article by Pedro Rolo](https://www.imaginarycloud.com/blog/queries-on-rails/)
  * [Mitch Crowe's Blog](http://www.mitchcrowe.com/10-most-underused-activerecord-relation-methods/)
