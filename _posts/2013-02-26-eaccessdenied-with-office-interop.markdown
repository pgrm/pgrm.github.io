---
layout: post
title: E_ACCESSDENIED with Office Interop
date: '2013-02-26T02:20:00.002+01:00'
tags:
- Exceptions
- Office Interop
- C#
- ".Net"
modified_time: '2013-02-26T02:20:45.862+01:00'
blogger_id: tag:blogger.com,1999:blog-8459046186574607112.post-9024554568488761247
blogger_orig_url: http://emptycode.blogspot.com/2013/02/eaccessdenied-with-office-interop.html
---

Once upon a time, I had a nice little function which would open an Word-Document via Microsoft-Interop, 
export all bookmarks as a Dictionary&lt;BookmarkName, StringBookmarkValue&gt;. 
I would modify this dictionary and by pass it on to another little function, which would afterwards set all the values of the bookmarks. 
This worked very well until .... One day the server was upgraded to Windows 2012 and Office 2010 and since than nothing seemed to work any more.

After getting the following error:

    System.UnauthorizedAccessException: Retrieving the COM class factory for component with CLSID {000209FF-0000-0000-C000-000000000046} failed due to the following error: 80070005 Access is denied. (Exception from HRESULT: 0x80070005 (E_ACCESSDENIED)).
       at System.Runtime.Remoting.RemotingServices.AllocateUninitializedObject(RuntimeType objectType)
       at System.Runtime.Remoting.Activation.ActivationServices.CreateInstance(RuntimeType serverType)
       at System.Runtime.Remoting.Activation.ActivationServices.IsCurrentContextOK(RuntimeType serverType, Object[] props, Boolean bNewObj)
       at System.RuntimeTypeHandle.CreateInstance(RuntimeType type, Boolean publicOnly, Boolean noCheck, Boolean& canBeCached, RuntimeMethodHandleInternal& ctor, Boolean& bNeedSecurityCheck)
       at System.RuntimeType.CreateInstanceSlow(Boolean publicOnly, Boolean skipCheckThis, Boolean fillCache, StackCrawlMark& stackMark)
       at System.RuntimeType.CreateInstanceDefaultCtor(Boolean publicOnly, Boolean skipCheckThis, Boolean fillCache, StackCrawlMark& stackMark)
       at System.Activator.CreateInstance(Type type, Boolean nonPublic)
       at System.Activator.CreateInstance(Type type)

...and spending roughly an hour on the Internet I found following link: [http://support.microsoft.com/kb/257757/en-us](http://support.microsoft.com/kb/257757/en-us)

Since non of the mentioned solutions worked for me in Word 2012 I decided to let Interop be and switch to OpenXML. [Here you can read how to do the same function to replace bookmarks in OpenXML]({% post_url 2013-02-26-handling-bookmarks-in-openxml-word %}) with the [OpenXML SDK](http://www.microsoft.com/en-us/download/details.aspx?id=30425)