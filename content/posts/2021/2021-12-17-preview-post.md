---
title: "SwiftUI Preview "
date: 2021-12-17
tags: ["iOS", "Xcode", "SwiftUI"]
author: ["Szabolcs Tóth"]
cover:
    image: /images/preview.png
draft: false
---

Preview in SwiftUI speeds up the development process as we don't need to build and run our application every time we make some changes. Here are some tips how I use the Preview function.

### Light and dark mode side-by-side

Change your code this way:
```swift
struct ContentView_Previews: PreviewProvider {
    static var previews: some View {
        Group {
            ContentView()
                .preferredColorScheme(.light)
            ContentView()
                .preferredColorScheme(.dark)
        }
    }
}

```
![](/images/preview_post_1.png)

### Focus on important part not the whole screen

Add this:
```swift
    .previewLayout(.sizeThatFits)
```
![](/images/preview_post_2.png)

I hope it helps and you will be more efficient.