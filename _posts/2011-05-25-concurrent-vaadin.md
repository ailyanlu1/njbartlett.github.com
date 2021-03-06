---
layout: post
title: Concurrent Vaadin
summary: Handling asynchronous events in a Vaadin application.
tags: java osgi eclipse
date: 2011-05-25T14:00:00+00:00
---

As my colleagues and regular readers know, I have been hugely impressed by <a href="http://www.vaadin.com">Vaadin</a>, a web framework for Java, and have been using it in many of my OSGi demos, tutorials and projects.

This blog post is not about OSGi, but the more general problem of dealing with concurrent updates in a Vaadin application. I describe a technique for updating the application display in response to asychronous events. My own use-case was handling service binding/unbinding events in OSGi, but the technique would equally apply to message processing applications, etc.

Vaadin's programming model feels very much like working with a conventional desktop GUI API such as Swing or SWT/JFace, though it is more elegant than either of those examples. However one aspect where it differs significantly is its lack of an "event thread", which in desktop APIs is responsible for continuously painting the screen and responding to keyboard and mouse clicks.

Rather, Vaadin controls are backed by a data structure on the server side, which is queried by the HTTP server thread when the browser makes a request -- the HTTP server thread is generally owned by the Application Server. Therefore at the first approximation, we merely need to mutate the data structure in response to our async event, and the HTTP request will see the changed value when the next request occurs (this would be when the user clicks a button or refreshes the page... I will ignore "push" technologies for now -- i.e., COMET, Web Sockets -- since they are not currently supported in core Vaadin).

![](/images/posts/vaadin-concurrency.png)

Unfortunately the first approximation quickly fails because it is not safe. The HTTP request might arrive while we are in the middle of mutating the data structure, so it will see data in an inconsistent state. Or it might see the old value of the data, even if the request occurs after the update in terms of wall-clock time, because of the way caches work on modern CPUs.

To ensure that the changes we make to the data are both *visible* and *consistent* when read by the request thread, we have to perform them in a `synchronized` block, and furthermore we must synchronize on the same object that the HTTP request thread synchronizes on. This topic is not well documented in the <a href="http://vaadin.com/book">Book Of Vaadin</a>, but several messages on the forums indicate that the main *Application* object is the one to synchronize on. Our life is easiest then if we **are** the main Application. For example:

{% highlight java %}
public class MyApp extends Application {

  private final Table table = new Table();
  
  @Override
  public void init() {
    setMainWindow(new Window("Demo App", table));
    table.setFullSize();
  }

  // This is our async handler
  public synchronized void onMessage(Message message) {
    Item item = table.addItem(message.getId());
    // ...
  }
}
{% endhighlight %}

Making the `onMessage` method synchronized is enough, because it locks the Application. Be careful with synchronized methods though... it is usually better to reduce the scope of the lock by doing the minimal UI changes in a synchronized block, rather than locking for the entire method.

Unfortunately things are not so easy when we are writing a CustomComponent, which we commonly do in order to implement a reusable "chunk" of the overall Application. The problem is, we may start receiving async events before the component has been bound into the Application, so we cannot synchronize on the Application because we don't have visibility of it yet (that is, `getApplication()` will return null). The solution is to synchronize on the application if it is bound, but if it is not yet bound then synchronize on the component itself. There are a number of tricky details to take care of though: what if the application becomes bound *while* we are handling the async event? We also need to make the `setParent()` method synchronized, since that is the method where we are notified of the application being bound.

To avoid repeating ourselves, we can encapsulate these details into an abstract base class `ConcurrentComponent`, as follows:

{% highlight java %}
public abstract class ConcurrentComponent extends CustomComponent {
  @Override
  public synchronized void setParent(Component parent) {
    super.setParent(parent);
  }  
  @Override
  public synchronized Component getParent() {
    return super.getParent();
  }
  
  protected void executeUpdate(Runnable update) {
    Application application = null;
    synchronized (this) {
      application = getApplication();
      if (application == null)
        update.run();
    }
    if (application != null) {
      synchronized (application) {
        update.run();
      }
    }
  }
}
{% endhighlight %}

This allows us to implement our own component as follows:

{% highlight java %}
public class MyComponent extends ConcurrentComponent {

  private final Table table = new Table();

  // Async handler
  public void onMessage(Message message) {
    executeUpdate(new Runnable() {
      public void run() {
        Item item = table.addItem(message.getId());
        // ...
      }
    });
  }
}
{% endhighlight %}

This code should be strongly reminiscent of `SwingUtilities.invokeLater()` or SWT's `Display.asyncExec()`, which is intentional. Note how the lock on the Component is used directly to execute the operation if the application is unbound, but that lock is released and an Application lock acquired instead when the application is bound.

I must admit that I am not entirely happy with the necessity of taking two locks in succession for the common case, and worry that it may have a performance impact. However I can't see an easy fix that retains correctness; perhaps somebody else will be able to suggest one.

On the chance that any of the above code proves useful, I hereby place it in the Public Domain -- use it however you wish.
