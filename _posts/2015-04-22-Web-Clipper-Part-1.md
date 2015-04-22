---
layout: post
date: 2015-04-22
title: Web Clipper Part 1
---

I have an idea of an iOS app that I've been working on for a few weekends. I am using an alpha implementation of that for months, and I've learnt quite a lot from trying to bring it to life. Some of that knowledge begs to be shared, so I'm going to write a series of articles explaining the evolution of it.

The app is very simple. You open a web page, choose a rectangular region and hit Done. This region appears in the Notification Center as a widget and gets updated every once in a while. This allows you to monitor certain websites that have infrequently updated data on a fairly static page like weather graphs or tracking a letter. A lot of useful services don't provide you with an API, an app, an RSS feed or any other means of conventional tracking, relying solely on a web interface. I found myself checking one such page for a few times a day, which gave me the idea of making a widget out of it.

My first approach was to show a web view in a widget, and it failed because of memory restrictions. My next idea was to refresh an image in background and show a screenshot from disk every time in the widget. I will go into further details in later posts.

So you need to load a web page and make a snapshot of it periodically. How do you do that? The first thing that comes to mind is Background Fetch. From [Apple docs](https://developer.apple.com/library/ios/documentation/iPhone/Conceptual/iPhoneOSProgrammingGuide/BackgroundExecution/BackgroundExecution.html): "The app regularly downloads and processes small amounts of content from the network." Sounds like a perfect fit for the job!

# Background Fetch
Making an app work with background fetches is very simple, the docs are very clear, and there is a ton of samples on the Web showing off the implementation. My personal favourite is [this objc.io article](http://www.objc.io/issue-5/multitasking.html). Go ahead and read it if you have no idea how this works.

The question is, how often does it actually fetch in my scenario, where the container app should never be opened manually? The docs are very vague and basically iOS gives you zero guarantees about how often it is going to actually happen.

I've written a [small demo app](https://github.com/stkhapugin/BackgrouindFetchInvestigator) to test background fetch frequency. All it does is it keeps track of all the background fetches and notifies you when there have been no fetch for 24 hours. It always tells system that new data have been downloaded. It also actually loads my blog's main page just to use some resources.

I've launched it yesterday evening. It fetched a few times while I was debugging the code, then it fetched once right after I closed the app. And then... 

![](/assets/article_images/2015-04-22-Web-Clipper-Part-1/bgfetch_fail.jpg)

it just never fetches again. So this clearly wasn't going to work for me. I had to find some other way to fetch my data.


