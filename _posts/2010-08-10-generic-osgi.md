---
layout: post
title: Generic OSGi
summary: Java Generics added to the OSGi R4.3 APIs
tags: osgi, eclipse
---

The next version of the OSGi specification — Release 4.3 — will finally support Java Generics. Neither the specification nor the RFC working document are publicly available yet, but the new API has recently been <a href="https://bugs.eclipse.org/bugs/show_bug.cgi?id=322007">checked into</a> Eclipse Equinox for all to see.

The most noticeable changes are to do with services, naturally. Both the `ServiceReference` and `ServiceRegistration` classes now have a single parameter for the service type, and there are new methods on `BundleContext` that take `Class<S>` to produce objects of those types. These new methods augment the old String-based methods that now return `ServiceReference<?>` and `ServiceRegistration<?>`.

But the biggest single set of changes is on the `ServiceTracker` class, which now takes **two** type parameters, `S` and `T`, where `S` is the type of service being tracked and `T` is the type of objects being produced.

This split has always been an important aspect of `ServiceTracker`, but it was not obvious before. When a tracker is used in its default form (i.e. without overriding or customising) the two type parameters are the same: we track a service of type `S` and then typically call `getService()` to get the current instance of type `S`. But the call to `getService()` in fact gives us whatever the tracker's `addingService()` method returns, and we can override that.

So in a sense, `ServiceTracker` is a **transformer** for services.

For example, the first code sample in my <a href="/2010/08/05/when-servicetrackers-trump-ds.html">previous post</a> was a `ServiceTracker` that transformed a service of type `IA` into a `ServiceRegistration<IC>`... but you would have had to read the code very carefully to realise that. The `ServiceRegistration` type wasn't even mentioned in the `addingService()` method, so you needed to know that was the return type of `registerService()`.

Here is what the same code will look like in OSGi R4.3 (N.B. I haven't actually compiled and tested this, but it should work):

{% highlight java %}
public class ForeachA
             extends ServiceTracker<IA, ServiceRegistration<IC>> {

    public ForeachA(BundleContext ctx) {
        super(ctx, IA.class, null);
    }

    @Override
    public ServiceRegistration<IC> addingService(
	                               ServiceReference<IA> ref) {
        IA a = context.getService(ref);
        return context.registerService(IC.class, new C(a), null);
    }

    @Override
    public void removedService(ServiceReference<IA> ref,
                               ServiceRegistration<IC> reg) {
        reg.unregister();
        context.ungetService(ref);
    }
}
{% endhighlight %}

Unfortunately this a little bit more verbose than last time but it is safer due to the lack of casts, and more importantly the purpose of the class is clearly stated on the first line, thanks to the type parameters.
