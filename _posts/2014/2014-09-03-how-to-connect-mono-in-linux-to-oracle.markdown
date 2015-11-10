---
title: "How to connect Mono in Linux to Oracle"
redirect_from:
  - /how-to-connect-mono
tags:
  - Linux
  - Oracle
  - Mono
  - Docker
---

> Long post about all the steps I did can be found here: [How I managed to connect from Mono in Linux to Oracle](http://peter.grman.at/how-i-managed-to-connect-from-mono-in-linux-to-oracle/)

If you are using `ServiceStack.OrmLite` it is extremely easy in the end:

> **Attention!** This solution might bring problems with it if you need timezone aware TimeStamps and TimeSpans in Oracle. Use it with caution.

After a lot of searching, the simplest solution was to modify `ServiceStack.OrmLite.Oracle` to use Oracle's manged driver. For it to work, I had to remove the `OracleTimestampConverter`, because Oracle's managed driver doesn't have the `GetClientInfo` function in the `OracleGlobalization` class. The code changes can be seen on [GitHub](https://github.com/Grman-IT-Solutions/ServiceStack.OrmLite/tree/OracleManagedDataAccess). I've also published these changes as a package on [NuGet](https://www.nuget.org/packages/ServiceStack.OrmLite.Oracle.Managed/).

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
