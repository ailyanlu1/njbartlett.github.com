---
layout: post
title: Bndtools Now Available in Eclipse Marketplace
summary: The easy way to get going with OSGi development in Eclipse
tags: [osgi, eclipse]
flattr: http://njbartlett.github.com/bndtools.html
---

<a href="/bndtools.html">Bndtools</a> — the OSGi development environment for Eclipse based on <a href="http://www.aQute.biz/Code/Bnd">Bnd</a> — can now be installed from the Eclipse Marketplace. This is substantially easier than the traditional install procedure for Eclipse plug-ins, which involved copying and pasting an update site URL ino the Update Manager.

Eclipse Marketplace client is available in all of the Eclipse 3.6 distribution packages except for the so-called "Eclipse Classic" distribution... if you are running Eclipse Classic then I strongly recommend installing the Marketplace client -- this can be done in the usual way from the Helios update site.

To install Bndtools, open the Help menu in Eclipse and select *Eclipse Marketplace*. If this is the first time you have used it you may have to select your solution catalog as *Eclipse Marketplace* (rather than *Yoxos Marketplace* or any of the other catalogs that may appear in the future) and click Next. Then in the search field type "bndtools" (just "bnd" will work also) and click Go:

<img src="/images/posts/eclipse-marketplace-bndtools.png" alt="Eclipse Marketplace Client"></img>

Now click the Install button to the right of the Bndtools entry and follow the rest of the on-screen instructions. These include accepting the licence (of course) and accepting the fact that the plug-in is unsigned (I would love to get rid of this warning, but the price for a code-signing certificate is quite outrageous).

The really neat thing about Marketplace Client is how easy it is to \_un\_install a plug-in. While I hope you find Bndtools useful enough to keep it around, it's good to know that it can be removed painlessly if things don't work out. Just reopen the Marketplace Client, select the Installed tab and click the Uninstall button next to Bndtools (and any other plug-ins that might be cluttering up your SDK):

<img src="/images/posts/eclipse-marketplace-bndtools-uninstall.png" alt="Eclipse Marketplace Client -- Uninstalling" title="Don't do it!"></img>
