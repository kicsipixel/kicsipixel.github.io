---
date: 2020-10-28
title:  "NSWindow without Storyboard but..."
categories: [coding]
tags: [macOS, swift, NSWindow, custom, Storyboard]
---

Earlier this year I wrote a short [blog post](https://kicsipixel.github.io/2020/nostoryboard/) about creating NSWindow without Storyboard with code only. While collecting information, I read several opinions why it was a tedious job, and it could be done better...

Maybe the easiest way to mix Storyboard and coding approach. As I pointed out, recreation of ```NSMenu``` is a real pain/mess. So, what about keeping Main.storyboard for NSMenu only and code the rest of the components?


**Step 1.**
Create a new macOS app.
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/i9m5w6nuz3imbwcb6d23.png)


**Step 2.**
Delete:
- Window Controller Scene
- View Controller Scene
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/0x0kezp2abi23hcfmn9u.png)

This way we can keep NSMenu and need to recreate NSWindow and NSView with their Controllers.


**Step 3.**
Add a new Cocoa (NSWindow) class with the following code:
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/a74g0sx96nqn756wufsn.png)


{% gist https://gist.github.com/kicsipixel/708f19d1a36b23f0c82667021e992d40 MainWindow.swift %}

Probably, the most important part of [NSWindow init](https://developer.apple.com/documentation/appkit/nswindow/1419477-init) method is the StyleMask.

I added these four constants:
- titled: The window displays a title bar
- closable: The window displays a close button
- miniaturizable: The window displays a minimize button
- resizable: The window can be resized by the user

It's worth checking the documentation to find out more about NSWindow [StyleMask](https://developer.apple.com/documentation/appkit/nswindow/stylemask).

**Step 4.**
Add a new Cocoa class, an ```NSWindowController```.

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/llabmy38jye6t7m21wfa.png)

In the Controller file we need to define the size and position of our new window. Beside these we need to prepare the window to hold a view too as content.

{% gist https://gist.github.com/kicsipixel/708f19d1a36b23f0c82667021e992d40 MainWindowController.swift %}

Note: the convinience init method is required as ```NSWindowController``` cannot load the relevant xib file, unless ```NSViewController```. Since we don't want to use any xib files, so we set empty string there.

**Step 5.**
Add a new Cocoa class, an ```NSViewController```.
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/en666ihennxw0zd8s2dg.png)

As I mentioned above, ```NSViewController``` will try to use xib file with the same name, we override this to set it to nil.
{% gist https://gist.github.com/kicsipixel/708f19d1a36b23f0c82667021e992d40 MyViewController.swift %}

It's important not to forget that ```NSViewController``` is not enough alone. We need an ```NSView``` to show the content, which size is the same as defined for ```NSWindow```.

Run the application. Everything seems to be fine but... we have no error and no window. Why? We have never told our application that it has an ```NSWindow```.

**Step 6.**
Go to AppDelegate.swift and define our newly created ```NSWindow```. Once we have it as variable, we need to show in ```applicationDidFinishLaunching``` method.

{% gist https://gist.github.com/kicsipixel/708f19d1a36b23f0c82667021e992d40 AppDelegate.swift %}


Run the application now, and you'll see the window.
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/pmktd4oxntd028csa5xy.png)

**Step 7.**
Add 'Hello, World' label as ```NSTextField``` to the View using Constraints.

{% gist https://gist.github.com/kicsipixel/708f19d1a36b23f0c82667021e992d40 label.swift %}

Note: We'll align the label to the center of the View, not important to give other x and y coordinates than 0. You use sizeToFit() method, so width and height of the NSTextfield are irrelevant as well.


Your final app:
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/84t7519xsdoy5ytvmp2g.png)

Source code: [Github](https://github.com/kicsipixel/Cocoa-Samples/tree/master/SemiStoryboard)