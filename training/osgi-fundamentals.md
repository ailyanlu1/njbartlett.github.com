---
layout: default
title: Fundamentals of OSGi
breadcrumbs: [ {url: /training.html, title: Training} ]
---

Fundamentals of OSGi -- Training
--------------------------------

This 2-day workshop covers the fundamentals of developing for OSGi, the Dynamic Module System for Java, and using OSGi's fine-grained services to produce highly decoupled and reusable software components.

On day one, you will be introduced to OSGi and learn how it meets the challenge of building modular, scalable applications for the Java Platform. Next you will review the three principal open source implementations of OSGi before focussing on one of them (Apache Felix). Then you will dive into the construction of modules, learning how to define dependencies between modules and manage versions of APIs.

Next you'll move onto OSGi Services, the lynchpin of OSGi's programming model. You will use the Declarative Services (DS) specification to build components that are able to react to their environment, configure themselves dynamically and call other components. Then you will look at one of the most important patterns used in constructing real applications with services, namely the "whiteboard" pattern.

On day two, you will learn first about practical topics. These include how to build and test modules using existing industry-standard tools (e.g., ANT, Maven, JUnit). Also, how to define and manage a runtime application using combinations of modules and configurations, and how to correctly evolve APIs and implementations over time.

Next you will review alternative component models including Blueprint (previously known as Spring Dynamic Modules), Guice/Peaberry and Apache iPOJO. Day two concludes with a look at distributed programming with Remote Services (also known as Distributed OSGi or D-OSGi) and the use of OSGi in enterprise applications. You will also experiment with deploying OSGi to "the Cloud", and look at the advantages of OSGi in that environment.

By the end of the "Fundamentals" course you will be aware of most of the core principles and techniques used in OSGi, and be able to develop highly decoupled, reusable and configurable software components.

The course is very practical and comprises hands-on programming exercises throughout.

Who Is This Course For?
-----------------------

If you are a Java developer interested in building highly modular, extensible applications using OSGi then this course is for you.

Course Prerequisites
--------------------

To benefit from this course you should be a competent Java developer or hands-on technical architect and you will need to have a good understanding of the core Java APIs. Some experience with using a build tool such as ANT and an IDE such as Eclipse will be useful, but not essential.

Course Labs & Exercises
-----------------------

30% Labs, 70% Presentation

Detailed Programme
------------------

#### BASICS

#### The Challenge of Modularity

-   The State of the Art in Standard Java
-   "JAR Hell"
-   What Is a Module?

#### OSGi Bundles

-   Nuts and Bolts: What Does a Bundle Look Like?
-   Building our First Bundle
-   Exporting and Importing
-   Keeping Internals Hidden

#### OSGi Implementations

-   Overview of Equinox, Knopflerfish and Felix
-   Getting Equinox
-   Launching Equinox
-   Using the OSGi Console
-   Installing, Updating and Uninstalling Bundles
-   Starting and Stopping Bundles

#### Dependencies and Versions Management

-   Importing Packages vs Requiring Bundles
-   Versioning of Bundles and Packages
-   Versioning of Imports/Requires

#### SERVICES

#### Introduction to Services

-   Late Binding in Java
-   Dependency Injection
-   Dynamic Dependency Injection
-   Declarative Services
-   Component Lifecycle
-   Laziness
-   Service Dependencies
-   Cardinality and Dynamics
-   Configurable Components

#### The Whiteboard Pattern

-   Review of the Classic Observer Pattern
-   Problems with the Observer Pattern
-   Fixing the Observer Pattern
-   Overview of the Whiteboard Pattern
-   Registering Listeners
-   Sending Events

#### OSGi IN PRACTICE

#### Building OSGi Bundles

-   Review of Approaches to Building Bundles
-   How BND Works
-   Using BND with Maven
-   Eclipse PDE

#### Testing Bundles

-   Unit Testing with JUnit
-   System Testing
-   Testing Frameworks

#### Building Applications

-   What is an Application?
-   Managing and Configuring OSGi Applications
-   Managing Versions and Evolving APIs
-   Bundle Repositories
-   Managing Bundle States
-   Start Levels

#### ADVANCED TOPICS

#### Alternative Component Frameworks

-   Blueprint/Spring Dynamic Modules
-   Google Guice + Peaberry
-   Apache iPOJO
-   Component Framework Interoperability

#### Remote Services and Enterprise OSGi

-   Overview of Remote Services/D-OSGi
-   Distribution Providers
-   Discovery Providers
-   Enterprise OSGi Motivation: Goals and non-Goals
-   Overview of Enterprise OSGi Services
-   OSGi in the Cloud
