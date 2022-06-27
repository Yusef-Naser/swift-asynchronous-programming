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

```swift

example(of: "PassthroughSubject") {
    // 1
    enum MyError: Error {
    case test
    }
    // 2
    final class StringSubscriber: Subscriber {
        typealias Input = String
        typealias Failure = MyError
        func receive(subscription: Subscription) {
            subscription.request(.max(2))
        }
        func receive(_ input: String) -> Subscribers.Demand {
            print("Received value", input)
            // 3
            return input == "World" ? .max(1) : .none
        }
        func receive(completion: Subscribers.Completion<MyError>) {
            print("Received completion", completion)
        }
    }
    // 4
    let subscriber = StringSubscriber()
    
    // 5
    let subject = PassthroughSubject<String, MyError>()
    // 6
    subject.subscribe(subscriber)
    // 7
    let subscription = subject
        .sink(
            receiveCompletion: { completion in
                print("Received completion (sink)", completion)
            },
            receiveValue: { value in
                print("Received value (sink)", value)
            }
        )
    
    
}

```

1. Define a custom error type.
2. Define a custom subscriber that receives strings and MyError errors.
3. Adjust the demand based on the received value.
4. Create an instance of the custom subscriber.
5. Creates an instance of a PassthroughSubject of type String and the custom error type you defined.
6. Subscribes the subscriber to the subject.
7. Creates another subscription using sink.


- Returning .max(1) in receive(_:) when the input is "World" results in the new max being set to 3 (the original max plus 1), Other than defining a custom error type and pivoting on the received value to adjust demand

- Passthrough subjects enable you to publish new values on demand. They will happily pass along those values and a completion event. As with any publisher, you must declare the type of values and errors it can emit in advance; subscribers must match those types to its input and failure types in order to subscribe to that passthrough subject.

- Now that you’ve created a passthrough subject that can send values and subscriptions to receive them, it’s time to send some values. Add the following code to your example:

```swift
subject.send("Hello")
subject.send("World")
```
> ——— Example of: PassthroughSubject ——— 
Received value Hello 
Received value (sink) Hello
Received value World
Received value (sink) World

Add the following code:

```swift
    // 8
    subscription.cancel()
    // 9
    subject.send("Still there?")
```
8. Cancel the second subscription.
9. Send another value.

- Run the playground. As you might have expected, only the first subscriber receives the value. This happens because you previously canceled the second subscriber’s subscription:

> ——— Example of: PassthroughSubject ——— 
Received value Hello
Received value (sink) Hello
Received value World
Received value (sink) World
Received value Still there?

- Add this code to the example:
```swift
subject.send(completion: .finished)
subject.send("How about another one?") 
```

- Run the playground. The second subscriber does not receive the "How about another one?" value, because it received the completion event right before that value was sent. The first subscriber does not receive the completion event or the value, because its subscription was previously canceled.

> ——— Example of: PassthroughSubject ——— 
> Received value Hello 
> Received value (sink) Hello
> Received value World
> Received value (sink) World
> Received value Still there?
> Received completion finished

- Add the following code immediately before the line that sends the completion event.

```swift
subject.send(completion: .failure(MyError.test))
```
- Instead of storing each subscription as a value, you can store multiple subscriptions in a collection of AnyCancellable. The collection will then automatically cancel each subscription added to it when the collection is about to be deinitialized.

```swift
example(of: "CurrentValueSubject") {
    // 1
    let subject = CurrentValueSubject<Int, Never>(0)
    // 2
    subject
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions) // 3
    
    subject.send(1)
subject.send(2)

}
```
1- Create a CurrentValueSubject of type Int and Never. This will publish integers and never publish an error, with an initial value of 0.
2- Create a subscription to the subject and print values received from it.
3- Store the subscription in the subscriptions set, which is passed as an inout parameter so that the same set is updated instead of a copy.

> ——— Example of: CurrentValueSubject ———
> 0
> 1
> 2

- Unlike a passthrough subject, you can ask a current value subject for its value at any time. Add the following code to print out the subject’s current value:

