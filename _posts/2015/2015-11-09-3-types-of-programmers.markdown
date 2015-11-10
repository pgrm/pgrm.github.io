---
title: "3 Types of Programmers and Why Programming Should Be Easy"
tags:
  - Other
---

Few days ago I listened to Scott Hanselman's 500th podcast (congratulations on that), [You don't know JS with Getify (Kyle Simpson)](http://hanselminutes.com/500/you-dont-know-js-with-getify-kyle-simpson).

# Not everyone needs to know, how to drive a stick

I had mixed feelings about that podcast and was thinking for a long time about Scott's analogy of driving a stick or driving an automatic. I've heard it from him before and I did agree, but now I came to a different conclusion. I used to drive a car with manual transmission, as I believe, most europeans do. I have an automatic now, and love it for most occasions. But sometimes (mostly for hills) the automatic just sucks and I override it, shifting the gears myself. That's me and I've been driving since 18 quite a bit. But I got the automatic in the hope, my fioncé would start driving more. She has her license for over 7 years but never really practiced and when she's coming to an intersection where she has to reduce the speed, shift, indicate turning and pay attention to other cars, ... let's say it's a bit too much. So I thought, without shifting it's one thing less to think about.

But lets take this analogy a bit further. Except shifting gears, I can change the lightbulbs on my car and that's pretty much it. I managed to change my tires once, with some success, but except that I'm pretty stuck. As it happens, I have a cousin, which learned to be a car mechanic. There are so many more things he understands about the car and can fix himself. On the other hand, I gave him my old laptop few years ago, as I knew it would be better than his last one. When I visited him a year later, I found out, he wasn't using it for all this time, because he didn't know how to log in. I gave him the laptop not realizing he wouldn't be able to reinstall Windows on it. It took me a couple of drinks at his place, before we were going out, to set up a fresh installation with his own user and password, that was not possible for him alone.

# 3 Types of Programmers

When I talk about programming with less technical friends, I coin the term *3 Types of Programmers*. These are:

## Type 1:

Type 1 is easy, these are people whom you'd think of when talking about software developers. I think these are the people Scott Hanselman and Kyle Simpson were thinking about when talking in their podcast. The profession of these people is of technical nature and IT related. One would expect them to know at least their preferred stack in and out and also know some (or many) things around it. This is the group I count myself to.

## Type 2:

This is where it gets tricky. If type 1 are all software developers, who is than a type 2 programmer according to me? Well, those are the often forgotten ones. My fioncé and her course mates would fall in this category for instance. They study/studied cartography. This has nothing to do with IT. Yet, most of them have to write code for their master thesis. Is it to implement an algorithm or create an application. Those people didn't know what a Version Control System (VCS) is, and what it is good for (comparet to Apple's Time Machine and Dropbox).

The other part in this group are people who did finish an IT related degree, but are much more interested in the domain problems of their customers. Many former colleague of mine would fit here. Back than, I didn't understand it well enough, so it was at times really frustrating to work with them, since their code was sometimes unnecessarily inefficient, as they cared much more about the domain solution than the technical implementation. After all, the implementation was just an implementation detail.

These are the people who don't want to have to understand how a framework or abstraction works, as long as it helps them solving their problems. So why would we force them? They are much better at what they are doing, at what they like, want and chose to do, than writing clean and efficient code and making sure that the underlying interpreters or compilers understand it correctly to make it extra efficient.

## Type 3:

Well, there is the possibility that one isn't interested too much in the technical, nor in the domain aspect. They just want to know what needs to be implemented and don't care about the big picture. This is fine as well. Probably it's easier for them to stop thinking about work and actually have quality time with people and hobbies which matter to them.

## What about Type 4?

I said it's 3 types, didn't I? But mathematically, looking at those possibilities there should be also type 4 as a great combination of type 1 and 2 and the opposite to type 3. So much for combinatorics, in praxis, I haven't met them (yet). I believe that people do focus on technical aspects or domain. This doesn't mean that you can't know both, but there will be one thing you focus on more than the other. So if you think you're type 4, give it a try and think again ;)

# Why programming should be easy

Going back to type 2, those for whom programming is just a tool. Actually the computer is the tool, but, unfortunately, nobody wrote the program they'd need yet, so they need to write it themselves. And specifically to people who have not studied an IT related field. Some of these were even suddenly surprised in their master studies with the demand for writing code. I thought for quite some time, what the best approach for people in these situations would be. I've more closely thought about 2 ways:

