# combine

- In Apple's own words: "The Combine framework provides a declarative approach for how your app processes events. Rather than potentially implementing multiple delegate callbacks or completion handler closures, you can create a single processing chain for a given event source. Each part of the chain is a Combine operator that performs a distinct action on the elements received from the previous step."

## Asynchronous programming

- In a simple, single-threaded language, a program executes sequentially line-by-line. For example, in pseudocode:

```swift
 
 begin
var name = "Tom"
print(name)
name += " Harding"
print(name)
end
 
```

- Synchronous code is easy to understand and makes it especially easy to argue about the state of your data. With a single thread of execution, you can always be sure what the current state of your data is. In the example above, you know that the first print will always print "Tom" and the second will always print "Tom Harding".

- Now, imagine you wrote the program in a multi-threaded language that is running an asynchronous event-driven UI framework, like an iOS app running on Swift and UIKit.

- Consider what could potentially happen:

```swift
--- Thread 1 ---
begin
var name = "Tom"
print(name)
--- Thread 2 ---
name = "Billy Bob"
--- Thread 1 ---
name += " Harding"
print(name)
end
```

- Here, the code sets name's value to "Tom" and then adds "Harding" to it, just like before. But because another thread could execute at the same time, it's possible that some other part of your program could run between the two mutations of name and set it to another value like "Billy Bob".

- When the code is running concurrently on different cores, it's difficult to say which part of the code is going to modify the shared state first.

- The code running on "Thread 2" in the example above might be:

1- executing at exactly the same time on a different CPU core as your original code.
2- executing just before name += " Harding", so instead of the original value "Tom", it gets "Billy Bob" instead.

- What exactly happens when you run this code depends on the system load, and you might see different results each time you run the program. Managing mutable state in your app becomes a loaded task once you run asynchronous concurrent code.

# Foundation and UIKit/AppKit

- Apple has been improving asynchronous programming for their platforms over the years. They've created several mechanisms you can use, on different system levels, to create and execute asynchronous code. You’ve probably used these in your projects without giving them a second thought because they are so fundamental to writing mobile apps.

- You’ve probably used most of the following:

1- **NotificationCenter:** Executes a piece of code any time an event of interest happens, such as when the user changes the orientation of the device or when the software keyboard shows or hides on the screen.
2- **The delegate pattern:** Lets you define an object that acts on behalf of, or in coordination with, another object. For example, in your app delegate, you define what should happen when a new remote notification arrives, but you have no idea when this piece of code will be executed or how many times it will execute.

3- **Grand Central Dispatch and Operations:** Helps you abstract the execution of pieces of work. You can use them to schedule code to be executed sequentially in a serial queue or to run a multitude of tasks concurrently in different queues with different priorities.

4- **Closures**: Create detached pieces of code that you can pass around in your code, so other objects can decide whether to execute it, how many times, and in what context.


# Combine basics
- In broad strokes, the three key moving pieces in Combine are publishers, operators and subscribers. There are, of course, more players in the team, but without those three you can't achieve much.


### **Publishers**
- Publishers are types that can emit values over time to one or more interested parties, such as subscribers. Regardless of the internal logic of the publisher, which can be pretty much anything including math calculations, networking or handling user events, every publisher can emit multiple events of these three types:

1. An output value of the publisher's generic Output type.
2. A successful completion.
3. A completion with an error of the publisher's Failure type.

- A publisher can emit zero or more output values, and if it ever completes, either successfully or due to a failure, it will not emit any other events.

- One of the best features of publishers is that they come with error handling built in; error handling isn't something you add optionally at the end, if you feel like it. The Publisher protocol is generic over two types, as you might have noticed in the diagram earlier:

• Publisher.Output is the type of the output values of the publisher. If the publisher is specialized as an Int, it can never emit a String or a Date value.
• Publisher.Failure is the type of error the publisher can throw if it fails. If the publisher can never fail, you specify that by using a Never failure type.
- When you subscribe to a given publisher, you know what values to expect from it and which errors it could fail with.

### **Operators**
- Operators are methods declared on the Publisher protocol that return either the same or a new publisher. That's very useful because you can call a bunch of operators one after the other, effectively chaining them together.