```swift
print(subject.value)
subject.value = 3
print(subject.value)

```

- Next, at the end of this example, create a new subscription to the current value subject:

```swift
subject
.sink(receiveValue: { print("Second subscription:", $0) })
.store(in: &subscriptions)
```
- You read a moment ago that the subscriptions set will automatically cancel the subscriptions added to it, but how can you verify this? You can use the print() operator, which will log all publishing events to the console.

- Insert the print() operator in both subscriptions, between subject and sink. The beginning of each subscription should look like this:

```swift
subject
.print()
.sink...
```
- Run the playground again and you’ll see the following output for the entire example:
> ——— Example of: CurrentValueSubject ———
> receive subscription: (CurrentValueSubject)
> request unlimited
> receive value: (0)
> 0
> receive value: (1)
> 1
> receive value: (2)
> 2
> 2
> receive value: (3)
> 3
> 3
> receive subscription: (CurrentValueSubject)
> request unlimited
> receive value: (3)
> Second subscription: 3
> receive cancel
> receive cancel

- values. Completion events must still be sent using send(_:).
```swift
subject.send(completion: .finished)
```

## Dynamically adjusting demand

```swift
example(of: "Dynamically adjusting Demand") {
final class IntSubscriber: Subscriber {
    typealias Input = Int
    typealias Failure = Never
    func receive(subscription: Subscription) {
        subscription.request(.max(2))
    }
    func receive(_ input: Int) -> Subscribers.Demand {
        print("Received value", input)
        switch input {
        case 1:
            return .max(2) // 1
        case 3:
            return .max(1) // 2
        default:
            return .none // 3
        }
    }
    func receive(completion: Subscribers.Completion<Never>) {
        print("Received completion", completion)
    }
}
let subscriber = IntSubscriber()
let subject = PassthroughSubject<Int, Never>()
subject.subscribe(subscriber)
subject.send(1)
subject.send(2)
subject.send(3)
subject.send(4)
subject.send(5)
subject.send(6)
}
```
1. The new max is 4 (original max of 2 + new max of 2).
2. The new max is 5 (previous 4 + new 1).
3. max remains 5 (previous 4 + new 0).

- Run the playground and you’ll see the following:

> ——— Example of: Dynamically adjusting Demand ———
> Received value 1
> Received value 2
> Received value 3
> Received value 4
> Received value 5

- As expected, five values are emitted but the sixth is not printed out.

## Type erasure

- There will be times when you want to let subscribers subscribe to receive events from a publisher without being able to access additional details about that publisher.
- This would be best demonstrated with an example, so add this new one to your playground:

```swift
example(of: "Type erasure") {
    // 1
    let subject = PassthroughSubject<Int, Never>()
    // 2
    let publisher = subject.eraseToAnyPublisher()
    // 3
    publisher
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
    // 4
    subject.send(0)
}
```
1. Create a passthrough subject.
2. Create a type-erased publisher from that subject.
3. Subscribe to the type-erased publisher.
4. Send a new value through the passthrough subject.

- AnyPublisher is a type-erased struct that conforms the Publisher protocol. Type erasure allows you to hide details about the publisher that you may not want to expose to subscribers — or downstream publishers, which you’ll learn about in the next section.
- AnyCancellable is a type-erased class that conforms to Cancellable, which lets callers cancel the subscription without being able to access the underlying subscription to do things like request more items.

- The eraseToAnyPublisher() operator wraps the provided publisher in an instance of AnyPublisher, hiding the fact that the publisher is actually a PassthroughSubject. This is also necessary because you cannot specialize the Publisher protocol, e.g., you cannot define the type as Publisher<UIImage, Never>.

- To prove that publisher is type-erased and cannot be used to send new values, add this code to the example.

```swift
publisher.send(1)
```
- You get the error Value of type 'AnyPublisher<Int, Never>' has no member 'send'. Comment out that line of code before moving on.


# Operators

