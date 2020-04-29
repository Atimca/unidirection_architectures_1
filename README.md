# How to cook reactive programming. Part 1: Unidirectional architectures introduction.

While ago I wrote an article [What is Reactive Programming? iOS Edition](https://medium.com/atimca/what-is-reactive-programming-43e60cc4c0f?source=friends_link&sk=4ab8aa82f6e669bad59be42cba67e0ef) where I in super basic way described how to build your own Reactive Framework and helped you to understand that nodoby should scary of reactive way. Previous arcticle now could be named **How to cook reactive programming. Part 0.** since this one is a sort of continuation of the previous one. I would recommend to read the previous one if you are not familiar with reactive programming concepts.

Today we are going to talk more about practical aspect of reactive programming. I've already mensioned that's it's too easy to shoot into the foot, after you started to use any reactive framework, so I want to show you a way how to secure yourself)

## The problems

For the beginning I want to describe some common problems with reactive programming and with the most common approaches nowadays.

Let's start with the reactive problems. The most common fear about reactive that when you start to use it you would reach a point when in your code hundreds of data sequences flying around and you as a developer stop to have any power over it.

### Unpredicted mutations

The first and the most problem of what reactive way could do for you is unpredicted mutations. Let me make an example:

```swift
class Foo {
    var val: Int?

    init(val: Int?) {
        self.val = val
    }
}

var array: [Foo] = [Foo(val: 1), Foo(val: 2), Foo(val: 3)]

array = array.map { element in
    array[0].val = element.val
    return element
}

// array == 3, 2, 3
```

For the simplicity reasons I've replaced reactive sequence with an array. Back to example. This is super oversimplified example of how could you screw up, doing any reactive things. Nobody could predict, where you change data, we have so many ways to it and they are so hidden under reactive framework implementations.

But we can do with this, how can we protect ourselfs? First of all we should move from `reference` types to `value` ones.

```swift
struct Foo {
    var val: Int?
}

var array: [Foo] = [Foo(val: 1), Foo(val: 2), Foo(val: 3)]

array = array.map { element in
    array[0].val = element.val
    return element
}

// array == 1, 2, 3
```

Just with this simple change, I protect my array from unpredicted mutation. As far as you know `structs` are value types and they like to copy thereself all the time.

However, wait for a second, let's try to move one step forward and try to make one radical movement. Let's make variable `val` constant.

```swift
struct Foo {
    let val: Int?
}

var array: [Foo] = [Foo(val: 1), Foo(val: 2), Foo(val: 3)]

array = array.map { element in
    array[0].val = element.val // Cannot assign to property: 'val' is a 'let' constant
    return element
}
```

Oh yeah, that's the protection I'm talking about. Now we don't have any chance to mutate our data on the way though the data Sequence.

But we are going to do with all this? All mobile apps are not pictures, they are not static, we need mutate data to `react` on user behaviour or any other external changes such as network data or even timers. The app is a living being. So we have two problems to solve we need to keep our value entities in the floating sequences and we need somehow to mutate data to show changes to the user. That's sounds like a chalange, but wait for it, let's talk a little bit of modern architectures. We have article about architecture over there.

### MV family

The problem above is not only about reactive. You can face it everywhere. Let's make a small step back and talk a little bit about modern app architectures.

![mvc](images/MVC.png)

I bet, you have seen this guy before. Actualy for my opinion most of the modern architectures looks like this. They could be called MVsomething family. MVP, VIPER, MVVM same but different. I don't want to go deep inside to try to understand what the difference between them, you've alredy know it without me. Let's talk about common things. They are all small peaces, separated by screens, single views or just peace of business logic. Whoever is familiar with the topic will understand in advance what I'm getting at. With all this architectures super easy to bump into inconsistent state of the app. Let me make an example imagine you have a home page in your app and this home page depends on the logged in user. You have had a quite complicated app already with possibility to sign in or sign out user from different parts of the app. Already got what I'm talking? Imagine you decided to sign out from one part of the app and your home page should `react` on this change. Let me just show a gif with it.

![homesync](images/homesync.gif)

https://twitter.com/mobileunderhood/status/1250773949287407617

https://twitter.com/mobileunderhood/status/1250805880813236225

https://twitter.com/mobileunderhood/status/1250831101247066112