- Because these methods, called "operators", are highly decoupled and composable, they can be combined (aha!) to implement very complex logic over the execution of a single subscription.

- It's fascinating how operators fit tightly together like puzzle pieces. They cannot be mistakenly put in the wrong order or fit together if one's output doesn't match the next one's input type:

- In a clear deterministic way, you can define the order of each of those asynchronous abstracted pieces of work alongside with the correct input/output types and built-in error handling. It's almost too good to be true!
- As an added bonus, operators always have input and output, commonly referred to as upstream and downstream — this allows them to avoid shared state (one of the core issues we discussed earlier).
- Operators focus on working with the data they receive from the previous operator and provide their output to the next one in the chain. This means that no other asynchronously-running piece of code can "jump in" and change the data you're working on.

### **Subscribers**
- Finally, you arrive at the end of the subscription chain: Every subscription ends with a subscriber. Subscribers generally do "something" with the emitted output or completion events.
- Currently, Combine provides two built-in subscribers, which make working with data streams straightforward:

1- The sink subscriber allows you to provide closures with your code that will receive output values and completions. From there, you can do anything your heart desires with the received events.

2- The assign subscriber allows you to, without the need of custom code, bind the resulting output to some property on your data model or on a UI control to display the data directly on-screen via a key path.

- Should you have other needs for your data, creating custom subscribers is even easier than creating publishers. Combine uses a set of very simple protocols that allow you to be able to build your own custom tools whenever the workshop doesn't offer the right one for your task.

### **Subscriptions**

- When you add a subscriber at the end of a subscription, it "activates" the publisher all the way at the beginning of the chain. This is a curious but important detail to remember — publishers do not emit any values if there are no subscribers to potentially receive the output.

- Subscriptions are a wonderful concept in that they allow you to declare a chain of asynchronous events with their own custom code and error handling only once, and then you never have to think about it again.

- If you go full-Combine, you could describe your whole app's logic via subscriptions and once done, just let the system run everything without the need to push or pull data or call back this or that other object:

# Publishers & Subscribers

## Publisher
- At the heart of Combine is the Publisher protocol. This protocol defines the requirements for a type to be able to transmit a sequence of values over time to one or more subscribers. In other words, a publisher publishes or emits events that can include values of interest.

- If you’ve developed on Apple platforms before, you can think of a publisher as kind of like NotificationCenter. In fact, NotificationCenter now has a method named publisher(for:object:) that provides a Publisher type that can publish broadcasted notifications.

- ### **So what’s the point of publishing notifications when a notification center is already capable of broadcasting its notifications without a publisher?**
- You can think of these types of methods as a bridge from the old to the new — a way to Combine-ify existing APIs such as NotificationCenter.

- A publisher emits two kinds of events:
1- Values, also referred to as elements.
2- A completion event.

- A publisher can emit zero or more values but only one completion event, which can either be a normal completion event or an error. Once a publisher emits a completion event, it’s finished and can no longer emit any more events.
- Before diving deeper into publishers and subscribers, you’ll first finish the example of using traditional NotificationCenter APIs to receive a notification by registering an observer. You’ll also unregister that observer when you’re no longer interested in receiving that notification.


## Subscriber
- Subscriber is a protocol that defines the requirements for a type to be able to receive input from a publisher
```swift

example(of: "Subscriber") {
let myNotification = Notification.Name("MyNotification")
let publisher = NotificationCenter.default
.publisher(for: myNotification, object: nil)
let center = NotificationCenter.default
}

```
- If you were to post a notification now, the publisher wouldn’t emit it — and that’s an important distinction to remember. A publisher only emits an event when there’s at least one subscriber.

### Subscribing with sink(_:_:)
- Continuing the previous example, add the following code to the example to create a subscription to the publisher:

```swift
let subscription = publisher
.sink { _ in
print("Notification received from a publisher!")
}
```
- With this code, you create a subscription by calling sink on the publisher — but don’t let the obscurity of that method name give you a sinking feeling. Option-click on sink and you’ll see that it simply provides an easy way to attach a subscriber with closures to handle output from a publisher. In this example, you ignore those closures and instead just print a message to indicate that a notification was received.

