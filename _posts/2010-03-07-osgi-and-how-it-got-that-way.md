---
layout: post
title: OSGi and How It Got That Way
summary: A thought experiment... if OSGi didn't exist, how would we design it?
tags: [osgi, eclipse]
---

> …some of you may have had occasion to run into mathematicians and to wonder therefore how they got that way. Here in partial explanation perhaps is the story of the great Russian mathematician Nikolai Ivanovich Lobachevsky. — Tom Lehrer.

Probably the most common complaint against OSGi is that it is too complex. For example, quoting Reza Rahman of Caucho:

> From a personal standpoint, I did explore OSGi with an open mind. To my dismay, what I found is a very complex, low-level specification that needs a lot of evolution/abstraction to be adapted to be digestible to most enterprise Java environments. It also seems quite overkill for most realistic enterprise needs. Jigsaw in comparison feels much more “clean room”, Java-centric, compact and easy to understand.

Honestly, this complaint always makes me scratch my head a little. If I imagine for a moment that OSGi does not exist and I have been entrusted with the task of designing a new module system for the Java Platform, then the most sensible set of requirements for such a module system lead directly to OSGi as pretty much the most simple possible solution that satisfies those requirements!

Is this just a failure of my imagination? Or is it perhaps a case where Einstein’s Razor (“things should be as simple as possible, but no simpler”) trumps Ockham’s?

On the other hand, I accept that OSGi may not appear particularly simple at first glance, especially if you are not aware of how it got that way. Therefore in this blog post I have attempted to walk through the above thought-experiment of designing OSGi from scratch. I will gloss over a lot of detail of course. I had also intended to perform a similar analysis on Jigsaw, but this post is already a screed of nearly Yeggian proportions, so I will save Jigsaw for a subsequent post.

So without further ceremony, just how did OSGi get that way?

Module Separation
-----------------

Our first requirement is to cleanly separate modules so that classes from one module do not have the uncontrolled ability to see and obscure classes from other modules. In traditional Java the so-called “classpath” is an enormous list of classes, and if multiple classes happen to have the same fully-qualified name then the first will always be found and the second and all others will be ignored. This is not as unusual a problem as it may appear; it’s quite common if you have a lot of libraries that depend on other libraries. Obscuration is an absolute killer because it can lead to weird errors such as LinkageError, IncompatibleClassChangeError, etc. In fact if we see those errors, we are lucky! If we are unlucky then our system will just quietly do the wrong thing, irrespective of all the up-front testing we may have done before deployment.

The way to prevent uncontrolled visibility and obscuring of classes is to create a class loader for each module. A class loader is able to load only the classes it knows about directly, which in our system would be the contents of a single module (though it can also choose on a class-by-class basis to ask another class loader to supply the class instead — this is known as delegation). Once we do this, each module contains the code that it needs to work, and classes developed together are guaranteed to obtain the classes they expect, even if another module in the system has heavily overlapping fully-qualified class names.

Getting Back Together
---------------------

If we stop here then modules will be completely isolated and unable to communicate with each other. To make the system practical we need to add back in the ability to see classes in other modules, but we do it in a careful and constrained way. At this point we input another requirement: modules would like the ability to hide some of their implementation details.

In Java there is a missing access modifier between protected/default and public. If I am writing a library and I wish one of my classes to be used by other packages in my library, I must make the class public. But then the class is visible to everybody, including clients external to my library; those clients can now use my internal parts directly. We would like to have a “module” access level, but the problem today is that the javac compiler has no idea where the module boundaries lie, so it cannot perform any checks against such a access modifier. In fact the existing “default” access modifier is also broken because it is supposed to offer access only to members of the same “runtime package” (i.e., a package as loaded by a particular class loader), but again javac has no idea what class loaders might exist at runtime. In this situation, javac punts: it allows the access even though it may result in IllegalAccessErrors later.

The solution we choose in our module system is to allow modules to “export” only portions of their contents. If some part of a module is non-exported then it simply cannot be seen by other modules. But what should be the default? Should we export everything except for some explicitly hidden parts, or should we hide everything except for some explicitly exported parts? Choosing the latter seems to result in more clarity: we can easily look at the export list to determine the visible parts, i.e. the “surface area”, of the module.

Notice that I have not yet specified what exactly is exported… that is deliberate, please bear with me.

