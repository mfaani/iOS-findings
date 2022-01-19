Discussion with E.K.

AnyCancellable, can be canceled i.e. you can call `cancel` on it. This gives you free capabilities. Any time that subscriptions goes out of scope e.g. in `deinit`, `cancel` will automatically be called on all the subscriptions.
 ```swift
 var subscriptions: Set<AnyCancellable> = []
 ```
 

 ```swift
 player.candidates
     // sink is a way that you can subscribe...
     .sink(receiveValue: { [weak self]  (localType: String, remoteType: String) in
         self?.didReceiveChosenCandidate(local: localType, remote: remoteType)
     })
     // we store it in subscriptions. If you don't keep a reference to it, it will go out of memmory immediately...
     /// alternate way is is to do `let subscription = player.chosenCandidateSignal; subscription.cancel() a few lines later when you're done.`, otherwise it will go out of memory. 
     /// tldr you need a reference so it won't go out of memory.
     .store(in: &subscriptions)
```

Random note: Typically you don't need to hold a reference to the cancellable object — because the lifecycle of the pub/sub are the same. But if they're not, then if you need to cancel before the publisher finishes, then you need to have a way/reference to call cancel
```swift
var x: AnyCancellable?
```

`eraseToAnyPublisher` hides a lot of detail. You don't see that big tree call...???
```swift
lazy public var chosenCandidate = chosenCandidatePrivate.eraseToAnyPublisher()
```

A publisher has a datatype and a fail type. Never means it will never fail. So here it means it will only generate streasm of this tuple.
```swift
lazy private var chosenCandidatePrivate = PassthroughSubject<(localType: String, remoteType: String), Never>()
```

- `CurrentValueSubject` maintains state. `PassthroughSubject` doesn't.
- `CurrentValueSubject` needs to be initialized with a value`. `PassthroughSubject` doesn't need to.
