---
title:  "NSWindow without Storyboard but..."
date: 2020-10-28
tags: ["macOS", "Swift", "NSWindow", "Storyboard"]
author: ["Szabolcs Tóth"]
draft: false
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

This way we can keep ```NSMenu``` and need to recreate ```NSWindow``` and ```NSView``` with their Controllers.


**Step 3.**
Add a new Cocoa (NSWindow) class with the following code:
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/a74g0sx96nqn756wufsn.png)

```swift
class MainWindow: NSWindow {
    override init(contentRect: NSRect, styleMask style: NSWindow.StyleMask, backing backingStoreType: NSWindow.BackingStoreType, defer flag: Bool) {
        super.init(contentRect: contentRect, styleMask: [.miniaturizable, .closable, .resizable, .titled],  backing: .buffered, defer: true)
         isMovableByWindowBackground = true
     }
}
```

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

```swift
class MainWindowController: NSWindowController {
    
    convenience init() {
        self.init(windowNibName: "")
    }
        
    override func loadWindow() {
        // MARK: Create window
        let windowSize = NSSize(width: 600, height: 300)
        let screenSize = NSScreen.main?.frame.size ?? .zero
        let rect = NSMakeRect(screenSize.width/2 - windowSize.width/2, screenSize.height/2 - windowSize.height/2, windowSize.width, windowSize.height)
        window = MainWindow(contentRect: rect, styleMask: [], backing: .buffered, defer: true)
        self.window?.title = "SemiStoryboard Window"
        self.window?.titlebarAppearsTransparent = true
        self.window?.styleMask.insert(.fullSizeContentView)
        self.window?.contentViewController = MyViewController()
    }
    
}
```

Note: the convinience init method is required as ```NSWindowController``` cannot load the relevant xib file, unless ```NSViewController```. Since we don't want to use any xib files, so we set empty string there.

**Step 5.**
Add a new Cocoa class, an ```NSViewController```.
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/en666ihennxw0zd8s2dg.png)

As I mentioned above, ```NSViewController``` will try to use xib file with the same name, we override this to set it to nil.
```swift

class MyViewController: NSViewController {
    
    init() {
         super.init(nibName: nil, bundle: nil)
     }
     
    required init?(coder: NSCoder) {
         fatalError()
     }

    override func viewDidLoad() {
    }
    
    override func loadView() {
        super.viewDidLoad()
        self.view = NSView()
        view.frame.size = CGSize(width: 600, height: 300)
        view.wantsLayer = true
        view.layer?.backgroundColor = NSColor.white.cgColor
    } 
}
```

It's important not to forget that ```NSViewController``` is not enough alone. We need an ```NSView``` to show the content, which size is the same as defined for ```NSWindow```.

Run the application. Everything seems to be fine but... we have no error and no window. Why? We have never told our application that it has an ```NSWindow```.

**Step 6.**
Go to AppDelegate.swift and define our newly created ```NSWindow```. Once we have it as variable, we need to show in ```applicationDidFinishLaunching``` method.

```swift
class AppDelegate: NSObject, NSApplicationDelegate {

    lazy var mainWindowController = MainWindowController()

    func applicationDidFinishLaunching(_ aNotification: Notification) {
        // Insert code here to initialize your application
        mainWindowController.showWindow(nil)
    }

    func applicationWillTerminate(_ aNotification: Notification) {
        // Insert code here to tear down your application
    }
}
```


Run the application now, and you'll see the window.
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/pmktd4oxntd028csa5xy.png)

**Step 7.**
Add 'Hello, World' label as ```NSTextField``` to the View using Constraints.

```swift

let label = NSTextField()
label.frame = CGRect(x: 0, y: 0, width: 100, height: 44)
label.stringValue = "Hello, World!"
label.backgroundColor = .white
label.isBezeled = false
label.isEditable = false
label.sizeToFit()
view.addSubview(label)
        
label.translatesAutoresizingMaskIntoConstraints = false
let labelConstraint = [
    label.centerXAnchor.constraint(equalTo: view.centerXAnchor),
    label.centerYAnchor.constraint(equalTo: view.centerYAnchor),
]
NSLayoutConstraint.activate(labelConstraint)
```

Note: We'll align the label to the center of the View, not important to give other x and y coordinates than 0. You use sizeToFit() method, so width and height of the NSTextfield are irrelevant as well.


Your final app:
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/84t7519xsdoy5ytvmp2g.png)

Source code: [Github](https://github.com/kicsipixel/Cocoa-Samples/tree/master/SemiStoryboard)