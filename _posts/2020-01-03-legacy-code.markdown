---
layout: post
title: "A Cool Thing I Learned from Michael Feathers' Book `Working Effectively with Legacy Code`"
date:   2020-01-03 12:00:00 -0400
categories: coding
---

I recently finished Michael Feathers' *[Working Effectively with Legacy Code](https://www.oreilly.com/library/view/working-effectively-with/0131177052/)*.  It's an impressive piece of work full of tips about...well...working with legacy code. Attempting a summary won't do the book justice, but to give it a go anyway:

Legacy code, under the book's definition, is any code that isn't supported by tests.  Not having tests means that when a programmer has to refactor legacy code, or add a new feature to legacy code, things become extremely difficult.  The programmer is forced to make changes to the code base, but the programmer has no easy or reliable way of knowing if the changes will cause bugs anywhere else.  To make matters worse, the legacy code that now must change was likely written in a way that makes testing difficult; a test suite cannot simply be slapped on somewhere.  For example, there could be long "monster" methods or classes, methods or classes with too many responsibilities, and/or methods or classes tightly coupled to dependencies.  Any of these obstacles makes the addition of tests mind-numbingly painful. Ultimately, the likely outcome of having to change code without tests is that the the programmer, under time pressure, follows patterns that the programmer sees in the current code base, trepidatious about making improvements.  If the patterns already in the code base are not healthy, this repetition will contribute to the degradation of the code base over time.  

The mission of *Working Effectively with Legacy Code* is to halt this downward spiral that typifies changing legacy code.  Michael Feathers fully recognizes that programmers are often stuck between a rock and a hard place when approaching legacy code, because its hard to change code without tests, but to put tests in place, the code has to change. To help alleviate this conundrum, much of the book is dedicated to describing techniques for changing legacy code *just enough* to get tests in place while remaining reasonably confident that bugs are not being introduced.  These techniques focus primarily on breaking dependencies, including nudging the code into more modular, independent methods or classes which lean on dependency injection.  Once a chunk of code is separated and dependency injection replaces reliance on concrete classes, it is easier to put tests in place for that chunk of code by replacing the injected dependencies with mocks or other helpful test objects.  Then, once the tests are in place, the programmer, and those who must change the code base after, can move forward with changes more confidently and with greater protection against introducing bugs.  

My favorite technique described in *Working Effectively with Legacy Code* is what Michael Feathers calls "Parameterize Constructor." I think it's my favorite because (a) it's really clever yet simple, (b) there's a "wow" factor because I never would have thought of it on my own, and (c) it offers a way to inject a dependency for testing purposes without changing any method signatures that clients are relying on.  Also, I personally wrestle a lot with how to write testable objects that instantiate other objects while not relying on concrete classes, and this technique offers an interesting way of solving this dilemma when having to update code whose pre-existing behavior can't change.

For the purpose of recapping the Parameterize Constructor technique, let's assume we're looking at Java code for a crossword puzzle game with just one player.  We have a CrosswordPuzzleGame class, and an instance of this class depends on an instance of a Player object.  Right now, the CrosswordPuzzleGame class has a concrete dependency on the Player class, creating a Player instance in its constructor, like so:

```java
public class CrosswordPuzzleGame {
    private Player player;

    public CrosswordPuzzleGame() {
        this.player = new Player();
    }

    // ...more stuff...
}
```

Clients who rely on this class have written their own code assuming all they have to do is call `new CrosswordPuzzleGame()`.  The problem we face here is that we want to somehow try to get the CrosswordPuzzleGame class under test to sense and verify how an instance of the class behaves with respect to a Player.  Unfortunately, with the code as-is, we have no way of injecting a mock Player object to assert against what happens to it.  We have to change the code here a bit, but how do we do that *just enough* to minimize the risk of introducing bugs?  

The Parameterize Constructor method has us first create a new constructor where we pass in our dependency as an argument.  It's a case of writing the code we wish we had.  So now our code looks like this:

```java
public class CrosswordPuzzleGame {
    private Player player;

    public CrosswordPuzzleGame() {
        this.player = new Player();
    }

    public CrosswordPuzzleGame(Player player) {
        this.player = player;
    }

    // ...more stuff...
}
```

But now this code looks weird and confusing.  Future programmers will be lost wondering why we have two public constructors.  Clients may not know which to use if they know both constructors are available.  Right now, we may not be able to change having two constructors (especially since clients rely on the first), but we can bring them closer together with a small tweak to the original constructor so that it simply calls the new one:

```java
public class CrosswordPuzzleGame {
    private Player player;

    public CrosswordPuzzleGame() {
        this(new Player());
    }

    public CrosswordPuzzleGame(Player player) {
        this.player = player;
    }

    // ...more stuff...
}
```

With this small tweak, the behavior of the CrosswordPuzzleGame hasn't changed, and clients can continue to rely on their calls to `new CrosswordPuzzleGame()` without passing in any arguments.  Further, keeping in mind our goal of getting the CrosswordPuzzleGame class under test, we can now write tests that, when instantiating a CrosswordPuzzleGame object, call the second constructor and pass in a mock Player object to test the effects that CrosswordPuzzleGame has on a Player object.  

And that is the "Parameterize Constructor" technique.  At the end of the day, it allowed us to change the code in a simple way to enable testing without changing any of the code's behavior and without causing headaches for clients.  "Parameterize Constructor" strikes me as such an elegant and simple solution, and I'm grateful to Michael Feathers for sharing it.  
