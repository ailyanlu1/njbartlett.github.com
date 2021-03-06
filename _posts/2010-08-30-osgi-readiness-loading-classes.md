---
layout: post
title: OSGi Readiness — Loading Classes
summary: Loading classes at runtime without breaking OSGi
tags: osgi
flattr: http://njbartlett.name/2010/08/30/osgi-readiness-loading-classes.html
---

In my previous post on "OSGi Compliance", I discussed the idea of making libraries ready for OSGi without depending on OSGi directly. In light of BJ's comment I will refer from now on to OSGi "Readiness" rather than Compliance, as the latter term is easily confused with the concept of a framework implementation that complies with the OSGi specification.

In each of these blog posts, I will state a requirement and then look at options for satisfying that requirement in an OSGi Ready way.

Problem Statement
-----------------

You are developing a Java library, "Framework X". Occasionally you must load and instantiate a class dynamically by name, where the name may originate from a configuration file or be passed into the framework as a parameter to an API call. You would like the framework to work cleanly both in OSGi and other environments, i.e. there must be no dependency on OSGi.

For example, suppose we are implementing an O/R mapping framework similar to Hibernate, and the user has supplied a configuration file such as the following:

{% highlight xml %}
<or-mapping>
   <class name="org.example.domain.Event" table="EVENTS">
      <id name="id" column="EVENT_ID"/>
   </class>
</or-mapping>
{% endhighlight %}

This tells our framework to create an instance of `org.example.domain.Event` for each row of the EVENTS table. Therefore we must dynamically load and instantiate the `org.example.domain.Event` class.

Discussion
----------

To dynamically load a class in Java we require **two** things: the fully qualified name of the class, and a `ClassLoader` from which to request it. If we have both of these then we can trivially load a class irrespective of whether we are in OSGi or not.

To reiterate: **OSGi does not interfere with the standard Java class loading API**. If we have the fully qualified name of a class and a `ClassLoader` that knows of that class, then we can always load and instantiate the class.

Problems arise only when we do not know the correct `ClassLoader`, and make the wrong guess about which one to use.

### First Wrong Guess: Class.forName()

Java provides us with a way to load a class without explicitly specifying a `ClassLoader`, using the single-argument version of the `Class.forName()` method:

{% highlight java %}
Class clazz = Class.forName("org.example.domain.Event");
{% endhighlight %}

Since we don't specify a `ClassLoader`, this method uses the loader that loaded the class in which the line of code appears. In other words, it uses the same loader that loaded the framework. In a traditional Java SE environment this usually works just fine, because both the framework and domain classes are probably installed as JARs on the classpath, and the classpath is loaded by a single `ClassLoader`, the so-called "system" loader.

But in OSGi, the domain classes are likely to be supplied in a different bundle, because there is a clear modular separation between the O/R framework code and the domain model classes, and therefore they will be loaded by a different `ClassLoader`. We can still make it work if the package `org.example.domain` is listed as an import of the framework bundle like so:

{% highlight make %}
Import-Package: org.example.domain, ...
{% endhighlight %}

However this is unsatisfactory because `Import-Package` is statically defined, and a general-purpose O/R framework does not know in advance which domain packages it will need to import.

(**NB** there are a couple of other workarounds for this problem in the case that the framework code cannot be changed. These include: `DynamicImport-Package`; shipping the domain classes as a fragment hosted by the O/R framework; and using the Equinox-only "buddy loading" feature. However all of these workarounds have their own severe problems, which I shall not discuss in depth here because the purpose of the post is to document how to best implement a framework such that hackish workarounds are **not** required.)

### Second Wrong Guess: Thread Context Class Loader

The `Class.forName()` approach fails not only in OSGi but also in other environments such as Java Enterprise Edition, for similar reasons. Because of this, another common pattern is to use the Thread Context Class Loader (TCCL), which was introduced relatively recently (i.e. Java 1.2):

{% highlight java %}
ClassLoader tccl = Thread.currentThread().getContextClassLoader();
Class clazz = Class.forName("org.example.domain.Event", true, tccl);
{% endhighlight %}

Some libraries use the TCCL exclusively, others use it as a last resort after failing to load from other candidate class loaders. Almost every library follows a different strategy because Java has provided almost no specification or guidance on when to use the TCCL... to quote a <a href="http://www.javaworld.com/javaworld/javaqa/2003-06/01-qa-0606-load.html">classic JavaWorld article</a>:

> Those class and resource loading strategies must be the most poorly documented and least specified area of J2SE.

