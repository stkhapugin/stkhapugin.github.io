---
layout: post
title: Back from vacation
image: /assets/article_images/2015-1-26-Back-from-vacation/alps.jpg
tags: personal
---

I've been snowboarding in Alpe d'Huez for a week. I decided to leave my laptop at home, so that there is no chance I start coding there. And my god was it good.

Even though the snow was not perfect and slopes were designed by skiers for skiers (lots of inclines where you are forced to walk), I had a grand time. I also meet some cool French folks who were sharing the apartment with us, and none of them was a techie. 

Today my detox is over. It was quite funny to pick up the project I left for my teammate and find out that it doesn't even build anymore. It turns out there was a complication with git submodules which he didn't notice in his local copy. We ended up migrating it to CocoaPods: I've set up and tested a private podspec repo while he filled in his podspec and updated the project. Right after he went offline, I've opened Podfile and found out that he actually used `:git` and `:tag` directives that didn't use the repo I've set up at all. He did add the podspec to it, though. It seems like I'm doing a presentation to properly introduce the team to CocoaPods :) 

Feels good to be back.
