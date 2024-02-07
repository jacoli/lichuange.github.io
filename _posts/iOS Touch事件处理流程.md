---
title: iOS Touch事件处理流程
date: 2015-9-16 23:07:56
---

### 概述

* 本文简单地梳理下Touch事件的处理流程。

* 参考文档：[Event Handling Guide for iOS](https://developer.apple.com/library/content/documentation/EventHandling/Conceptual/EventHandlingiPhoneOS/Introduction/Introduction.html)

* Touch事件流程可以分成，找第一响应和事件传递处理。

### Hit-Test找第一响应视图

1. 用户在屏幕上的点击、滑动等都会转换成Touch事件，并加到待处理队列。
2. 接着，RunLoop设置的回调`__CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__`被触发，然后由`_UIApplicationHandleEventQueue`从待处理队列取出事件并分发给`UIApplication`。
3. `-[UIApplication sendEvent:]`分发事件到`UIWindow`。
4. `UIWindow`先通过`hit-test`递归找出第一响应的视图，然后再从第一响应视图开始处理该事件。这里可以重载这两个函数改变第一响应视图的查找，`- [pointInside:withEvent:]`，`-[hitTest:withEvent:]`。

### 基于响应链的事件传递

1. 第一响应视图通过重载以下几个方法`-[touchesBegan:withEvent:]`，`-[touchesMoved:withEvent:]`，`-[touchesEnded:withEvent:]`开始处理touch事件。
2. 如果第一响应视图上存在Gesture，则Gesture也会收到touch事件并将自己置为`UIGestureRecognizerStatePossible`状态。
3. Gesture识别到对应手势后，则切换到`UIGestureRecognizerStateBegan`状态，并cancel掉第一响应视图的事件处理`-[touchesCancelled:withEvent:]`。
4. 如果存在多个Gesture都在识别手势，则可以通过Gesture的属性和委托处理多个Gesture的协同问题。
5. 如果第一响应视图没有处理，则事件沿着响应链继续传递。（默认的响应链是当前视图->父视图->当前视图控制器->`UIWindow`->`UIApplication`）




