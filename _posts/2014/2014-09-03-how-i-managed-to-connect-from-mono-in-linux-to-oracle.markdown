---
title: How I managed to connect from Mono in Linux to Oracle
date: 2014-09-03 00:00:00 Z
tags:
- Linux
- Oracle
- Mono
- Docker
---

> **TL;DR** [How to connect Mono in Linux to Oracle](http://peter.grman.at/how-to-connect-mono-in-linux-to-oracle/)

I'm in the process of migrating a ASP.NET MVC application to Mono and Linux. We made the decision to have easier deployment for our clients based on Docker-containers. Looking at [Mono's compatibility list](http://www.mono-project.com/docs/about-mono/compatibility/), .Net 4.0 and MVC 3  are all supported. Exactly what we needed.

While most parts of the website worked just fine in Mono, data access didn't. At first we needed to migrate from Entity Framework 5 to ServiceStack OrmLite. This turned out to be in many ways also an advantage in the end. `UPDATE` and `DELETE` statements as well as general CRUD was much easier. We implemented most of the rest via Views and Functions in Oracle directly.

At first I was looking at some StackOverflow questions how I could start this:

- [How to configure oracle instantclient for mono?](http://stackoverflow.com/questions/2422316/how-to-configure-oracle-instantclient-for-mono)
- [Can I connect to an Oracle database in a way that will work in a Mono environment (on Linux) and in a ClickOnce deployment (on Windows)?](http://stackoverflow.com/questions/2645934/can-i-connect-to-an-oracle-database-in-a-way-that-will-work-in-a-mono-environmen)
- [How do I use Oracle from .NET?](http://stackoverflow.com/questions/11366695/how-do-i-use-oracle-from-net)

I tried to use `ServiceStack.OrmLite.Oracle` and all the information I got from StackOverflow to access the Oracle database. But it turned out, I was totally of the track. I was in fact so far off that it took me almost half a week to get back and solve this, rather trivial problem. Forget everything written in those posts, the reality is much simpler.

The problem for `ServiceStack.OrmLite.Oracle` was, that they referenced the old Oracle client. Not the managed version but the one which exists in an x64 and x86 version and requires the full Oracle client to be installed. Trying out all the tricks didn't help me. I ended up with different exceptions in Linux with libraries missing, file headers being wrong,...

## The Solution

> **Attention!** This solution might bring problems with it if you need timezone aware TimeStamps and TimeSpans in Oracle. Use it with caution.

In the end, I took `ServiceStack.OrmLite.Oracle` and modified it to reference Oracle's new managed driver. For it to work, I had to remove the `OracleTimestampConverter`, because Oracle's managed driver doesn't have the `GetClientInfo` function in the `OracleGlobalization` class. The code changes can be seen on [GitHub](https://github.com/Grman-IT-Solutions/ServiceStack.OrmLite/tree/OracleManagedDataAccess). I've also published these changes as a package on [NuGet](https://www.nuget.org/packages/ServiceStack.OrmLite.Oracle.Managed/).

The beauty of this solution is that you don't need anything extra anymore. You can take for instance my [Ubuntu-mono Docker image](https://hub.docker.com/u/pgrm/ubuntu-mono), configure your driver and ConnectionString in `web.config`

{% highlight xml %}
<system.data>
  <DbProviderFactories>
    <remove invariant="Oracle.ManagedDataAccess.Client" />
    <!-- If any should be in the machine.config -->
    <add name="Oracle Data Provider for .NET" invariant="Oracle.ManagedDataAccess.Client" description="Oracle Data Provider for .NET" type="Oracle.ManagedDataAccess.Client.OracleClientFactory, Oracle.ManagedDataAccess, Version=4.121.1.0, Culture=neutral" />
  </DbProviderFactories>
</system.data>
<connectionStrings>
  <clear />
  <add name="OracleContext" providerName="Oracle.ManagedDataAccess.Client" connectionString="DATA SOURCE=<IP_ADDRESS>:1521/XE;PASSWORD=<PASSWORD>;USER ID=<USER_ID>;Connection Timeout=600;Validate Connection=true" />
</connectionStrings>
{% endhighlight %}

... and it is **WORKING**!