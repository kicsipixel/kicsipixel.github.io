---
date: 2020-01-11
title:  "NSWindow without Storyboard"
categories: [coding]
tags: [macOS, swift, NSWindow, custom, Storyboard]
---

Time to time the debate restarts whether using Storyboard is the proper way of interface desing. On iPhone it is quite simple and you can find several articles or tutorials. 
I wanted to try it on macOS and hardly found anything up-to-date information. Hereby I share what I found, maybe it can help others. 

At the end of the article I indicated all the sources I used.

**Step 1.**
Create a new Swift project with Storyboard. 
![create](/images/NoStoryboard1.png)

**Step 2.**
Remove the Main.Storyboard and ViewController.swift, then delete "Main Interface".
![remove](/images/NoStoryboard2.png)

**Step 3.**
Add a new swift file called: main.swift. 
![main.swift](/images/NoStoryboard3.png)
Add the following code to it, this is require to execute AppDelegate.swift.

{% gist 22ba4e36489f7cb17fbe30ed06fbe546 main.swift %}

**Step 4.**
Insert the following into ```AppDelegate.swift``` and comment out ```@NSApplicationMain```

{% gist 22ba4e36489f7cb17fbe30ed06fbe546 AppDelegate.swift %}

Run our app now.
![firstrun](/images/NoStoryboard4.png)
Looks good but we need to add a NSViewController to show some content.

**Step 5.**
Add MyViewController.swift as NSViewController.
![myviewcontroller](/images/NoStoryboard5.png)

Connect our new NSWindow with the new MyViewController:
```swift
window?.contentViewController = MyViewController()
```

Color the background of the NSViewController to see if we managed to add to NSWindow.

{% gist 22ba4e36489f7cb17fbe30ed06fbe546 MyViewController.swift %}

![secondrun](/images/NoStoryboard6.png)

**Step 6.**
Add some more code to MyViewController.swift to show a label.

{% gist 22ba4e36489f7cb17fbe30ed06fbe546 Label.swift %}

![thirdrun](/images/NoStoryboard7.png)

There is only one big issue with our new app, there is no Menu as we deleted it with the Main.Storyboard.

**Step 7.**
Go back to ```AppDelegate.swift``` and insert the code for menu.

{% gist 22ba4e36489f7cb17fbe30ed06fbe546 Menu.swift %}

Now, you can quit from menu.

Source code: [Github](https://github.com/kicsipixel/Custom-Cocoa-Controllers/tree/master/NoStoryboardWindow)

_Sources_:
1. [Malik Alayli - Create a macOS app without a Main Storyboard](https://medium.com/@MalikAlayli/create-a-macos-app-without-a-main-storyboard-81f4ee8be1d)
2. [Christoffer Winterkvist - How to write an NSViewController without interface builder on macOS](https://medium.com/hyperoslo/how-to-write-an-nsviewcontroller-without-interface-builder-on-macos-760283648f12)
3. [Christoffer Winterkvist - How to make a label in macOS](https://medium.com/hyperoslo/how-to-make-a-label-in-macos-150f53ce86fd)
4. [Stackoverflow](https://stackoverflow.com/questions/28254377/get-app-name-in-swift)
5. [Stackoverflow](https://stackoverflow.com/questions/6706806/creating-nsmenu-with-nsmenuitems-in-it-programmatically?rq=1)
6. [Stackoverflow](https://stackoverflow.com/questions/3436395/how-i-can-get-the-application-menu-in-cocoa/33581939#33581939)
7. [Gene De Lisa - Swift script to create a Cocoa window](http://www.rockhoppertech.com/blog/swift-script-to-create-a-cocoa-window/)
8. [Lukas Thoms - AppDelegate is Not The First Code to Run](https://medium.com/@lukasthinks/appdelegate-is-not-the-first-code-to-run-54c1535db443)
9. [Stackoverflow](https://stackoverflow.com/questions/34538112/nswindowcontroller-without-a-nib-file-in-swift?rq=1)