Even in J (2)EE the specifications do not lay out precisely which set of classes should be visible through the TCCL. In practice, a J (2)EE application server is able to provide a sensible set of classes via the TCCL because the programming model is highly constrained: each application runs in an isolated silo, is not allowed directly to create threads or network sockets, and threads cannot cross application boundaries. Because it controls all the entry points, the application server can provide a TCCL that is appropriate for the running code.

OSGi's programming model is far less constrained. Bundles are free to create threads, to open network sockets, and to call APIs exposed by any other bundle. There is no way in Java to intercept a direct method call that happens to cross a bundle boundary, so the OSGi framework cannot ensure the TCCL is relevant to the current code. As a result, the TCCL is not defined by the OSGi specification, and it may be `null`. Therefore, we cannot rely on the TCCL for anything.

Solutions
---------

Now let's discuss possible solutions. To be clear, all of the following suggestions are intended to supplement rather than replace existing approaches. **If your framework is working in J2SE/J2EE with `Class.forName()` or TCCLs then you should continue to use those approaches**. But, please provide additional options for runtime environments such as OSGi where those approaches fail.

If you feel these suggestions are trivial or obvious — that is exactly the point. There is nothing complicated about making frameworks that work well in OSGi. Indeed, you may find you prefer using one of these patterns as they reduce the complexity and testing burden of your code.

### Option 1 — Instance Factory

The first option is to avoid dynamic by-name class loading altogether. Although not always practical, it may be possible in some cases to allow clients to supply their own callback or factory to create objects as required.

For example our O/R mapper could define the following interface:

{% highlight java %}
public interface DomainObjectFactory {
    Object createInstance(String tableName);
}
{% endhighlight %}

Clients could supply an instance of this factory when initialising a session, or we could wire it in with our favourite Dependency Injector. This has the advantage that clients can construct objects however they like, rather than placing restrictions on the permitted constructors in the domain classes.

On the other hand it may be risky since a client factory implementation could return shared instances, whereas the framework expects new objects on each call to `createInstance()`. So long as such requirements are documented, the framework should permit clients to do what they like.

### Option 2 — Register Classes

Another option is to add API for clients to register loaded classes in advance. This is best illustrated in client code:

{% highlight java %}
session.registerClassForTable("EVENTS", Event.class);
List events = session.createQuery("from Event").list();
{% endhighlight %}

Here the client code has direct visibility of the `Event` domain class, so it can use a class-literal to pass a `Class` to the framework API. The framework can now use the supplied class whenever it reads from the EVENTS table.

We can even register classes directly against their names:

{% highlight java %}
session.registerClass("org.example.domain.Person", Person.class);
{% endhighlight %}

The framework can use the registered class whenever it needs to create an instance of `org.example.domain.Person`, but fall back on `Class.forName()`/TCCL for unregistered names. No other changes are required in the framework.

Note that with this option, the client loses some flexibility and control. By passing a class rather than instances, the client has no opportunity to construct objects in the way it wants them, but instead relies on the framework to construct them.

This is not so bad in the running example of domain objects, but suppose we are talking about some kind of "service" object. Without control over instantiation, clients cannot make the service aware of its "environment" or "context". Sometimes this forces us to use static fields, thread-local variables or other hack to get hold of our "context" after having been instantiated by a framework.

If the framework allows pre-instantiated objects, then those objects can be pulled out of the OSGi service registry, a Dependency Injection container, or even generated on the fly. So when designing an API, please consider whether the framework really needs to take control of instantiation by requiring `Class` objects — if not, at least allow the option of passing pre-instantiated objects.

### Option 3 — Pass the Loader

This option is the conceptually the simplest, and also has the least impact on existing code: allow clients to pass a `ClassLoader` into your API. The appropriate time to do this is highly dependent on the nature of the API, but in the O/R mapper it might be done during initialisation of a session:

{% highlight java %}
SessionFactory.createSession(MyClass.class.getClassLoader());
// ... OR ...
session.setDomainClassLoader(MyClass.class.getClassLoader());
{% endhighlight %}

To minimise the impact on existing code, this method can be overloaded with another version that takes no argument. Alternatively, clients could pass `null` to indicate that the framework should attempt to find the right `ClassLoader`.

Again there is a loss of flexibility for clients since this solution assumes all classes can be loaded from one class loader, which may not actually be the case, but in practice it doesn't create any real problems.

Summary
-------

Although this post has been rather long-winded, the message is very simple: **stop guessing**.

**If you are writing a library that performs dynamic by-name class loading, do not make assumptions about which `ClassLoader` to use.** Allow client code to specify a class loader, or perform its own class loading. By all means retain your `Class.forName()` or TCCL-based lookups, but please provide an override for environments in which they will not work.
