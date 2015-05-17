---
layout: post
date: 2015-05-17
title: Web Clipper Part 2
---

This is a second part of a series dedicated to iOS backgrounding. This one is about Silent Push Notifications and background NSURLSession. But first let me share a small update on the [first post]({% post_url 2015-04-22-Web-Clipper-Part-1 %}).

# Background fetch (continued)
Since the previous post, I've found something I can't help but share. It seems like running apps with common bundle identifiers will **trigger background fetch on each other**. 

Latest article ends on background fetches not getting called for over 24 hours. A few days later, I've received this "no background fetches" notification again. And then, in a few days, again. At first I struggled to find any logical explanation for this, but later I discovered that I'm using my company's dev program, and my bundle id was `com.companyname.BackgroundFetchInvestigator`. It so happened that I've built and ran a few apps with `com.companyname.SomeOtherApp` bundles. These moments correspond with a new cycle of background fetch requests for my test app: normally it would launch it immediately after I quit the other app, then make two to four fetch attempts later. Then the updates stop forever once again.

# Silent push notifications
This is by far **the most reliable approach** that I've tried. You sign up for silent push notifications from a server that regularly sends you silent pushes. You wake your app and fetch necessary data, update the web page snapshot and call it a day. 

In my scenario, I used [Parse](http://parse.com) (which I think is very unreliable and poorly engineered as a BAAS, yet it works just fine for a simple task like this). I scheduled a background job that sent me pushes every 15 minutes. The pushes arrived fairly reliably, occasionally skipping an update a couple of times a day. I have never seen two consequent updates failing.

Battery usage was consistently reading 4% a day in this mode. 

Basically a cron job sending you silent pushes is a cron replacement for iOS. At least as for iOS 8.3, this is the best you can get (unless you're writing a VOIP app and have full background capabilities).

I decided to opt-out of this model for several reasons:

- It sounds hacky
- It can easily fail an App Store review
- Push notifications do not come for free. You need to either run your own server or pay some push notification service.
- I also don't like the fact that the updates happen at regular intervals, not taking user's actual interest in account. 

# Better silent push notifications
I figured that one can actually offload most of the work to the server and only send push notifications when your snapshot has actually changed. This is a far cleaner approach and saves precious battery life and traffic. 

If only I had resources to learn web development, I'd love to try that out. There are some obvious problems with this approach: the web pages are rendered differently in different browsers; the web server load can easily get out of hand, plus web hosting is not free; backend has to store user's URLs.

# Background URL Sessions
I am currently testing an absolutely different approach. The idea is to start a background URL session and fire up a connection at such a moment that the extension gets killed before the request finishes. 
From [Extension programming guide](https://developer.apple.com/library/prerelease/ios/documentation/General/Conceptual/ExtensibilityPG/ExtensionScenarios.html) :

>In iOS, if your extension isn’t running when a background task completes, the system launches your containing app in the background and calls the application:handleEventsForBackgroundURLSession:completionHandler: app delegate method.

So you schedule the request, and the `NSURLSession` launches your app in background for you to act on the downloaded data. From there you have something like 30 seconds of background execution and a reasonable amount of resources, such as RAM.

I've tried scheduling a network request to a background session in `viewWillDisappear:` of my widget and strangely *it worked*. I've been running this funky build for a week and the widget's info is almost always not older then an hour or two. The updates are about as recent as I look at Today extension tab, which is good enough. As for the battery usage, it seems to be about 2% (but then it actually launches far fewer times a day then the silent push build, which runs every 15 minutes)

I'm currently looking for beta-testers for my app because this approach looks very promising to me. It is almost perfect in my scenario: the updates are failry recent (multiple times a day), do not require any external services and as secure as your phone itself. I need some field testing before rushing to the App Store though 😃. 


