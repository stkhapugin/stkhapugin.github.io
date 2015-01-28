---
layout: post
title: Fetched properties and sorting by to-many
---

Imagine you're writing a Core Data-backed chat app and you have two entities: `Chat` and `Message`. You are implementing a screen with a list of all chats. You wanna show last message preview of every chat and sort them by last message's date.

## Fetched properties
This is what I started with a few days ago, and my first approach to the sorting problem and last message preview was a fetched property. I haven't used these for quite a while, but I figured that it is exactly what I'd need, since I wanna fetch the last message in a chat and sort by its date, i.e., a property that is fetched.

However, soon I realised that this is not exactly what I need, since fetched properties can only be arrays. I don't need an array, I need a `Message`, right? So I named property `lastMessageArray`(sounds funny, but at that point I **really** wanted to use that fetched property mechanism) and implemented a custom getter for `lastMessage`, returning the `firstObject`. I also ended up adding the fetched property in code to managed model because the model editor [does not support sorting](http://stackoverflow.com/questions/1071765/how-to-sort-core-data-fetched-properties) (and as a cool exercise, surely). I've thrown in a bunch of tests for new property and it all was looking nice so far.

Not for long. Once I started building the datasource for the chat list, I realised that you can't use your own getter in fetch requests. But I still have my fetched property, right? It is added to object model, so Core Data surely is aware of it? Nope, you can't sort by it either, since both collections KVC sugar and to-many keys are not supported by fetch request predicates, so I couldn't access my only object in `lastMessageArray` in any way from there. 

## How to finally do this

It turns out your [best choice](http://stackoverflow.com/questions/3943572/sorting-a-to-many-relationship-when-calling-nsfetchrequest) is to implement an actual to-one relationship if you need it in predicates. In my case it was relatively easy: I removed my fetched property, added a `lastMessage` relationship, and then used `messages` accessors overload where I updated `lastMessage` (so that I can rely on it being always up-to-date and save myself time remembering to update it manually). The accessors overload is pretty [straightforward][1]: you just write regular collection accessors and remember to support KVO, plus instead of ivar, the property is backed by primitive accessors. 

[1]: https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CoreData/Articles/cdAccessorMethods.html#//apple_ref/doc/uid/TP40002154-SW4 "Core Data Programming Guide"

## Today I learned
Today I learned that fetched properties are probably only [suitable for playlists](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CoreData/Articles/cdRelationships.html#//apple_ref/doc/uid/TP40001857-SW7) and were created by iTunes devs for iTunes devs. I also was surprised to learn that sorting by `Chat.messages.creationDate` would result in ` 'NSInvalidArgumentException', reason: 'to-many key not allowed here` .

Core Data is so big and cool and it will never cease to impress me with all  those things I will probably never need. From now on, in my brain fetched properties will rest together with multiple managed stores and child contexts in a box labeled "esoteric Core Data".
