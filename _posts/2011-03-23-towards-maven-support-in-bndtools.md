---
layout: post
title: Towards Maven Support in Bndtools
summary: Building Bndtools Projects with Maven
tags: java osgi bndtools maven eclipse
date: 2011-03-02T14:00:00+00:00
---

A commonly requested enhancement for Bndtools is to enable integration with Maven. Although I don't tend to use Maven myself, I do recognise that it's a very popular build tool, especially in big companies with lots of developers.

That's why I've been working on Maven support for Bndtools, which is now available to try out as an "Alpha" quality pre-release. While not yet particularly stable, I need to get some feedback from real Maven users to guide the rest of the implementation.

Before going into the details, first a word of reassurance to the non-Maven-users: you absolutely will **not** have to adopt Maven in order to use Bndtools, and I will **never** make it a requirement. Bndtools projects can still be built with <a href="http://www.aQute.biz/Code/Bnd">Bnd</a>, which is a very capable tool: for example, the OSGi Alliance uses Bnd to build over 1300 bundles every night.

On the other hand if you are a committed Maven user already, then Bndtools' support will make it possible for you to take advantage of powerful OSGi-specific development features in your IDE, without changing your build processes.

It's easiest to explain what I have done in a Q&A style. Please add your own questions in the comments.

How Does it Work?
-----------------

An important design goal is to avoid duplicating any configuration data. Dependencies and bundle contents/configuration are each defined in exactly *one* place only.

Bndtools should be installed alongside M2Eclipse in the same Eclipse instance. There is a clear division of responsibilities between the two tools:

-   Maven and M2Eclipse are responsible for build-time dependencies and performing the canonical "offline" build. This in turn uses the <a href="http://felix.apache.org/site/apache-felix-maven-bundle-plugin-bnd.html">Maven Bundle Plugin</a>, which is the most popular way to build OSGi bundles with Maven; for example, Apache Felix and all its subprojects are built this way. The Bundle Plugin uses Bnd internally, just like Bndtools.
-   Bndtools is responsible for "on-the-fly" developer builds, launching OSGi frameworks and test runs, and configuring bundle contents.

Why Should I Use This?
----------------------

If you are a current Maven user, then the approach outlined here allows you to easily adapt your existing Maven builds to develop OSGi bundles, using the most popular approach for building bundles with Maven. It does not require major changes to your project structure.

At the same time you will gain the OSGi specific ease-of-use features of Bndtools (see next question) while still allowing users of other Java IDEs such as NetBeans to easily work on your project.

What Does Bndtools Offer Over M2Eclipse?
----------------------------------------

M2Eclipse is a fully-featured development environment for working with any kind of Maven-built project. However it offers few ease-of-use features for working with *specific* project types.

Bndtools offers a rich and easy-to-use environment specifically for developing OSGi bundles. You can take advantage of the following features:

-   On-the-fly generation of the OSGi bundle, and deployment to a running OSGi container, immediately upon saving the source of a Java file.
-   Powerful tools to visualise and manipulate the package-level dependencies of bundles.
-   Tools to configure, run and test an OSGi application, which also work in exactly the same way from the command line.
-   ... and more!

For example, when using the raw Bundle Plugin, the contents of a bundle are controlled via a rather arcane set of instructions embedded in the POM, such as the following:

{% highlight xml %}
<instructions>
    <_versionpolicy>
        [$(version;==;$(@)),$(version;+;$(@)))
    </_versionpolicy>
    <Export-Package>
        org.apache.felix.fileinstall;version=${project.version}
    </Export-Package>
    <Import-Package>
        org.osgi.service.log;resolution:=optional,
        org.osgi.service.cm;resolution:=optional,
        *
    </Import-Package>
    <Private-Package>
        org.apache.felix.fileinstall.internal,
        org.apache.felix.utils.properties,
    </Private-Package>
    <Bundle-Activator>
        org.apache.felix.fileinstall.internal.FileInstall
    </Bundle-Activator>
    ...
{% endhighlight %}

In contrast Bndtools gives us the following editor (click for full size). Note the Calculated Imports list in the top right, which is recalculated on the fly as we adjust the bundle contents and customise imports; with Maven Bundle Plugin we must do a full build to obtain this information.

