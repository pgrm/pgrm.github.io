---
layout: post
title: Problem Adding Ribbon-Control to Office AddIn Projects
date: '2013-01-07T06:48:00.000+01:00'
tags:
- Ribbon
- Namespaces
- C#
- Office AddIn
- ".Net"
modified_time: '2013-01-07T06:48:12.464+01:00'
blogger_id: tag:blogger.com,1999:blog-8459046186574607112.post-4714018850222278104
blogger_orig_url: http://emptycode.blogspot.com/2013/01/problem-adding-ribbon-control-to-office.html
---

> ThisRibbonCollection' does not contain a definition for 'GetRibbon'....

If you see this error, don't panic, the solution is rather simple.

To make matters easier for you, Office AddIns have a partial class "ThisRibbonCollection" which is automatically used to define your Ribbon Control as the *MainRibbonControl* of the application. This might seem, after the creation and a first run rather as magic, unless of course, you want to have all your application controls in a separate folder and therefore separate namespace from *ThisAddIn.cs.* (thx for the hint to [http://qa.social.msdn.microsoft.com/Forums/en-SG/vsto/thread/3b117e7a-e3d5-4f10-8262-358b04494230](http://qa.social.msdn.microsoft.com/Forums/en-SG/vsto/thread/3b117e7a-e3d5-4f10-8262-358b04494230))

However there is also a solution to this, allowing you to keep your RibbonControl in a separated folder and namespace:

1. Open *[RibbonControl]Designer.cs*
2. Go to the end of the file where you see *partial class ThisRibbonCollection*
3. Create at the end of the file a new namespace section, with the root-namespace of your AddIn
4. Move the class into the new namespace
5. Since you are in a different namespace now, don't forget to add the necessary usings.