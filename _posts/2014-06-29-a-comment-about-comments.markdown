---
layout: post
title:  "A comment about comments"
date:   2014-06-29 12:00:00
---

To make your code maintainable by others you just add lots and lots of comments. 
Simple right? 
Unfortunately this isn't true. 
A few problems with comments:

1.  Comments have to be maintained too.
2.  Partly due to point 1, comments don't necessarily tell the truth.
3.  Comments can clutter up your classes.
4.  We programmers are better trained to read code than comments.

<!--more-->

When you think it's useful to explain a certain part of your code, you might first want to rethink your programming decisions and try to find a solution to explain yourself using code.

An example of code in need of explanation:

{% highlight java linenos %}
//Check if user has administrative rights.
if (User.securityLevel() > 10) {
{% endhighlight %}

The above code could be refactored to something like this:

{% highlight java linenos %}
if (User.hasAdministrativeRights()) {
{% endhighlight %}

This single line of code tells you everything you need to know.
I'm not preaching to avoid all comments by definition,
but with a mindset to avoid comments as much as possible you'll be forced to write smaller methods with descriptive method names. 
This way instead of having highly commented unmaintainable code you end up with an actual maintainable code base. 
Which is after all the reason we would have used comments anyway.