What’s the opposite of exporting? Importing, of course. A module that wants to use code from another module can import from it. Now we have another choice… should we import everything that is exported by the other module, or should we import just the parts of it that we need? Again we choose the latter since it results in more clarity: the important thing is what we are importing, not whom we import from.

A Strained Shopping Analogy
---------------------------

This point about imports is very important, so I will digress for a moment into a hopefully humourous, if somewhat exaggerated, analogy about shopping.

My wife and I have different approaches to shopping. I regard it as an onerous chore. When I must purchase something, I find a shop (or group of shops) that sells the items I need and I buy just those items and take them home. I don’t care which shop(s) I visit so long as I get the items I need.

In contrast, my wife goes to a shop she likes and basically buys everything in that shop.

Obviously I believe my approach is superior, because my wife has no control over what she brings home. If her favourite shop du jour changes its inventory then she ends up with different stuff… certainly lots of stuff she doesn’t need and perhaps missing some of the stuff she really does need.

It gets even worse… sometimes she ends up with an item that will not work on its own, because it has a dependency on something else, like batteries perhaps. So she has to go shopping again for batteries, and of course ends up buying the entire contents of the battery shop. Straining the analogy a little now, we could imagine that something else acquired from the battery shop also depends on something else, so she makes yet another trip to another shop, and another, purely to satisfy the requirements of a bunch of items that were not even needed to begin with! This problem is known as “fan-out”.

Hopefully the analogy with a module system is clear. The pathological shopping behaviour is equivalent to a system that forces us to import everything from the modules we declare a dependency on. When importing, we should import what we actually need to use, irrespective of where it comes from and ignoring all the things that happen to be packaged alongside it. We see the fan-out problem acutely when using the Maven build tool, which supports only whole-module (i.e. “buying-the-whole-shop”) dependencies, and as a result must download the entire Internet before it can compile a 200-byte source file.

Granularity of Exports and Imports
----------------------------------

What should be the granularity of the things that we import and export from modules? There are many levels of granularity in Java due to the various nesting levels: methods and fields are nested in classes, which are nested in packages, which are nested in modules or JAR files.

It shouldn’t be too hard to see that the sharing level should not be methods and fields. It would be simply insane to export some methods of a class but not others. Not only this, but we could write some methods/fields of a class in one module and some methods/fields in another module! Imagine writing the lists of exports and imports for every method that we wished to share across a module boundary. The complexity of checking this and diagnosing problems at runtime would be horrible, and lots of things would break because classes are just not designed to be split apart at runtime.

At the other extreme, the sharing level should not be whole modules because then modules would not be able to hide parts of their implementation, and importers would routinely suffer from bought-the-whole-shop syndrome.

So the only sane choices are classes and packages… and frankly, choosing classes just isn’t all that sane. Though not as bad as methods/fields, classes are too numerous and too heavily dependent on other classes in the same package for it to be sensible either to list classes as our imports and exports or to split some classes of a package into one module and other classes of the same package into another module.

As a result, OSGi chooses packages. The contents of a Java package are intended to be somewhat coherent, but it is not too onerous to list packages as imports and exports, and it doesn’t break anything to put some packages in one module and other packages in another module. Code that is supposed to be internal to our module can be placed in one or more non-exported packages.

What we lose is the ability to handle so-called “split-packages” cleanly. In OSGi, packages are the fundamental unit of sharing: when you import a package, you get the whole of a package as exported by one module but NO other. There are some strategies for dealing with legacy libraries that insist on smearing the contents of a package across many modules, but it is far better to adjust the modules so that each package is exported in its entirely by only one module.

Package Wiring
--------------

Now that we have a model for how modules isolate themselves and then reconnect, we can imagine building a framework that constructs concrete runtime instances of these modules. It would be responsible for installing modules and constructing class loaders that know about the contents of their respective modules.

Then it would look at the imports of newly installed modules and trying to find matching exports. Suppose module A exports package com.foo and module B imports the same package. The framework would notify B that it can obtain classes for the com.foo from module A — this is called wiring. Now when the class loader for B tries to load the class com.foo.Bar, it will delegate to the class loader for A. The process of wiring up imports for a whole module is known as resolution and if all imports are successfully wired then the bundle is resolved, which makes it fully available for use.