- **Transforming Operators**: Before a subscriber receives values from a publisher, you’ll often want to manipulate those values in some way. One of the most common things you’ll want to do is transform those values into some form that is ideal for use by the subscriber. By the end of this chapter you’ll be transforming all the things.

- **Filtering Operators** :you'll learn about filtering values from Combine publishers, so you can easily control the values published by the upstream and only deal with the ones you care about.

- **Combining Operators** : Publishers are extremely powerful, but they're even more powerful when composed together! This chapter will teach you about Combine's combining operators which let you take multiple publishers and create meaningful logical relationships between them.

- **Time Manipulation Operators** : A large part of asynchronous programming relates to processing values over time. This chapter goes into the details of performing complex time-based tasks that would be hard to do without Combine.

- **Sequence Operators** : When you think about it, publishers are merely sequences. As such, there are many useful operators that let you target specific values, or gather information about the sequence as a whole, which you'll learn about in this chapter.

# Transforming Operators

## Operators are publishers

- In Combine, methods that perform an operation on values coming from a publisher are called operators.
- Each Combine operator actually returns a publisher. Generally speaking, that publisher receives the upstream values, manipulates the data, and then sends that data downstream. To streamline things conceptually, the focus will be on using the operator and working with its output. Unless an operator’s purpose is to handle errors, if it receives an error from an upstream publisher, it will just publish that error downstream.

## Collecting values **collect()**
- The `collect` operator provides a convenient way to transform a stream of individual values from a publisher into an array of those values

```swift
example(of: "collect") {
["A", "B", "C", "D", "E"].publisher
.sink(receiveCompletion: { print($0) },
receiveValue: { print($0) })
.store(in: &subscriptions)
}
```
- This is not using the collect operator yet. Run the playground, and you’ll see each value is emitted and printed individually followed by the completion:

> ——— Example of: collect ———
> A
> B
> C
> D
> E
> finished

- Now insert the use of collect before the sink. Your code should look like this:

```swift
["A", "B", "C", "D", "E"].publisher
.collect()
.sink(receiveCompletion: { print($0) },
receiveValue: { print($0) })
.store(in: &subscriptions)
```
> ——— Example of: collect ———
> ["A", "B", "C", "D", "E"]
> finished

> **Note**: Be careful when working with collect() and other buffering operators that do not require specifying a count or limit. They will use an unbounded amount of memory to store received values.

- There are a few variations of the collect operator. For example, you can specify that you only want to receive up to a certain number of values.

- Replace the following line:
```swift
.collect()
```
- with:
```swift
.collect(2)
```
- Run the playground, and you’ll see the following output:
> ——— Example of: collect ———
> ["A", "B"]
> ["C", "D"]
> ["E"]
> finished


## Mapping values **map(_:)**

- The first you’ll learn about is map, which works just like Swift’s standard map, except that it operates on values emitted from a publisher. In the marble diagram, map takes a closure that multiplies each value by 2.

- Add this new example to your playground:

```swift
example(of: "map") {
    // 1
    let formatter = NumberFormatter()
    formatter.numberStyle = .spellOut
    // 2
    [123, 4, 56].publisher
    // 3
    .map {
    formatter.string(for: NSNumber(integerLiteral: $0)) ?? ""
    }
    .sink(receiveValue: { print($0) })
    .store(in: &subscriptions)
}
```
1. Create a number formatter to spell out each number.
2. Create a publisher of integers.
3. Use map, passing a closure that gets upstream values and returns the result of using the formatter to return the number’s spelled out string.

> ——— Example of: map ———
> one hundred twenty-three
> four
> fifty-six

## Map key paths
- The map family of operators also includes three versions that can map into one, two, or three properties of a value using key paths. Their signatures are as follows:

• map<T>(_:)
• map<T0, T1>(_:_:)
• map<T0, T1, T2>(_:_:_:)

- The T represents the type of values found at the given key paths.

