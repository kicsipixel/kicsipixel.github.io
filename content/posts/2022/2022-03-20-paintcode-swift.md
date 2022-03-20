---
title: "PaintCode on macOS with Swift"
date: 2022-03-20
tags: ["macOS", Xcode", "Swift", "Cocoa"]
author: ["Szabolcs Tóth"]
draft: false
---

I already wrote about [PaintCode] in this [post]. This time I will show you how to use PaintCode without Storyboard. I will add the [Figma] file and PaintCode file to GitHub repository. The code itself is very simple.
![][PaintCodemacOS]

**Step 1.**
Create a new macOS app in Xcode.
![][PaintCodemacOS1]

**Step 2.**
Open the PaintCode file and set the following:
![][PaintCodemacOS2]

**Step 3.**
Export the drawing code from PaintCode and add it to your Xcode project.

**Step 4.**
Add a new NSView class.
![][PaintCodemacOS3]

**Step 5.**
The following code will draw for us:
```swift
import Cocoa

class GaugeView: NSView {

    override func draw(_ dirtyRect: NSRect) {
        super.draw(dirtyRect)

        // We can use the dirtyRect, which we will define during the initialization of the NSView.
        GaugeCode.drawTemperature(frame: dirtyRect, resizing: .aspectFit)
    }
    
}
```

**Step 6.**
Add our new custom NSView to the `ViewController`.
```swift
import Cocoa

class ViewController: NSViewController {
    
    let gaugeView: GaugeView = {
        let view = GaugeView(frame: NSRect(x: 0, y: 0, width: 400, height: 400))
        return view
    }()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        view.addSubview(gaugeView)
        NSLayoutConstraint.activate([
            gaugeView.centerXAnchor.constraint(equalTo: view.centerXAnchor),
            gaugeView.centerYAnchor.constraint(equalTo: view.centerYAnchor)
        ])
    }
}
```

As I promised it was very simple even without using Storyboard.


[PaintCode]:		https://www.paintcodeapp.com/
[post]:				https://kicsipixel.github.io/posts/2018/2018-11-18-paintcode/
[Figma]:			https://figma.com
[PaintCodemacOS]: 	/images/PaintCodemacOS.png
[PaintCodemacOS1]: 	/images/PaintCodemacOS1.png
[PaintCodemacOS2]: 	/images/PaintCodemacOS2.png
[PaintCodemacOS3]: 	/images/PaintCodemacOS3.png