An unexpected benefit from this is we can dynamically install, update and uninstall modules. Installing a new module has no effect on those modules that are already resolved, though it may enable some previously unresolvable modules to be resolved. When uninstalling or updating, the framework knows exactly which modules are affected and it will change their state if necessary. There are some additional details to make this work smoothly, for example if a module is doing something important before it is uninstalled or unresolved then we need to send it a notification so it can shutdown cleanly. So, dynamic modules in OSGi do not come for free, since there is no magic involved, but OSGi at least makes it possible.

Some OSGi users prefer to avoid dynamic loading, and that is fine. It is not the most important feature of OSGi, but because it is unique to OSGi it tends to get a disproportionate degree of attention. Nevertheless, nobody will force you to use it, and you will still derive plenty of benefits from OSGi even if you never take advantage of dynamics.

Versions
--------

Our module system is looking good, but we cannot yet handle the changes that inevitably occur in modules over time. We need to support versions.

How do we do this? First, an exporter can simply state some useful information about the packages it is exporting: “this is version 1.0.0 of the API”. An importer can now import only the version that is compatible with what it expects and has been compiled/tested against, and refuse to accept e.g. version 3.0.0. But what if the importer wants version 1.0.0 and only version 1.0.1 is available… the slightly higher version doesn’t sound like it should be a breaking change, so the importer should probably accept 1.0.1 as well as 1.0.0. In fact the importer should specify a range of versions that it will accept, which could be something like “version 1.0.0 through to 2.0.0 but excluding 2.0.0 itself“. The package wiring process supports this and will only wire up an import to an export where the exported version falls within the range specified by the import. For this to work, version numbers need to be ordered and comparable.

How do we know that version 1.0.1 is not a breaking change against 1.0.0? Unfortunately we cannot for sure. OSGi strongly encourages, but cannot enforce, the following semantics on version numbers:

-   Increments to the Major segment (the first) should be made for non-backwards-compatible changes
-   Increments to the Minor segment (the middle) should be made for feature enhancements that are backwards compatible.
-   Increments to the Micro segment (the last) should be made for bug-fixes that do not produce visible feature changes.

If everybody stuck to these semantics then specifying import ranges for any module would be a breeze. Unfortunately the real world is not so simple, so we have to look carefully at compatibility issues when using any external library.

Packaging Modules and Metadata
------------------------------

Our module system will need a way to package the contents of a module along with metadata describing the imports and exports into a deployable unit.

Java already has a standard unit of deployment: the JAR file. JAR files may not be fully-fledged modules but they work just fine for moving around chunks of compiled code, so we have no need to invent something new. So the only question is, where should we put the metadata, i.e. the lists of imports and exports, versions and so on?

Configuration formats seem to be strongly influenced by fads; if we designed our module system between 2000 and 2006 we would probably choose to put the metadata in an XML file somewhere in the JAR file. This would work but it has a number of issues: XML is not particularly efficient to process, especially when we must find it somewhere inside a JAR file and decompress it before parsing. A JAR is just a ZIP, so finding a particular file means reading to the end to find the trailing central directory, then jumping back to the offset indicated by the directory. In other words, we always have to read the whole JAR file, which is a pain for tools that might want to scan large directories containing many modules, for example to search for available modules to satisfy a dependency.

XML is also barely editable by human beings; we need custom editing tools to do it reliably.

On the other hand if we were designing after 2006 then our first impulse might be to use Java annotations. I’m a big fan of annotations when used appropriately, and it’s easy to see the attraction of putting something like @Export(version="1.0.0") on a package declaration in the Java source rather than maintaining it in a separate file. But wait a minute… the package declaration is repeated in every source file in the package; must we put the annotation on all of them??

To address this problem, the Java Language Specification (JLS) suggests the use of a special source file named package-info.java. Now what about the metadata that does not belong to any specific package, e.g. the list of imported packages or the name and version of the module itself? This suggests we need another special source file called something like module-info.java.

So far so good, but now consider how the module will be processed. Those special source files will be compiled to bytecode in package-info.class and module-info.class, so you can forget about opening up the JAR file in WinZip to take a squint at the metadata! Any module-scanning tool will not only have to read whole JAR files but also have to be able to process bytecode. And the runtime module system itself will have to create a class loader for the module immediately, just to read its metadata; as it turns out this eliminates a large class of optimisations that are possible if we can defer class loader creation until a class is actually loaded from the module.

As it happens OSGi was designed before 2000, so it did choose either of these solutions. Instead it looked back at the JAR File Specification, where the answer is spelled out: META-INF/MANIFEST.MF is the standard location for arbitrary application-specific metadata. In the words of the specification: “Attributes which are not understood are ignored. Such attributes may include implementation specific information used by applications.”

