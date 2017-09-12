---
layout: post
title: Are We There Yet?
summary: Using the OSGi Coordinator Service
tags: java osgi
date: 2012-03-21 12:00:00 +00:00
---

Peter [wrote a blog post](http://www.osgi.org/blog/2012/03/coordinator.html) yesterday about the OSGi Coordinator service, which is new in the R4.3 Compendium. I wanted to give a more concrete example of how it is used.

Sometimes we want to perform an expensive operation after a bunch of things have changed. For example, in the writable OBR repositories used by [Bndtools](http://bndtools.org/) we need to regenerate the index XML file after the user deploys a new resource into the repository. However, what if the user is deploying a hundred resources? It would be wasteful to regenerate the index after each of those deployments, because 99 times out of the hundred we will throw away the result.

The Coordinator service is therefore used as a way of chunking operations. When deploying a file to the repository, we check if a coordination is ongoing; if one is, then we join it and defer the regeneration of the index. Otherwise, we just regenerate the index immediately.

Here is the abbreviated code for `put()`, i.e. the method that does the deployment:

{% highlight java %}
public File put(Jar bundle) throws Exception {
	File newFile = storageRepo.put(bundle);
	newFilesInCoordination.add(newFile);

	if (coordinator == null || !coordinator.addParticipant(this)) {
		finishPut();
	}
	return newFile;
}
{% endhighlight %}

When the coordinator service exists AND there is a current coordination, indicated by `addParticipant` returning `true`, we simply begin the put operation and remember the file that we have created in the context of this coordination. However when there is no coordinator or current coordination, we both begin and immediately finish the put operation. The `finishPut` method looks like this:

{% highlight java %}
private void finishPut() throws Exception {
	regenerateIndex();
	newFilesInCoordination.clear();
}
{% endhighlight %}

In order to join the coordination we have to implement the `Participant` interface, which has two methods `ended` and `failed`. The `ended` method is very simple, we just call `finishPut`:

{% highlight java %}
public void ended(Coordination coordination) throws Exception {
	finishPut();
}
{% endhighlight %}

In the `failed` method we clean up the changes that we started to make. The repository will be back in the state it was in before the `put` method was called:

{% highlight java %}
public void failed(Coordination coordination) throws Exception {
	for (File file : newFilesInCoordination) {
		file.delete();
		// omitted some extra error handling and logging here
	}
	newFilesInCoordination.clear();
}
{% endhighlight %}

The thing I like about Coordinator is that it provides a very useful optimisation, but things can still work without it. In this case, when there is no Coordinator service available the repository regenerates the index on every `put`... which is exactly the behaviour that it had before I added Coordination support. Simply by adding a Coordinator bundle I can improve the performance of bulk repository updates.

There is a Coordinator implementation [available in Equinox 3.8M6](http://download.eclipse.org/equinox/drops/S-3.8M6-201203141800/download.php?dropFile=org.eclipse.equinox.coordinator_1.1.0.v20120219-1616.jar). As far as I know Felix does not have its own implementation yet, but the Equinox Coordinator works fine on Felix in my testing.

Incidentally, the use-cases for the Coordinator may remind you of the archetypal chunking problem in OSGi: when should the framework resolve bundles after a series of install, update and uninstall operations? Bundle resolution is not merely expensive but may actually produce spurious errors if done at inappropriate times, so the OSGi framework has always had the ability to chunk these operations: we call `FrameworkWiring.refreshBundles` method (or `PackageAdmin.refreshPackages()` prior to R4.3) to signal when we are done. The Coordinator service generalises this idea to make it available for our own applications.
