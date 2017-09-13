---
layout: post
title: Bndtools and Maven&#58; A Brave New World
summary: Tutorial at OSGi Community Event 2017
tags: osgi eclipsecon conference
date: 2017-09-13 00:00:00
comments: true
---

At this year's OSGi Community Event -- co-hosted with EclipseCon Europe 2017 -- I will present a new tutorial titled [Bndtools and Maven: A Brave New World](https://www.eclipsecon.org/europe2017/session/bndtools-and-maven-brave-new-world). My friend and [Paremus](https://paremus.com) colleague [Tim Ward](https://blogs.paremus.com/author/tim-ward/) will co-present and will provide support during the exercises.

Bndtools has long provided the premier development experience for writing OSGi bundles and applications. However there was a significant gap: until recently it has been very difficult to use Maven to build Bndtools projects. This was partially because of legacy issues (after starting with ANT as our standard build tool, we eventually migrated to Gradle). Partially it was because Maven is far more prescriptive in terms of the structure and lifecycle of a project, and it took a long time to find a way to fit our models together. These problems meant that Bndtools was simply unusable for a large number of developers, in particular those working for organisations that have stnadardised on Maven -- and also of course those developers who simply prefer Maven!

**These problems are now essentially solved and, from Bndtools version 3.4, Maven is a fully supported build environment.** Gradle also remains fully supported.

In this tutorial, we will walk through all of the day-to-day development tasks in the new Bndtools/Maven toolchain. These include:

1. Starting a new project from scratch;
2. Adding bundle modules to the project;
3. Managing library dependencies;
4. Integration testing (i.e. in-container testing);
5. Running bundles and tests from the IDE, and the rapid code/run cycle enabled by Bndtools;
6. Indexing, resolving and assembling applications for deployment;
7. Managing bundle and package versions;
8. Configuring a Continuous Integration build.

The following Maven plugins will be covered:

* `bnd-maven-plugin`: the core plugin that generates OSGi bundle JARs;
* `bnd-testing-maven-plugin`: executes OSGi in-container tests;
* `bnd-indexer-maven-plugin`: indexes OSGi bundles and dependencies for assembly and/or runtime resolving;
* `bnd-export-maven-plugin`: assembles a standalone OSGi application from its minimal bundles.

We do hope that you will attend this tutorial armed with lots of questions! Prerequisites are Bndtools 3.5 (which will have been released before the conference), JDK 8 and Maven 3. Also make sure you sign up to attend the tutorial on the EclipseCon website, and of course you also need a ticket to the main conference.