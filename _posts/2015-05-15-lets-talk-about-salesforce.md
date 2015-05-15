---
layout: post
title:  "Lets talk about Salesforce"
date:   2015-05-15 15:00:00
---

As we know all languages has thier quirks and odd/unexpected/undefined behaviours. [Gary Bernhardt][gary-bernhardt] pointed out some of those in his [lighting talk][jsc-wat].

As i'm currently tided up to <s>Slaveforce</s> [Salesforce][sfdc] and in my opinion there is lots things to fix and improve. I'll put here some of inconsistances in code and it's behaviour. If you know more of them, give me a shout.

"Lest talk about salesforce" :)

<!-- more -->

![WAT](/assets/wat.png)

## PageReference comparision

While wrtiting tests for software we assume a lot of things. Most basic assumption is that objects are properly compared. In test i belive object should be compared by it's content and it works most of the time. But when you want to assert equality of PageReference the magic happens :)

{% highlight java %}

PageReference reference1 = new PageReference('/apex/some_page');
PageReference reference2 = new PageReference('/apex/some_page');

System.assertEquals(reference1, reference2);

{% endhighlight %}

In my opinion this should execute nicely and let the test pass but hey, it's Salesforce :)

<pre>
32.0 APEX_CODE,DEBUG
Execute Anonymous: PageReference reference1 = new PageReference('/apex/some_page');
Execute Anonymous: PageReference reference2 = new PageReference('/apex/some_page');
Execute Anonymous:
Execute Anonymous: System.assertEquals(reference1, reference2);
14:40:42.037 (37207059)|EXECUTION_STARTED
14:40:42.037 (37218368)|CODE_UNIT_STARTED|[EXTERNAL]|execute_anonymous_apex
14:40:42.038 (38306527)|EXCEPTION_THROWN|[4]|System.AssertException: Assertion Failed: Expected: System.PageReference[/apex/some_page], Actual: System.PageReference[/apex/some_page]
14:40:42.038 (38457751)|FATAL_ERROR|System.AssertException: Assertion Failed: Expected: System.PageReference[/apex/some_page], Actual: System.PageReference[/apex/some_page]

AnonymousBlock: line 4, column 1
14:40:42.038 (38502059)|CODE_UNIT_FINISHED|execute_anonymous_apex
14:40:42.039 (39853255)|EXECUTION_FINISHED
</pre>

Of course there is workaround. But it should work out of the box.

{% highlight java %}

PageReference reference1 = new PageReference('/apex/some_page');
PageReference reference2 = new PageReference('/apex/some_page');

System.assertEquals(reference1, reference2);

{% endhighlight %}

## This is not the end

This post is going to be published 'rolling release' manner. So there will be updates for sure. Keep in touch :)

[gary-bernhardt]:https://twitter.com/GaryBernhardt
[jsc-wat]:https://www.destroyallsoftware.com/talks/wat
[sfdc]:http://www.salesforce.com/
