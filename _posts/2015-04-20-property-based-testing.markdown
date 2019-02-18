---
layout: post
title:  "Uncovering Defects with Property Based Testing"
date:   2015-04-20 20:07:08 -0400
categories: jekyll update
published: true
---

<em>Note: I originally published this piece in March of 2015 to the former blog of BoldRadius Solutions (later acquired by <a href="http://www.lightbend.com">Lightbend</a>). I am re-publishing it here with permissions for archival reasons.</em>

It’s been a few years since test-driven development changed the way I come think about software development. Testing and verification became a first and foremost consideration, while the practice of adding or changing functionality without having a way to validate it came to be seen as something outside the realm of professional software development.

While I still strongly believe in TDD’s benefits, experience has taught me that it’s not without its challenges. Maintaining tests has a cost. Despite writing tests while we code, corner cases are not always identified. As the complexity of our code increases, so does the possibility that we’ll fail to account for a key scenario or that a legacy function we’re using won’t behave as expected.

In a recent project, we utilized property-based testing as a means of mitigating these challenges - in particular, providing greater coverage with fewer tests and the ability to discover hidden corner cases. This created the perfect opportunity to explore how property-based testing compares with the more well known xUnit style of tests.

While this post will focus on Scala and ScalaCheck, which was inspired by the Haskell library QuickCheck, comparable property-based testing frameworks are available for most major programming languages out there. These principles can be applied to any language or domain.
<h2><strong>Traditional Unit Tests</strong></h2>
With traditional xUnit-style tests, a set of pre-defined inputs are fed into a function and a set of concrete outputs are specified as the expected result. If the output doesn’t match the expected result, an error is thrown and you know you have some code to change.

https://gist.github.com/longmuir/e5ce4b4f2b8167b6d64d#file-scalacheck-blog-1

A common TDD strategy is to start with the simplest test scenarios, work your way to developing the core functionality, and then add additional tests to cover error and corner cases - <em><strong>at least the ones you know about!</strong></em>
<h2><b>Generalized Assertions</b></h2>
<span style="font-weight: 400;">Property based testing, on the other hand, starts with generalized statements about how a function should behave. Rather than testing how a function will respond to specific inputs, we try to reason about general behaviour over </span><b><i>ranges of inputs</i></b><span style="font-weight: 400;">! </span>

<span style="font-weight: 400;">For example, in testing a function that computes the square root of a number, we might assert that for all integers, the result will be less than or equal to the original input value.</span>

https://gist.github.com/longmuir/20bc829a55e4dc364097#file-scalacheck-blog-2

<span style="font-weight: 400;">Is this a correct assertion? Let’s find out! When we execute this property test, we get following:</span>

https://gist.github.com/longmuir/586287e996d44e57dfe3#file-scalacheck-blog-3

<span style="font-weight: 400;">This tells us that after the test passed 4 times, it failed with an input value of -1 (more on what 30 shrinks means later). We can then decide how to handle this case - Did we really understand how our function worked? In this case, obviously not.</span>

<img class="aligncenter wp-image-716" src="http://www.jamielongmuir.com/wp-content/uploads/2016/12/scalacheck-equation-1-300x30.png" alt="scalacheck-equation" width="620" height="62" />

The challenge in property based testing is to come up with properties that can be proven ove ranges of data. These proofs can be seen as analogous to mathematical statements about properties of formulas.

<span style="font-weight: 400;">Or for functions...</span>

https://gist.github.com/longmuir/7dae9091de49e8b5fdaa#file-scalacheck-blog-4

<span style="font-weight: 400;">Property tests could also involve performing one action, then the performing the opposite action - known as “round trip testing.” For example, with a heap data structure, we may assert that deleting a heap with one element should always yield an empty heap. This should happen regardless of the data type we insert into the heap. Doing this provides some validation over both the insertion and deletion methods.</span>

https://gist.github.com/longmuir/59309df6ae41f7686b21#file-scalacheck-blog-5

<span style="font-weight: 400;">This can progress into more complex scenarios, such as inserting more elements, dealing with duplicates, sorting, combining - all with a random collection of data. It can also be useful to compare two different implementations of the same function to see if they are really the same.</span>

<span style="font-weight: 400;">Even very weak assertions (e.g. “will never return None”) made over a broad range of data are often very effective at revealing corner cases that we may not have considered when we designed our solution.</span>
<h2><b>ScalaCheck Mechanics</b></h2>
<span style="font-weight: 400;">ScalaCheck tests centre around the forAll method, which takes a function with arbitrary input values and returns a Boolean. ScalaCheck includes built-in generators that provide arbitrary input values for common types such as Int, String, List, etc. For other types, you can define your own generators, which are passed into forAll as an argument. </span>

<span style="font-weight: 400;">One of the challenges of property based testing is coming up with appropriate generators. Luckily, Gen is a Monad, which means we can use the power of Scala’s ‘for’ expressions to build custom generators like the following one.</span>