MANIFEST.MF is designed to be efficient to process, and it is faster than XML, at least. It is somewhat readable; at least as readable as XML, and obviously more readable than compiled Java bytecode. Also the standard jar command line tool always places MANIFEST.MF as the first entry in the JAR file, so tools can scan just the first few hundred bytes of the file in order to obtain the metadata.

Unfortunately MANIFEST.MF is not perfect. For one thing it is quite hard to write by hand due to a rule requiring each line to be no longer than 72 bytes, which becomes problematic if one considers that a single UTF-8 character can be between 1 and 6 bytes. A better approach is to generate MANIFEST.MF using a template in an alternative format; the Bnd tool does this, and so do Maven’s Bundle Plugin and SpringSource’s Bundlor.

In fact, Bnd even includes experimental support for processing annotations such as @Export on source code annotations. These may give us the best of both worlds: the convenience of annotations, combined with the efficiency and runtime readability/toolability of MANIFEST.MF.

Late Binding
------------

The final piece of modularity puzzle is late binding of implementations to interfaces. I would argue that it is a crucial feature of modularity, even though some module systems ignore it entirely, or at least consider it out of scope.

It is well known that interfaces in Java can break the coupling between consumers and producers of functionality. By defining an interface that serves as the contract between a consumer and a producer, neither of them require direct knowledge of the other, so we can put them in different modules that do not depend on each other. Instead they will each have a dependency on the interface, which can live in a third module if we choose. The only question is how to supply instances of the interface to the consumer class, and the most common answer these days is with a Dependency Injection (DI) framework such as Spring or Guice.

Therefore to complete our module system we could simply defer to an existing DI framework. We are seeking simplicity after all, and there can be nothing simpler than declaring a problem to be out of scope, to be solved by somebody else. However this is not quite satisfactory because the DI framework really needs to be aware of the module boundaries. The problem with a traditional approach to DI is that it tends to create huge, centralised configurations which cut across all modules. Peter Kriens calls this the “God Class” problem, in which one component knows everything about every module and requires them all to do its bidding (as an atheist I find this unsatisfactory, but even if you are a theist then I’m sure you agree that we should not create any more Gods than currently exist). These God Classes — or XML configurations — are fragile, difficult to maintain, and negate most of the benefits of separating our code into modules.

Instead we should look for a decentralised approach. Rather than being told what to do by the God Class, let us suppose that each module can simply create objects and publish them somewhere that the other modules can find them. We call these published objects “services”, and the place where they are published the “service registry”. The most important information about a service is the interface (or interfaces) that it implements, so we can use that as the primary registration key. Now a module needing to find instances of a particular interface can simply query the registry and find out what services are available at that time. The registry itself is still a central component existing outside of any module, but it is not “God”… rather, it is like a shared whiteboard.

We do not need to abandon DI, indeed it is still very useful: existing DI frameworks can be used to inject services into other services and to declaratively publish certain objects as services. The DI framework no longer directs the whole system, instead it is a utility employed within individual modules. We could even use multiple DI frameworks, e.g. Spring and Guice together in the same application, which might be useful if we want to integrate a third-party component that uses a different framework from our own preferred one. Finally, the service registry will have a programmatic API for publishing and lookups, but it will only be used for low-level work such as implementing a new DI framework.

Conclusion
----------

Hopefully this explains in broad stokes the reasons why OSGi is the way it is. OSGi will still be accused of being complex, but I believe that any complexity that does exist is there to handle the corner cases in what I have described.

Of course it has its warts. For example, versioning could stand to be improved, especially in the face of third-party libraries with weird versioning schemes. It’s still the right choice to assign meaning to version numbers, but more assistance needs to be available for managing versions and compatibility of APIs. Then there are the legacy libraries that make dodgy assumptions about the existence of a flat system classpath. In my opinion, any library that takes in a class name as a String and calls Class.forName() to obtain an object is broken, since it assumes that all classes are visible to all modules, which is simply not true in any kind of modular system. Unfortunately they cannot all be fixed overnight, so we need strategies to deal with such broken libraries. However they need to be dealt with in exceptional ways that do not compromise the rules of modularity for everybody else.

Coming up next: the motivation and design of Jigsaw, along with a technical comparison of its features alongside those of OSGi. I also intend to point out where it may be appropriate for OSGi to borrow some ideas from Jigsaw.
