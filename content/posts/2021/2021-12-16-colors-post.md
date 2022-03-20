---
title: "Handling colors in SwiftUI easily"
date: 2021-12-16
tags: ["iOS", "Xcode", "SwiftUI"]
author: ["Szabolcs Tóth"]
cover:
    image: /images/colors.png
draft: false
---

Since Dark mode was introduced in iOS, the easiest was to define colors in Xcode/Assets. This way our app will change the colors automatically.

I have these three colors for my project:
![](/images/color_post_1.png)

Define them in Assets: ![](/images/color_post_2.png)

Using ```AccentColor``` is easy, you need to type only:
```swift
.foregroundColor(Color.accentColor)
```

but for pastelGreen, you need to type:
```swift
.foregroundColor(Color("pastelGreen"))
```


What if there is an eaier way and you can use your owned defined colors as SwiftUI built-in ```Color```.

Create a new file ```ColorExtension.swift``` and add the following:
```swift
import SwiftUI

extension Color {
    static let pastelGreen = Color("pastelGreen")
}
```

Now, you can use your defined color as:
```swift
.foregroundColor(Color.pastelGreen)
```