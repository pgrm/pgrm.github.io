---
title: Dynamic Controllers in AngularJS
date: 2014-10-15 00:00:00 Z
tags:
- AngularJS
---

I have an AngularJS application, which offers a set of different tasks. In order not to limit myself in the future, I have one main controller, and sub-controllers for each task. The correct sub-controller is selected based on the task's URL.

## However, how do you inject a dynamic controller into the HTML?

**Static controllers are easy,** `<ANY ng-controller="ControllerName"></ANY>`

### So how do you do dynamic ones?

In StackOverflow there is a very helpful question and answer to this: [From an AngularJS controller, how do I resolve another controller function defined in the module?](http://stackoverflow.com/questions/21204371/from-an-angularjs-controller-how-do-i-resolve-another-controller-function-defin). However it is quite long and slightly incomplete, so here for myself, and anybody out there, the two simple versions which **just work**:

1. **Don't!** Don't mess with `ng-controller` around, let Angular handle it. In my case, every controller had anyway a different view, which was downloaded, and I just defined the controller statically inside the view (`<div class="ViewContainer" ng-controller="SpecificViewController">...</div>`)

2. The first option is really nice and simple, but doesn't always work. The 2nd option is, what you're probably rather looking for:

You can load the controller via the [Controller Service](https://docs.angularjs.org/api/ng/service/$controller) like this: `$controller('ControllerName')` (optionally, you can inject variables as a second parameter). However, if you want to have the scope injected into the controller, this will throw some exception like 

> unknown provider scopeprovider ...

For it to work, you need to create the scope yourself, so the correct version is:

{% highlight javascript %}
$controller('ControllerName', { $scope: $scope.$new() });
{% endhighlight %}

This will now successfully create the new controller. (Of course you can add additional local variables, if necessary.) The second problem I had, how to use this code together with `ng-controller`?

First, you need to add the code into the scope, as a function:

{% highlight javascript %}
$scope.SubController = function() {
  return $controller('ControllerName', { $scope: $scope.$new() }).constructor;
}
{% endhighlight %}
    
Afterwards, you can bind it in HTML simply like this:

{% highlight xml %}
<ANY ng-controller="SubController"></ANY>
{% endhighlight %}
    
and everything should work.