```swift
example(of: "map key paths") {
    // 1
    let publisher = PassthroughSubject<Coordinate, Never>()
    // 2
    publisher
    // 3
    .map(\.x, \.y)
    .sink(receiveValue: { x, y in
    // 4
    print(
    "The coordinate at (\(x), \(y)) is in quadrant",
    quadrantOf(x: x, y: y)
    )
    })
    .store(in: &subscriptions)
    // 5
    publisher.send(Coordinate(x: 10, y: -8))
    publisher.send(Coordinate(x: 0, y: 5))
}
```

1. Create a publisher of Coordinates that will never emit an error.
2. Begin a subscription to the publisher.
3. Map into the x and y properties of Coordinate using their key paths.
4. Print a statement that indicates the quadrant of the provide x and y values.
5. Send some coordinates through the publisher.

- Run the playground and the output from this subscription will be the following:

>——— Example of: map key paths ———
>The coordinate at (10, -8) is in quadrant 4
>The coordinate at (0, 5) is in quadrant boundary

## tryMap(_:)

- Several operators, including map, have a counterpart try operator that will take a closure that can throw an error. If you throw an error, it will emit that error downstream. Add this example to the playground:

```swift
example(of: "tryMap") {
    // 1
    Just("Directory name that does not exist")
    // 2
    .tryMap { try
    FileManager.default.contentsOfDirectory(atPath: $0) }
    // 3
    .sink(receiveCompletion: { print($0) },
    receiveValue: { print($0) })
    .store(in: &subscriptions)
}
```
1. Create a publisher of a string representing a directory name that does not exist.
2. Use tryMap to attempt to get the contents of that nonexistent directory.
3. Receive and print out any values or completion events.

- Notice that you still need to use the try keyword when calling a throwing method.
- Run the playground and observe that tryMap outputs a failure completion event with the appropriate “folder doesn’t exist” error (output abbreviated):

> ——— Example of: tryMap ———
>failure(..."The folder “Directory name that does not exist” doesn't exist."...)

## Flattening publishers

- The flatMap operator can be used to flatten multiple upstream publishers into a single downstream publisher — or more specifically, flatten the emissions from those publishers.
- A common use case for flatMap in Combine is when you want to subscribe to properties of values emitted by a publisher that are themselves publishers.

- Time to implement an example to see this in action. Start by taking a look at the code in Sources/SupportCode.swift. It includes the definition of a Chatter structure with two properties:
1. name is a regular string.
2. message is a CurrentValueSubject subject that is initialized with the message string passed in.

```swift
public struct Chatter {
    public let name: String
    public let message: CurrentValueSubject<String, Never>
    public init(name: String, message: String) {
        self.name = name
        self.message = CurrentValueSubject(message)
    }
}
```

```swift
example(of: "flatMap") {
    // 1
    let charlotte = Chatter(name: "Charlotte", message: "Hi, I'm
    Charlotte!")
    let james = Chatter(name: "James", message: "Hi, I'm James!")
    // 2
    let chat = CurrentValueSubject<Chatter, Never>(charlotte)
    // 3
    chat
    .sink(receiveValue: { print($0.message.value) })
    .store(in: &subscriptions)
}
```

- From the top, you:
1. Create two instances of Chatter: charlotte and james.

2. Create a chat publisher initialized with charlotte.

3. Subscribe to chat and print out the message of any received Chatter structure.

- Run the playground, and as you might expect, Charlotte’s message is printed out:
> ——— Example of: flatMap ———
> Charlotte wrote: Hi, I'm Charlotte!

- Now add this code to the example:

```swift
// 4
charlotte.message.value = "Charlotte: How's it going?"
// 5
chat.value = james
```
- You don’t see Charlotte’s new message, but you do see James’ initial message. That’s because you’re subscribed to chat, which is a Chatter publisher. You are not subscribed to the message publisher property of each emitted Chatter. What if you wanted to subscribe to the message of every chat? You can use flatMap.

- Find the following code:
```swift
chat
.sink(receiveValue: { print($0.message.value) })
.store(in: &subscriptions)
```
- And replace it with:

```swift
chat
// 6
.flatMap { $0.message }
// 7
.sink(receiveValue: { print($0) })
.store(in: &subscriptions)
```
6. Flat map into the Chatter structure message publisher.

