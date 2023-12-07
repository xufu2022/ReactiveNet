# Fundamental Interfaces
There are two essential types types in Rx: the IObservable<T> and IObserver<T> interfaces. These are important enough to be in the System namespace. Example 2-1 shows their definitions

```cs
public interface IObservable<out T>
{
    IDisposable Subscribe(IObserver<T> observer);
}

public interface IObserver<in T>
{
    void OnCompleted();
    void OnError(Exception error);
    void OnNext(T value);
}
```

The fundamental abstraction in Rx, IObservable<T>, is implemented by event sources. Instead of using the event keyword, it models events as a sequence of items. An IObservable<T> provides items to subscribers as and when it’s ready to.

> Observable<T> Interface:

- Role: The IObservable<T> interface represents the class that sends notifications (the provider).
- Function: It's responsible for sending messages to IObserver<T> instances. These messages can represent the occurrence of an event, an error, or the completion of a sequence of events.
- Methods: The key method in IObservable<T> is Subscribe. When an observer **subscribes** to an IObservable<T>, it means the observer wants to receive notifications from the IObservable<T>.
- Usage: Typically, classes that implement IObservable<T> create a sequence of data or events to be observed, such as a stream of sensor readings or user interactions.

> IObserver<T> Interface:

- Role: The IObserver<T> interface represents the class that receives the data or notifications (the subscriber).
- Function: It's designed to react to the data being received from IObservable<T>. It listens for various types of messages sent by IObservable<T> instances, including new data, an error, or completion of the data sequence.
- Methods: The IObserver<T> has three main methods: OnNext (called when new data arrives), OnError (called when an error occurs), and OnCompleted (called when the sequence of data or events is complete).
- Usage: Classes implementing IObserver<T> are typically concerned with what to do with the streamed data, like updating a user interface, processing data, or triggering other events.

In summary, the IObservable<T> is like a broadcaster or publisher, emitting notifications or data streams, while the IObserver<T> is like a listener or subscriber, receiving and responding to these notifications. The relationship between them is fundamental to the observer pattern, which is central to reactive programming.


As you can see from the **out** keyword, the type argument for IObservable<T> is **covariant**, meaning that if you have a type Base that is the base type of another type Derived, then just as you can pass a Derived to any method expecting a Base


> Covariance:

- Covariance is a type relationship term in C# that allows you to use a more derived type (such as a subclass) in place of a less derived type (such as a base class).
In simpler terms, if Class B is derived from Class A, covariance allows you to use an instance of B wherever an instance of A is expected.
out Keyword in Generics:

The out keyword in C# is used with generic type parameters to specify that a generic type is covariant.
When you mark a generic type parameter with out, it can only be used in output positions such as return types but not in input positions (like method parameters).

> Application in IObservable<T>:

The IObservable<T> interface in C# is defined with the out keyword, making the T covariant. This means that IObservable<T> can be more flexible in the types it can handle.
For example, if Cat is a subclass of Animal, and you have an IObservable<Cat>, covariance allows you to treat it as an IObservable<Animal>. This is because every Cat is an Animal, so an observable of Cat can be seen as an observable of Animal.

> Practical Implication:

This feature is particularly useful in scenarios where you need to deal with a hierarchy of types and want to create generic methods or classes that can work with these types in a flexible way.
For instance, if you have a method that expects an IObservable<Animal>, you can pass an IObservable<Cat> or IObservable<Dog> to it, assuming Cat and Dog are derived from Animal. This promotes code reusability and flexibility.

> In summary, the out keyword in IObservable<T> makes T covariant, allowing you to use an IObservable of a derived type wherever an IObservable of a base type is expected. This aligns with the principle of substitutability in object-oriented programming, where a derived type should be substitutable for its base type.


Conversely, items go into a subscriber’s IObserver<T> implementation, so that has the in keyword, which denotes contravariance—you can pass an IObserver<Base> to anything expecting an IObserver<Derived>.

