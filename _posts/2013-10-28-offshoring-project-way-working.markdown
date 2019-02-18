---
layout: post
title:  "OffShore Software Development: A Project or a Way of Working?"
date:   2013-10-28 20:07:08 -0400
categories: jekyll update
published: true
---

> This was originally posted on my previous blog from one of my experiences with a fantastic team in India.

I went to India with the assignment to ramp up a team on a new project.  I could have written a detailed spec, given them some very tight guidelines to work under, and told them that I had to approve every line of code.  Those have all been done.  I wanted to try something different...

<strong>An Experiment</strong>

Like most projects, this one started out as an experiment.  I wanted to learn about web development, and felt that I could replace many of our manually produced reports with a database and web-front end.  I installed a web development framework, plugged in some sample data and generated a web report that I could show to people.  The response was very favourable, so I decided to develop it further using <a href="http://en.wikipedia.org/wiki/Test-driven_development" target="_blank">Test Driven Development (TDD)</a> as much as I could.

Why TDD?  Partly because I knew it was the professional way to develop software, but more because - once you get into the habit of writing code with unit tests at the same time, developing any other way just feels dangerous, risky...like I am probably wasting my time because I have no clue if my code actually works.  Once you're hooked on TDD, there's no going back.

We could have started the project with a discussion on the architecture, database design, etc.  Had we known what we were building we probably would have, but we didn't.  So we continued to develop the system based on crude sketches with stakeholders, writing unit tests, making them pass and giving frequent demos. Eventually, we had a pretty cool looking prototype.

<strong>A Product or a Way of Working?</strong>

The developers in India were generally young and highly enthusiastic about developing code.  They were awesome, but sometimes over enthusiastic.  They had a history of developing quick solutions to complete current work items, without considering that the code may need to change again.  As a result, several recent requests for modifications to existing work had met with "It's difficult" or "not possible".  This pattern had to change.

I could have just sent them the code, a big document or a diagram of what I had done, but I knew this would not in any way enable them to continue building this project as we had started it.  For this project to have a long term future, I would have to teach them <strong>NOT</strong> about <strong>WHAT</strong> we had done, <strong>BUT HOW</strong> we did it.

<strong>Mob Programming</strong>

I'm not going to discuss the reasons, but at the time, only one developer on the team in India had access to the source code repository.  This was a great excuse to try a technique known as <a href="http://mobprogramming.org/mob-programming-basics/" target="_blank">Mob Programming</a> that is gaining more publicity on the internet and twitter.  Basically, the entire team solves one problem at a time, while they take turns entering the solution into a single keyboard while the display is projected on a screen.

Given the fluid nature of the project, we had to find ways that the team could: A) Quickly learn technologies they had never worked with before, and B) Learn new ways of working they had never done before.  The Mob programming format seemed perfect for this.

We started off the two weeks with a requirements workshop to establish a project vision, and where they wanted it to go.  The end result was an ordered product backlog of requirements that we could start on immediately.  We chopped off the top items, and went to work!

<strong>Jumping into TDD</strong>

First up was an addition the developers thought would be a one line change to the website - add a link to a given page.  How hard can that be?   Harder than we thought!  Some logic would be required to construct the link for different types of data.  Lucky for us, this was a fantastic first exercise for TDD!

I had the team define a new test class, and write their first test case which called a currently undefined function to generate one variation of the URL.  They did all of this without complaining!  It wasn't until they started writing the 'make it pass' product code that team started going down the classic 'TDD-newbie' path of trying to write more code than was necessary to make the test pass.  I gave them a verbal slap for that, and told them to only write what was necessary to make the test pass.  After one or two more slaps, the team quickly got into the <a href="http://www.jamesshore.com/Blog/Red-Green-Refactor.html">fail (red)-pass (green)-refactor</a> rhythm of TDD.  They got it fast, and it was a thing of beauty.

<strong>Getting Feedback... FAST!</strong>

We were all feeling great about a productive first day - the team had delivered new functionality on a product and in a language most of the team had never worked in before!  So it was with great pride that the team called in one of the key stakeholders to examine their results.

Surprise!

