---
title: iOS Push相关的资料
date: 2016-06-18
---

* 苹果官方的APNS，支持App不同状态下（前台、后台、未启动等）的消息推送。

* 另外个方案是，App和应用服务间维护一个长链接，应用服务通过心跳感知App状态。当App为在线状态时，应用服务可以通过该长链接推送消息。一般跟APNS配合使用。

* 自己的项目中使用的是友盟推送。

* 在Push消息中设置自定义参数，App在`- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo`收到消息后解析参数并做相应逻辑（如跳转到相应页面等）


```
- (void)application:(UIApplication *)application didRegisterForRemoteNotificationsWithDeviceToken:(NSData *)deviceToken {
    NSLog(@"didRegisterForRemoteNotificationsWithDeviceToken");
    
    // 注册用户登录登出监听
    [self handleRemoteNotificationDidRegistered];
}

- (void)application:(UIApplication *)application didFailToRegisterForRemoteNotificationsWithError:(NSError *)error {
    NSLog(@"didFailToRegisterForRemoteNotificationsWithError");
}

- (void)application:(UIApplication *)application didReceiveRemoteNotification:(NSDictionary *)userInfo {
    NSLog(@"didReceiveRemoteNotification");
    
    [UMessage didReceiveRemoteNotification:userInfo];
    
    NSString *jsonString = [userInfo objectForKey:@"custom"];
    NSData *jsonData = [jsonString dataUsingEncoding:NSUTF8StringEncoding];
    NSDictionary *dic = [NSJSONSerialization JSONObjectWithData:jsonData
                                                        options:NSJSONReadingMutableContainers
                                                          error:nil];
    if (dic==nil || [dic isEqual:[NSNull null]]) {
        return;
    }

    [self handleRemoteNotification:dic];
}
```


### 其他一些资料

* [一步一步实现iOS应用PUSH功能 - 咪咕咪咕](http://www.tuicool.com/articles/Y77Ffe)
* [iOS上简单推送通知（Push Notification）的实现](http://blog.csdn.net/daydreamingboy/article/details/7977098)
* [十大豪门推送sdk，哪个更适合你](http://blog.csdn.net/c101012221/article/details/34433425)
* [个推](http://www.getui.com/)
* [腾讯信鸽](http://www.qcloud.com/product/XGPush.html)
* [友盟U-Push](http://mobile.umeng.com/push?spm=0.0.0.0.pphrqW)

