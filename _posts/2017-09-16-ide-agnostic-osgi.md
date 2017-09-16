---
layout: post
title: March of the IDEs — IDE Agnostic OSGi Development
summary: Presenting at the OSGi Users Forum Germany Tooling Workshop
tags: osgi eclipsecon conference
date: 2017-09-16 00:00:00
comments: true
---

For a long time Eclipse has been, in practice, the only widely used and supported IDE for developing OSGi applications. Both of the premier IDE plugins — namely [Bndtools](http://bndtools.org/) and the older [PDE](https://www.eclipse.org/pde/) — supported only Eclipse. While some tooling existed in IntelliJ and NetBeans it was nowhere near as comprehensive or usable.

This wasn't much of a problem for me personally. I have been a fan of Eclipse since I first saw it in the early 2000s, and indeed the first IDE I ever used was VisualAge for Java, which later morphed into Eclipse. I've tried other IDEs but I always end up reverting to the familiar.

However IDE choice is very personal, and the devotion of some developers to their preferred IDE can make religious fanatics seem positively lukewarm. So if you met an IntelliJ fan and told her that she would need to switch to Eclipse in order develop for OSGi... well, it's unsurprising that a common response was that OSGi could get stuffed.

So while I continue to use and support Bndtools in Eclipse, for the sake of OSGi adoption I have long wished for there to be a better development experience in the other popular IDEs. I'm happy to report that substantial progress has been made towards this goal. The latest version of bnd includes a suite of plugins for both Maven and Gradle that make development of OSGi applications practical irrespective of the IDE. Combined with some minimal IDE-specific tooling, such as that included in the latest IntelliJ builds, it is now a productive and even pleasurable activity.

On 23 October I will present a talk at the [OSGi Users Forum Germany Tooling Workshop](http://germany.osgiusers.org/Main/UFG2017) titled **March of the IDEs — IDE Agnostic OSGi Development**. In my talk I will demonstrate the development of a single OSGi application using Eclipse, IntelliJ, Visual Studio Code and a variety of command-line tools. This will highlight the opportunities for developers of all creeds to collaborate on high-quality modular code.

The workshop takes place all day on 23 October and is located at the Forum am Schlosspark, Ludwigsburg, Germany. This is the same venue as [EclipseCon Europe and OSGi Community Event 2017](https://www.eclipsecon.org/europe2017/) but it is organised as a separate event. Participation is free and you do not need a ticket for EclipseCon but you do need to register with the OSGi User Forum Germany by emailing [germany-info@osgiusers.org](mailto:germany-info@osgiusers.org). I hope to see you there!