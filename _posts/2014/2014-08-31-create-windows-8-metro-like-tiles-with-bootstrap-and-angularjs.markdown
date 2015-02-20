---
title: "Create Windows 8 Metro-like tiles with Bootstrap and AngularJS"
tags:
  - HTML5
  - AngularJS
  - Bootstrap
---

There are many JavaScript libraries out there to create layouts with tiles like they are on the Windows 8 start screen. But when you already use Bootstrap and AngularJS, do you really need to add further libraries? - No, you don't.

Here is a simple AngularJS directive which, will allow you to create tiles together with Bootstrap. This way you can make them with just those technologies, you are already using anyway.

HTML-code:

{% highlight html %}
<div class="row">
  <div class="col-md-3 col-sm-4 col-xs-6" data-bind-height-to-width="0.5">
    <div style="height:100%; width:100%; background-color:orange"></div>
  </div>
  <div class="col-md-3 col-sm-4 col-xs-6" data-bind-height-to-width="1.0">
    <div style="height:100%; width:100%; background-color:blue"></div>
  </div>
  <div class="col-md-3 col-sm-4 col-xs-6" data-bind-height-to-width>
    <div style="height:100%; width:100%; background-color:green"></div>
  </div>
  <div class="col-md-3 col-sm-4 col-xs-6" data-bind-height-to-width="2">
    <div style="height:100%; width:100%; background-color:yellow"></div>
  </div>
</div>
{% endhighlight %}

The `bindHeightToWidth`-directive used in the divs will automatically keep the height updated based on `width * factor`. Default factor is `1`, which creates a square, the other factors used in the example are `0.5` and `2`.

AngularJS-directive (written in TypeScript):

{% highlight javascript %}
.directive('bindHeightToWidth', () => {
  var directive: ng.IDirective;

  directive = {
    restrict: 'A',
    link: (scope, instanceElement, instanceAttributes, controller, transclude) => {
      var heightFactor = 1;

      if (instanceAttributes['bindHeightToWidth']) {
        heightFactor = instanceAttributes['bindHeightToWidth'];
      }

      var updateHeight = () => {
        instanceElement.height(instanceElement.width() * heightFactor);
      }

      scope.$watch(instanceAttributes['bindHeightToWidth'], (value) => {
        heightFactor = value;
      });

      $(window).resize(updateHeight);
      updateHeight();

      scope.$on('$destroy', () => {
        $(window).unbind('resize', updateHeight);
      });
    }
  };

  return directive;
})
{% endhighlight %}

As you can see, the directive is very easy and straight forward to start with. This might not have the same feature richness of other libraries which are several KB large, but you see what it does. You don't have unnecessary clutter around your code and it just works.

And if you want your tiles to organise automatically like in Google+ for instance, you can combine it with [Masonry](http://masonry.desandro.com/) and this awesome and very lightweight directive to use Masonry together with AngularJS: https://github.com/klederson/angular-masonry-directive.

I've also created a plunk so you can directly play with a sample version: http://plnkr.co/edit/Ffgrxq?p=preview Feel free to fork it and post your variations in the comments below. Looking forward to many different tiles :)
