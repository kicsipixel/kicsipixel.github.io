---
date: 2018-11-18
title:  "PaintCode, the mindblowing tool"
categories: [coding]
tags: [paintcode, swift, design]
comments: true
---
There are several fantastic tools, I will write some of them later. I would like to start with [PaintCode].
Drawing simple shapes (e.g. ovals, rectangles, etc) can be easy, but complex shape drawings are very tedious.

This ugly app we will build today. 
![app][PaintCode1]

I would like to draw this origami bird, and add to our app in two different way.
<iframe style="border: none;" width="800" height="450" src="https://www.figma.com/embed?embed_host=share&url=https%3A%2F%2Fwww.figma.com%2Ffile%2FTIkZmR1l12bUiigjTUOmrTAg%2FGithub-pages-blog%3Fnode-id%3D6%253A2"></iframe>

One way is to use Custom View and draw method, while the other is ImageView using image method. 

PaintCode allows us to draw shapes inside or drop .svg and converts it to Swift or Obejctive-C code. After downloading, let's open it.
![paintcode][PaintCode1-1]

Let's drop our .svg image and be sure the highlighted settings are the same.
![draw][PaintCode1-2]

Later, I'd like to use the colors as well, so let's add them to our file.
![colors][PaintCode1-3]

[PaintCode]:	https://www.paintcodeapp.com/
[PaintCode1]: /images/PaintCode1.png
[PaintCode1-1]: /images/PaintCode1-1.png
[PaintCode1-2]: /images/PaintCode1-2.png
[PaintCode1-3]: /images/PaintCode1-3.png

