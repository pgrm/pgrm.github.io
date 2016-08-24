---
title: How To Stop a Storyboard
date: 2012-08-12 08:50:00.004000000 Z
tags:
- XAML
- C#
- ".Net"
layout: post
modified_time: '2012-08-12T20:18:26.667+02:00'
blogger_id: tag:blogger.com,1999:blog-8459046186574607112.post-4057643004216424340
blogger_orig_url: http://emptycode.blogspot.com/2012/08/how-to-stop-storyboard.html
---

I have an application with a storyboard, which turns a icon that indicates that the application is currently busy. Because I don’t have any chance to know, how long the application will be busy, the RepeatBehavior on the application is set to Forever and I have to stop it manually.

OK, seams to be easy, the storyboard has everything needed to stop it, a Pause-, Stop- and Remove-method, but somehow, nothing works. After some time I detected one line in the Output-Window of Visual Studio:

    System.Windows.Media.Animation Warning: 6 :
    Unable to perform action because the specified Storyboard was never applied to
    this object for interactive control.; Action='Remove'; ...

After some research the answer was very easy, when you start the storyboard with the Begin-method, there is a second parameter of type Boolean, **isControllable**. This has to be set to true, afterwards all the methods mentioned above will work.