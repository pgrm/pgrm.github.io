---
title: "Request Authorization in ASP.NET Web API in Mono"
tags:
  - Asp.Net
  - WebAPI
  - Permissions
  - Mono
---

Few days ago I was struggling to set up authorization on requests to my Web API. The major Problem was missing sessions. You can read more here: [Request Authorization in ASP.NET Web API]({% post_url /2015/2015-02-21-request-authorization-in-asp.net-web-api %}). After I set it up, it was running and everything seemed great until I found out that it didn't work in [mono](http://www.mono-project.com/). So back to square one (or [StackOverflow](http://stackoverflow.com/questions/28667957/access-session-in-asp-net-web-api-in-mono)). It took me a while, until I discovered this 3 years old bug report:

> [SessionStateBehaviour in HttpContext is ignored](https://bugzilla.xamarin.com/show_bug.cgi?id=3012)

The fact that it was 3 years old, had only 1 comment (additionally to the reporter) and the last update has been 2 years ago didn't give me too much hope, it would be fixed. In fact, I felt confirmed that this was the problem in mono and I'd need to find a workaround myself.

First of all, let's consider, do I need a `Session` object? What am I using it for?

1. it caches the list of permissions
2. it does this automatically per user
3. it should be cleaned up automatically

Ok, so is there any other way to solve those 3 points?

### @1. it caches the list of permissions

Although this post is from 2010, it gave a nice overview of possible solutions for caching: [Four Methods Of Simple Caching In .NET](http://www.jondavis.net/techblog/post/2010/08/30/Four-Methods-Of-Simple-Caching-In-NET.aspx). And I didn't like it. Seems like there isn't really something simple out there which will work for sure. I didn't want to try the .Net default implementations, because I didn't know if they'd work in mono. But even on that post, the last option was **Build One Yourself**. And that one I'll be able to fix, as well as make sure that it runs in mono. So I did. It is basically a class with a static `ConcurrentDictionary`, but what to put inside?

### @2 it caches the list of permissions automatically per user (==per session)

So, how else can I identify users except on their session? Turns out, a user can actually have multiple sessions, but his permissions will be always the same, so why not just identify him by their **unique** ID or name. I went for **name** as it is easily accessed and stored in `HttpContext.Current.User.Identity.Name`.

Plugging this together with **1.** we know, that our `ConcurrentDictionary` is going to have the user's **name** as the **key** (`string`) and the list of permissions as a value (doesn't have to be a list, can be also a set, or any other data-structure you prefer). But is this going to be enough? What about memory leaks, how will I empty the cache?

### @3 it should be cleaned automatically

Please, pay close attention to the **should**. A bug report suggests that this isn't the case anyway for the normal `Session` objects: [Memory leak in asp.net with sessions enabled](https://bugzilla.xamarin.com/show_bug.cgi?id=381). And do I really need it? The maximum number of space allocated would be the number of different users in my database multiplied with the space required by all available permissions. Turns out this really isn't too much for me. It is less than 10MB and in a normal scenario will be probably around 1MB. And because I have a dictionary, I know I won't be able to store a user multiple times. So, probably, I even save memory compared to the old solution.

### Summary

I have one class which has a `static ConcurrentDictionary<string, ...>` (the value is up to you, how you want to store your permissions or other data). The dictionary stores one object (or list of objects) per user. Here you need to make sure, that you don't have too many users, otherwise you should implement a way to get rid of old unused entries.

This dictionary is a cache and before I try to read from it, I first check if it contains an entry for the current user. If not, first I have to retrieve it from the database and fill up the cache.