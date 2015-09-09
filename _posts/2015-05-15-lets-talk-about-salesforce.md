---
layout: post
title:  "Lets talk about Salesforce"
cover: https://static.pexels.com/photos/6508/nature-laptop-outside-macbook.jpg
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

System.assertEquals(reference1.getUrl(), reference2.getUrl());

{% endhighlight %}

## StandardSetController pagination

Let's say we have component for StandardSetController pagination. Component's page is simple and it's similar to standard pagination used by Salesforce. Controller for this component contains few methods to traverse it's records collection. Also it has additional feature to handle collection with edited objects.

{% highlight java %}

public with sharing class StandardSetPaginationController {
    public ApexPages.StandardSetController setController { get; set; }
    public Boolean needUpdate { get; set; }

    public void first() {
        if(needUpdate) setController.save();
        setController.first();
    }

    public void last() {
        if(needUpdate) setController.save();
        setController.last();
    }
    public void next() {
        if(needUpdate) setController.save();
        setController.next();
    }

    public void previous() {
        if(needUpdate) setController.save();
        setController.previous();
    }
}

{% endhighlight %}

Looks legit, right? And it works flawlessly except one case. When you save controller traversal methods doesn't works as excepted. What i noticed and as i understand is that after save methos is executed set controller reset it's current page number to 1 and you can view only 1 and 2 pages.

Why? I do not know.

How to deal with this? I've got workaround solution. I had to get one. You have to switch page manually. And this woks like a charm.

{% highlight java %}

public with sharing class StandardSetPaginationController {
    public ApexPages.StandardSetController setController { get; set; }
    public Boolean needUpdate { get; set; }

    public Integer currentPageStart {
        get { return (currentPage - 1) * pageSize + 1; }
    }
    public Integer currentPageEnd {
        get { return Math.min(setController.getResultSize(), currentPage * pageSize); }
    }
    public Integer currentPage {
        get { return setController.getPageNumber(); }
    }
    public Integer totalPages {
        get { return (Integer) Math.ceil((Decimal)setController.getResultSize() / (Decimal) pageSize); }
    }
    public Integer pageSize {
        get { return setController.getPageSize(); }
    }

    public Boolean hasPreviousPage { get { return setController.getHasPrevious(); } }
    public Boolean hasNextPage     { get { return setController.getHasNext(); } }

    public void first()    { goToPage(1); }
    public void last()     { goToPage(totalPages); }
    public void next()     { goToPage(currentPage + 1); }
    public void previous() { goToPage(currentPage - 1); }

    private void goToPage(Integer pageNumer) {
        if(needUpdate) setController.save();
        setController.setPageNumber(pageNumer);
    }
}
{% endhighlight %}

And here it is component itself. Neat, right? :)

{% hightlight html %}

<apex:component controller="StandardSetPaginationController" allowDML="true">
    <apex:attribute name="standardSetController" type="ApexPages.StandardSetController"
        description="Paginated controller"
        assignTo="{!setController}" required="true"
    />
    <apex:attribute name="rerenderComponents" type="String"
        description="Id of components to be rerendered on parent's page"
        required="true"
    />
    <apex:attribute name="updateBeforePageChange" type="Boolean"
        description="If controller needs to save records before page change"
        assignTo="{!needUpdate}" required="false" default="false"
    />

    <div class="bottomNav">
        <div class="paginator">
            <span class="left">
                <apex:outputlabel value="{!currentPageStart} - {!currentPageEnd} of {!setController.resultsize}" />
            </span>
            <span class="prevNextLinks">
                <apex:outputPanel rendered="{!hasPreviousPage}">
                    <span class="prevNext">
                        <apex:commandlink action="{!first}" rerender="{!rerenderComponents}">
                            <img src="/s.gif" class="first" />
                        </apex:commandlink>
                    </span>
                    <span class="prevNext">
                        <apex:commandlink action="{!previous}" rerender="{!rerenderComponents}">
                            <img src="/s.gif" class="prev" />
                            Previous
                        </apex:commandlink>
                    </span>
                </apex:outputPanel>

                <apex:outputPanel rendered="{!hasNextPage}">
                    <span class="prevNext">
                        <apex:commandlink action="{!next}" rerender="{!rerenderComponents}">
                            Next
                            <img src="/s.gif" class="next" />
                        </apex:commandlink>
                    </span>
                    <span class="prevNext">
                        <apex:commandlink action="{!last}" rerender="{!rerenderComponents}">
                            <img src="/s.gif" class="last" />
                        </apex:commandlink>
                    </span>
                </apex:outputPanel>
            </span>
            <span class="right">
                <apex:outputlabel value="Page {!currentPage} of {!totalPages}" />
            </span>
        </div>
    </div>
</apex:component>

{% endhighlight %}

## This is not the end

This post is going to be published 'rolling release' manner. So there will be updates for sure. Keep in touch :)

[gary-bernhardt]:https://twitter.com/GaryBernhardt
[jsc-wat]:https://www.destroyallsoftware.com/talks/wat
[sfdc]:http://www.salesforce.com/