- Run the playground and you’ll see the following:
> ——— Example of: Publisher ——— 
> Notification received from a publisher!

- The sink operator will continue to receive as many values as the publisher emits. This is known as unlimited demand,  And although you ignored them in the previous example, the sink operator actually provides two closures: one to handle receiving a completion event, and one to handle receiving values.

- To see how this works, add this new example to your playground:

```swift
example(of: "Just") {
    // 1
    let just = Just("Hello world!")
    // 2
    _ = just
        .sink(
        receiveCompletion: {
            print("Received completion", $0)
        },
    receiveValue: {
        print("Received value", $0)
    })
}
```
Here, you:
1. Create a publisher using Just, which lets you create a publisher from a primitive value type.
2. Create a subscription to the publisher and print a message for each received event. 
Run the playground. You’ll see the following:

> ——— Example of: Just ———
Received value Hello world!
Received completion finished

- Option-click on Just and the Quick Help explains that it’s a publisher that emits its output to each subscriber once and then finishes.
- Try adding another subscriber by adding the following code to the end of your example:

```swift
_ = just
    .sink(
    receiveCompletion: {
        print("Received completion (another)", $0)
    },
    receiveValue: {
        print("Received value (another)", $0)
    })
```
- Run the playground. True to its word, a Just happily emits its output to each new subscriber exactly once and then finishes.

> Received value (another) Hello world!
Received completion (another) finished

### Subscribing with assign(to:on:)

- In addition to sink, the built-in assign(to:on:) operator enables you to assign the received value to a KVO-compliant property of an object.

- Add this example to see how this works:

```swift

example(of: "assign(to:on:)") {
    // 1
    class SomeObject {
        var value: String = "" {
            didSet {
                print(value)
            }
        }
    }
    // 2
    let object = SomeObject()
    // 3
    let publisher = ["Hello", "world!"].publisher
    // 4
    _ = publisher
    .assign(to: \.value, on: object)
}

```
> ——— Example of: assign(to:on:) ———
Hello
world!

### Cancellable
- When a subscriber is done and no longer wants to receive values from a publisher, it’s a good idea to cancel the subscription to free up resources and stop any corresponding activities from occurring, such as network calls.
- Subscriptions return an instance of AnyCancellable as a "cancellation token," which makes it possible to cancel the subscription when you’re done with it. AnyCancellable conforms to the Cancellable protocol, which requires the cancel() method exactly for that purpose.

Finish the Subscriber example from earlier by adding the following code:

```swift
// 1
center.post(name: myNotification, object: nil)
// 2
subscription.cancel()
```
- Cancel the subscription. You’re able to call cancel() on the subscription because the Subscription protocol inherits from Cancellable.
- If you don’t explicitly call cancel() on a subscription, it will continue until the publisher completes, or until normal memory management causes a stored subscription to be deinitialized. At that point it will cancel the subscription for you.

> **Note**: It’s also fine to ignore the return value from a subscription in a
playground (for example, _ = just.sink...). However, one caveat: if you
don’t store a subscription in full projects, that subscription will cancel as soon
as the program flow exits the scope in which it was created!


- Take a look at the Publisher protocol and one of its most crucial extensions:

```swift
public protocol Publisher {
    // 1
    associatedtype Output
    // 2
    associatedtype Failure : Error
    // 4
    func receive<S>(subscriber: S)
    where S: Subscriber,
    Self.Failure == S.Failure,
    Self.Output == S.Input
}
extension Publisher {
    // 3
    public func subscribe<S>(_ subscriber: S)
    where S : Subscriber,
    Self.Failure == S.Failure,
    Self.Output == S.Input
}

```

- Here’s a closer look:
1- The type of values that the publisher can produce.
2- The type of error a publisher may produce, or Never if the publisher is guaranteed to not produce an error.
3- A subscriber calls subscribe(_:) on a publisher to attach to it.
4- The implementation of subscribe(_:) will call receive(subscriber:) to attach the subscriber to the publisher, i.e., create a subscription.

