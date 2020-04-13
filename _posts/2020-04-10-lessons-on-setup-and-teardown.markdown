---
layout: post
title: "Rails/Rspec/Jest: A Few Lessons Learned on Setting Up and Tearing Down Unit Tests to Avoid Test Pollution"
date:   2020-04-10 12:00:00 -0400
categories: coding
---

Recently I've been working on an app with a [Rails backend](https://github.com/breadoliveoilsalt/tinndarp-backend) and a [React frontend](https://github.com/breadoliveoilsalt/tinndarp-frontend) (in separate repos facilitate playing with cross-origin requests between different hosts).  While TDD'ing the code and working on unit tests, I've come across various bugs, but some of the most perplexing ones were due to bugs in the tests themselves. While diving into these bugs out with my mentors, it became apparent that most of the problems, in one form or another, were attributable to improper setup or teardown of unit tests.  This, in turn, was leading to varying forms of test pollution or database pollution, causing my tests to malfunction. 

The "four phases" of testing -- setup, exercise, verify, and teardown -- have been written about extensively (see, for example, [here](http://xunitpatterns.com/Four%20Phase%20Test.html) and [here](https://8thlight.com/blog/thomas-countz/2019/02/19/essential-and-relevant-unit-tests.html)).  Oddly enough, when reflecting and doing research on my errors with setups and teardowns, I found it difficult to find concrete discussions of what could go wrong, particularly with teardowns.  I figured this would be a good place to throw my two cents in and note some of the main things I learned.

-------
<p/>
<p class="text-centered bold">Rails/Rspec Test Setup: Watch out where you instantiate objects. Do it only within `it` or `before(:each)` blocks.</p>

<span class="underlined">The situation</span>:  I had about 50 unit tests passing. I started TDD'ing some new tests before writing the actual code, ran all of the tests expecting to see only the new ones fail, and all of a sudden, a slew of my old tests were failing.  I had no idea what was going on, since I hadn't touched the old tests or their classes.  

<span class="underlined">Assessing the situation</span>: I turned to the Rspec failure logs.  There was so much red that, at first, I didn't know where to start.  Going line by line, though, I began to notice a pattern: many of the failures involved tests manipulating my test database and, directly or indirectly, asserting against an expected amount of return records.  My assertions were failing because, all of a sudden, a few more database records were showing up in the actual returned collection.  Funny enough, there were never *less* records then expected....

<span class="underlined">A relevant aside</span>: A mentor sat down with me shortly after I noticed this and explained that the situation smelled like some sort of test pollution, or rather, database pollution coming from my test setup.  We poked around for a bit and he helpfully pointed out that there is a [config setting](https://relishapp.com/rspec/rspec-rails/docs/transactions) in `rails_helper.rb` specifying that when something is added to the database during a test, it is automatically rolled back (i.e., deleted from the database) after the test:

```ruby
RSpec.configure do |config|
  config.use_transactional_fixtures = true
end
```

I had no idea this was there.  I had been taking it for granted the whole time.  In any event, we checked out  `rails_helper.rb` and everything looked ok.  BUT if you face a similar problem, please be sure to check this out. 

<span class="underlined">What the problem was</span>: My mentor was right that there was some sort of database pollution going on from my test setup.  And it was hidden in plain sight. After scratching my head with the code for a while and testing out prior commits, I knew the problem had to come from the tests I had just written.  What did I do wrong there?  Then it dawned on me: moving too quickly, I had attempted a test setup that created objects right after the start of a `describe` block. Now, everytime the entire test suite was running, these new objects were being created in the database, but never rolled back.  The setting in `rails_helper.rb` wasn't rolling back these transactions automatically because they weren't happening in a test itself (i.e., inside an `it` block) or in a `before(:each)` block typical of test setups.  

<span class="underlined">The solution</span>: I dropped my entire test database (usually done through `rake db:drop RAILS_ENV=test`), re-ran the migrations, and made sure my test setup was properly within an `it` block or `before(:each)` block.

-------
<p/>
<p class="text-centered bold">React/Jest Test Teardown: Watch out for mocking imported modules, particularly in ES6. Make sure you restore any mocks or spies.</p>

<span class="underlined">The situation</span>: I mentioned above that, with my backend tests, I had been taking for granted that Rspec was rolling back database transactions automatically after each test.  It turns out that, on the frontend, I was also taking for granted that Jest was restoring mocks when I imported a module and mocked it. Left to its own devices, Jest wasn't. 

I discovered this in the context of testing code that made calls to my backend api.  For the working code, I leaned on [axios](https://github.com/axios/axios) to make the backend calls, but I placed each axios function in a wrapper function.  I did this for two reasons: (1) to be able to mock the wrapped methods in my tests and make sure the correct arguments were being passed in, and (2) to be able to stub out api return values from these mocks in other tests. In the case where the problem arose, I was testing functions that called my axios wrappers .  At the top of the test suite, I imported my wrapper functions, and my tests would check the call count and that the proper arguments were being passed to the wrappers.  

<span class="underlined">A relevant aside</span>: How do you mock something you've imported into a test file when working in ES6? Say my axios wrappers looked like this...

```javascript
// axiosWrappers.js

export function axiosWrapperGet(url, params) {
  // axios get method
}

export function axiosWrapperPost(url, params) {
  // axios post method
}
````

...and say I wanted to mock my `axiosWrapperGet()` using Jest, something like this...

```javascript
// test file
import { axiosWrapperGet } from './axiosWrappers'

// ... stuff here
    axiosWrapperGet = jest.fn()
```

If I tried this, I would quickly get an error.  Part of the problem is that JavaScript imports are [read-only](https://stackoverflow.com/questions/38060519/es6-import-as-a-read-only-view-understanding) and are ["live connections"](https://exploringjs.com/es6/ch_modules.html#sec_imports-as-views-on-exports) to the exports, not copies of the exports. The work around, although a bit clunky, is to import all of the exports within one big object and then to assign the mock to the key that represents your function, like so:

```javascript
// test file
import * as axiosWrappers from './axiosWrappers'

// ... stuff here
    axiosWrappers.axiosWrapperGet = jest.fn()
```

<span class="underlined">The (hidden) problem</span>: While talking to another mentor about comparisons between Jest and Rspec and the database pollution problem I had faced with Rspec above, the mentor pointed out that there was no default behavior in Jest to restore a mocked function to its original state after each test, as there might be with Rspec. And he noticed that, while I was mocking my wrappers, I was never restoring the mocks.  This baffled me at first, because (as of this writing) the Jest doc's [Introduction to Mock Functions](https://jestjs.io/docs/en/mock-functions) never mentions having to restore mocks. I had assumed no such teardown procedure was necessary. But sure enough, as this mentor pointed out to me, there are all kinds of mock-restoration functions in the [Jest mock API Reference](https://jestjs.io/docs/en/mock-function-api). So, after each test that mocked something, I should have been restoring the mock, like so:

```javascript
// test file
import * as axiosWrappers from './axiosWrappers'

// ... test stuff here
    axiosWrappers.axiosWrapperGet = jest.fn()
// ... more test stuff here and then teardown
    axiosWrapper.axiosWrapperGet.mockRestore()
```

I hadn't noticed any problems with my tests up to this point when I wasn't restoring mocks, but that's likely because I was re-assigning my imports to a new mock before each test with `beforeEach()` AND it just so happens that none of my other tests depended on an imported functions behaving normally.  If, conversely, I had a test file that needed to mock an imported function in one test and then needed the imported function to behave normally in another test, things would have blown up fast.

Upon some further digging, I learned that Jest is configurable to restore all mocks automatically (just as `rails_helper.rb` is configurable to rollback database transactions automatically), but the Jest default is NOT to restore automatically. [Here](https://jestjs.io/docs/en/configuration.html#restoremocks-boolean) are the Jest docs on how to change the configuration.

I feel lucky to have mentors to point out the problems with my test setups and teardowns, as well as that I was able to recover fairly quickly from these problems.  The biggest lessons I learned overall are (1) to "color within the lines" of a `before(:each)` block or an actual test when it comes to test setup, and (2) to not take for granted that a testing framework will automatically handle teardowns.  This latter point must always be verified, even if the tests seem to work well without any explicit teardown in the tests themselves.  
