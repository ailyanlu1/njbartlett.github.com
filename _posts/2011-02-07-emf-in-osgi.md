---
layout: post
title: Using EMF in OSGi
summary: Modular modelling? 
tags: java emf osgi eclipse
date: 2011-07-02T14:30:00+00:00
---

It's official -- I ♥ [EMF](http://www.eclipse.org/emf)!

This is a surprise to me. I previously regarded modelling in general and EMF specifically as a sort of cult... obscure jargon, terrible documentation, unbearably aloof and patronising [talk abstracts](http://www.eclipsecon.org/2009/sessions?id=358..). but all that's behind me now. I see the light: it's all about the industrialisation of software, which requires both *componentisation* and *automation*. OSGi provides admirably for the first, and perhaps EMF can provide for the second.

However I wouldn't say I'm an *entirely* orthodox EMFite. I've had to hack it about a bit, primarily to get it to work properly OSGi. At first blush using EMF in OSGi may seem trivial, after all, Eclipse is an OSGi application and EMF is designed to work in Eclipse, so EMF must work well in OSGi. Also: all cats have four legs; my dog has four legs; therefore my dog is a cat. Ahem.

![](/images/posts/emfosgi/semiskimmed.jpg)

EMF supports running in two distinct environments, neither of which matches my requirements. The first is in a "full-fat" Eclipse SDK or Eclipse RCP application. The second is in a traditional non-OSGi Java runtime, which the EMF docs refer to as "standalone". However the middle ground -- an OSGi environment that is not Eclipse -- is not supported. That is, you cannot drop EMF into a lightweight OSGi runtime, even allowing for a few dependencies. It depends on the following eight bundles as an absolute minimum:

-   org.eclipse.core.runtime
-   org.eclipse.equinox.common
-   org.eclipse.core.jobs
-   org.eclipse.equinox.registry
-   org.eclipse.equinox.preferences
-   org.eclipse.core.contenttype
-   org.eclipse.equinox.app
-   org.eclipse.osgi

The last one is the killer: Equinox itself, meaning that EMF will run only on Equinox, not any other OSGi framework implementation such as Apache Felix or Knopflerfish.

There's something fishy going on, though. How can EMF run in a "standalone" non-OSGi environment -- with *no* other JARs on the classpath -- but not run in Felix??

![](/images/posts/emfosgi/deps.png)

The explanation is that the EMF bundles have poor internal cohesion. In fact only the activators have a hard dependency on Eclipse classes; other parts of EMF reference them only by reflection. Since bundle activators are ignored in non-OSGi runtimes, EMF works fine there without having to put big chunks of Eclipse on the classpath. However it fails in an OSGi environment where the activator is not ignored -- actually it fails during resolution, long before the activators are triggered. As far as I can tell the activators do nothing that is useful outside of Eclipse.

Hopefully the diagram clarifies the problem. We want the green bits but not the red bits; since the green is packaged into the same bundle as part of the red, we have to get all of the red.

The worst consequence is that if you use EMF to generate code for your model then your model API -- even the *interfaces*! -- will be inextricably linked to EMF and will therefore be encumbered with all those dependencies as well. As [Alex discovered](http://alblue.bandlem.com/2010/11/using-emf-for-osgi-service-creation.html), the links to EMF can be minimised but not completely eliminated, and anyway to remove them would be to throw away some of the most useful functionality of EMF.

Frankly this is the kind of screw-up that makes me want to strangle Eclipse, if only it didn't have so many necks. But can it be fixed?

If backwards compatibility were not a concern then the fix would be very simple: first, refactor the activator out into a separate bundle; then mark the dependencies optional. However, backwards compatibility is **always** a concern, so the EMF project is [unlikely ever to do this](https://bugs.eclipse.org/bugs/show_bug.cgi?id=328227).

The alternative is to repackage EMF into new bundles with better OSGi manifests. So that is what I've done, and I'm making them available so that others can enjoy the benefits of EMF without hitching themselves to the Eclipse juggernaut. [Bnd](http://www.aQute.biz/Code/Bnd) makes it so simple to slice n' dice the JARs into proper bundles. Everything's available from my GitHub page at <http://github.com/njbartlett/emf-osgi>.

It's a work in progress; the bundles available so far are as follows:

-   `name.njbartlett.osgi.emf.minimal` contains what appears to be the minimal set of packages required for any EMF-based model. This corresponds to the original `org.eclipse.emf.ecore` and `org.eclipse.emf.common` bundles.

<!-- -->

-   `name.njbartlett.osgi.emf.xmi` contains classes for XMI marshalling/unmarshalling. It has the same content as the original `org.eclipse.emf.ecore.xmi` bundle.

As the bundle symbolic names are different, you're going to be out of luck using them if you use Require-Bundle for your dependencies... but if you do use Require-Bundle then you're not probably not interested in minimising dependencies and being portable, so you might as well use the original bundles.

Be sure not to be misled by the name "minimal" -- this is still a fairly chunky bundle, weighing in at over 1Mb. That appears to be the minimum cost of using EMF, and certainly in some scenarios it is too much, but then again in many others it is a drop in the ocean. There is no free lunch, after all.

That's it for now. I'm only at the start of my EMF journey, and I fully expect to be whacked off course a few more times. Still it seems to be a journey worth taking.
