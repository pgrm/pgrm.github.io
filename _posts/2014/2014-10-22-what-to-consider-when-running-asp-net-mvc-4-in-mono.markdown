---
title: What to Consider when running ASP.NET MVC 4 in Mono
date: 2014-10-22 00:00:00 Z
tags:
- ASP.NET MVC
- Mono
---

I'm fighting for the last 2 days to deploy my ASP.NET MVC 4 application to a Linux VM running Mono again. - Again because it already worked, but in the meanwhile Microsoft updated the MVC 4 library and something broke (read more: [Microsoft Asp.Net MVC Security Update MS14-059 broke my build!](http://blogs.msdn.com/b/webdev/archive/2014/10/16/microsoft-asp-net-mvc-security-update-broke-my-build.aspx))

Basically the most common error was, that my view, which existed in the path with the correct case sensitive name, couldn't be found by the ASP.NET application. I've found several more or less useful posts about this, but finally it was about one library which I forgot to **remove** from the deployed application: **Microsoft.Web.Infrastructure.dll**.

Mono seems to have a custom implementation of this library, which works smoothly, while Microsoft's implementation is specific to IIS. After removing that library everything worked again. I also switched the views to be precompiled, so if removing the library isn't enough, you might want to precompile the views during the publishing process.

Additionally, I've also included some other libraries, here is a list of the default .Net libraries which I'm copying together with my application:

- System.Data.Entity.dll
- System.IO.dll
- System.Net.Http.dll
- System.Net.Http.Extensions.dll
- System.Net.Http.Formatting.dll
- System.Net.Http.Primitives.dll
- System.Net.Http.WebRequest.dll
- System.Runtime.dll
- System.Threading.Tasks.dll
- System.Web.Entity.dll
- System.Web.Helpers.dll
- System.Web.Http.dll
- System.Web.Http.WebHost.dll
- System.Web.Mvc.dll
- System.Web.Optimization.dll
- System.Web.Razor.dll
- System.Web.Routing.dll
- System.Web.WebPages.dll
- System.Web.WebPages.Deployment.dll
- System.Web.WebPages.Razor.dll

Maybe not all libraries are necessary, but this works for now for me.