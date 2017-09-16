---
layout: default
title: Advanced OSGi
breadcrumbs: [ {url: /training.html, title: Training} ]
---

Advanced OSGi -- Training
-------------------------

This 2-day workshop is a follow-on to the OSGi Fundamentals course, and fills in many of the details that will enable you to become a highly skilled OSGi developer.

On day one you will take a deep dive into OSGi services, looking at the details of how they work at the lowest level using the OSGi APIs directly. You will learn about Service Trackers, including when they should be used (and when they should be avoided!), about cardinality and selection rules, service properties and filtering. You will also look at the problems associated with Java multi-threading and how concurrency impacts the correct use of services.

Next you will learn about the full module lifecycle, and about another very common and useful pattern known as the "extender" pattern. You will look at existing examples of extenders such as the Eclipse Extension Registry, and build your own extender.

At the end of day one and continuing into day two, you will review the Compendium of useful services offered by the OSGi specification, and also useful third-party modules. This will include: Event Admin (for asynchronous event delivery); Configuration Admin (for managing configuration data); Metatype Service (for defining metadata about services); HTTP Service (for building lightweight web servers without a traditional servlet container); and User Admin (for administering users, roles and authorisations).

The course concludes with a look at the most advanced OSGi topics including service hooks, integrating legacy and/or native code, embedding OSGi into an existing Java application or server, and the use of alternative JVM languages (e.g., Scala, Groovy, Clojure).

The course is very practical and comprises hands-on programming exercises throughout.

Who Is This Course For?
-----------------------

If you are a Java developer interested in gaining a thorough understanding of OSGi and how to use it to build highly modular, extensible applications, then this course is for you.

Course Prerequisites
--------------------

To benefit from this course you must have completed the [Fundamentals of OSGi](osgi-fundamentals.html) course.

Also you should be a competent Java developer or hands-on technical architect and you will need to have a good understanding of the core Java APIs. Some experience with using a build tool such as ANT and an IDE such as Eclipse will be useful, but not essential.

Course Labs & Exercises
-----------------------

30% Labs, 70% Presentation

Detailed Programme
------------------

#### OSGi SERVICES IN DEPTH

#### The Services APIs

-   Bundle Contexts
-   Registering and Unregistering Services
-   Looking up Services
-   Listening to Services
-   Properties and Filters

#### How Services Really Work

-   The Service Registry
-   Service Selection and Ranking
-   Service APIs, Versions and Compatibility
-   Service Factories
-   Using Service Listeners

#### Tracking Services

-   Introduction to ServiceTracker
-   Extending the ServiceTracker Class
-   Cardinality and Selection Rules
-   Composing Trackers

#### CONCURRENCY AND OSGi

#### Concurrency

-   The Price of Freedom
-   Review of Java Concurrency Practices
-   Safe Publication in OSGi
-   Avoiding Deadlock
-   GUI Development

#### BUNDLE LIFECYCLE AND EXTENDERS

#### Bundle Lifecycle

-   Installation and Uninstallation
-   Resolving
-   The Active State
-   Bundle Activators

#### Extending Without Code

-   The Extender Model
-   Inspecting Bundles
-   Tracking Bundles
-   Synchronous and Asynchronous Bundle Listeners
-   The Eclipse Extension Registry
-   Acting on Behalf of Bundles

#### THE OSGi COMPENDIUM SERVICES

#### Event Admin

-   Overview of Event Admin
-   The Event Object
-   Receiving Events
-   Synchronous vs Asynchronous Delivery

#### Declarative Services in Depth

-   Component Factories
-   Component Contexts
-   Service Factories
-   The Lookup Strategy vs the Event Strategy
-   Configuration Policy

#### Configuration Admin

-   Managed Services
-   Managed Service Factories
-   Creating and Updating Configurations
-   Building a Configuration Agent

#### Metatype Service

-   Declaring Configuration Types
-   Static vs Dynamic Metatype Data
-   The Metatype Service
-   Building a Configuration Client

#### HTTP Service

-   Registering Servlets
-   Registering Resources
-   The Http Context

#### Log Service

-   Logging from Bundles
-   Listening To and Reading the Log

#### User Admin Service

-   Users, Groups and Roles
-   Authentication
-   Authorization
-   JAAS

#### ADVANCED TOPICS

#### Service Hooks

-   Symmetric Services
-   Proxying Services
-   Providing Services on Demand

#### Legacy and Native Code

-   Class Loading Issues with Legacy Code
-   Calling Class.forName()
-   Thread Context Class Loaders
-   Singletons
-   Loading Native Code
-   Limitations of OSGi for Native Code

#### Embedding OSGi

-   Why Embed OSGi?
-   The Launch API
-   Building a Launcher
-   Using the Equinox Servlet Bridge

#### Alternative JVM Languages

-   Scala
-   Groovy
-   Clojure
