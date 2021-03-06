---
layout: post
title: When ServiceTracker Trumps Declarative Services
summary: Illustrating a use-case where ServiceTracker is a better fit than DS.
tags: osgi, eclipse
---

I normally recommend using the Declarative Services (DS) specification for working with OSGi services, and in general I agree with the recommendation voiced by others such as <a href="http://martinlippert.blogspot.com/2010/03/slides-on-osgi-best-and-worst-practices.html">Martin</a> and <a href="http://www.slideshare.net/caniszczyk/osgi-best-and-worst-practices">Chris</a> that one should avoid programming against the OSGi APIs.

However I'd like to highlight a use case in which the good old OSGi `ServiceTracker` API can produce code that is more readable and robust than anything we could achieve with DS.

In DS, we declare components. A component can bind to services with either a unary reference, where the component binds to a single selected instance, or a multiple reference, where the component binds to all available instances. However the component itself remains a singleton.

For example, suppose we declare a component as follows:

{% highlight java %}
@Component
public class C implements IC {
	@Reference
	public void setA(IA a) { /* ... */ }
	public void unsetA(IA a) { /* ... */ }
}
{% endhighlight %}

This component binds to the service "IA" with a unary reference. If there are three instances of IA available then the component will bind as follows:

![](/images/posts/ds-bind-unary.png)

Alternatively if we declare the component like this:

{% highlight java %}
@Component
public class C {
	@Reference (multiple = true)
	public void setA(IA a) { /* ... */ }
	public void unsetA(IA a) { /* ... */ }
}
{% endhighlight %}

Then the bind will look like this:

![](/images/posts/ds-bind-multi.png)

But here is a binding that we cannot achieve in DS (at least, not declaratively):

![](/images/posts/bind-per.png)

Fortunately the ServiceTracker code for this is actually quite simple. Suppose we want to create one instance of C for each available instance of IA, and also register those C instances as services under the IC interface. We assume that the C class has a public constructor that takes a parameter of type IA. Here is the complete code (excluding imports and package declaration):

{% highlight java %}
public class ForeachA extends ServiceTracker {
    public ForeachA(BundleContext ctx) {
        super(ctx, IA.class.getName(), null);
    }
    
    @Override
    public Object addingService(ServiceReference ref) {
        IA a = (IA) context.getService(ref);
        return context.registerService(IC.class.getName(),
                                       new C(a), null);
    }
    
    @Override
    public void removedService(ServiceReference ref, Object svc) {
        ServiceRegistration reg = (ServiceRegistration) svc;
        reg.unregister();
        context.ungetService(ref);
    }
}
{% endhighlight %}

Of course, we **could** have achieved this with DS, by writing a component to manage C instances. We would have had to keep track of the instances of IA and the corresponding Cs and ServiceRegistrations, remembering to watch out for concurrency issues. If we were to go that route then we would essentially have reinvented ServiceTracker inside a DS component. The code above is much easier and less error-prone to write.

Things get even more interesting if we need to reference **two** service types, IA and IB, such that one instance of C is created for each combination of an IA and an IB, i.e.:

![](/images/posts/bind-per-combine.png)

To do this in DS is very hard — think about all the things we must keep track of to ensure that we create a C for every IA and every IB irrespective of whether an IA or an IB appears first... and now think about making it thread-safe!

However we can implement this pattern quite easily by chaining together a pair of ServiceTrackers. Here is the code, note that we are reusing the `ForeachA` class from above but adding to it a constructor parameter of type IB:

{% highlight java %}
public class ForeachB extends ServiceTracker {
    public ForeachB(BundleContext ctx) {
        super(ctx, IB.class.getName(), null);
    }
    
    @Override
    public Object addingService(ServiceReference ref) {
        IB b = (IB) context.getService(ref);
        ForeachA foreachA = new ForeachA(context, b);
        foreachA.open();
        return foreachA;
    }
    
    @Override
    public void removedService(ServiceReference ref, Object service) {
        ForeachA foreachA = (ForeachA) service;
        foreachA.close();
        context.ungetService(ref);
    }
}
{% endhighlight %}

Note that while using ServiceTracker necessarily means having a dependency on the OSGi APIs, it's possible to completely isolate our usage of those APIs from our business logic. The class C is a POJO and receives instances of IA and IB through its constructor, so it remains useable and testable outside of an OSGi environment.

In summary, there are still use cases where the low-level OSGi APIs such as ServiceTracker have an advantage over abstractions such as Declarative Services. I would like to see future versions of the DS specification support this specific use case, but I anticipate that will be difficult because of other things in the spec like Component Factories and factory configurations.

**UPDATED**: Heiko Seeberger pointed out that the null-checks in my `addingService` methods were not needed. This makes the code even more concise... thanks Heiko!