7. Change the handler to print the value received, which is now a string, not a Chatter instance.

- Run the playground again, and now you’ll see Charlotte’s new message printed.

> Hi, I'm Charlotte!
> Charlotte: How's it going?
> Hi, I'm James!

- Now add the following code to the example, which changes each Chatter’s message value:

> james.message.value = "James: Doing great. You?"
>charlotte.message.value = "Charlotte: I'm doing fine thanks."

- Run the playground, and you may be surprised by what you see:

> James: Doing great. You?
> Charlotte: I'm doing fine thanks.

- To help you manage flatMap’s memory footprint, you can optionally specify how many publishers flatMap will receive and buffer using its maxPublishers parameter.

- Find the following line:

```swift
.flatMap { $0.message }
```
- And replace it with:

```swift
.flatMap(maxPublishers: .max(2)) { $0.message }
```

- You specify that flatMap will receive a maximum of two upstream publishers. It will ignore any additional publishers. If not specified, maxPublishers defaults to `.unlimited`.

- You previously set flatMap’s maxPublishers parameter to two. Now add the following code to the bottom of the example to see how this affects the output:

```swift
// 8
let morgan = Chatter(name: "Morgan",
message: "Hey guys, what are you up to?")
// 9
chat.value = morgan
// 10
charlotte.message.value = "Did you hear something?"
```
8. Create a third Chatter instance.
9. Add that Chatter onto the chat publisher.
10. Change Charlotte’s message.

- Run the playground and see if you were right:

> ——— Example of: flatMap ———
> Hi, I'm Charlotte!
> Charlotte: How's it going?
> Hi, I'm James!
> James: Doing great. You?
> Charlotte: I'm doing fine thanks.
> Did you hear something?

- Morgan’s message is not printed, because flatMap will only receive up to a max of two publishers.

## replaceNil(with:)
- replaceNil will receive optional values and replace nils with the value you specify:

- Add this new example to your playground:
```swift
example(of: "replaceNil") {
    // 1
    ["A", nil, "C"].publisher
    .replaceNil(with: "-") // 2
    .sink(receiveValue: { print($0) }) // 3
    .store(in: &subscriptions)
}
```

> ——— Example of: replaceNil ———
> Optional("A")
> Optional("-")
> Optional("C")

- The optional values were not converted to non-optional ones. True to its name, replaceNil(with:) replaced a nil with a non-nil value. One way that you could convert the output from replaceNil to a non-optional value is by inserting the use of map to force-unwrap it. Do so by changing your code to the following:

> ["A", nil, "C"].publisher
> .replaceNil(with: "-")
> .map { $0! }
> .sink(receiveValue: { print($0) })
> .store(in: &subscriptions)

## replaceEmpty(with:)
- You can use the replaceEmpty(with:) operator to replace — or really, insert — a value if a publisher completes without emitting a value.

```swift

example(of: "replaceEmpty(with:)") {
    // 1
    let empty = Empty<Int, Never>()
    // 2
    empty
    .sink(receiveCompletion: { print($0) },
    receiveValue: { print($0) })
    .store(in: &subscriptions)
}

```
1. Create an empty publisher that immediately emits a completion event.
2. Subscribe to it, and print received events.

- The Empty publisher type can be used to create a publisher that immediately emits a .finished completion event. It can also be configured to never emit anything by passing false to its completeImmediately parameter, which is true by default. This publisher is useful for demo or testing purposes, or when all you want to do is signal completion of some task to a subscriber. Run the playground and its completion event is printed:

> ——— Example of: replaceEmpty ———
> finished

- Now, insert this line of code before calling sink:

```swift
.replaceEmpty(with: 1)
```

- Run the playground again, and this time you get a 1 before the completion:

> 1
> finished

## scan(_:_:)

- A great example of this in the transforming category is scan. It will provide the current value emitted by an upstream publisher to a closure, along with the last value returned by that closure.

