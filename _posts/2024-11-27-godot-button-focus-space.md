---
title: 解决Godot因Button获取焦点触发误点击的问题
date: 2024-11-27 10:37:00 +0800
tags: [Godot, Focus]
categories: [Godot]
---

开发过程中，发现先点击某一按钮，再按空格键后，会莫名其妙触发按钮被点击的信号。一开始以为是按钮设置shortcut的问题，后来经观察，发现点击该按钮后颜色会略有变化，怀疑是获取了键盘焦点，而**空格键会自动点按获取键盘焦点的按钮**。在按钮主题中将Focus Color设置得更为显眼证明了这一点。

解决方法：button.set_focus_mode(0)，禁止按钮获取焦点。这不影响点按功能，只是禁止按钮捕获键盘事件。

测试阶段可将按钮的各种颜色（Focus、Hover、Hover Pressed等）设置得明显一些，便于debug。