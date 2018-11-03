---
layout: post
title:  "React/Redux/Thunks: Getting a Thunk to Re-Execute a Fetch Request When the First Try Retrieved Invalid Data"
date:   2018-07-31 12:00:00 -0400
categories: coding
---

I build a React app called [Browseum](https://github.com/breadoliveoilsalt/browseum). Redux is used to maintain the app's state, and Thunks are used to handle the asynchronicity of `fetch()` requests.  At it's core, the app is intended to help a user learn more about art by allowing the user to view works of art selected randomly, along with some accompanying information about each art piece. The inspiration for the app came from the fact that I don't know a lot about art, but wish I did. Once a work of art is displayed, navigation buttons permit the user to generate other works of art at random or, if the user takes interest in the currently displayed art work, permit the user to look for more art with the same artist, time period, or  culture. A user can also mark an art piece as a favorite and view his or her browsing history for the past 30 days.

Before displaying a work of art, the app needs to populate its Redux state with information about the work.  To retreive such information, I turned to the [Harvard Art Museum API](https://github.com/harvardartmuseums/api-docs).  For example, when a user clicks a button to get a piece of art, the app makes a specific `get` request to the Harvard Museum API.  The API then responds to the request by sending back information about a work of art, such as the title, author, and a URL for an image of the piece. The app's JavaScript checks that the information is complete, populates its Redux state with the information, and then uses the updated state to render the art piece and its information to the user via React.  

Thunk middleware is responsible for handling this dance between the Harvard Museum API and the app’s Redux state.  You can read lots about “thunks” [here](https://github.com/reduxjs/redux-thunk), but in my mind, I think of a thunk as a function that

* (1) is responsible for doing something asynchronous in Javascript, like a fetch request,
* (2) makes use of Redux’s dispatch function to manipulate an app’s Redux-governed state, but
* (3) in order to accomplish (1) and (2), has to have a return value of another function, with `dispatch` as an argument.  

This other function then does the asynchronous action and dispatching.  To illustrate, here’s one of the examples from the redux-thunk [README](https://github.com/reduxjs/redux-thunk):

![](/assets/images/2018-07-31-getting-a-thunk-to-re-execute/image-1.jpg)

When the correct parameters are used in a `get` request to the Harvard Museum API, the API will gladly provide a JSON object containing information about a random piece of art. However, no matter what I tried, I found that the parameters of the request could not be set to guarantee that the random piece of art had an image associated with it.  The parameters could be set to guarantee that a returned JSON object had a key called `primaryimageurl,` but the value for that key might be `null`, rather than a URL for the image. In order to ensure that Browseum did not get stuck displaying information about a piece of art without an image, I wanted to draft a thunk that would make a `fetch()` request to the Harvard Museum API to retrieve a JSON object about a random work of art, but then re-run the `fetch()` request if the JSON object returned did not have a `primaryimageurl` key or a had a `primaryimageurl` key with a null value.  

After some trial and error, I got the thunk to re-run the `fetch()` request by breaking the thunk into two separate functions, separating the asynchronous action (#1 above) from the returned function (#3 above).  Below is a much-simplified version of what I came up with.  Assume for illustrative purposes that the `fetch()` request has the proper parameters to return a JSON object (a “record”) with information about a single piece of art.

```javascript

  // `getRandomArt()` is the thunk that gets called when the app is retreiving
  // a random work of art. But its async `fetch()` request is basically moved
  // to a separate function that can re-execute itself, fetchBasicData().
  // Let's call `getRandomArt()` the "main thunk" and `fetchBasicData()` the
  // "sub-thunk"
export function getRandomArt() {
  return function(dispatch){
    return fetchBasicData(dispatch)
    }
  }

  // Note that we had to pass `dispatch` from the main thunk to the sub-thunk
  // so the sub-thunk could call `dispatch` later.
function fetchBasicData(dispatch) {
    // Here is the `get` request to the Harvard Museum API to request a record
    // for a randomly-selected work of art:
  fetch(harvardUrlWithProperParameters)
    .then(response => response.json())
    .then(record => {
        // If the record has an image (i.e., primaryimageurl is present and
        // not 'null'), then return our record to be dispatched to the
        // Redux state in the next `.then()`:
      if (record.primaryimageurl) {
        return record
        // If the record does not have an image, then throw a custom error,
        // and the flow of exeuction skips the next `.then()` to hit `.catch()`.
        // I used a Redux-style "plain old JS object" with a "type" and "data"
        // keys so that `.catch()` below could distinguish between an image
        // error and any other error, such as the Harvard API being down.
      } else {
        throw {errorType: "INVALID_RECORD", data: record}
      }
    })
        // If the record from the Harvard API has an image, then it's good.
        // Dispatch the record to the Redux reducer, inserting it into the
        // Redux state and rendering the new information for the user:
    .then(record => dispatch(loadArtObjectIntoState(record)))
        // Here's where we jump to if there are any errors along the way:
    .catch(error => {
        // If the record has returned without a valid primaryimageurl, then
        // re-run the fetch request by calling the sub-thunk again. Note how
        // again we have to pass `dispatch` to `fetchBasicData()`, otherwise
        // the dispatching above won't work.
      if (error.errorType === "INVALID_RECORD") {
        fetchBasicData(dispatch)
        // If there are other errors, such as the Harvard API is down, then
        // dispatch an error message, which will render in the DOM to alert
        // the user:
      } else {
        dispatch(loadError("Sorry, error with the Harverd API.")))
      }
    })
}

```


I hope this conveys the flow of how the thunk was able to re-run a `fetch()` request in the event the Harvard Museum API returned a record (no error per se), but the record wasn’t what the app needed.  Please note, too, how the thunk had to pass `dispatch` as an argument to the sub-thunk every time it was called, so that the sub-thunk could ultimately dispatch approved data to the Redux state.  

One thing that worried me for a while was whether re-calling the sub-thunk would leave the Promise (`fetch()`) of the original sub-thunk outstanding.  In other words, if the Harvard API kept returning incomplete records and multiple `fetch()` requests had to be made, one after the other, would we wind up with a stack of unresolved Promises (`fetches`)? After some research, I believe that the use of `.catch()` closes each Promise (see generally [here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch)), so this problem should be avoided, but it's something worth chewing on a bit more.

I hope this helps if you find yourself in a similar situation. If you’re interested in the complete code for Browseum, it can be found [here](https://github.com/breadoliveoilsalt/browseum), with the more fulsome version of the code above [here](https://github.com/breadoliveoilsalt/browseum/blob/master/client/src/actions/harvardApiThunks.js).
