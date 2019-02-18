---
layout: post
title:  "Lessons Learned in Failure to Drive Change"
date:   2013-12-16 20:07:08 -0400
categories: jekyll update
published: true
---

> Aside: While this post was written earlier in my career and trends have moved on from the early adopters of agile software development, many of the same principles hold true today for the next waves of change - be it technical or process oriented.

This was my story of failure and lessons learned in trying to drive changes in a large organization.

It started out with me as a developer working in a large telecom company.  I was highly rated by management.  I was great at completing my work before their deadlines, and great at following all of the corporate processes, like noting all of non-automated tests I had run on my code exactly once.

Yet, I was extremely unhappy.  Our development cycle was measured in years, and I knew that if I was going to improve my skills, I would need to move to a company with a much shorter development cycle and a greater culture of learning.

<strong>Then along came something called 'Agile'...</strong>

Or was it 'Scrum'?  We were all given 4 days of instructor led training - tons of fun!  I was easily the most enthusiastic participant.  This stuff made so much more sense than the way we had been working!  Everything in Scrum seemed to fit so well into a nice package that your project had to be a huge success if you followed all of the rules.  They even gave me a certificate of some kind!

The week after they would throw the Agile equivalent of 'the works' at us.  We were collocated into a single room.  All of our work was put on a board with stickies and we began having all of the prescribed meetings.

<strong>Not everyone was thrilled about this...
</strong>

We had several pissed off team members... nobody stopped to ask the team how they actually felt about the changes, and even if they were asked - what could they say?  Every level of management - up to executives, would come into our room on a daily basis to tell us how important what we were doing was for the company.  We were pioneers doing this Agile thing (which of course didn't involve these managers)!

We hit the ground running - largely thanks to two experienced Agile coaches that set us on the right track.  Within a few weeks, we had a better shared understanding and more work completed on the project than we had accomplished in the previous 6 months. We were cranking out code like never before!

<strong>Many learn this the hard way: The cost of hiring an experienced Agile Coach is far less than the cost of NOT hiring one!</strong>

> Aside: The key word above is experienced. Since this post was written, the Agile coaching field has become highly commoditized with certifications and processes (e.g. SAFe) trumping experience and know how. Be sure to check references.

That wasn’t enough though.  We were told that we had to start doing things like ‘automated testing’, ‘continuous integration’ and using something called ‘test driven development’.  None of us had ever done these things before.  We barely knew what they were, but our management was convinced that these things needed to be done.  "I read about TDD online, and it seemed like a great idea... you guys must do it now!"  I jumped on the bandwagon as well - pushing many of these practices, while starting several 'pet projects' that I would push my team to adopt.  Unfortunately, much of it was on the side and adoption was low.

At the same time, were getting pushed to keep this metric called ‘velocity’ increasing every sprint.  Someone told our management that this number should be increasing continuously or our Scrum Master was failing.

<b>Have you ever tried to change a flat tire while doing 100kph (~62 in American) on the highway?  We were trying to change all 4!
</b>

As you can imagine, a couple of things happened:
<ol>
	<li>The team began to burn out.  Stress took a huge toll on the team members.  So much so, our Scrum Master would eventually have to take stress leave.</li>
	<li>Product quality suffered.  Sure, we delivered a lot of product quickly, but the product had yet to hit the market and it was clear our technical debt was mounting as were our defects.  This way of working was NOT sustainable.</li>
</ol>
<b>Scrum had shown us that developing software sustainably required development skills that our team did NOT possess at the time.
</b>

I became frustrated with the slow pace of change, so when the opportunity came up, I jumped at the chance to become...

<i><b>An Agile Coach pushing Technical Change!
</b></i>

You'd think I would have learned my lessons with my failures to drive change as a developer.  As an Agile coach, I was brought in to push practices such as TDD, unit testing, automation, etc. across the company.  So, push I did!

We did a lot of preaching, and we finally brought in an experienced expert to teach all of our developers Test-Driven Development.  He was one of the Agile co-founders, and someone who I can highly recommend for teams looking to <a href="http://www.renaissancesoftware.net/">get started with TDD in embedded systems</a>. For those with an open mind, in three days he would take a developer who had never written a single unit test to a point where she could begin using basic TDD in their own work.  I learned a ton from him, and still count him as a key influence in my career.

<strong>Unfortunately, back in the real world...</strong>

Few of our development teams picked up on these practices.  Who could blame them when they were being pushed to work evenings and weekends to complete their existing work?

It was an odd situation.  I recall one meeting where a director was begging his team to write unit tests, while their front-line manager pushed them to get their work done 'at all costs', tests 'at the end'.  Sadly, I understand this situation is quite common in large companies attempting Agile transformations.

<b>Once again, we were trying to make too many changes without providing an environment for the change to take place.
</b>

We thought we had one success... though it turned out to be a failure.  We had a team that was given a side project to write unit tests and refactor existing code.  That's all they did - write tests and refactor code.  I even pair programmed with their team members for a few sprints and had a blast!  We submitted tens of thousands of unit tests, and a lot of refactored code - all without ever breaking anything.  We thought it was a big success!  But alas, it was a big failure.  As soon as the core development teams began working on the test covered code, they ignored the tests, and turned them off rather than work with them.

<b>Successful side projects can make us feel like we're seeing real improvements, but until we are using our new practices on real customer driven work, we can't consider our improvements successful.</b>

<b>Back to the Code... :)
</b>