1. They should learn to code, before they can write their thesis. Well, they tried it. They had some classes on coding, but those were a mixture of not useful and too advanced. They were doing for instance web applications, by copying their professor (and his code wasn't that great either). In the end, they need to implement their own algorithms or applications in Python or Java, based on what they've learned before, when they copied a PHP/MySQL application. Some have troubles understanding the difference between classes and objects - which is tricky, unless you've had a class on OO-Programming. So learning to code is clearly not going to work for them.
2. They shouldn't code at all. This might work for non technical studies, but doesn't for technical ones. A friend of mine, who is doing a PhD in Physics at KTH, once stated, all engineers should know how to code. Either you need to run simulations, analyze your data or create and implement a model. All of these will be much easier with a computer, if you're able to tell it, what you want it to do. Hypothetically speaking, there could be a kind of symbiosis between IT students and other technical students, where the IT students would implement the ideas of everyone else, but let's face it, this isn't going to work.

After the, above mentioned, podcast I thought, actually no - Just like I wouldn't know how to fix a broken car and my fioncé is overwhelmed when driving a stick, others don't need to know how JavaScript works differently on different browsers or even different Chrome versions. It is a black box and it is good so, since you can't know everything anyway.

My former colleagues, who would mostly refer to themselves as consultants, could write code. And in some languages, they were much much better at it than me. They had clever ways to prevent stupid mistakes in C and C++. For instance instead of

    if (i == 1)

they'd write

    if (1 == i)

This way, the compiler would immediately throw an error in case they forgot an `=` sign, as you can't assign anything to `1` (on the other hand `if (i = 1)` would be correct). However, this was much more based on experience and former mistakes, than on the understanding of the underlying technology, framework or libraries.

When I was analyzing a C# code, which happened to be extremely slow, I discovered the use of `List` in a way which would have been much better fitting for a `HashSet`. When the application was new, the amount of items in the list was small, so nobody noticed anything and everyone kept using it. Once it grew to tens of thousands of items (maybe event hundreds of thousands) the application became unusable on older hardware. The most common response from everyone on the team was something like, *well C# is slow, what did you expect?* Once I switched to use a HashSet the application got quite a boost. I do think, they remember since than, what a `HashSet` is good for, but they also didn't seem to care too much for the **why**. As in *Why is there this difference?* - Again, quite frustrating back than, but in retrospect, they understood the domain problems really well and, so why would they need to know the different types of collections, or other classes, which exist in .Net and other open source projects written for .Net? `List` had all the necessary methods, so why wouldn't it do?

# How programming could be easy

There is already a solution for programmers who prefer domain problems over technical ones, it's a Domain Specific Language (DSL). These days it doesn't even have to be a language. A high level framework with a lot of templates which allows the definition and implementation of business use cases could already work. Visual Studio does the first step by hiding advanced type members by default. But more should be done in this direction. Before I left my former work, I had worked on enhancing the common framework, everyone was using to create new business logic, and tried to make it as easy to use and as efficient as possible. Shortly before leaving I've also added templates to add new code in Visual Studio with ease. I think this is the best way to go about here. These people understand code and abstract thinking and have at times even over 20 years of experience. They just don't want to learn new frameworks over and over again, that's what a DSL or a high level framework can abstract from them.

Programming is hard, and Jeff Atwood has a lot of posts on his [blog](http://blog.codinghorror.com/) devoted to this fact. Especially for the young engineers, which need to write code for few and very specific use cases, it is unreasonable to expect them to learn programming for years. The wrong assumption here would be, that they want to be programmers - no, they don't. They want to have a tool which works or a model they can test. So instead of telling them how hard it is and that they need to understand the machine code of their Python interpreter before they can start printing a line on the screen, maybe we should look for a different approach. I don't know how such one would look like, since it seems to me true, that the only way to learn programming is by doing it over and over again for years. But maybe that's where I'm wrong, as those people don't want to learn programming, they just want to use computers. Maybe we should have a way to teach people how to change their motor oil, without expecting them to know how to change the breaks first.

------

I hope to get some more interaction with this post. What do you think? Should we make it easier for others to write code without the need of understanding everything, and how could this look like? DSL for every little thing or something else? Or is it necessary for people to first go through thousands of hours of experience and always keep learning?

Looking forward to your replies!
