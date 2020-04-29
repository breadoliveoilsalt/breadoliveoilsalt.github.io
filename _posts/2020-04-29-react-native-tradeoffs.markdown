---
layout: post
title: "React Native/Expo/Jest/Enzyme: Some Considerations when Deciding How to Develop a React Native App"
date:  2020-04-30 12:00:00 -0400
categories: coding
---

One of my recent projects is an iOS app, built with React Native, called [TINNDÅRP](https://github.com/breadoliveoilsalt/TINNDARPiOS). The app allows a user to review IKEA furniture by swiping left or right and then to see whether the user has "liked furniture" in common with another user. 

There were many times while working on the app that I came to forks in the road and had to choose one way or the other.  I thought this post would be a good opportunity to discuss some of the questions around Expo and testing that swam around my head, and to summarize the tradeoffs behind going one way or the other, in case others find themselves in a similar spot. 

----------
<p/>
<p class="text-centered bold">Should I develop the app using Expo?</p>

The React Native docs describe different ways of getting a React Native app set up, but they proclaim that you "[can get you writing a React Native app within minutes](https://reactnative.dev/docs/environment-setup)" if you use Expo. What is [Expo](https://docs.expo.io/)?
Expo is basically a tool that layers on top of React Native.  Its stated intention is to make developing, building, and deploying React Native applications easier (although it is not without [some bad press](https://medium.com/better-programming/how-expo-is-fooling-everyone-adf7f34d7528)).  Expo offers two different, mutually exclusive layers to sit on top of React Native, one called a "bare workflow" and the other called a "managed workflow," each offering different functionality and tools.  See [here](https://docs.expo.io/versions/latest/introduction/managed-vs-bare/) for a fulsome comparison between the two in the Expo docs.  

I had to get moving fast when building TINNDÅRP, so I chose to use Expo's "managed workflow."  What were the benefits of using this approach?  To me the greatest ones are that it provides:

- a browser-based [Expo Developer Tool](https://blog.expo.io/announcing-expo-dev-tools-beta-c252cbeccb36) to manage the process of opening the app in an iOS Simulator or Android simulator, or opening the app as a browser-based, debuggable application, in a quick and easy fashion, with real time updates as the code changes;
- access to an Expo Client App on an iPhone, iPad, or Android device, to experience the app on one of these devices as it is being built, so long as the device is on the same network as the computer running the Expo Developer Tool; and
- a Expo-managed website that allows a user or tester to play with the app via a web browser without having to download the app from the Apple Store or Google Play (referred to below as the "Web Based Demo" and further described in [the README to TINNDÅRP](https://github.com/breadoliveoilsalt/TINNDARPiOS)).  

All of these things indeed helped me develop and debug the app quickly. There were tradeoffs to taking this path, however. The Expo managed workflow prohibited me from:

- writing anything in a native language such as Swift (you can only write in JavaScript/JSX if you want to use the Expo Developer Tool);
- downloading the binaries and playing with the app in Xcode; and 
- opening the app in an iOS Simulator directly (I could only do it through the Expo Developer Tool).

In addition: 

- even though TINNDÅRP was meant to be an iOS app, the Web Based Demo was only available on an Android device, whose styling didn't quite match up to the iOS-based styling; 
- Expo is opinionated about the app's file structure.  Initializing a React Native app with the Expo CLI produces a directory structure that is different from initialization with the React Native, npm-based set up;
- Installing Expo and Expo CLI as dependencies takes a while.  This could impact the build times in your CI/CD pipeline if your deploy tools cannot cache them.

I could live with these limitations given the constraints of the app and the need to move quickly. But one more thing worth noting, particularly if an app is expected to grow in complexity, is that Expo is not perfectly synced with the React Native libraries. It is sort of "behind the times," riding on the coattails of React Native, and that can lead to problems.  One problem I encountered, for example, arose because TINNDÅRP relies on storing a [JSON web token](https://jwt.io/) locally on the iOS device for user authentication.  React Native includes an AsyncStorage module currently sourced to the [the React Native community](https://github.com/react-native-community/async-storage) to handle locally storing such items.  Expo's managed workflow, however, prohibits any extension of Expo's native module packages and accordingly prohibits any sourcing to the React Native community.  Instead, the Expo docs force a developer to rely on a depreciated version of AsyncStorage sourced to the React Native core library.  It seems to work...but when running tests with Jest, nasty warnings like this show up:

<p style="text-align:center"><img src="/assets/images/2020-04-30-asyncStrorageWarning.png" width="800" /> </p>

For now, I am living with the warnings since there does not seem to be a workaround using Expo's managed workflow.  And I put a module wrapper around TINNDÅRP's AsyncStorage commands so any changes down the line will not cause changes elsewhere in my code. But seeing the warning does make me uncomfortable every time I run tests. For further discussions of this and similar issues, see [here](https://github.com/react-native-community/async-storage/issues/89), [here](https://github.com/react-native-community/async-storage/issues/304), and [here](https://stackoverflow.com/questions/56029007/nativemodule-asyncstorage-is-null-with-rnc-asyncstorage). 

While I encountered downsides such as this in using Expo's "managed workflow," it should also be noted that the Expo provides a way of ["ejecting" to its "bare workflow"](https://docs.expo.io/workflow/customizing/), which could give you more control and alleviate some of the issues above.  Unfortunately I cannot vouch for whether an ejection goes smoothly since I haven't tried it yet.  And it's worth noting that if a developer "ejects" in order to use native code, the developer will not be able to use the Expo Developer Tool, which can be particularly useful for debugging.

----------

<p/>
<p class="text-centered bold">What testing libraries should I use for unit testing?</p>

Like current iterations of React, React Native comes with the [Jest](https://jestjs.io/) testing framework out of the box.  There are upsides to using Jest, such as:

  - there is nothing further to install; 
  - it is *meant* to work with React and React Native, having its start from the same organization that created React and React Native; and
  - it has fairly extensive [mocking capabilities](https://jestjs.io/docs/en/mock-functions) to mock functions and imports.

When reviewing [Jest's React Native tutorial](https://jestjs.io/docs/en/tutorial-react-native), however, I became a bit wary.  The tutorial focuses not so much on testing the logic of the component or its state and how this impacts rendering, but rather what a snapshot of the component looks like, which has to be updated every time you have to make a change to the layout, however small.  Further research indicated that Jest indeed has a [snapshot-heavy favoritism](https://medium.com/javascript-in-plain-english/should-i-be-writing-snapshot-tests-47da13a62085), and when I read Justin Searls' thoughts on snapshot testing documented [here](https://kentcdodds.com/blog/effective-snapshot-testing), I decided to avoid them.  Long story short, Justin points out that snapshot tests don't really tell you why something fails and encourages developers to simply regenerate and re-commit snapshots whenever they fail. I didn't want to set this tone for TINNDÅRP, so I began to consider other testing frameworks to work with alongside Jest's assertion methods. The two other libraries I considered were [Enzyme](https://enzymejs.github.io/enzyme/) and [React Native Testing Library](https://callstack.github.io/react-native-testing-library/docs/getting-started).  

I was biased toward using Enzyme because, well, I was already familiar with it from working on React apps. Plus I had to move quickly with developing TINNDÅRP and use my time efficiency, so turning to Enzyme's familiar face seemed like the sensical choice.  As expected, however, additional considerations arose.  The Enzyme API offers two primary means of rendering components to test them: a `shallow` render which is quicker and allows limited testing of the rendered component's immediate children, and a `mount` render which is slower and costs more in efficiency but is a deep render that allows better testing of the component's children and nested children.  Unfortunately for TINNDÅRP, these rendering methods were meant to work in a pure React app context, not in a React Native context.  Configuring a React Native app to work with Enzyme can be done, but as [the Enzyme docs](https://enzymejs.github.io/enzyme/docs/guides/react-native.html) concede, it is tricky and difficult. After I followed the configuration advice in the Enzyme docs (and did some additional googling), rendering with `shallow` seemed to work just fine.  Rendering with `mount` however, only worked some of the time, and even worse, my test results were buried under a mountain of warnings.  Hundreds and hundreds of lines of warnings.  The common workaround, discussed [here](https://github.com/enzymejs/enzyme/issues/831), is to put a script in a test setup file that effectively cancels the console-logged warnings when Jest runs its tests.  This approach made me very nervous.  What if I missed other warnings, serious ones, that might arise in the tests?

This is when I began to investigate [React Native Testing Library](https://callstack.github.io/react-native-testing-library/docs/getting-started), which, unlike Enzyme, was designed with React Native in mind.  The major upside of using React Native Testing Library is that deep rendering is not a problem.  On its face, this seemed to solve every problem I might have with Enzyme.  Certain things gave me pause, however:

- At the time, I was anticipating having a Redux-managed state (in addition to some traditional component states), and the [Redux docs on testing](https://redux.js.org/recipes/writing-tests) assume Enzyme is available.  I wasn't sure if it was worth investing in learning a new API only to find out I couldn't make it work with Redux.  Some time-boxed googling didn't turn up anything helpful on this question.
- Unlike Enzyme, React Native Testing Library does not allow you to manipulate a component's state when setting up tests.  See [here](https://stackoverflow.com/questions/54104547/how-to-set-components-local-state-while-testing-using-jest-and-react-testing-li), for example. 

The inability to manipulate a component's state in React Native Testing Library seems rooted in a philosophy that a developer should write tests that simulate user interactions, followed by assertions on what a component renders based on those interactions. This approach should make tests less brittle and more "real-world." 

I can see the nobility behind this philosophy.  But questions kept lurking in the back of my mind: is a test, set up purely by user interactions, clear?  Does it require multiple simulated interactions to get to the punchline, diminishing clarity? The answer to these questions may be that if my tests simulate too many interactions, it's a smell that the component under test does too many things. But given the time constraints at hand, there wasn't time to test that theory with an experiential deep dive.  So what to do?

I ended up sticking with Enzyme, and using `shallow` rendering exclusively.  In exchange for saving time by sticking with a familiar face and for avoiding warnings caused by `mount`, I had to use non-intuitive Enzyme methods or esoteric component properties to get at the innards of shallowly rendered components.  For example, at one point, I wanted to test a `<FlatList />` which was a child in `<ComparingContainer />` component, to make sure the `<FlatList />` was rendering a list of homegrown `<ItemDisplay />` components.  Assume the code looked something like this, simplified for the sake of illustration:

```javascript
export default class ComparingContainer extends Component {

  constructor(props) {
    super(props)
    this.state = {
      commonItems: [{name: "box", name: "square"}]
    }
  }

  render() {
    return (
      <View>
        <View>
          {this.state.commonItems ? 
              <FlatList 
                data={this.state.commonItems}
                renderItem={({item}) => <ItemDisplay item={item} />}
              />
            : 
              null
          }
        </View>
      </View>
    )
  }
}
  
export default ItemDisplay = ({item}) => {
  return (
    <View>
      <Text>{item.name}</Text>
    </View>
  )
}
```
With only the ability to `shallow` render `<ComparingContainer />` and not deeply render children such as the `<FlatList />`, my Enzyme tests could not directly assert against the inner workings of what was inside the `<FlatList />`.  For my tests to give me some comfort that what I expected was there, I had to test other properties of the shallowly-rendered `<FlatList />` like so (inspired by Stack Overflow discussions like [this one](https://stackoverflow.com/questions/52017521/how-to-test-that-the-renderitem-function-returns-a-listitem)):   

```javascript
describe("<ComparingContainer />", () => {

  let wrapper
  beforeEach(() => {
    wrapper = shallow(<ComparingContainer />)
  })

  const mockData = [
    {
      name: "triangle",
      name: "diamond"
    }
  ]
  
  describe("the <FlatList />", () => {

  it("looks to commonItems in the state for data", () => {
    wrapper.setState({commonItems: mockItemData})
    const flatList = wrapper.find(FlatList)

    expect(flatList.props().data).toEqual(wrapper.state().commonItems)
  })

  it("its children are <ItemDisplay />s, and it passes to an <ItemDisplay /> an item in the data as a prop", () => {
    wrapper.setState({commonItems: mockItemData})
    const flatList = wrapper.find(FlatList)

    const itemElement = flatList.props().renderItem({item: flatList.props().data[0]})

    expect(itemElement.type).toEqual(ItemDisplay)
    expect(itemElement.props.item).toEqual(mockItemData[0])
  })

})
```

These tests told me two things: that the `<FlatList />` was looking to an expected set of data to render items (the `commonItems` in the state) and that when it rendered a child element, it was rendering an `<ItemDisplay />` and passing the expected prop to the `<ItemDisplay />`.  These tests, together with other tests on how the commonItems were manipulated in `<ComparingContainer />` and on how `<ItemDisplay />` rendered its `item` prop, gave me a fair amount of comfort that the rendered `<Flatlist />` would render what I expected to be there.  While this solution wasn't ideal and I wish circumstances had been otherwise, it seemed sufficient to me given the circumstance. 

----------
<p />

Best of luck if you are developing a React Native app as well.  I hope the discussions above were helpful if you are getting started and weighing your options.  It's really great to see an app come together.  Wishing you tons of fun in your journey.


