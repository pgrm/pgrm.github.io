---
title: Simple Zoom for WPF Controls
date: 2012-08-12 16:07:00 Z
tags:
- XAML
- C#
- ".Net"
layout: post
modified_time: '2012-08-12T20:09:07.818+02:00'
blogger_id: tag:blogger.com,1999:blog-8459046186574607112.post-2164028543641204210
blogger_orig_url: http://emptycode.blogspot.com/2012/08/hallo-again-recently-i-had-to-implement.html
---

Recently, I had to implement a zoom – in / out – functionality into a WPF application. After some research I found out that there is a RenderTransform on the Grid-Control. Here you can define a ScaleTransform which is able to scale everything in the grid by a value of type double (so 0.5 makes everything half the size, 2 makes everything double the size). One more great but probably not often needed feature is, that you can define different values for the horizontal and the vertical scale.To get an easy and quick return, just use the following XAML-Code:


{% highlight xml %}
<Grid x:Name="LayoutRoot">
  <Grid.RenderTransform>
    <ScaleTransform>
      <ScaleTransform.ScaleX>
        <Binding Path="ScaleFactor" ElementName="Window"/>
      </ScaleTransform.ScaleX>
      <ScaleTransform.ScaleY>
        <Binding Path="ScaleFactor" ElementName="Window"/>
      </ScaleTransform.ScaleY>
    </ScaleTransform>
  </Grid.RenderTransform>
 
  <Grid.RowDefinitions>
    <RowDefinition Height="66"/>
    <RowDefinition Height="*"/>
    <RowDefinition Height="30"/>
  </Grid.RowDefinitions>
 
  <!-- enter your controls here -->
</Grid>
{% endhighlight %}

As you see there is a binding to ScaleFactor, which is Dependency-Property on the Window element. Now you only have to set the ScaleFactor to change the zoom-factor, this was easy.

When you want to implement this on your main-window, which doesn’t have any scroll-bars you won’t be very. You get the zoom-functionality but when you zoom-in, you won’t see everything anymore, and when you zoom-out, there will be unused space.

I can’t help you with the unused space, because this can only be filled, when you actually have scroll bars, because your control ist too large for it’s container. So the best thing is, just ignore the zoom-out and set the minimum zoom value to 1 (100%).

But for the zoom-in, there is a very simple trick, just add 2 more RowDefinitions. Why 2? Because the RowDefinitions with static values (pixel, auto) change by a slightly other factor then the dynamic ones (star). So now, our definition should look something like this:

{% highlight xml %}

<Grid x:Name="LayoutRoot">
  <Grid.RenderTransform>
    <ScaleTransform>
      <ScaleTransform.ScaleX>
        <Binding Path="ScaleFactor" ElementName="Window"/>
      </ScaleTransform.ScaleX>
      <ScaleTransform.ScaleY>
        <Binding Path="ScaleFactor" ElementName="Window"/>
      </ScaleTransform.ScaleY>
    </ScaleTransform>
  </Grid.RenderTransform>
 
  <Grid.RowDefinitions>
    <RowDefinition Height="66"/>
    <RowDefinition Height="*"/>
    <RowDefinition Height="30"/>
    <!-- zooming row definitions -->
    <RowDefinition Height="0"/>
    <RowDefinition Height="0"/>
  </Grid.RowDefinitions>
   
  <!-- enter your controls here -->
</Grid>
{% endhighlight %}

As you can see, I added 2 more RowDefinitions with Height=”0″ but this will change, in the code, this is the definition of the ScaleFactor and what happens, when it changes: 

{% highlight c# %}

public double ScaleFactor
{
  get { return (double)GetValue(ScaleFactorProperty); }
  set { SetValue(ScaleFactorProperty, value); }
}
 
// Using a DependencyProperty as the backing store for ScaleFactor. 
// This enables animation, styling, binding, etc…
public static readonly DependencyProperty ScaleFactorProperty =
  DependencyProperty.Register("ScaleFactor", typeof(double), 
    typeof(MainWindow), new FrameworkPropertyMetadata(1.0,
      FrameworkPropertyMetadataOptions.BindsTwoWayByDefault,
      ScaleFactorPropertyChangedCallback));
 
private static void ScaleFactorPropertyChangedCallback(
  DependencyObject d, DependencyPropertyChangedEventArgs e)
{
  MainWindow me = d as MainWindow;
 
  if (me != null)
  {
    double starSize = 0, staticSize = 0;
 
    for (int i = 0; i < me.LayoutRoot.RowDefinitions.Count – 2; i++)
    {
      if (me.LayoutRoot.RowDefinitions[i].Height.IsStar)
        starSize += me.LayoutRoot.RowDefinitions[i].Height.Value;
      else if (me.LayoutRoot.RowDefinitions[i].Height.IsAuto)
        staticSize += me.LayoutRoot.RowDefinitions[i].MinHeight;
      else
        staticSize += me.LayoutRoot.RowDefinitions[i].Height.Value;
    }
 
    me.LayoutRoot.RowDefinitions
      [me.LayoutRoot.RowDefinitions.Count - 2].Height = 
        new GridLength(staticSize * (me.ScaleFactor – 1), GridUnitType.Pixel);
    me.LayoutRoot.RowDefinitions
      [me.LayoutRoot.RowDefinitions.Count - 1].Height = 
        new GridLength(starSize * (me.ScaleFactor – 1), GridUnitType.Star);
  }
}
{% endhighlight %}

Now, you can see, that I dynamically sum up the dynamic and static heights of all rows.
In the end, I set the height of the row before last to `Sum(height of all RowDefinitions with a static size) * (ScaleFactor – 1)` and I do the same with the last row, just for all dynamic RowDefinitions.

But why ScaleFactor – 1 ?

The answer is easy, so I can assure, that everything else my other rows are still fully visible, how?

Because the ScaleFactor changed, everything got bigger, so for example it changed to 1,5 so now everything is 150% of the normal size. But in the window only fits 100% so I can only see 2/3 of my grid. By resizing the 2 rows to the half of the grid (1,5 – 1 = 0,5) the rest still fits into the 100% and the 2 dummy rows are hidden in an unknown space.