I was getting tired of preaching.  I wanted to be involved again.  I saw an opportunity to make a huge improvement on an internal system and had also been playing around with database backed website design for a pet project.  This was an opportunity to put that knowledge to work...at work.  I threw together a quick prototype, and after receiving some positive feedback, I was cleared to spend my time at work on this new project.

I wanted to do this one properly - applying the development skills I had learned, while using a popular open source web framework.  I communicated with my stakeholders daily to ensure that I was building what they wanted.  In the end, the project was such a success that, in typical global company fashion, I was told to hand off the project to a larger team in India while I went to work on something else.  So, I flew to India and spent a few weeks <a href="http://mobprogramming.org/">mob-programming </a>with them using test driven development, continuous integration all the while displaying things I head learned from "Uncle Bob's" <a href="http://www.amazon.ca/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882">Clean Code</a>.  The difference was, this time, I was able to sit down and do it with them... and they actually picked up on it.  I flew home and (three months later), they continue to improve their development skills.

Now, I still feel like a software development novice - and probably made lots of mistakes, but I had a blast passing on my knowledge to the team in India.  In many ways, it has felt like the most successful software project I've worked on to date.

<strong>Biggest Lesson: How you go about making your change is as important, if not more important than the change itself.</strong>

Being prescriptive, telling people what you want them to do, only works if you are prepared to dive in and work with them to implement the changes you're proposing.  If you aren't prepared to be an active participant in the change you're proposing, you have no business proposing it.<b>
</b>

As an aside, there seems to be a growing acknowledgement in the world of Agile transformation and Change Management that successful change initiatives must heavily involve or even be driven by the change recipients.  There is also a growing number of strategies for driving changes incrementally, and avoiding some of the change burnout that our teams ran into above.  Jason Little's <a href="http://leanchange.org/">Lean Change </a>is a refreshing work describing some strategies for doing both.

<strong>Also.. Technical practices matter...big time! </strong>

You <em>should</em> be thinking 'Isn't that obvious?', but you'd be surprised how many coaches and trainers try to sell Scrum or other Agile practices as something that can be used with existing technical practices.  In my opinion, <a href="http://blog.8thlight.com/uncle-bob/2013/12/10/Thankyou-Kent.html">the XP guys had it right all along... </a>

If you're developing software, your source code is the single most important thing you produce.  Most traditional organizations make up for their poor code with a massive <a href="http://en.wikipedia.org/wiki/Quality_assurance">QA </a>effort and lengthy defect backlog, but Scrum brings these problems significantly forward.  It takes strong leadership, and highly motivated developers to avoid the temptation to puke at what Scrum is showing you, and say it was 'better before'.  As a result, companies moving to Scrum or other Agile practices must be prepared to invest serious time and money in their developer's skills.  On the flip side, developers must be willing to spend significant time improving their skills as well (see <a title="Software Developers: The Road to Professionalism" href="http://www.jamielongmuir.com/2013/11/09/software-developers-as-professionals-its-a-long-road/">Software Craftsmanship</a>).  If they're not, you have the wrong developers...