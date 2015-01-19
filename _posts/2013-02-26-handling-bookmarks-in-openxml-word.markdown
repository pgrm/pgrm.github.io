---
layout: post
title: Handling Bookmarks in OpenXML Word-Documents
date: '2013-02-26T02:17:00.001+01:00'
tags:
- OpenXML
- C#
- ".Net"
modified_time: '2013-02-26T02:19:29.946+01:00'
blogger_id: tag:blogger.com,1999:blog-8459046186574607112.post-3710213993555581570
blogger_orig_url: http://emptycode.blogspot.com/2013/02/handling-bookmarks-in-openxml-word.html
---

I needed to reimplement my function of exporting all bookmarks of a word-document into a dictionary and than setting them based on the changes in the dictionary from Word-Interop to OpenXML SDK.

I've found a very helpful answer on [stackoverflow](http://stackoverflow.com/a/3318381/621366) suggesting the following solution:

{% highlight c# %}
IDictionary<string, BookmarkStart> bookmarkMap = new Dictionary<string, BookmarkStart>();
 
foreach (BookmarkStart bookmarkStart in file.MainDocumentPart.RootElement.Descendants<BookmarkStart>())
{
    bookmarkMap[bookmarkStart.Name] = bookmarkStart;
}
 
foreach (BookmarkStart bookmarkStart in bookmarkMap.Values)
{
    Run bookmarkText = bookmarkStart.NextSibling<Run>();
    if (bookmarkText != null)
    {
        bookmarkText.GetFirstChild<Text>().Text = "blah";
    }
}
{% endhighlight %}

Seems very easy, right? It was a great starting point however only a starting point.

## First thing I've discovered:

There are **hidden** bookmarks. This was weird in the beginning however you can see quickly the pattern, all of them start with an _ (underscore) and after finding [this trustworthy page](http://www.computorcompanion.com/LPMArticle.asp?ID=320) as the first answer from Google, to confirm if my assumption was correct, I didn't bother looking further, so just add

{% highlight c# %}
bookmarkStart.Name.StartsWith("_")
{% endhighlight %}

... and the problem is *solved*.

## Next Problem occurred:

You can define bookmarks for **Cells** and than they behave totally different. So how do they behave? 

> They are all stuck in the first cell of each row.

So how do I know to which cell they belong? Word seems to know it.

> BookmarkStart has a property ColumnFirst. Normally the value is NULL, however in this case it has the *0*-based index of the column it refers to. If your bookmarks stretch over multiple cells, there is also a ColumnLast (for my case ColumnFirst == ColumnLast).

However retrieving the data now is a bit tougher, so let's take a step back. First I created some Extension-Methods to  make my functions smaller:

{% highlight c# %}
public static T GetFirstDescendant<T>(this OpenXmlElement parent) where T : OpenXmlElement
{
    var descendants = parent.Descendants<T>();
 
    if (descendants != null)
        return descendants.FirstOrDefault();
    else
        return null;
}
 
public static T GetParent<T>(this OpenXmlElement child) where T : OpenXmlElement
{
    while (child != null)
    {
        child = child.Parent;
 
        if (child is T)
            return (T)child;
    }
 
    return null;
}
{% endhighlight %}

Now having those helpful methods let's start solving the actual problem.
First we need to add another condition.

{% highlight c# %}
if (bookmarkStart.ColumnFirst != null)
    return FindTextInColumn(bookmarkStart);
{% endhighlight %}

And actually implement FindTextInColumn

{% highlight c# %}
private Text FindTextInColumn(BookmarkStart bookmark)
{
    var cell = bookmark.GetParent<TableRow>().GetFirstChild<TableCell>();
 
    for (int i = 0; i < bookmark.ColumnFirst; i++)
    {
        cell = cell.NextSibling<TableCell>();
    }
 
    return cell.GetFirstDescendant<Text>();
}
{% endhighlight %}

As you can see, I'm looking for the Parent of type `TableRow` and take the first `TableCell`-child of this row. Afterwards I take the `NextSibling` of type `TableCell` until I reach the necessary column. Than I just need to return the first `Text` which can be found in this column. I myself don't really care how many texts exist, since I need only one to replace the content and keep the formatting. Later you will see that I delete additional `Text`-elements.

So, problem *solved* one more time. What else could there be?

While it's not a problem to read bookmark-values, it is one, when you are trying to set them:

## Bookmarks can be empty - not having any element:

However once you figured out that the bookmark really is empty it is quite easy and straight forward to add a simple `Run` with a `Text` after the `BookmarkStart`, the following function takes care of this very easily:

{% highlight c# %}
private void InsertBookmarkText(BookmarkStart bookmark, string value)
{
    bookmark.Parent.InsertAfter(new Run(new Text(value)), bookmark);
}
{% endhighlight %}

This is *solved* very easily, however as I suggested, the problem is not to insert the value, but to figure out if it needs to be inserted. For this I present you the last problem I've found and solved for retrieving the values from bookmarks:

## How to find Text and Run if they are not siblings of the bookmark as suggested by the initial solution?

For this, I expanded the simple search for Run from the initial solution into something more sophisticated. I don't know the specification of OpenXML-Documents so it might be unnecessary, but it provides also the information if the bookmark as such is empty.

First, here are 2 new helping Extension-Methods, I will use later on:

{% highlight c# %}
public static bool IsEndBookmark(this OpenXmlElement element, BookmarkStart startBookmark)
{
    return IsEndBookmark(element as BookmarkEnd, startBookmark);
}
 
public static bool IsEndBookmark(this BookmarkEnd endBookmark, BookmarkStart startBookmark)
{
    if (endBookmark == null)
        return false;
 
    return endBookmark.Id == startBookmark.Id;
}
{% endhighlight %}

And now the little magic...

{% highlight c# %}
var run = bookmarkStart.NextSibling<Run>();
 
if (run != null)
    // I've found a run and suppose it has a Text
    return run.GetFirstChild<Text>(); 
else
{
    // I will go through all the siblings and try to find any Text
    Text text = null;
    var nextSibling = bookmarkStart.NextSibling();
    while (text == null && nextSibling != null)
    {
        if (nextSibling.IsEndBookmark(bookmarkStart))
            // I've reached the end of the bookmark and couldn't find any Text
            return null;
 
        text = nextSibling.GetFirstDescendant<Text>();
        nextSibling = nextSibling.NextSibling();
    }
 
    return text;
}
{% endhighlight %}

Having this defined I managed to retrieve and replace correctly all bookmarks. We just forgot to solve the last issue - removing unnecessary Text-elements. In the following function, I want to remove all Text-elements within my bookmark except the parameter keep:

{% highlight c# %}
private void RemoveOtherTexts(BookmarkStart bookmark, Text keep)
{
    if (bookmark.ColumnFirst != null) return;
 
    Text text = null;
    var nextSibling = bookmark.NextSibling();
    while (text == null && nextSibling != null)
    {
        if (nextSibling.IsEndBookmark(bookmark))
            break;
 
        foreach (var item in nextSibling.Descendants<Text>())
        {
            if (item != keep)
                item.Remove();
        }
        nextSibling = nextSibling.NextSibling();
    }
}
{% endhighlight %}

Now all my problems are **SOLVED** :)
Hope it could help you as well, here you can find the whole code with all the functions, I used:

{% gist pgrm/5034752 %}