---
title: "Always check the ModelState"
tags: 
  - ASP.NET MVC
  - ModelState
---

Ever came across weird behaviour in your ASP.NET Views? - I did already several times, and this is what I've learned:

> __Always__ check the ModelState.

Most of you probably remember to _clear_ the _ModelState_ when you modify _POST_ed values. But I had to find out, that HttpPost isn't the only way to get a messed up ModelState.
I had a simple Action:

{% highlight c# %}
[HttpGet]
public ActionResult Edit(string username)
{
    return View(ComponentFactory
        .GetAdapter<IUserListAdapter>()
        .Get(username));
}
{% endhighlight %}

and in the view, I just wanted to show a form for editing those details:

{% highlight c# %}
@using (Html.BeginForm())
{
    @Html.AntiForgeryToken()
    @Html.ValidationSummary(true)
    <fieldset>
        <legend>Globale Benutzer</legend>
        <div class="editor-label">
            @Html.LabelFor(model => model.UserId)
        </div>
        <div class="editor-field">
            @Html.EditorFor(model => model.UserId)
            @Html.ValidationMessageFor(
                model => model.UserId)
        </div>
        <div class="editor-label">
            @Html.LabelFor(model => model.UserName)
        </div>
        <div class="editor-field">
            @Html.EditorFor(model => model.UserName)
            @Html.ValidationMessageFor(
                model => model.UserName)
        </div>
        <div>
            <input type="submit"
                value="@LocalizationHelper
                .LocalizedLiteral("Save").ToString()"
            />
        </div>
    </fieldset>
}
{% endhighlight %}

While the value for `Model.UserName` was the correct name of the user, the text-box (created by `@Html.EditorFor(model => model.UserName)`) contained the user's id. The same value as `Model.UserId`. Everything else (label, validation rules, ...) was correct and I couldn't figure out __WHY__.


What else could modify it?
-------------------------

After no answer from Stack Overflow and some more googleing I came across the term _"ModelState"_. Since I had no other clue, I thought to give it a try:

{% highlight c# %}
[HttpGet]
public ActionResult Edit(string username)
{
    ModelState.Clear();
    return View(ComponentFactory
        .GetAdapter<IUserListAdapter>()
        .Get(username));
}
{% endhighlight %}

And .... __It Worked!__


How does ModelState work?
-------------------------

I didn't give too much attention to ModelState before, just cleared it if it solved my problems and that's all. Yet, I didn't understand, why I would need `ModelState.Clear()` in a an action as simple as this one.

So I had a deeper look, and it is actually very interesting what happens in the background.
Ok what ASP.NET does is:

1. Taking all parameters you passed to the action (via ``GET`` or `POST` - that doesn't matter)
2. Storing them in a dictionary, the ModelState (which you can empty with ``ModelState.Clear()``).

Later, when the view is created and ``@Html.EditorFor`` is called, ASP.NET checks if it has a parameter with that name. (Name checks are case-__in__-sensitive.) If the parameter is found in the ModelState, it is retrieved from there, instead of evaluating the Model.


### So why did I have problems with it?

Now the solution is simple: My Action had the parameter named _username_. Even I stored the UserId inside. Later when ```@Html.EditorFor(model => model.UserName)``` was called, _"username"_ from the ModelState was retrieved. However this _"username"_ contained the UserId, passed to the action.


Possible solutions:
-------------------

- Call `ModelState.Clear()` (really bad solution, don't like it)
- Rename `username` to `userid`, than the ModelState will only replace one userId, for the same from a different source.