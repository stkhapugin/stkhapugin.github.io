---
layout: post
title: Upload screenshots to imgur automatically
---

Today I found myself violating DRY principle outside of programming. I realized that I am constantly repeating the following steps:

1. create a screenshot
2. move it to dropbox/tmp
3. copy share link
4. paste it to Skype

I started doing this a lot more lately, and grew tired of repeating all those steps. Plus you have to manually keep track of the files you can safely delete from dropbox and those that may still be required when you're cleaning your tmp folder up.

So I came up with this automator script that you can find [here](/assets/article_files/2015-02-12-Upload-screenshots-to-imgur-automatically/upload_screenshots_to_imgur.zip). It watches your desktop for new screenshots, and every time a new one is created, it is uploaded to imgur with a simple curl call (thanks for the snippet and API token, [Charlie Meyer](http://charliemeyer.net/2014/04/uploading-images-to-imgur-from-the-linuxmac-command-line/)!)and parses the original image link out from output. The link is then placed into your clipboard and a notification is posted. Very simple yet saves me a minute many times every day. Watch out though: **it will replace your clipboard contents every time you take a screenshot**.
