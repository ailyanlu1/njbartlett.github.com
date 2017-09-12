---
layout: post
title: The JDK8 Module System Requirements
summary: My take on things
tags: java osgi eclipse
date: 2011-05-27T06:30:00+00:00
---

> "There's something very important I forgot to tell you. Don't cross the streams... It would be bad... Try to imagine all life as you know it stopping instantaneously and every molecule in your body exploding at the speed of light."
> -- Egon Spengler (Harold Ramis), <a href="http://www.imdb.com/title/tt0087332/quotes">Ghostbusters</a>.

I have reviewed the <a href="http://openjdk.java.net/projects/jigsaw/doc/draft-java-module-system-requirements-12">JDK8 Module System Requirements</a> document just published by Mark Reinhold.

The first thing to note is that this is a *much* more positive and constructive document than anything I have seen previously from Mark on this subject. At last the need to interoperate with the existing standard for Java modularity -- that is, OSGi -- is recognised and specified as a non-negotiable requirement. So that's excellent news.

The second point of note is that the requirements for this new module system are really not that far from OSGi itself! Aside from cosmetic differences, such as the location and format of module metadata, the general goals are quite closely aligned and achievable with OSGi. For example, I think that if there were a strong desire to encode metadata in a file named "module-info.java" instead of a file named "MANIFEST.MF" then an agreement could be reached within the auspices of the OSGi Alliance.

In other areas, the proposed module system -- which I'll call "Jigsaw" for now, though it's unclear if that will be the proper name going forward -- is a more permissive superset of OSGi and would be compatible with it. For example it imposes no semantics on versions except that they must be totally ordered, whereas OSGi additionally specifies how many version segments should be used and what are the semantics of an increment in each segment. OSGi's semantics provide excellent guidance for developing and evolving software, whereas Jigsaw's are more adaptable to the version schemes used by existing software (actually most existing software uses insane, arbitrary and inconsistent versioning schemes that are driven more by Marketing than by reality). Nevertheless this represents progress, since previous Jigsaw proposals did not even allow for two versions to be compared with each other.

So why create a new module system at all? Why not just implement a few small changes in OSGi to support these new requirements? After all, Oracle and IBM working together probably have the clout within the OSGi Alliance to achieve this, whereas Sun working alone did not.

There is to my mind one big difference that justifies a continued separation. Jigsaw will be used to modularise the Java platform itself, which means it must have features capable of dissecting the tangled mess of packages in the JRE standard library. Even I, a total OSGi bigot, have accepted for some time now that a non-OSGi solution may be necessary for this one specific use-case. That is because, in the Java world, the JRE is unique.

The JRE is an old API that has evolved organically and with very little planning, and as a result it is a tangled mess that cannot be separated cleanly along package lines. In this regard at least, it is not unique at all, but depressingly normal. However when other APIs get into this state we refactor them... which leads to breakages of downstream libraries and applications, but we can cope if we have a functional version system. Unfortunately refactoring the JRE now would literally **break every Java application and library in the entire world**. So we need a way to modularise without moving classes between packages. In other words we need to split packages across module boundaries.

It's a common misconception that OSGi does not support split packages; it does. There is a form of module dependency called Require-Bundle in which the depending bundle sees an aggregate view of all the packages exported from its dependencies. The thing is, splitting packages is usually such a bad idea that any OSGi egghead will simply tell you: **don't split the packages**.

Don't split the packages, and **don't cross the streams**... except right at the end of the movie, when the Ghostbusters do cross the streams because they're in the one situation where nothing else will work.

One reason why split packages are bad is because they break Java accessibility rules. Remember that default accessibility in Java is "package private": a member or a type that can only be accessed by other types in the same package. The current <a href="http://java.sun.com/docs/books/jvms/second_edition/html/ConstantPool.doc.html#75929">JVM specification (section 5.4.4)</a> says that types or members with default accessibility can be accessed by other members of the same **runtime** package, where the runtime package is defined as the combination of the package name and the class loader. In OSGi each module has its own class loader, so the runtime package of classes in each module is always different, even if some classes have the same package name. So applying OSGi with Require-Bundle to the JRE would result in IllegalAccessErrors.

Jigsaw's solution is to provide a mechanism -- the details of which are not yet specified -- for certain modules to share a class loader. Remarkably, this is really the only major requirement in the Jigsaw document not already supported by OSGi. It would fix the accessibility problem, and also certain other problems where a class expects to be able to load other classes dynamically through its own class loader.

However, as necessary as it may be, Jigsaw's shared-class-loader solution creates more problems than it solves. OSGi's class loaders serve to create a barrier between modules, allowing private implementation to be hidden. Jigsaw tears down those boundaries, allowing non-modular code to be repackaged into module-like deployment units. This allows it to be applied to the toughest of legacy code, but at the same time it does nothing to discourage the ill-disciplined development practices that got us into this mess.

It's like trying to lose weight through liposuction, rather than through diet and exercise.

Also the access rule problem is far from the only issue arising from split packages. See the OSGi core specification, section 3.12.3, for some more good reasons not to do it. Admittedly none are as bad as making your molecules explode at the speed of light, but they're pretty bad.

Of equal concern is tooling: I believe that tooling to develop and manage modules with fine-grained sub-package splitting will be extremely complex and nigh unusable. In OSGi with our focus on coherent packages, it has still taken us a very long time to develop tools that are easy to use for the majority of developers. This is not due to accidental complexity in OSGi, but inherent complexity in developing modular, reusable software. A module system that encourages split packages will require tools that can manage modularity at the level of types rather than packages. How many types are there in an average Java package... twenty, thirty? That is how many **times** more difficult it will be to design and assemble coherent modules with Jigsaw.

Sadly when you put a powerful yet dangerous tool in the hands of developers and warn them to never, ever use it except as a last resort, many of them use it as a first resort just to prove what geniuses they are. Lest you think I'm being sanctimonious, I know that I'm guilty of riding this power trip as well, from time to time.

So while I accept the need for Jigsaw in modularising the Java platform, and am happy for certain Jigsaw requirements to migrate into OSGi, I remain opposed to its use anywhere else. OSGi's Module Layer provides all the power anybody could reasonably need to build highly modular software, and I have not even talked yet about the OSGi Service Layer, where OSGi's power really shines -- something entirely absent from Jigsaw.

OSGi has rules, and those rules provide guidance and a framework for creating modular software. Jigsaw's permissiveness will be seductive because it will allow painful decisions to be deferred indefinitely. Its additional power will cause great damage if applied to the wrong problems in the wrong way. No wonder it is named after a tool notorious for chopping off fingers.
