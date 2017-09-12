---
layout: post
title: The Eclipse/Java 6u21 Blame Game
summary: What would you have done differently?
tags: [eclipse, oracle]
---

Ed Burnette published an <a href="http://www.zdnet.com/blog/burnette/oracle-rebrands-java-breaks-eclipse/2012">informative article on ZDNet</a> about the recent incompatibility problem with Eclipse and Java 6 update 21. Unfortunately the comments contain a lot of nasty accusations and finger-pointing, with people variously blaming Oracle or Eclipse or both for the mess... and even Ed's article title seems a trifle unfair on Oracle.

To address those who accuse Eclipse of inappropriately using an internal variable, consider for a moment why this was done. Here are the facts:

1.  The Sun JVM as shipped does not allow enough PermGen memory for a large application such as Eclipse to run. Incidentally many server applications also need more PermGen than is available by default.
2.  There is no standard option to expand the PermGen. The option on the Sun JVM is `-XX:MaxPermSize` but the "XX" part indicates this option is specific only to the Sun JVM and is not available on other JVMs such as JRockit, IBM J9 etc. In fact using it with those JVMs can cause them to simply emit an error and exit.
3.  Therefore Eclipse needs to detect whether it is on the Sun JVM specifically before it adds the `-XX:MaxPermSize` option. This cannot be done in Java because by the time the JVM has started it is too late, so it has to be done in the native launcher executable, i.e. eclipse.exe, which is coded in C. At this point the Java system properties are not yet available, but the properties in the exe/dll file **are** visible.

So I don't think Eclipse has done anything particularly wrong here. Neither has Oracle, there was no reasonable way for them to know that a simple rebranding would cause such a problem, and they responded very quickly to fix it.

Of course the fix is only a temporary one, as Oracle are going to want to change that variable eventually. So rather than pointing fingers, perhaps the Java community could try to come up with a real long-term solution. Such as a way to manage the JVM's memory from within the JVM... or getting rid of the cursed PermGen altogether?
