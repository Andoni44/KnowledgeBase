# Basic closures
## Now, I will explain how closures works using a simple example

///A -> B
///B -/> A
//A closure is a piece of code that can be passed around and used in any part of your code. In this example we have two classes, Receiver which has a function that executes a closure and the Sender, that is a class that has a strong refrence to the receiver and stores its response. Notice that the response of the receiver is stored in Sender but the receiver does not know anything about the Sender
```Swift
final class Receiver {

    func messagesHandler(message: String, handlingCode: (String) -> ()) {
       handlingCode("This is receiver response to message: " + message)
   }
}

final class Sender {

    var response: String = "" {
        didSet {
            print(response)
        }
    }
    let r = Receiver()
}

let sender = Sender()

sender.r.messagesHandler(message: "Hello") { message in
    sender.response = message
}
//Response: "This is receiver response to message: Hello"
```
## @escaping vs @nonescaping
By default, from Swift 3, all closures are @nonescaping, this means that the code contained in a closure is executed inside the function that is receiving a closure as parameter. Thus, the ARC is no incrementing its value by 1.

@escaping closures are pieces of code that could be executed outside the function context, for example, when the caller function ends and the closure is executed later. So it retainCount is incremented by one in order to store the code and execute it later.

Example:
```Swift
final class Receiver {

    func messagesHandler(message: String, handlingCode: @escaping (String) -> ()) {
        DispatchQueue.main.asyncAfter(deadline: .now() + 3) {
            handlingCode("This is receiver response to message: " + message)
        }
   }
}

final class Sender {

    var response: String = "" {
        didSet {
            print(response)
        }
    }
    let r = Receiver()
}

let sender = Sender()

sender.r.messagesHandler(message: "Hello") { message in
    sender.response = message
}
//Response: "This is receiver response to message: Hello"
```