> Contravariance:
    Contravariance is the opposite of covariance. In the context of C# generics, it allows a function to accept arguments of less derived types than specified by the generic parameter type.
    In simpler terms, if Class B is derived from Class A, contravariance allows you to use an instance of A where an instance of B is expected.

> in Keyword in Generics:

The in keyword in C# is used with generic type parameters to specify that a generic type is contravariant.
When you use in, the generic type can be used in input positions (like method parameters), but not in output positions (like return types).

> Application in IObserver<T>:

In the IObserver<T> interface, the T parameter is marked with in, making it contravariant. This means the IObserver<T> can be more flexible in the types of data it can observe and react to.
For example, if Animal is a base class and Cat is a subclass, a method that expects an IObserver<Cat> can also accept an IObserver<Animal>. This is because every Cat is an Animal, but not every Animal is a Cat. An IObserver<Animal> can handle any Animal, including Cat.

> Practical Implication:

This is particularly useful when you have a method or a class that needs to deal with a family of related types in a generic way.
For instance, if you have a method designed to work with an IObserver<Dog>, and you have an IObserver<Animal>, you can pass the IObserver<Animal> to it. This is because the IObserver<Animal> can handle Dog instances along with other Animal types.

> In summary, the in keyword in IObserver<T> makes T contravariant, allowing you to use an IObserver of a base type wherever an IObserver of a derived type is expected. This concept is essential in scenarios where you need to handle a variety of data types that share a common base in a flexible and type-safe manner. Contravariance is particularly useful in designing frameworks and libraries where such flexibility is necessary.

We can subscribe to a source by passing an implementation of IObserver<T> to the Subscribe method. The source will invoke OnNext when it wants to report events. Rx has a basic rule that the source is required to wait until OnNext returns before either calling OnNext again, or calling OnError or OnComplete. This rule keeps things simple for observers: even in multithreaded applications, any single observer will only ever have to deal with one thing at a time. (More subtly, it may also require a source to detect re-entrancy: if the observer’s OnNext performs some action that indirectly causes the source to emit another item, the source is not allowed to make a recursive call into the observer. It has to wait until the OnNext in progress returns.) The source can call OnCompleted to indicate that there will be no further activity, and if it wants to report an error, it can call OnError. Both OnCompleted and OnError indicate the end of the stream, so another basic Rx rule is that an observable should not call any further methods on the observer after that.

- Sequential Processing Rule:

    - A fundamental rule in Rx is that calls to the observer's methods must be sequential. This means the observable must wait for OnNext to return before it can call OnNext again, or call OnError or OnComplete.
    - This rule simplifies the observer's implementation, especially in multithreaded scenarios, as it ensures that an observer doesn't have to handle simultaneous invocations of its methods.
- Handling Re-entrancy:

    - The rule against simultaneous invocations also implies that observables must handle re-entrancy. If an action within OnNext indirectly causes the same observable to emit another item, the observable can't just recursively call into the observer.
    - Instead, the observable needs to queue or otherwise manage these additional emissions until the current OnNext call completes.
- Finality of OnError and OnCompleted:

    - Both OnError and OnCompleted signal the end of the data stream.
    - Another basic rule in Rx is that once an observable calls either OnError or OnCompleted, it should not call any further methods on the observer. The stream is considered finished, and no more data should be pushed to the observer.


```cs
class MySubscriber<T> : IObserver<T>
{
    public void OnNext(T value) => Console.WriteLine("Received: " + value);
    public void OnCompleted() => Console.WriteLine("Complete");
    public void OnError(Exception ex) => Console.WriteLine("Error: " + ex);
}

public class SimpleColdSource : IObservable<string>
{
    public IDisposable Subscribe(IObserver<string> observer)
    {
        observer.OnNext("Hello,");
        observer.OnNext("World!");
        observer.OnCompleted();
        return EmptyDisposable.Instance;
    }

    private class EmptyDisposable : IDisposable
    {
        public readonly static EmptyDisposable Instance = new();
        public void Dispose() { }
    }
}

var source = new SimpleColdSource();
var sub = new MySubscriber<string>();
source.Subscribe(sub);
```