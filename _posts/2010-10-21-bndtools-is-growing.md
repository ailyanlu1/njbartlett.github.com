---
layout: post
title: Bndtools is Growing
summary: More users, more contributors... more bugs...
tags: osgi eclipse bndtools
---

<a href="/bndtools_intro.html">Bndtools</a> is starting to gather momentum and attract a real community. The best thing about this from my point of view is that I have started to receive significant contributions.

The biggest so far has come from Per Kristian Søreide of <a href="http://comactivity.net/">ComActivity</a>. Per and his team are hugely experienced in OSGi and Eclipse, and they have developed a tool for releasing bundles into a repository, with in-depth analysis of compatibility and versioning. Essentially, it helps you to comply with the <a href="http://www.osgi.org/wiki/uploads/Links/SemanticVersioning.pdf">OSGi Semantic Versioning</a> (PDF) guidelines.

When you make changes to an API, the tool calculates whether those changes are backwards compatible for consumers, producers, both, or neither. It then adjusts the version numbers of each package accordingly. It also bumps the version number of your source `.bnd` and `packageinfo` files.

For example, suppose I make my first release of a new API, version 1.0.0. Here is what the tool displays:

![](/images/posts/bndtools-release-1.png)

Now I add a method to the `AuctionService` interface and release again:

![](/images/posts/bndtools-release-2.png)

This tells me that I have made a change that is non-backwards compatible for producers, and therefore the "minor" version segment needs to be incremented. That is, I need to update from 1.0.0 to 1.1.0. Of course it also highlights the reason. If I were to make an even more drastic change such as deleting a method, the tool would tell me that I need to bump to version 2.0.0, indicating a breaking change for both consumers and providers.

Using this tool, alongside bnd's existing support for generating import ranges, means that (in theory) **you should never have to manually maintain version numbers!** Now, that's a pretty bold statement and we need to see how things works out in practice, but I strongly feel that version management is one of those extremely important — but also extremely tedious — tasks that can be performed far better by machines than by humans..

The release feature is now included in the main Bndtools update site, so if you have already installed Bndtools, please update it via the Eclipse update manager.

Many thanks to Per and his team! And thanks also to all the others who reported bugs and contributed fixes... please keep them coming.
