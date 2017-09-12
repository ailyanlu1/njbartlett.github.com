---
layout: post
title: Solving OSGi "Uses" Constraint Violations
summary: Untangling the plate of spaghetti. 
tags: java osgi eclipse
date: 2011-09-02T21:45:00+00:00
---

One of the thornier deployment problems we sometimes come across in OSGi is the dreaded "uses constraint violation". I recently helped a client to solve one of these, and it occurred to me to document the problem and the approach to solving it.

Be warned that this post is quite long and boring. You may not want to read it all right now, but perhaps Google will bring you back here when you first encounter a uses-constraint violation in the wild.

What is a Uses Constraint Anyway??
----------------------------------

You may have already read [Glyn Normington's excellent article](http://blog.springsource.com/2008/10/20/understanding-the-osgi-uses-directive/), which goes into quite a bit of depth on the subject, but regretably without the assistance of pretty pictures, so if you are a visual thinker like me then you might find the following explanation more accessible.

Starting with the basics, in OSGi we have dependencies based on Java packages. Some bundle exports a package, possibly with a version number, and another bundle imports it, possibly specifying an acceptable range of versions:

![](/images/posts/usesconstraint/1.png)

This notation is borrowed from the OSGi specification: a black rectangle is an export and a white rectangle is an import. The surrounding yellow blobs are bundles. The line between the export and the import means that OSGi has chosen that specific export as a match for the import -- they have been "wired" together. So B imports package foo (version 1.0) from A.

Let's build this up. Suppose bundle B, in addition to importing package foo 1.0, also exports package bar (the version of bar is unimportant). The C bundle imports package bar and gets wired to B:

![](/images/posts/usesconstraint/2.png)

So far so good. Now let's complicate things a bit more: let's say that bundle C, in addition to importing package bar, also imports package foo... BUT it imports version 2.0. Wait, we don't have foo version 2.0! Never fear, there's another bundle that does export foo 2.0. We'll call him D:

![](/images/posts/usesconstraint/3.png)

Are you surprised that this works? Perhaps not, you may have heard that OSGi supports multiple versions of the same library at the same time, and here it is in action. However there are limits: while we can have multiple versions of the foo package, we must still be able to construct a consistent "class space" for each bundle that has exactly one version of every class. The "class space" for bundle C is shown by the shaded blue area:

![](/images/posts/usesconstraint/4.png)

Notice how the shaded area avoids the import of foo inside bundle B. This is only possible if the package bar exported by B ~~has no internal dependency on foo~~ does not "expose" foo via its signature (thanks to BJ for this correction).

Exposing foo means that a type in foo is visible through the signature of a type in bar, for example:

{% highlight java %}
package bar;

import foo.Foo;

public class Bar extends Foo {   // exposure via subclassing
    public Foo getFoo();         // exposure via method return type
    public void setFoo(Foo foo); // exposure via method parameter
}
{% endhighlight %}

If package bar *does* expose package foo, as in this example, then we have a "uses constraint". We illustrate this with a little rubber band, like so:

![](/images/posts/usesconstraint/5.png)

Now bundle C cannot be resolved. The rubber band means we cannot exclude the import of foo 1.0 from the blue shaded area, i.e. C's class space must contain foo 1.0. But it is not allowed to contain both foo 1.0 and foo 2.0, so C's second import cannot be satisfied.

I hope this is clear enough, but I'm sure you're wondering: how the hell do I fix this? We should certainly avoid tempting runtime options that turn off the uses constraint validation. Neither should we start mucking about trying to "fix" the manifests of third-party bundles. Both of these are hacks that only mask the problem. After all, package bar in bundle B really does depend on a specific version of foo, so we shouldn't blindly force it to use a different version.

In this example the proper solution is to find an alternative provider of the bar package, one that is compatible with foo 2.0. For example maybe there is a bar version 2.0 available somewhere, that happens to be compatible with foo 2.0. We may need to change the import in bundle C to make sure it gets bar 2.0. If bundle C is our own bundle then this is a reasonable change to make.

How About a Real Example?
-------------------------

Good idea! Let's move away from foos and bars and look at a real uses-constraint problem. We'll take the specific example of the client I mentioned: he was trying to use Apache ActiveMQ from an Eclipse RCP application. In the RCP application bundle there was a single import of the `org.apache.activemq` package. At runtime the RCP bundle failed to resolve with the following error:

{% highlight text %}
!MESSAGE Bundle org.example.rcp.activemq_1.0.0.qualifier [37]
was not resolved.
!SUBENTRY 2 org.example.rcp.activemq 2 0 2011-02-08 14:45:37.513
!MESSAGE Package uses conflict: Import-Package: org.apache.
activemq; version="5.4.2"
{% endhighlight %}

Here we see the most frustrating aspect of uses-constraint violations: the almost total lack of information about the cause of the problem! We need to put our detective hat on.

Our bundle, `org.example.rcp.activemq`, is failing to resolve because it cannot import package `org.apache.activemq` version 5.4.2. That package exists, it is exported by bundle number 33, `org.apache.activemq.activemq-core`:

{% highlight text %}
osgi> packages org.apache.activemq
org.apache.activemq; version="5.4.2"<org.apache.activemq.activemq
-core_5.4.2 [33]>
{% endhighlight %}

The uses constraint violation tells us that something *used by* the package `org.apache.activemq` clashes with something else that is imported into our bundle. We can look at the imports of the ActiveMQ bundle:

{% highlight text %}
osgi> bundle 33
...
Imported packages
  javax.annotation; version="1.0.0"<com.springsource.javax.annotation...
  javax.jms; version="1.1.0"<org.apache.geronimo.specs.geronimo-jms_...
  javax.management; version="0.0.0"<org.eclipse.osgi_3.6.1.R36x_...
  javax.management.j2ee.statistics; version="1.1.0"
       <org.apache.geronimo.specs.geronimo-j2ee-management_1.1_...
  javax.management.openmbean; version="0.0.0"<org.eclipse.osgi_...
  javax.management.remote; version="0.0.0"<org.eclipse.osgi_...
  javax.naming; version="0.0.0"<org.eclipse.osgi_...
  javax.naming.directory; version="0.0.0"<org.eclipse.osgi_...
  javax.naming.event; version="0.0.0"<org.eclipse.osgi_...
  javax.naming.spi; version="0.0.0"<org.eclipse.osgi_...
  javax.net; version="0.0.0"<org.eclipse.osgi_...
  javax.net.ssl; version="0.0.0"<org.eclipse.osgi_...
  javax.security.auth; version="0.0.0"<org.eclipse.osgi_...
  javax.security.auth.callback; version="0.0.0"<org.eclipse.osgi_...
  javax.security.auth.login; version="0.0.0"<org.eclipse.osgi_...
  javax.security.auth.spi; version="0.0.0"<org.eclipse.osgi_...
  javax.sql; version="0.0.0"<org.eclipse.osgi_...
  javax.transaction.xa; version="0.0.0"<org.eclipse.osgi_...
  javax.xml.parsers; version="0.0.0"<org.eclipse.osgi_...
  org.apache.commons.logging; version="1.1.1"<jcl.over.slf4j...
  org.apache.kahadb.index; version="5.4.2"
       <org.apache.activemq.kahadb_5.4.2 [30]>
  org.apache.kahadb.journal; version="5.4.2"
       <org.apache.activemq.kahadb_5.4.2 [30]>
  org.apache.kahadb.page; version="5.4.2"
       <org.apache.activemq.kahadb_5.4.2 [30]>
  org.apache.kahadb.util; version="5.4.2"
       <org.apache.activemq.kahadb_5.4.2 [30]>
  org.osgi.framework; version="1.5.0"<org.eclipse.osgi_...
  org.w3c.dom; version="0.0.0"<org.eclipse.osgi_...
  org.xml.sax; version="0.0.0"<org.eclipse.osgi_...
  org.w3c.dom.traversal; version="0.0.0"<org.eclipse.osgi_...
  javax.xml.stream; version="0.0.0"<org.eclipse.osgi_...
...
{% endhighlight %}

Wow, quite a long list, and I've not even included the optional imports! In the worst case we need to review all of these, and then possibly the transitive dependencies of the bundles that export those packages, in order to find the conflict. Nightmare! Fortunately there is a shortcut we can take.

Recall the conflict in the simplistic example arose because there were two versions of the foo package available. This is a necessary precondition for a uses constraint violation, and it gives us a really big clue:

**Look for packages that have more than one exporter.**

I'd like to be able to say that I jumped straight to the answer at this point, but truthfully I hummed and hawwed a bit first (oh well, it was billable time). I speculated that there might be two copies of the Commons Logging APIs, so I asked the OSGi shell:

{% highlight text %}
osgi> packages org.apache.commons.logging
org.apache.commons.logging; version="1.1.1"<jcl.over.slf4j_1.6.1 [31]>
  org.apache.activemq.kahadb_5.4.2 [30] imports
  org.apache.activemq.activemq-core_5.4.2 [33] imports
{% endhighlight %}

Nope, just one export, and it's imported by both `org.apache.activemq.kahadb` and `org.apache.activemq.activemq-core`. No problem there.

Then I realised: we're running on Java 6! That's significant because Java 6 gratuitously added a bunch of APIs to the base JRE library... APIs that were previously available as optional libraries. This is a rich seam of duplication! So my next guess was `javax.xml.stream`:

{% highlight text %}
javax.xml.stream; version="0.0.0"<org.eclipse.osgi_3.6.1.R36x_... [0]>
  org.eclipse.ui_3.6.1.M20100826-1330 [2] imports
  org.eclipse.core.expressions_3.4.200.v20100505 [7] imports
  org.eclipse.ui.workbench_3.6.1.M20100826-1330 [10] imports
  org.eclipse.core.runtime_3.6.0.v20100505 [11] imports
  org.eclipse.help_3.5.0.v20100524 [23] imports
  org.apache.activemq.activemq-core_5.4.2 [33] imports
{% endhighlight %}

Again no. Cutting to the chase, the culprit was `javax.annotation`:

{% highlight text %}
javax.annotation; version="0.0.0"<org.eclipse.osgi_3.6.1.R36x_... [0]>
  org.eclipse.ui_3.6.1.M20100826-1330 [2] imports
  org.eclipse.core.expressions_3.4.200.v20100505 [7] imports
  org.eclipse.ui.workbench_3.6.1.M20100826-1330 [10] imports
  org.eclipse.core.runtime_3.6.0.v20100505 [11] imports
  org.eclipse.help_3.5.0.v20100524 [23] imports
javax.annotation; version="1.0.0"<com.springsource.javax.annotation >
  org.apache.activemq.activemq-core_5.4.2 [33] imports
{% endhighlight %}

Bingo! The package is exported by two bundles: the first is the system bundle itself, which is responsible for exporting all the JRE packages (aside from `java.*`), and this version is being imported by all the Eclipse plug-ins.

The other version is exported by `com.springsource.javax.annotation`, which is one of the wrapper bundles available from the [repository](https://ebr.springsource.com/repository/app/) hosted by SpringSource. Our RCP application naturally depends on the Eclipse stuff, therefore it cannot get the ActiveMQ library because of this conflict. Time for another pretty diagram:

![](/images/posts/usesconstraint/6.png)

How Can This Be Fixed?
----------------------

Another fine question! In our abstract foo/bar case the solution involved finding a provider of "bar" that was compatible with foo version 2.0.0, and then changing our bundle C to make sure it used that new "bar". In this case, both ActiveMQ and the Eclipse bundle are third-party binaries that we should not change.

Our goal is to make both ActiveMQ and Eclipse import the same version. ActiveMQ cannot import version 0.0.0 from the JRE, but Eclipse can import version 1.0.0.

But wait a moment, there is no such thing as version 0.0.0 of `javax.annotation`! It's nonsense. The OSGi system bundle's job is to export all of the JRE packages but it has no idea what version numbers to use for those exports. Indeed, **nobody** knows what the version numbers of all the JRE packages are. There is documentation for some of them, but none of that documentation is normative, and anyway it doesn't cover everything in the JRE. We really need a JSR or OSGi specification that tells us definitively. Anyhow because of the lack of information, the OSGi framework clumsily exports everything as version 0.0.0.

Therefore the solution involves configuring the OSGi framework in one of the following ways:

-   Removing `javax.annotation` from the exports of the system bundle altogether; this will make both ActiveMQ and Eclipse resolve from the SpringSource bundle.
-   Exporting the `javax.annotation` package from the system bundle as version 1.0.0; then we don't need the SpringSource bundle.
-   Using a special bundle that pulls the `javax.annotation` package from the JRE (using Require-Bundle!) and reexports it as version 1.0.0.

The first two approaches require us to create a "profile", i.e. a list of JRE packages, and pass it to Equinox using the `osgi.java.profile` configuration setting.

The third approach is quite tricky if we have to create the bundle ourselves, but there is an existing bundle in most Eclipse IDE distributions that does exactly this. Unfortunately the bundle doesn't appear in the RCP target platform downloads, which is why my client went to the SpringSource bundle repository to find a provider for the package and ended up in the mess he was in.

What's my recommendation? Really I prefer the second option because the third just feels like a hack to me. There is a further issue though: if you use Eclipse PDE and click the "Validate Plug-ins" button in the Run Configuration dialog, then PDE will report a missing import, because it doesn't know about the JRE profile setting. Clearly a missing feature or even a bug in PDE, but because of this you may prefer the third option.

Another Rant About Require-Bundle
---------------------------------

Notice the arrows in the previous diagram, they indicate a Require-Bundle dependency rather than an import/export wiring. It turns out that Require-Bundle is responsible for this uses constraint problem, so I might as well have another rant about it.

The `packages` command told us that `javax.annotation` from the JRE was imported by Eclipse core runtime, core expressions, UI, workbench, and help. But I happen to know that **none of those bundles actually use the `javax.annotation` package!!** I know this because Eclipse still runs on Java 1.4, where those annotation classes won't even load. But Eclipse uses Require-Bundle as follows:

{% highlight text %}
Require-Bundle: org.eclipse.osgi
{% endhighlight %}

By requiring the system bundle, Eclipse implicitly imports the entire JRE. This is a terrible idea, and it is only done in Eclipse because of backward compatibility and because the PDE tooling is so inadequate for working with Import-Package dependencies.

(In fact, the uses constraint problem in this example can be solved by switching the RCP plug-in to use Import-Package to get hold of its dependencies on the Eclipse APIs. Sadly PDE makes Require-Bundle **so** convenient, and Import-Package so inconvenient, that very users are disciplined enough. I admit that even I use it when I'm feeling lazy.)

In my opinion Eclipse 4 should be used as an opportunity to break with the uglier parts of Eclipse's legacy. It's admittedly hard to do that when you have an ecosystem of literally thousands of 3rd party plug-ins that would stop running on the new version... but must this crap really be maintained forever? Will Eclipse 5 make the break, or Eclipse 6? A big part of what ails Java itself is the legacy compatibility burden; I hope Eclipse can be bolder.

Anyway, ranting over, here comes the recommendation:

**Eradicate Require-Bundle dependencies from your bundles.**

Installation Order
------------------

Consider the following modification to the original abstract scenario. Bundle B now accepts a range for the foo package: either version 1 **or** 2. If these bundles are all installed at the same time, then C is now resolvable because both C and B can wire to the version 2.0 foo offered by bundle D:

![](/images/posts/usesconstraint/7.png)

In this scenario bundle A will be ignored. However now suppose that only bundles A and B are available at startup, with C and D being installed later.

At startup B will resolve to the single version of foo available, which is version 1.0.0 from bundle A. When C and D are installed, C will be unresolvable, since it cannot import bar from B at the same time as it imports foo from D. OSGi tries to keep the existing resolved bundles stable, so it does not force B to rewire its import of foo to the new version.

![](/images/posts/usesconstraint/8.png)

This behaviour can be counter-intuitive. OSGi **never** dynamically rewires imports when new bundles are installed, it only tries to find wirings for the newly installed bundles. This is because rewiring is expensive: it can force the shutdown and restart of many bundles, and just calculating all the wirings again takes some time ([it is NP-complete](http://stackoverflow.com/questions/2085106/is-the-resolution-problem-in-osgi-np-complete)).

We can force OSGi to rewire everything by issuing the `refresh` command at the shell. If we do this, B's wiring will switch to use foo 2.0 from D, and C will resolve. Therefore to check whether you have a problem with installation order:

**Just type `refresh` and see whether the problem goes away**.

Any More General Advice?
------------------------

Solving a uses-constraint violation requires us to find an accurate diagnosis first, before we attempt to prescribe a cure.

Once the diagnosis is confirmed, seek a cure based on configuring the platform rather than messing with bundle manifests: uses-constraint violations are signals of an invalid deployment, i.e. a set of bundles that cannot work together. They are usually not signs of a development problem to be fixed within the bundles themselves -- unless the bundles can be shown conclusively to have inaccurate or invalid manifests.

Regarding diagnosis, a necessary precondition for any uses-constraint violation is multiple exporters of the same package. So the first question to ask ourselves is:

**Are there any obvious sources of multiple exports of the same package?**

For example, have we included two versions of the same bundle? Are there bundles with overlapping exports?

Using the `packages` command in the Equinox shell can help to find multiple exporters, and tells us which bundles are using each export.

**If running on Java 6, could one of the new JRE packages be the source of the problem?**

Here's a list of all the packages in Java 6 that were not in Java 5:

{% highlight text %}
javax.activation                    javax.xml.crypto.dom
javax.annotation                    javax.xml.crypto.dsig
javax.annotation.processing         javax.xml.crypto.dsig.dom
javax.jws                           javax.xml.crypto.dsig.keyinfo
javax.jws.soap                      javax.xml.crypto.dsig.spec
javax.lang.model                    javax.xml.soap
javax.lang.model.element            javax.xml.stream
javax.lang.model.type               javax.xml.stream.events
javax.lang.model.util               javax.xml.stream.util
javax.script                        javax.xml.transform.stax
javax.tools                         javax.xml.ws
javax.xml.bind                      javax.xml.ws.handler
javax.xml.bind.annotation           javax.xml.ws.handler.soap
javax.xml.bind.annotation.adapters  javax.xml.ws.http
javax.xml.bind.attachment           javax.xml.ws.soap
javax.xml.bind.helpers              javax.xml.ws.spi
javax.xml.bind.util                 javax.xml.ws.wsaddressing
javax.xml.crypto                    org.w3c.dom.xpath
{% endhighlight %}

With regard to prescribing a cure, the general approach is to find an alternative bundle or set of bundles that can provide the dependencies we need.

**Does an alternative non-conflicting source for the dependency exist?**

**If JRE packages are involved, can the additional exporters be removed? Try specifying package versions in the Java profile.**

If the JRE is not involved and no alternative supply of the dependencies can be found, then maybe these bundles really cannot be used together: they are simply incompatible. This is bad news to be sure, but it's better to find it out early from the OSGi resolver than from weeks of debugging a misbehaving runtime.

Does It Really Have to Be This Hard?
------------------------------------

I hope not! Clearly this kind of analysis is still based on too much in-depth knowledge of OSGi's internal workings, which puts it beyond the capabilities of most developers. This could be a real problem for adoption of OSGi in complex enterprise applications.

As ever, we need better tools. My dream is to create tooling in [Bndtools](http://bndtools.org) that will help to quickly diagnose and offer fixes for problems like this. Here I'd like your suggestions... what kind of tooling would help?

I'm thinking about some kind of graphical tool that can build diagrams on the fly similar to those in this blog post, including the "class space" visualisation. My concern is that it might not scale to a realistic scenario: perhaps the diagrams would just be too big and complex to be any use. I'm just not sure at this point.

Anything Else to Say?
---------------------

Yes: well done for making it all the way through, and happy hunting.
