---
title:  "PaintCode, the mindblowing tool"
date: 2018-11-18
tags: ["PaintCode", "Swift", "Design"]
author: ["Szabolcs Tóth"]
draft: false
---

There are several fantastic tools, I will write some of them later. I would like to start with [PaintCode].
Drawing simple shapes (e.g. ovals, rectangles, etc) can be easy, but complex shape drawings are very tedious.

This ugly app we will build today. 
![app][PaintCode1]

I would like to draw this origami bird, and add to our app in two different way.
<iframe style="border: none;" width="800" height="450" src="https://www.figma.com/embed?embed_host=share&url=https%3A%2F%2Fwww.figma.com%2Ffile%2FTIkZmR1l12bUiigjTUOmrTAg%2FGithub-pages-blog%3Fnode-id%3D6%253A2"></iframe>

One way is to use Custom View and draw method, while the other is Image View using image method. 

PaintCode allows us to draw shapes inside or drop .svg and converts it to Swift or Obejctive-C code. 
**Step 1. After downloading, let's open it.**
![paintcode][PaintCode1-1]

Let's drop our .svg image and be sure the highlighted settings are the same.
![draw][PaintCode1-2]

Later, I'd like to use the colors as well, so let's add them to our file.
![colors][PaintCode1-3]

**Step 2. Open Xcode and create a macOS project.**
We will need one Custom View and one Image View. Please, preapere your Storyboard as it's seen below.
![storyboard][PaintCode1-4]

I have added two Labels under the views and used Stack View for easier organization. 

**Step 3. Export your project at PaintCode and import it to Xcode.**
![export][PaintCode1-5]
![addfile][PaintCode1-6]

**Step 4. Add a new Cocoa class to your project**
![class][PaintCode1-7]
You need to add one single line to make it work:
``` swift
PaintCode.drawOrigami()
```

Before testing, don't forget to add your new class to your Custom View on Storyboard.
![customclass][PaintCode1-8]

Let's test our code:
![firsttest][PaintCode1-9]

**Step 5. Create IBOutlets for the Image View and the Labels.**

After connecting them on the Storyboard you will see the following in the ViewController:
``` swift
@IBOutlet var origamiImage: NSImageView!
@IBOutlet var customImageTextField: NSTextField!
@IBOutlet var imageViewTextField: NSTextField!
```

**Step 6. Add value to these variables**

First line of code set an image to the Image Viewm while the second and third use the colors from PaintCode project.

```swift
origamiImage.image = PaintCode.imageOfOrigami

// Using the colors defined in PaintCode
customImageTextField.textColor = PaintCode.fillColor5
imageViewTextField.textColor = PaintCode.fillColor6
 ```

Now, you will see both images as we planned.
![app][PaintCode1]


Source code: [Github](https://github.com/kicsipixel/Cocoa-Samples/tree/master/PaintCode)

[PaintCode]:	https://www.paintcodeapp.com/
[PaintCode1]: 	/images/PaintCode1.png
[PaintCode1-1]: /images/PaintCode1-1.png
[PaintCode1-2]: /images/PaintCode1-2.png
[PaintCode1-3]: /images/PaintCode1-3.png
[PaintCode1-4]: /images/PaintCode1-4.png
[PaintCode1-5]: /images/PaintCode1-5.png
[PaintCode1-6]: /images/PaintCode1-6.png
[PaintCode1-7]: /images/PaintCode1-7.png
[PaintCode1-8]: /images/PaintCode1-8.png
[PaintCode1-9]: /images/PaintCode1-9.png