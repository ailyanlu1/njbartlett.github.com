---
layout: post
title: How To Embed OSGi
summary: A quick tutorial
tags: java osgi eclipse
date: 2011-07-03T09:00:00+00:00
---

A popular question on mailing lists and [Stack Overflow](http://stackoverflow.com/questions/4673406/programatically-start-osgi-equinox) is how to embed OSGi in a larger application. This is really easy to do, especially since a standard API for it was introduced in OSGi Release 4.1. Because of the standard, switching between Equinox, Felix and Knopflerfish can be as simple as a configuration change.

(Tangent: another popular question is "which OSGi framework implementation should I choose?". The only possible answer is: "it depends". Each framework has different non-functional characteristics, e.g. with respect to memory consumption, scalability, and so on. By sticking to the standard and remaining agnostic, we can develop our entire application and then test which framework best fits our specific needs).

Here's how to do it.

Create and Start the Framework
------------------------------

The framework launching API uses the Java SPI mechanism to load a "framework factory". Assuming we have some R4.1-compliant OSGi framework on the classpath, the following will work:

{% highlight java %}
FrameworkFactory frameworkFactory = ServiceLoader.load(
                     FrameworkFactory.class).iterator().next();
Map<String, String> config = new HashMap<String, String>();
// TODO: add some config properties
Framework framework = frameworkFactory.newFramework(config);
framework.start();
{% endhighlight %}

We now have an OSGi framework running, but it's not very useful until we add some bundles. Let's do that next.

Add Bundles
-----------

The `Framework` object that was returned from `newFramework` is actually a sub-interface of `Bundle`, from which we can get a `BundleContext`. If you have ever written a bundle activator then you are familiar with `BundleContext` already: it's the central point of access to the entire OSGi API.

Let's install the Felix Shell bundles (NB these bundles work just fine on Equinox and Knopflerfish also):

{% highlight java %}
BundleContext context = framework.getBundleContext();
List<Bundle> installedBundles = new LinkedList<Bundle>();

installedBundles.add(context.installBundle(
                     "file:org.apache.felix.shell-1.4.2.jar"));
installedBundles.add(context.installBundle(
                     "file:org.apache.felix.shell.tui-1.4.1.jar"));

for (Bundle bundle : installedBundles) {
    bundle.start();
}
{% endhighlight %}

Note that as a general principle, when we want to install a set of bundles we should install them all **first** and then start them. If you tried to install and start them individually then you would have to be careful about ordering... much better to let the framework sort it all out!

Also bear in mind that there is no error handling in the above code. To improve this we should catch any exceptions on install or start, so that we can still install/start the other bundles.

**UPDATE:** another thing to be careful of is fragment bundles. Fragments cannot be started, so the `bundle.start()` line will throw an exception if the installed bundle was a fragment. You may want to add the following check to avoid those exceptions:

{% highlight java %}
if (bundle.getHeaders().get(Constants.FRAGMENT_HOST) == null)
	bundle.start();
{% endhighlight %}

Handling Shutdown
-----------------

Once the framework is running with the bundles you want, you should just let it run until it finishes. Usually the shutdown signal should come from within OSGi, for example the user might type `shutdown` at the Felix shell. This would stop the framework, and you would normally want to do something after this has happened.

For example, if you are writing a straightforward launcher then you probably want to call `System.exit()` after OSGi stops. The cleanest way to do this is as follows:

{% highlight java %}
try {
    framework.waitForStop(0);
} finally {
    System.exit(0);
}
{% endhighlight %}

The `waitForStop` method simply blocks the main thread until OSGi stops, so it is the last thing we do after performing all of our initialisation steps. We can't just allow the main thread to finish and expect the JVM to shut down, because any of the bundles in the OSGi framework might have started a non-daemon thread, so we need `System.exit()` in order to actually shut down.

Exposing Application Packages
-----------------------------

When embedding OSGi we might want the bundles inside the OSGi framework to have visibility of types from the parent application. The way to do this is to expose those types as exports of the system bundle; then the bundles that wish to see those types can import them with `Import-Package` in the usual way.

For example suppose our parent application contains a domain model in the package `org.example.mydomain`. This package needs to be added to the system bundle exports, so go back to the first code sample and insert the following where we had the TODO marker:

{% highlight java %}
config.put(Constants.FRAMEWORK_SYSTEMPACKAGES_EXTRA,
           "org.example.mydomain");
{% endhighlight %}

The bundle that uses this package should have the following in its manifest:

{% highlight text %}
Import-Package: org.example.mydomain
{% endhighlight %}

The framework will wire up this import to the export offered by the system bundle. Note that the importing bundle doesn't care that the package comes from the system bundle, it could just as easily import from an ordinary bundle! This gives us the flexibility to refactor our application. In the future we might want to do more in OSGi, so the domain model packages would be inside an ordinary bundle in OSGi rather than outside it.

The technique works for any package that is on the classpath of the launcher class. I once had a requirement to embed OSGi in a J2EE application server and expose some EJBs to the OSGi bundles: so `javax.ejb` and a few other packages had to go on the system bundle exports. The EJB instances were published into OSGi as services.

Other Configuration
-------------------

There are other configuration changes that can be made in a standard way using the properties you pass to `newFramework`. Here are a couple of the more useful ones:

{% highlight java %}
// Control where OSGi stores its persistent data:
config.put(Constants.FRAMEWORK_STORAGE, "/Users/neil/osgidata");

// Request OSGi to clean its storage area on startup
config.put(Constants.FRAMEWORK_STORAGE_CLEAN, "true");

// Provide the Java 1.5 execution environment
config.put(Constants.FRAMEWORK_EXECUTIONENVIRONMENT, "J2SE-1.5");
{% endhighlight %}

Of course there are a few framework-specific properties as well, for example:

{% highlight java %}
// Turn on the Equinox console on port 1234 (Equinox only)
config.put("osgi.console", "1234");
{% endhighlight %}