They had not added the functionality that the stakeholder expected - they were linking to a completely different page!  Luckily, a relatively easy fix was available that satisfied both team and stakeholder.  This served as a powerful lesson about obtaining regular feedback.  This is especially important when several stakeholders may be overseas.
<blockquote><em>The importance of frequent structured feedback INCREASES with the distance between the team and their stakeholders!</em></blockquote>
Over the next two days, we would get into more complex TDD scenarios. We would introduce the Model-View-Controller architecture of the web development framework. We would use principles such as <a href="http://en.wikipedia.org/wiki/Don%27t_repeat_yourself" target="_blank">DRY (don't repeat yourself)</a> to refactor out duplicate code.... and hold daily stakeholder demos.  I was very impressed with how quickly the team was learning these concepts, and getting work done at the same time - all in an environment that they had never worked in before.

Thing were going so well that the team continued to deliver content without me after I left early on Friday (I flew to Delhi to visit the Taj Mahal - future blog post!).

<strong>Breaking Old Habits</strong>

On Monday, I would move them away from the web development we'd been doing, to using their existing and more familiar development environment to add some back-end data parsing to existing functionality.  I was expecting the team do even better using TDD in their existing domain, because they should have been more familiar with the language and the type of work.

I was wrong - they struggled!

Moving them back to their more familiar environment caused them to revert back to their old work-flow of attempting to construct large solutions before writing a test.  The entire day was a battle of me reminding them to go back to their unit tests!  It didn't help that their development environment was extremely very slow, causing them significant frustrations when editing and executing tests.
<blockquote><em>It is far easier to teach new habits than to break old ones!  Budget time for that...
</em></blockquote>
<strong>The (Software Development) Environment Matters.. A LOT</strong>

Luckily, things did get better.  I made some tweaks to improve their development environment (namely give them a faster, offline one), and the team got back into TDD's rhythm of <a href="http://www.jamesshore.com/Blog/Red-Green-Refactor.html"><em>red-green-refactor</em></a> with their code.
<blockquote><em>A fast, responsive development environment is essential for frequent testing.  Pain and suffering during edits, compilations and tests promotes longer work cycles - and in longer work cycles, the first thing to be cut is writing test cases.
</em></blockquote>
<strong>Observations - Mob Programming, TDD, Learning
</strong>

The Mob Programming format was clearly a big factor in the success of the two weeks. The developers were quick to get into the habit of rotating who was at the computer.  It was a pleasure to watch the more senior developer tutoring the more junior developer at the keyboard.  The energy in the room was high all day long, only dropping off with an hour or so to go in the day - as a result, I usually ended the day about an hour earlier than they normally would have headed home.

One of the developers remarked that this was excellent for learning, but as the project continued it would be faster if they each worked separately.  I partly agreed, but with a big caveat.  This project had been about continuous learning from day one, and would likely continue to be so.  For our customers, it is only by continuous learning that we can hope to build a product that they really want.  Sacrificing the flow of learning for the flow of code production would likely be a <a href="http://en.wikipedia.org/wiki/False_economy">false-economy</a>.  He agreed - and the team agreed to continue with the format.

The developers commented that they were particularly impressed with the test-driven development technique.  The most senior designer said that everyone had told him that writing tests took longer, and more overhead (common misunderstandings), but he had learned otherwise this week.  He learned that it did not slow him down at all - in fact, it took less time writing code with TDD, as he would spend less time debugging work and have an automated set of unit tests at the end to show for it.

This was music to my ears.  As I reminded them throughout the week - we may not have chosen the best solutions, and it would be up to them to decide how to do things in the future.  The future success of this project  would not be determined by the strength of the initial design, but by the ability of the developers to evolve the system as they better learned the needs of their customers.  Quick learning, test-driven development, a responsive <a href="http://en.wikipedia.org/wiki/Software_development_environment">SDE</a> and frequent stakeholder feedback would all be key to that.

Now, I'm under no illusions that the job was done after my two weeks in India - these guys will require lots of ongoing support, but I had come to understand that for this team was to be successful - what we would have to offshore was not so much the implementation of a particular product, but a way of working and supportive environment that would put them in a position to develop a great product for their customers.

Jamie.