- scan begins by storing a starting value of 0. As it receives each value from the publisher, it adds it to the previously stored value, and then stores and emits the result:

> **Note**: If you are using the full project to enter and run this code, there’s no straightforward way to plot the output — as is possible in a playground. Instead, you can print the output by changing the sink code in the example below to .sink(receiveValue: { print($0) }).

- For a practical example of how to use scan, add this new example to your playground:

```swift

example(of: "scan") {
    // 1
    var dailyGainLoss: Int { .random(in: -10...10) }
    // 2
    let august2019 = (0..<22)
    .map { _ in dailyGainLoss }
    .publisher
    // 3
    august2019
    .scan(50) { latest, current in
    max(0, latest + current)
    }
    .sink(receiveValue: { _ in })
    .store(in: &subscriptions)
}

```
1. Create a computed property that generates a random integer between -10 and 10.
2. Use that generator to create a publisher from an array of random integers representing fictitious daily stock price changes for a month.
3. Use scan with a starting value of 50, and then add each daily change to the running stock price. The use of max keeps the price non-negative — thankfully stock prices can’t fall below zero!


# Filtering Operators

## Filtering basics

```swift
example(of: "filter") {
// 1
let numbers = (1...10).publisher
// 2
numbers
.filter { $0.isMultiple(of: 3) }
.sink(receiveValue: { n in
    print("\(n) is a multiple of 3!")
})
.store(in: &subscriptions)
}
```
1. Create a new publisher, which will emit a finite number of values — 1 through 10, and then complete, using the publisher property on Sequence types.
2. Use the filter operator, passing in a predicate where you only allow through numbers that are multiples of three.

> ——— Example of: filter ———
> 3 is a multiple of 3!
> 6 is a multiple of 3!
> 9 is a multiple of 3!


## removeDuplicates()

```swift
example(of: "removeDuplicates") {
// 1
let words = "hey hey there! want to listen to mister mister ?"
.components(separatedBy: " ")
.publisher
// 2
words
.removeDuplicates()
.sink(receiveValue: { print($0) })
.store(in: &subscriptions)
}
```
1. Separate a sentence into an array of words (e.g., Array<String>) and then create a new publisher to emit these words.
2. Apply removeDuplicates() to your words publisher

- Run your playground and take a look at the debug console:

> ——— Example of: removeDuplicates ———
> hey
> there!
> want
> to
> listen
> to
> mister
> ?

> **Note**: What about values that don’t conform to Equatable? Well, removeDuplicates has another overload that takes a closure with two values, from which you'll return a Bool to indicate whether the values are equal or not.


** Compacting and ignoring

```swift
example(of: "compactMap") {
    // 1
    let strings = ["a", "1.24", "3",
    "def", "45", "0.23"].publisher
    // 2
    strings
    .compactMap { Float($0) }
    .sink(receiveValue: {
    // 3
    print($0)
    })
    .store(in: &subscriptions)
}
```
1. Create a publisher that emits a finite list of strings.
2. Use compactMap to attempt to initialize a Float from each individual string. If Float’s initializer doesn’t know how to convert the provided string, it returns nil.
3. Only print strings that have been successfully converted to Floats.

> ——— Example of: compactMap ———
> 1.24
> 3.0
> 45.0
> 0.23

- All right, why don’t you take a quick break from all these values... who cares about those, right? Sometimes, all you want to know is that the publisher has finished emitting values, disregarding the actual values. When such a scenario occurs, you can use the ignoreOutput operator:

## ignoreOutput

```swift
example(of: "ignoreOutput") {
    // 1
    let numbers = (1...10_000).publisher
    // 2
    numbers
    .ignoreOutput()
    .sink(receiveCompletion: { print("Completed with: \($0)") },
    receiveValue: { print($0) })
    .store(in: &subscriptions)
}
```

1. Create a publisher emitting 10,000 values from 1 through 10,000.

2. Add the ignoreOutput operator, which omits all values and emits only the completion event to the consumer.

> ——— Example of: ignoreOutput ———
> Completed with: finished

## Finding values