- The associated types are the publisher’s interface that a subscriber must match in order to create a subscription.

- look at the Subscriber protocol: 

```swift
public protocol Subscriber: CustomCombineIdentifierConvertible {
    // 1
    associatedtype Input
    // 2
    associatedtype Failure: Error
    // 3
    func receive(subscription: Subscription)
    // 4
    func receive(_ input: Self.Input) -> Subscribers.Demand
    // 5
    func receive(completion: Subscribers.Completion<Self.Failure>)
} 
```
- Here’s a closer look:
1- The type of values a subscriber can receive.
2- The type of error a subscriber can receive; or Never if the subscriber won’t receive an error. 
3- The publisher calls receive(subscription:) on the subscriber to give it the subscription.
4- The publisher calls receive(_:) on the subscriber to send it a new value that it just published.
5- The publisher calls receive(completion:) on the subscriber to tell it that it has finished producing values, either normally or due to an error.


- The connection between the publisher and the subscriber is the subscription. Here’s the Subscription protocol:
```swift
public protocol Subscription: Cancellable, CustomCombineIdentifierConvertible {
    func request(_ demand: Subscribers.Demand)
}
```
- The subscriber calls request(_:) to indicate it is willing to receive more values, up to a max number or unlimited.

> **Note**: The concept of a subscriber stating how many values it’s willing to receive is known as backpressure management. Without it, or some other strategy, a subscriber could get flooded with more values from the publisher than it can handle, and this can lead to problems. Backpressure is also covered in depth in the next.

- In Subscriber, notice that receive(_:) returns a Demand. Even though the max number of values a subscriber is willing to receive is specified when initially calling subscription.request(_:) in receive(_:), you can adjust that max each time a new value is received.

> Note: Adjusting max in Subscriber.receive(_:) is additive, i.e., the new max value is added to the current max. The max value must be positive, and passing a negative value will result in a fatalError. This means that you can increase the original max each time a new value is received, but you cannot decrease it.


## Creating a custom subscriber

```swift
example(of: "Custom Subscriber") {
    // 1
    let publisher = (1...6).publisher
    // 2
    final class IntSubscriber: Subscriber {
        // 3
        typealias Input = Int
        typealias Failure = Never
        // 4
        func receive(subscription: Subscription) {
            subscription.request(.max(3))
        }
        // 5
        func receive(_ input: Int) -> Subscribers.Demand {
            print("Received value", input)
            return .none
        }
        // 6
        func receive(completion: Subscribers.Completion<Never>) {
            print("Received completion", completion)
        }
    }
    
    let subscriber = IntSubscriber()
    publisher.subscribe(subscriber)
    
}
```
- What you do here is: 
1- Create a publisher of integers via the range’s publisher property.
2- Define a custom subscriber, IntSubscriber.
3- Implement the type aliases to specify that this subscriber can receive integer inputs and will never receive errors.
4- Implement the required methods, beginning with receive(subscription:), which is called by the publisher; and in that method, call .request(_:) on the subscription specifying that the subscriber is willing to receive up to three values upon subscription.

5- Print each value as it’s received and return .none, indicating that the subscriber will not adjust its demand; .none is equivalent to .max(0).
6- Print the completion event.

- In this code, you create a subscriber that matches the Output and Failure types of the publisher. You then tell the publisher to subscribe, or attach, the subscriber.

- Run the playground. You’ll see the following printed to the console:

> ——— Example of: Custom Subscriber ———
> Received value 1
> Received value 2
> Received value 3

- You did not receive a completion event. This is because the publisher has a finite number of values, and you specified a demand of .max(3).

- In your custom subscriber’s receive(_:), try changing .none to .unlimited, so your receive(_:) method looks like this:

```swift
func receive(_ input: Int) -> Subscribers.Demand {
    print("Received value", input)
    return .unlimited
}
```

- Run the playground again. This time you’ll see that all of the values are received and printed, along with the completion event:

> ——— Example of: Custom Subscriber ———
> Received value 1
> Received value 2
> Received value 3
> Received value 4
> Received value 5
> Received value 6
> Received completion finished

- Try changing .unlimited to .max(1) and run the playground again.

