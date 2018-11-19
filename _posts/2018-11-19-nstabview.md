---
date: 2018-11-19
title:  "NSTabView controlled by NSSegmentedControl"
categories: [coding]
tags: [macOS, cocoa, swift, NSTabView, NSSegmentedControl]
---

Recently I found two very interesting articles about NSToolbar and NSSegmentedControl. One was written by [Christian Tietze](https://christiantietze.de/posts/2016/06/segmented-nstoolbaritem/) and the other by [sanjeetsuhag](https://www.saowen.com/a/78bee8d420f09c15a6c48040d14274a054b0cbd12bdb98d4ff708dd033b9b9c7).

One of my projects I followed Chritian's tutorial. Today, for another project I'd have liked to follow sanjeetsuhag's one but found myself a little lost, so decided to create a little more detailed article, I hope he won't mind.

On the left hand side, our project and on the right hand side hwo it would look with standard AppKit elements.
![final][NSTabview]

**Step 1. Create a project and name it**
![create][NSTabView1-1] 
![nameit][NSTabView1-2]

**Step 2. Delete View Controller**
![delete][NSTabView1-3]

**Step 3. Add NSToolbar but remove all items**
![remove][NSTabView1-4]
![segmentedcontrol][NSTabView1-5]

**Step 4. Add NSSegmentedControl and decrease the number of segments to 2**
![decrease][NSTabView1-6]

Setting the "Segment 0" to "Selected", make the first "tab" or "segment" active.
![selected][NSTabView1-7]

**Step 5. Add TabView Controller**
![addtabview][NSTabView1-8]

**Step 6. Add TabView Controller to NSWindow as content view**
![addcontentview][NSTabView1-9]

**Step 7. Add two images to the views to check if they work properly**
![addimage][NSTabView1-10]

**Step 8. Add "New file", an NSWindowController to our project**
![addnewfile][NSTabView1-11]
![namenewfile][NSTabView1-12]

**Step 9. Make your main window use this newly created class**
![addclass][NSTabView1-13]

**Step 10. Connect your NSSegmentedControl to your new WindowController class**
![addaction][NSTabView1-14]

``` swift
// This is your newly added action
@IBAction func segmentedControlSwitched(_ sender: Any) {
}
```

Implement the following inside the method:
``` swift
 @IBAction func segmentedControlSwitched(_ sender: Any) {
  let segmentedControl = sender as! NSSegmentedControl

  self.tabViewController?.selectedTabViewItemIndex = segmentedControl.selectedSegment
}
```

**Step 11. Add two more line to WindowController class, to match the code below**

``` swift
import Cocoa

class WindowController: NSWindowController {
    
var tabViewController: NSTabViewController?

  override func windowDidLoad() {
    super.windowDidLoad()

    self.tabViewController = self.window?.contentViewController as? NSTabViewController

  }
  @IBAction func segmentedControlSwitched(_ sender: Any) {
    let segmentedControl = sender as! NSSegmentedControl

    self.tabViewController?.selectedTabViewItemIndex = segmentedControl.selectedSegment
  }
 ```   
}

When you run the code, you realize something went wrong. This is quite far from what we wanted...
![bhua][NSTabView1-15]


[NSTabView]:   /images/NSTabView.png
[NSTabView1-1]: /images/NSTabView1-1.png
[NSTabView1-2]: /images/NSTabView1-2.png
[NSTabView1-3]: /images/NSTabView1-3.png
[NSTabView1-4]: /images/NSTabView1-4.png
[NSTabView1-5]: /images/NSTabView1-5.png
[NSTabView1-6]: /images/NSTabView1-6.png
[NSTabView1-7]: /images/NSTabView1-7.png
[NSTabView1-8]: /images/NSTabView1-8.png
[NSTabView1-9]: /images/NSTabView1-9.png
[NSTabView1-10]: /images/NSTabView1-10.png
[NSTabView1-11]: /images/NSTabView1-11.png
[NSTabView1-12]: /images/NSTabView1-12.png
[NSTabView1-13]: /images/NSTabView1-13.png
[NSTabView1-14]: /images/NSTabView1-14.png
[NSTabView1-15]: /images/NSTabView1-15.png
