---
title: Deep Linking
date: 2016-04-12
---

### 概念理解

* `Deep Link`深度链接，打破APP沙盒壁垒，链接到App内具体的内容。另外，iOS9 `CoreSpotlight.framework`提供了些APP内容搜索的API，也可以认为是深度链接。
* `Defferred Deep Link`延迟深度链接，在深度链接基础上改进，如果链接的App未安装，则先跳转到应用中心或AppStore提示安装App。
* `Universal Link`，iOS9上提供了Associated－Domain关联域名，用于在网页或App内唤起其他App。[Universal Links通用链接应用跳转总结以及坑](http://www.jianshu.com/p/16374288c976)
* `Custom URL Scheme`，iOS提供的App间跳转的协议。

### 产品

* [DeepShare](http://deepshare.io/deeplink/)

### 参考
* [iOS 中的 Deferred Deep Linking](http://www.cocoachina.com/ios/20160105/14871.html)
* [实战iOS 9：开发者必须掌握的三种搜索API](http://www.csdn.net/article/2015-07-16/2825222-search-apis)