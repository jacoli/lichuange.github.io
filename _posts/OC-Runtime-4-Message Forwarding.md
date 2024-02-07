---
title: OC Runtime(4)之Message Forwarding
date: 2014-03-21
---

### Runtime系列笔记

* [Runtime之对象和类](/2014/03/07/OC-Runtime-1-Object&Class/)
* [Runtime之属性和方法](/2014/03/12/OC-Runtime-2-Ivar&Property&Method&Protocol/)
* [Runtime之关联对象](/2014/03/18/OC-Runtime-3-Associated%20Objects/)
* [Runtime之消息转发](/2014/03/21/OC-Runtime-4-Message%20Forwarding/)
* [Runtime之方法替换](/2014/04/02/OC-Runtime-5-Method%20Swizzling/)
* [Runtime之Load&Initialize](/2014/04/09/OC-Runtime-6-Load&Initialize/)

![OC-Runtime](/images/OC-Runtime.png)




### 消息发送

* 对象方法的调用就是向该对象发送消息。


```
    OCHelper *object = [[OCHelper alloc] init];
    
    // 以下两句代码效果是相同的。
    [object onHandler];
    objc_msgSend(object, @selector(onHandler));
```

* 编译阶段，会自动将OC方法调用转化成内联的`objc_msgSend`方法调用。



### 消息转发

* 有了OC的消息机制，就可以在运行时修改消息执行路径。
* `objc_msgSend`会调用`lookUpImpOrForward`从类（或父类）的`methodsCache`或`methodList`寻找方法。

* `lookUpImpOrForward`的源码如下：

```
IMP lookUpImpOrForward(Class cls, SEL sel, id inst, 
                       bool initialize, bool cache, bool resolver)
{
    Class curClass;
    IMP methodPC = nil;
    Method meth;
    bool triedResolver = NO;

    methodListLock.assertUnlocked();

    // Optimistic cache lookup
    if (cache) {
        methodPC = _cache_getImp(cls, sel);
        if (methodPC) return methodPC;    
    }

    // Check for freed class
    if (cls == _class_getFreedObjectClass())
        return (IMP) _freedHandler;

    // Check for +initialize
    if (initialize  &&  !cls->isInitialized()) {
        _class_initialize (_class_getNonMetaClass(cls, inst));
        // If sel == initialize, _class_initialize will send +initialize and 
        // then the messenger will send +initialize again after this 
        // procedure finishes. Of course, if this is not being called 
        // from the messenger then it won't happen. 2778172
    }

    // The lock is held to make method-lookup + cache-fill atomic 
    // with respect to method addition. Otherwise, a category could 
    // be added but ignored indefinitely because the cache was re-filled 
    // with the old value after the cache flush on behalf of the category.
 retry:
    methodListLock.lock();

    // Ignore GC selectors
    if (ignoreSelector(sel)) {
        methodPC = _cache_addIgnoredEntry(cls, sel);
        goto done;
    }

    // Try this class's cache.

    methodPC = _cache_getImp(cls, sel);
    if (methodPC) goto done;

    // Try this class's method lists.

    meth = _class_getMethodNoSuper_nolock(cls, sel);
    if (meth) {
        log_and_fill_cache(cls, cls, meth, sel);
        methodPC = method_getImplementation(meth);
        goto done;
    }

    // Try superclass caches and method lists.

    curClass = cls;
    while ((curClass = curClass->superclass)) {
        // Superclass cache.
        meth = _cache_getMethod(curClass, sel, _objc_msgForward_impcache);
        if (meth) {
            if (meth != (Method)1) {
                // Found the method in a superclass. Cache it in this class.
                log_and_fill_cache(cls, curClass, meth, sel);
                methodPC = method_getImplementation(meth);
                goto done;
            }
            else {
                // Found a forward:: entry in a superclass.
                // Stop searching, but don't cache yet; call method 
                // resolver for this class first.
                break;
            }
        }

        // Superclass method list.
        meth = _class_getMethodNoSuper_nolock(curClass, sel);
        if (meth) {
            log_and_fill_cache(cls, curClass, meth, sel);
            methodPC = method_getImplementation(meth);
            goto done;
        }
    }

    // No implementation found. Try method resolver once.

    if (resolver  &&  !triedResolver) {
        methodListLock.unlock();
        _class_resolveMethod(cls, sel, inst);
        triedResolver = YES;
        goto retry;
    }

    // No implementation found, and method resolver didn't help. 
    // Use forwarding.

    _cache_addForwardEntry(cls, sel);
    methodPC = _objc_msgForward_impcache;

 done:
    methodListLock.unlock();

    // paranoia: look for ignored selectors with non-ignored implementations
    assert(!(ignoreSelector(sel)  &&  methodPC != (IMP)&_objc_ignored_method));

    return methodPC;
}
```

* 如果找到方法，则直接执行。
* 如果没有找到方法，则提供一套Hock机制，允许运行时解决。
* `NSObject`提供了些可重载的方法，如下：

```
@interface NSObject
- (id)forwardingTargetForSelector:(SEL)aSelector;
- (void)forwardInvocation:(NSInvocation *)anInvocation;
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector;
+ (BOOL)resolveClassMethod:(SEL)sel;
+ (BOOL)resolveInstanceMethod:(SEL)sel;
@end
```

* 如果没有找到方法，会先调用`resolveInstanceMethod`，在该方法中可动态添加方法到类。

![resolveInstanceMethod](/images/messaging.png)

* 如果不行，接下来会调用`forwardingTargetForSelector`转发，可动态替换掉响应对象。
* 如果还不行，则调用`methodSignatureForSelector`和`forwardInvocation`转发`Invocation`。
* 如果还不行，则会抛异常`doesNotRecognizeSelector`。

### 参考

* [understanding-objective-c-runtime](http://cocoasamurai.blogspot.jp/2010/01/understanding-objective-c-runtime.html)
* [ObjCRuntimeGuide](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008048)
* [runtime源码](http://opensource.apple.com/tarballs/objc4/)
* [Objective-C 的动态提示和技巧](http://blog.jobbole.com/45963/)
* [NSObject的load和initialize方法](http://www.cocoachina.com/ios/20150104/10826.html)
* [associated-objects](http://esoftmobile.com/2014/02/18/associated-objects/)
* [method-swizzling](http://esoftmobile.com/2014/02/19/method-swizzling/)

