![Bndtools Bundle Editor](/images/bndtools-editor-small.png "Bndtools Bundle Editor"):/images/bndtools-editor.png

Do I Have to Use M2Eclipse?
---------------------------

Yes, for now: to enable Maven support in Bndtools, M2Eclipse is required.

M2Eclipse provides the build-time classpath that allows Eclipse to compile the Java sources, before Bnd takes over to build the OSGi bundles.

I am investigating the possibility of directly processing the POM and local repository from within Bndtools. This could provide a lightweight alternative for users that do not need the full suite of functionality offered by M2Eclipse. However, this feature is not yet available.

Why Not Use Tycho?
------------------

There is nothing wrong with Tycho, but it is a tool for a different job: it is intended for building projects that have been developed using the Eclipse Plug-in Development Environment (PDE). Projects developed in PDE are very inflexible and require specialised build tools -- as a result they have never worked properly with "vanilla" Maven. Tycho works by adapting Maven to the PDE project structure.

In contrast, Bndtools is more flexible and adapts to your existing build system, whether it is vanilla Maven or something else. To summarise the advantages of using Bndtools with Maven versus Tycho:

-   Bndtools uses the standard Maven project structure; Tycho uses the PDE project structure.
-   Tycho projects are Eclipse PDE projects, which are difficult (though not impossible) to work on using IDEs other than Eclipse. Bndtools/M2Eclipse projects can be easily worked on by developers using other IDEs, though of course they will miss the ease-of-use features offered by Bndtools.
-   Tycho requires Maven 3, whereas Bndtools works with both Maven 2 and 3.

These points are distinct from the advantages of Bndtools versus Eclipse PDE, which is a subject for another blog post.

To be clear: if you are currently using Eclipse PDE to develop your OSGi bundles then you should *definitely* use Tycho to build them! On the other hand if you are:

-   a Maven user interested in trying OSGi, or;
-   looking to "OSGify&quot; (or "bundleize&quot;) an existing Maven build, or;
-   already using the Bundle Plugin

...then I believe Bndtools is more likely to fit your needs.

How are Dependencies Specified?
-------------------------------

In Bndtools/Maven projects the build dependencies are specified entirely in the POM. I.e., the POM is the *sole source* of build-time dependencies, and Bndtools uses the build classpath as defined by M2Eclipse to perform its on-the-fly builds.

The `-buildpath` feature of Bnd, which is ordinarily specified in `bnd.bnd`, should not used when building with Maven.

How Are Bundle Contents Specified?
----------------------------------

The instructions required by Bnd to specify bundle contents and calculate imports are given in the `bnd.bnd` file, just as in any other Bndtools project. These instructions are referenced from the POM using the following instruction to the Bundle Plugin:

{% highlight xml %}
<instructions>
    <_include>bnd.bnd</_include>
</instructions>
{% endhighlight %}

This means that `bnd.bnd` is the *sole source* of bundle content and configuration instructions.

Note that some parameters in the output bundle can be inherited from the POM. For example, we typically want to use the Artifact ID as the symbolic name of the bundle and the POM version as the bundle version. This can be achieved by referencing these properties from `bnd.bnd` as follows:

{% highlight text %}
-include: pom.xml

Bundle-SymbolicName: ${pom.artifactId}
Bundle-Version: ${pom.version}
{% endhighlight %}

Can I Build Multi-Bundle Projects?
----------------------------------

Bnd supports a feature called "sub-bundles", in which multiple bundles are built from a single project.

While this feature does still work with a Maven-based build, it is *not recommended* because it works against the Maven philosophy of one output artifact per project. Downstream tools and Maven plugins, such as those that assemble the output bundles into a packaging artifact, are unlikely to work correctly with multi-bundle projects.

How Do On-the-Fly Builds Work?
------------------------------

Bnd uses the project classpath -- as created by M2Eclipse from the POM -- to perform on-the-fly builds of bundles. As ever when using Bndtools, your bundles are always built, and if you are running an OSGi framework from Bndtools they will be immediately deployed into it.

