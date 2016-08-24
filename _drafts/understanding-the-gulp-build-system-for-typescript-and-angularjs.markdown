---
title: Understanding the Gulp Build-System for TypeScript and AngularJS
date: 2015-11-10 00:00:00 Z
tags:
- TypeScript
- AngularJS
- Gulp
---

> Last week, I've published the first part of this series, *[Getting started with Visual Studio Code, Atom or Sublime Text 3]({% post_url /2015/2015-11-03-getting-started-with-typescript-and-angularjs %})*. If you haven't read it yet, you should go and check it out, before you keep on reading here.

First question you might have - why should I care about the underlying build system, as long as it works? Great question! There are plenty of analogies I've heard from different people, but I don't think they do it justice so let me try mine:

I tend to say, there are [3 types of programmers]({% post_url /2015/2015-11-10-3-types-of-programmers %}):

1. Those, who are interested in technology. Those, who want to make an algorithm as efficient as possible and use the latest and greatest fitting technology to solve a given problem. These people usually also understand deep details of the technology they use and want to understand why things work the way they do. (I count myself into this group, and I don't like people to call me "just" a programmer).
2.  Those, which are interested in the domain. I can't really talk about these people, as I'm not them. I've worked with people like that, and some of my friends would be counted in this group. They might not be great programmers, but they understand the problem domain better than anyone else. They know what a solution should do and they are capable implementing it in a programming language, which the've mastered before. The solution might not be the most efficient one, but it works and does exactly what they (or their client) want it to do. (Many of these people are also consultants which create the birdge between business people and developers.)
3. Those which are neither. Yes those people also exist, and that's fine. They have other passions and interests, while still being able to do the work, they have to do.
4. If you want to read the whole story, try my other article - [3 Types of Programmers]({% post_url /2015/2015-11-10-3-types-of-programmers %})

So, obviously you don't need to know how the underlying build system works. But maybe you want to have your own application in a different environment and than you can learn from it a lot. Or, you want to tweak and improve it, than you of course need to understand it first. I, myself, took this generated code and build system to design a gulp build system for a NancyFX web application which can be build with VS 2013, VS 2015 but also with Visual Studio Online. And once .Net Core comes out, I should be able to build the whole application in a Linux or OS X command line. That makes my life of automation much easier, so I had to go through all the code provided here, to learn how a great gulp system could be created.
