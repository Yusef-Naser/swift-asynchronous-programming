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