Since both Bndtools and the Maven Bundle Plugin use Bnd internally, the on-the-fly bundles are exactly the same as those built by Maven in a full offline build.

What Is The Bnd OSGi Repository Used For?
-----------------------------------------

When using Bndtools without Maven, Bnd Repository is used to supply bundles for both building and running. However when using Maven, Bndtools is no longer responsible for build dependencies but the Bnd Repository is still used to supply bundles for running and testing OSGi applications.

This is important because the Maven repositories contain a mixture of different artifact types, and most of the JARs it contains are not OSGi bundles. The Bnd Repository contains exclusively OSGi bundles and so is very useful for actually *running* OSGi applications.

Having said that, a subset of JARs in the Maven repositories are OSGi bundles, and work is in progress to build a Bnd Repository that wraps the Maven local repository to offer those bundles for use at runtime.

How Do I Use It?
----------------

First ensure that both M2Eclipse and Bndtools Alpha are installed in your Eclipse IDE. Here is the update site for Bndtools Alpha:

{% highlight xml %}
http://bndtools-alpha-updates.s3.amazonaws.com/
{% endhighlight %}

The next step depends on whether you are creating a new project, or adapting an existing one.

### Creating a New Maven/Bndtools Project

To create a new Maven project with Bndtools support, use <a href="http://njbartlett.name.s3.amazonaws.com/bndtools-component-archetype-0.0.2-SNAPSHOT.jar">`bndtools-component-archetype`</a>. This archetype corresponds to Bndtools' "Component Template", and creates a project with support for Declarative Services annotations. The OSGi Core and Compendium APIs are added as dependencies, in addition to JUnit.

As soon as the Bndtools Maven support is stable, this archetype will be submitted to Maven Central so that it can be directly accessed by Maven/M2Eclipse. For now, you need to click the link to download it, then install into your local repository as follows:

{% highlight text %}
mvn install:install-file \
    -DgroupId=org.bndtools.archetypes \
    -DartifactId=bndtools-component-archetype \
    -Dversion=0.0.2-SNAPSHOT \
    -Dpackaging=jar \
    -Dfile=./bndtools-component-archetype-0.0.2-SNAPSHOT.jar \
    -DgeneratePom=true
{% endhighlight %}

You can now create a new project from the archetype as follows:

{% highlight text %}
mvn archetype:create \
    -DarchetypeGroupId=org.bndtools.archetypes \
    -DarchetypeArtifactId=bndtools-component-archetype \
    -DarchetypeVersion=0.0.2-SNAPSHOT \
    -DgroupId=org.example \
    -DartifactId=comp1 \
    -Dversion=0.0.1-SNAPSHOT
{% endhighlight %}

Then you can import the project into Eclipse using **File** &gt; **Import** &gt; **Existing Projects into Workspace**. You can also use M2Eclipse to create a project from the archetype without leaving Eclipse, but due to an apparent bug in M2Eclipse this throws an error (`NullPointerException`). In fact the project is still created, so the error can be safely ignored.

### Adapting an Existing Maven Project

If you have an existing "vanilla" Java Maven project, you can adapt it for use with Bndtools by right-clicking on the `pom.xml` and selecting **Add OSGi/Bnd Support**.

![](/images/bndtools-maven1.png)

This starts a refactoring wizard that performs the following changes:

-   Creates a `bnd.bnd` file and adds the Bnd "nature" to the Eclipse project;
-   Adds the Maven Bundle Plugin to the build plugins section of the POM, if not already present;
-   Changes the Maven packaging type to "bundle" (required by the Bundle Plugin).

Note that the changes to the POM will be previewed in the wizard, so you can make sure you are happy with them before clicking Finish.

![](/images/bndtools-maven2-small.png):/images/bndtools-maven2.png

Is That It?
-----------

Yes, for now.

Please give it a try, but don't expect everything to work. If something that seems like it should work doesn't, then <a href="mailto:njbartlett@gmail.com">drop me a line</a>. If you think my whole approach is wrong, then tell me how I should be doing this. My goal is to make Bndtools **the easiest** and fastest way to develop OSGi applications -- irrespective of how they are built.