https://gist.github.com/longmuir/702d4bcc3d901c6ff9c5#file-scalacheck-blog-6

<span style="font-weight: 400;">It’s also possible to limit the range of data in a generator, using a for-expression with a condition, the suchThat operator or Scala’s implication operator.</span>

https://gist.github.com/longmuir/4fac58e918ba4c506c02#file-scalacheck-blog-7

<span style="font-weight: 400;">Limiting techniques should be used with extreme care, as a filter that is too restrictive may end up filtering out too much generated data, which end with an ‘undecided’ property (considered a failure).</span>

<span style="font-weight: 400;">As mentioned, ScalaCheck supports many test frameworks and styles. The following shows an entire class structure using WordSpec. It puts a lot of what we’ve discussed together and shows the creation of a nested generator with dependent fields.</span>

https://gist.github.com/longmuir/b87b5eee3dba4459d823#file-scalacheck-blog-8
<h2><b>Shrinking</b></h2>
<span style="font-weight: 400;">When we get into testing large data structures over thousands of values, it can be challenging to determine why an example failed. To aid with this, ScalaCheck applies strategies to reduce the problem data to the simplest inputs that produce the same failure. This was evident in the “30 shrinks” comment in the failure message above. Shrinking strategies are provided for the most common data types, and again you can implement your own for custom data types.</span>
<h2><b>More than Unit Tests</b></h2>
<span style="font-weight: 400;">While I’ve been discussing property testing in the context of unit testing individual functions, similar strategies are employed to test much larger systems - including parallelized and distributed systems that use randomized testing means of find anomalies. John Hughes gave a great talk on using property based tests to validate systems with random data: </span><a href="https://www.youtube.com/watch?v=zi0rHwfiX1Q"><span style="font-weight: 400;">John Hughes - Testing the Hard Stuff and Staying Sane</span></a><span style="font-weight: 400;">.</span>
<h2><b>Why are Property-Based Tests Awesome?</b></h2>
<span style="font-weight: 400;">In our experience, it is rare that a handful of property-based tests do not catch a scenario had not been considered even after writing several unit tests. As discussed, these are potentially catastrophic defects that could bring down your application.</span>

<span style="font-weight: 400;">Property tests encourage a different way of thinking about problems. We don’t just try to enumerate common scenarios (though this is useful too), but we try to really understand how a system behaves in the general sense - what are we really trying to accomplish? For example, rather than discussion “f(4) returns 12”, we discuss the transformation that f would apply on its input. This leads to richer discussions, and greater test coverage with fewer tests to maintain.</span>
<h2><b>What to watch out for?</b></h2>
<span style="font-weight: 400;">As the complexity of generators and property tests increases, so does the possibility that the same error in logic may be repeated in both your function and generator/test code. As input and outputs are not in plain site, these errors may not be as obvious as with hard coded unit tests. Generator code must also be maintained, and thus complex generator code can become a liability in much the same way that complex product code can be. </span>

<span style="font-weight: 400;">When browsing API documentation, I often find myself skipping over the descriptions (after all a well named function shouldn’t require a lengthy description) and going straight to a concrete examples to see how a function should be used. If you believe - as I do, that good unit tests should act as executable documentation for your code, then unit tests involving concrete inputs are still an important part of your executable documentation.</span>

<span style="font-weight: 400;">Finally, our experience has been that when property tests fail - a good old xUnit style test is more effective at debugging and diagnosing the exceptional case than rerunning the same property test that likely only fail sporadically. </span>

<span style="font-weight: 400;">For those reasons, I would advocate having property based testing as a compliment, rather than a complete replacement for traditional unit tests. Thankfully, all suites that support property-based testing also support traditional unit testing, so it’s very easy to do both!</span>
<h2><b>In Closing</b></h2>
<span style="font-weight: 400;">Property-based tests are a fantastic complement to TDD and traditional x-Unit testing. They help you cover significantly more input combinations with fewer tests, while helping you discover corner cases that you may not have considered in your initial design or test scenarios. They provide a powerful guard against unexpected behaviours while documenting behaviours that help us better understand how our functions work in the general sense.</span>
<h2><b>Further Reading</b></h2>
<ul>
	<li>ScalaCheck.org: <a href="https://www.scalacheck.org">https://www.scalacheck.org</a></li>
	<li><span style="font-weight: 400;">ScalaCheck User Guide: </span><a href="https://github.com/rickynils/scalacheck/wiki/User-Guide"><span style="font-weight: 400;">https://github.com/rickynils/scalacheck/wiki/User-Guide</span></a></li>
	<li><span style="font-weight: 400;">ScalaCheck: The Definitive Guide: </span><span style="font-weight: 400;"><a href="http://www.artima.com/shop/scalacheck">http://www.artima.com/shop/scalacheck</a></span></li>
</ul>