---
layout: post
title: OSGi Compliance Levels
summary: Just what does "OSGi Compliant" mean, anyway?
tags: osgi
flattr: http://njbartlett.name/2010/08/24/osgi-compliance-levels.html
---

Almost every week, I hear about new and existing Java libraries or frameworks becoming "OSGi Compliant". This makes me happy as it indicates the traction that OSGi is getting as a platform. The only problem is that "OSGi Compliant" could mean almost anything... and therefore it means precisely nothing.

For example, many libraries can be easily used in OSGi even if they don't provide an OSGi descriptor (i.e., a `MANIFEST.MF` containing `Bundle-SymbolicName` etc.), so long as they steer clear of certain problematic coding patterns such as classpath scanning, unconstrained use of `Class.forName()` and so on. Such libraries could reasonably be termed "compliant". But another framework may have been deeply integrated into OSGi so that it can make use of services, receive configuration through Config Admin, and so on.

It seems that we need a vocabulary that will help us distinguish these two cases, and those in between. We need to define, at least informally, some kind of Beaufort Scale for levels of OSGi compliance (if you're feeling uncharitable towards OSGi you may prefer to call it a <a href="http://en.wikipedia.org/wiki/Schmidt_Sting_Pain_Index">Schmidt Pain Index</a>).

The purpose of the exercise would certainly not be to admonish authors of libraries that rate near the bottom of the scale, nor to congratulate those at the top. Rather, it would help authors to communicate the depth of their support for OSGi in a more precise way, and it would help the OSGi community communicate to library authors how they can make their products more useful to us.

To get the ball rolling, here are my initial thoughts. I do this merely to invite feedback rather than to attempt to impose my own classification.

| Level | Name           | Notes                                                                                                                                                                                                                                                                                                                                               |
|-------|----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| -1    | Non-compliant. | Difficult or impossible to use in OSGi. May be possible to use via an adaption layer or patching of the sources or binaries. For example, the library may rely on classpath scanning or invalid assumptions about the classloading hierarchy.                                                                                                       |
| 0     | Neutral.       | Ships as a plain (i.e., non-bundle) JAR, but can be used straightforwardly in OSGi if wrapped as a bundle or added to `Bundle-ClassPath`.                                                                                                                                                                                                           |
| 1     | Compliant      | Shipped as one or more OSGi bundles with correctly defined `MANIFEST.MF`s. All dependencies are also Compliant and either shipped with the library or available separately from a repository. Package versions follow the <a href="http://www.osgi.org/wiki/uploads/Links/SemanticVersioning.pdf">Semantic Versioning guidelines</a> (pdf warning). |

For many libraries, there is no need to go any higher than level 1. The higher levels are applicable to frameworks, which are more complex and may feature extensibility through plug-ins, custom configuration, logging, etc. Obviously I would like to see **all** libraries reach at least level 1, but the OSGi community needs to do more to make this easier.

| Level | Name          | Notes                                                                                                                                                                                                                                                                                                                                                                      |
|-------|---------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 2     | OSGi-Assisted | The framework is aware of OSGi, either in its core or via optional modules. Areas of the framework that are open to extensibility can use services for this, in addition to other mechanisms. The framework can cope with service dynamics without restarting the JVM.                                                                                                     |
| 3     | OSGi-Powered  | The framework is based on OSGi. It publishes services that may be used by other parts of our system. Its own services may be replaced by our own service implementations. It takes advantage of standard OSGi services such as Log Service, Config Admin, Event Admin, HTTP Service. However, despite all this it may still provide a way to run in non-OSGi environments. |

A key concern for many frameworks as they seek to rise this pyramid is to remain usable outside OSGi, which is a goal I support. There appears to be very little information available on how to develop frameworks that are able to take advantage of OSGi without introducing a hard dependency on it. This is something I hope to remedy in future posts.

What do you think of this taxonomy? Is it useful, or a waste of time? Should there more levels, or fewer? Please comment below, or link to your own blog.

**UPDATE** <a href="http://tux2323.blogspot.com/2010/08/osgi-bundle-quality-or-bundle.html">Christian Baranowski blogged</a> some thoughts about independent verification of bundle compliance. That's a great idea, but my initial goal is less ambitious, i.e. simply to allow library authors to assert something meaningful about their level of OSGi compliance. I think that is a prerequisite before looking to verify those assertions.

**UPDATE 2** BJ makes some excellent points in the comments below, which I entirely agree with. Be sure to scroll down and take a look.
