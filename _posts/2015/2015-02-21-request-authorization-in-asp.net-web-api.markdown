---
title: "Request Authorization in ASP.NET Web API"
tags:
  - Asp.Net
  - WebAPI
  - Permissions
---

When you use ASP.NET Web API as a back-end for your JavaScript heavy Single Page Application (SPA), you might want to check who is performing requests, before you answer with a result. In my case, the different permissions are a bit bulky and I'm storing them in the user's `Session` object. On each request, I check which permissions are necessary and if the user has those permissions. If not, I return a [403 Forbidden](http://en.wikipedia.org/wiki/HTTP_403). To do this, you simply extend the [AuthorizeAttribute](https://msdn.microsoft.com/en-us/library/system.web.http.authorizeattribute%28v=vs.118%29.aspx?f=255&MSPPError=-2147217396) and perform the necessary checks in [IsAuthorized](https://msdn.microsoft.com/en-us/library/system.web.http.authorizeattribute.isauthorized(v=vs.118).aspx). Peace of cake, [how hard can it be](https://www.youtube.com/watch?v=uL0ROeZw7wA)?

## Turns out, pretty hard...

# 1. [AuthorizeAttribute](https://msdn.microsoft.com/en-us/library/system.web.mvc.authorizeattribute%28v=vs.118%29.aspx?f=255&MSPPError=-2147217396) != [AuthorizeAttribute](https://msdn.microsoft.com/en-us/library/system.web.http.authorizeattribute%28v=vs.118%29.aspx?f=255&MSPPError=-2147217396)

Click on the links if you don't believe me. There are two classes with the same name but different namespace:

- [System.Web.Mvc.AuthorizeAttribute](https://msdn.microsoft.com/en-us/library/system.web.mvc.authorizeattribute%28v=vs.118%29.aspx?f=255&MSPPError=-2147217396)
- **[System.Web.Http.AuthorizeAttribute](https://msdn.microsoft.com/en-us/library/system.web.http.authorizeattribute%28v=vs.118%29.aspx?f=255&MSPPError=-2147217396)**

When you are working with Web API, you want to use **System.Web.Http.AuthorizeAttribute** instead of the one in the Mvc namespace.

# 2. Where is my Session?

This was probably the hardest part. Turns out, when an API Controller is launched, the session is not accessible. However, there is a workaround to this, which I've found here: [https://soabubblog.wordpress.com/2013/07/10/web-api-sessions/](https://soabubblog.wordpress.com/2013/07/10/web-api-sessions/)

{% highlight c# %}
protected void Application_PostAuthorizeRequest()
{
  if (IsWebApiRequest())
  {
    HttpContext.Current.SetSessionStateBehavior(
      System.Web.SessionState.SessionStateBehavior.Required);
  }
}

private static bool IsWebApiRequest()
{
  return HttpContext.Current.Request.AppRelativeCurrentExecutionFilePath.StartsWith(@"~/api");
}
{% endhighlight %}

These few lines will save you your sanity when you'll be looking for the session object everywhere.

# 3. I want to return 403 instead of 401

The `AuthorizeAttribute` returns [401 Unauthorized](http://en.wikipedia.org/wiki/List_of_HTTP_status_codes#4xx_Client_Error) but I want to return 403. It is a small difference but it matters to me. You can read more about HTTP Codes [401 VS 403 here](http://stackoverflow.com/questions/3297048/403-forbidden-vs-401-unauthorized-http-responses). The summary:

> ... In summary, a 401 Unauthorized response should be used for missing or bad authentication, and a 403 Forbidden response should be used afterwards, when the user is authenticated but isn’t authorized to perform the requested operation on the given resource.

To do this, you need to override `HandleUnauthorizedRequest` and set the response.

# The final solution:

{% highlight c# %}
[AttributeUsageAttribute(AttributeTargets.Class | AttributeTargets.Method, Inherited = true, AllowMultiple = true)]
public class PermissionsAttribute : AuthorizeAttribute
{
    public string[] AllowedPermissions { get; private set; }

    public PermissionsAttribute(params string[] allowedPermissions)
    {
        AllowedPermissions = allowedPermissions;
    }

    protected override bool IsAuthorized(HttpActionContext actionContext)
    {
        foreach (var perm in AllowedPermissions)
        {
            if (HasPermission(perm))
            {
                return true;
            }
        }
        return false;
    }

    protected override void HandleUnauthorizedRequest(HttpActionContext actionContext)
    {
        // optionally remember that somebody tried accessing a forbidden url
        actionContext.Response = new HttpResponseMessage(HttpStatusCode.Forbidden);
    }

    private bool HasPermission(string permission) {
        // Check here if the permission is stored in HttpContext.Current.Session
    }
}
{% endhighlight %}

However, turns out, this solution doesn't work in mono. There is a new post about that particular problem: [Request Authorization in ASP.NET Web API in Mono]({% post_url /2015/2015-02-23-request-authorization-in-asp.net-web-api-in-mono %})