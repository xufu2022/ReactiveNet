# Chapter 2. Reactive Extensions

The Reactive Extensions for .NET (usually shortened to Rx) are designed for working with asynchronous and event-based information sources. **Rx** provides services that help you orchestrate and synchronize the way your code reacts to data from these kinds of sources

It provides an abstraction that has a steeper learning curve than events, but it comes with a powerful set of operators that makes it far easier to combine and manage multiple streams of events than is possible with the free-for-all that delegates and .NET events provide. Microsoft has also made an associated set of libraries called Reaqtor available that builds on the foundation of Rx to provide a framework for reliable, stateful, distributed, scalable, high-performance event processing in services.

Rx’s fundamental abstraction, **IObservable<T>**, represents a sequence of items, and its operators are defined as extension methods for this interface. This might sound a lot like LINQ to Objects, and there are similarities—not only does IObservable<T> have a lot in common with IEnumerable<T>, but Rx also supports almost all of the standard LINQ operators.

Unlike IEnumerable<T>, Rx sources do not wait to be asked for their items, nor can the consumer of an Rx source demand to be given the next item. Instead, Rx uses a push model in which the source notifies its recipients when items are available.

For example, if you’re writing an application that deals with live financial information, such as stock market price data, IObservable<T> is a much more natural model than IEnumerable<T>. Because Rx implements standard LINQ operators, you can write queries against a live source—you could narrow down the stream of events with a where clause or group them by stock symbol. Rx goes beyond standard LINQ, adding its own operators that take into account the temporal nature of a live event source. For example, you could write a query that provides data only for stocks that are changing price more frequently than some minimum rate.

Rx’s push-oriented approach makes it a better match than IEnumerable<T> for event-like sources. But why not just use events, or even plain delegates? Rx addresses four shortcomings of those alternatives. 

-   First, it defines a standard way for sources to report errors. 
-   Second, it is able to deliver items in a well-defined order, even in multithreaded scenarios involving numerous sources. 
-   Third, Rx provides a clear way to signal when there are no more items. 
-   Fourth, because a traditional event is represented by a special kind of member, not a normal object, there are significant limits on what you can do with an event—you can’t pass a .NET event as an argument to a method, store it in a field, or offer it in a property. You can do these things with a delegate, but that’s not the same thing—delegates can handle events but cannot represent a source of them. **There’s no way to write a method that subscribes to some .NET event that you pass as an argument**, because you can’t pass the actual event itself. Rx fixes this by representing event sources as objects, instead of a special distinctive element of the type system that doesn’t work like anything else.

By providing a coherent abstraction that addresses these problems, Rx is able to bring all of the benefits of LINQ to event-driven scenarios. Rx does not replace events; I wouldn’t have dedicated one-fifth of Chapter 9 to them if it did. In fact, Rx can integrate with events. It can bridge between its own abstractions and several others, not just ordinary events but also IEnumerable<T> and various asynchronous programming models. Far from deprecating events, Rx raises their capabilities to a new level. It’s considerably harder to get your head around Rx than events, but it offers much more power once you do.

Two interfaces form the heart of Rx. 
-   **Sources** that present items through this model implement IObservable<T>. 
-   **Subscribers** are required to supply an object that implements IObserver<T>. 

These two interfaces are built into .NET. The other parts of Rx are in the System.Reactive NuGet package. That package is an open source project (https://github.com/dotnet/reactive) that was originally written by Microsoft (hence the System namespace), but which is now a community-maintained .NET Foundation project