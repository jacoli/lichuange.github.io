---
title: OC Runtime(5)之Method Swizzling
date: 2014-04-02
---

### Runtime系列笔记

* [Runtime之对象和类](/2014/03/07/OC-Runtime-1-Object&Class/)
* [Runtime之属性和方法](/2014/03/12/OC-Runtime-2-Ivar&Property&Method&Protocol/)
* [Runtime之关联对象](/2014/03/18/OC-Runtime-3-Associated%20Objects/)
* [Runtime之消息转发](/2014/03/21/OC-Runtime-4-Message%20Forwarding/)
* [Runtime之方法替换](/2014/04/02/OC-Runtime-5-Method%20Swizzling/)
* [Runtime之Load&Initialize](/2014/04/09/OC-Runtime-6-Load&Initialize/)

![OC-Runtime](/images/OC-Runtime.png)

### Method的几个概念

* `Method`包含方法名`Selector`、函数地址`IMP`、参数类型`TypeEncoding`。


```
NSLog(@"    method name = %s", sel_getName(method_getName(method)));
NSLog(@"    method type encode = %s", method_getTypeEncoding(method));
NSLog(@"    NumberOfArguments = %d", method_getNumberOfArguments(method));
NSLog(@"    ReturnType = %s", method_copyReturnType(method));
```

```
2016-04-08 20:54:59.351 OCRuntime[54195:953880]     method name = setNumberProperty:
2016-04-08 20:54:59.352 OCRuntime[54195:953880]     method type encode = v24@0:8@16
2016-04-08 20:54:59.352 OCRuntime[54195:953880]     NumberOfArguments = 3
2016-04-08 20:54:59.352 OCRuntime[54195:953880]     ReturnType = v
```


### 动态修改Method的几个接口

```
// 设置或交换IMP
IMP method_setImplementation(Method m, IMP imp);
void method_exchangeImplementations(Method m1, Method m2);
```

```
BOOL class_addMethod(Class cls, SEL name, IMP imp, const char *types);
                                 
// 如果原来没有这个方法，则直接添加Method，效果同class_addMethod
// 如果原来有，则替换掉原来的IMP，忽略参数类型
IMP class_replaceMethod(Class cls, SEL name, IMP imp, const char *types);
```


* 例子：

```
+ (void)load {
    static dispatch_once_t onceToken;
    dispatch_once(&onceToken, ^{
        Class class = [self class];

        // When swizzling a class method, use the following:
        // Class class = object_getClass((id)self);

        SEL originalSelector = @selector(viewWillAppear:);
        SEL swizzledSelector = @selector(xxx_viewWillAppear:);

        Method originalMethod = class_getInstanceMethod(class, originalSelector);
        Method swizzledMethod = class_getInstanceMethod(class, swizzledSelector);

        BOOL didAddMethod =
            class_addMethod(class,
                originalSelector,
                method_getImplementation(swizzledMethod),
                method_getTypeEncoding(swizzledMethod));

        if (didAddMethod) {
            class_replaceMethod(class,
                swizzledSelector,
                method_getImplementation(originalMethod),
                method_getTypeEncoding(originalMethod));
        } else {
            method_exchangeImplementations(originalMethod, swizzledMethod);
        }
    });
}
```



### 参考

* [understanding-objective-c-runtime](http://cocoasamurai.blogspot.jp/2010/01/understanding-objective-c-runtime.html)
* [ObjCRuntimeGuide](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008048)
* [runtime源码](http://opensource.apple.com/tarballs/objc4/)
* [Objective-C 的动态提示和技巧](http://blog.jobbole.com/45963/)
* [NSObject的load和initialize方法](http://www.cocoachina.com/ios/20150104/10826.html)
* [associated-objects](http://esoftmobile.com/2014/02/18/associated-objects/)
* [method-swizzling](http://esoftmobile.com/2014/02/19/method-swizzling/)

















