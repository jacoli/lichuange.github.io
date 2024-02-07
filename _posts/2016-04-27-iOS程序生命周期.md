---
title: iOS程序生命周期
date: 2016-04-27
---
### 程序的生命周期

* 程序的生命周期，包含非运行、非激活、激活、后台、挂起五个状态

* 状态间切换都会有响应的回调和通知

* 程序挂起状态将接收不到Terminate通知，因此进入后台，能够处理程序随时被结束。 applicationWillTerminate:—Lets you know that your app is being terminated. This method is not called if your app is suspended.

### 程序如何在后台运行

* 1、短时间运行，进入后台后，可以调用beginBackgroundTaskWithName方法启动有限时间的任务。
* 2、NSURLSession支持后台下载。
* 3、长时间运行，支持以下几种模式，音频及视频（airplay）播放、更新定位、VOIP、下载Newsstand、外部附件、蓝牙、push，在info.plist添加相应配置，代码中引入相应的framework并调用相应接口。
* 调用application:performFetchWithCompletionHandler: 在后台获取少量数据。