- You’ll see the same output as when you returned .unlimited, because each time you receive an event, you specify that you want to increase the max by 1.

- Change .max(1) back to .none, and change the definition of publisher to an array of strings instead. Replace:

```swift
let publisher = (1...6).publisher
```
with 

```swift
let publisher = ["A", "B", "C", "D", "E", "F"].publisher
```

- Run the playground. You get an error that the subscribe method requires types String and IntSubscriber.Input (i.e., Int) to be equivalent. You get this error because the Output and Failure associated types of a publisher must match the Input and Failure types of a subscriber in order for a subscription between the two to be created.

- Change the publisher definition back to its original range of integers to resolve the error.

## Hello Future
- Much like you can use Just to create a publisher that emits a single value to a subscriber and then complete, a Future can be used to asynchronously produce a single result and then complete. Add this new example to your playground:

```swift
example(of: "Future") {
    func futureIncrement(
        integer: Int,
        afterDelay delay: TimeInterval) -> Future<Int, Never> {
        
    }
}
```
- Here, you create a factory function that returns a future of type Int and Never; meaning, it will emit an integer and never fail.

- You also add a subscriptions set in which you’ll store the subscriptions to the future in the example. For long-running asynchronous operations, not storing the subscription will result in the cancelation of the subscription as soon as the current code scope ends. In the case of a Playground, that would be immediately.

- Next, fill the function’s body to create the future:

```swift
Future<Int, Never> { promise in
    DispatchQueue.global().asyncAfter(deadline: .now() + delay) {
        promise(.success(integer + 1))
    }
}
```
- This code defines the future, which creates a promise that you then execute using the values specified by the caller of the function to increment the integer after the delay.

- A Future is a publisher that will eventually produce a single value and finish, or it will fail. It does this by invoking a closure when a value or error is made available, and that closure is referred to as a promise. Command-click on Future and choose Jump to Definition. You’ll see the following:

```swift
final public class Future<Output, Failure> : Publisher
where Failure: Error {
    public typealias Promise = (Result<Output, Failure>) -> Void
    ...
}
```

- Promise is a type alias to a closure that receives a Result containing either a single value published by the Future, or an error.

- Head back to the main playground page, and add the following code after the definition of futureIncrement:

```swift
// 1
let future = futureIncrement(integer: 1, afterDelay: 3)
// 2
future
    .sink(receiveCompletion: { print($0) },
    receiveValue: { print($0) })
    .store(in: &subscriptions)
```
1- Create a future using the factory function you created earlier, specifying to increment the integer you passed after a three-second delay.
2- Subscribe to and print the received value and completion event, and store the resulting subscription in the subscriptions set. You’ll learn more about storing subscriptions in a collection later in this chapter, so don’t worry if you don’t entirely understand that portion of the example.

- Run the playground. You’ll see the example title printed, followed by the output of the future after a three-second delay:

> ——— Example of: Future ———
2
finished

- Add a second subscription to the future by entering the following code in the playground:

```swift
future
    .sink(receiveCompletion: { print("Second", $0) },
    receiveValue: { print("Second", $0) })
    .store(in: &subscriptions)
```
- Before running the playground, insert the following print statement immediately before the DispatchQueue block in the futureIncrement function:

```swift
print("Original")
```

- Run the playground. After the specified delay, the second subscription receives the same value. The future does not re-execute its promise; instead, it shares or replays its output.

> ——— Example of: Future ———
Original
2
finished
Second 2
Second finished


- Also, Original is printed right away before the subscriptions occur. This happens because a future executes as soon as it is created. It does not require a subscriber like regular publishers.

- In the last few examples, you’ve been working with publishers that have a finite number of values to publish, which are sequentially and synchronously published.

- The notification center example you started with is an example of a publisher that can keep on publishing values indefinitely and asynchronously, provided:

1- The underlying notification sender emits notifications.
2- There are subscribers to the specified notification.

- What if there was a way that you could do the same thing in your own code? Well, it turns out, there is! Before moving on, comment out the entire "Future" example, so the future isn’t invoked every time you run the playground — otherwise its delayed output will be printed after the last example.

## Hello Subject
