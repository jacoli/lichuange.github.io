---
title: OC Runtime(6)之Load&Initialize
date: 2014-04-09
---

### Runtime系列笔记

* [Runtime之对象和类](/2014/03/07/OC-Runtime-1-Object&Class/)
* [Runtime之属性和方法](/2014/03/12/OC-Runtime-2-Ivar&Property&Method&Protocol/)
* [Runtime之关联对象](/2014/03/18/OC-Runtime-3-Associated%20Objects/)
* [Runtime之消息转发](/2014/03/21/OC-Runtime-4-Message%20Forwarding/)
* [Runtime之方法替换](/2014/04/02/OC-Runtime-5-Method%20Swizzling/)
* [Runtime之Load&Initialize](/2014/04/09/OC-Runtime-6-Load&Initialize/)

![OC-Runtime](/images/OC-Runtime.png)


### +load

* `+load`方法只会被执行一次，且是在程序加载阶段，这时运行环境并不完备。
* 不同类或类别的`+load`方法会按一定顺序被执行。

```
1、依赖库的`+load`。
2、当前库`+load`，`class`的`+load`优先于`category`，父`class`优先于子`class`。
3、当前库的`All C++ static initializers and C/C++ __attribute__(constructor) functions`。
4、被依赖库的`+load`。

```

* 在`+load`方法中可以安全地发送消息给当前库内其他类的消息，虽然这些类的`+load`可能还没有执行过。

* 从源码中，也可以看出`class`的`+load`优先于`category`，父`class`优先于子`class`。

```
void prepare_load_methods(const headerType *mhdr)
{
    size_t count, i;

    runtimeLock.assertWriting();

    classref_t *classlist = 
        _getObjc2NonlazyClassList(mhdr, &count);
    for (i = 0; i < count; i++) {
        schedule_class_load(remapClass(classlist[i]));
    }

    category_t **categorylist = _getObjc2NonlazyCategoryList(mhdr, &count);
    for (i = 0; i < count; i++) {
        category_t *cat = categorylist[i];
        Class cls = remapClass(cat->cls);
        if (!cls) continue;  // category for ignored weak-linked class
        realizeClass(cls);
        assert(cls->ISA()->isRealized());
        add_category_to_loadable_list(cat);
    }
}
```

```
static void schedule_class_load(Class cls)
{
    if (!cls) return;
    assert(cls->isRealized());  // _read_images should realize

    if (cls->data()->flags & RW_LOADED) return;

    // Ensure superclass-first ordering
    schedule_class_load(cls->superclass);

    add_class_to_loadable_list(cls);
    cls->setInfo(RW_LOADED); 
}
```

### initialize

* 向类或对象发送消息时，会调用`lookUpImpOrForward`查找，并会检查是否初始化过，如果没有，则会调用`_class_initialize`做初始化。


```
IMP lookUpImpOrForward(Class cls, SEL sel, id inst, 
                       bool initialize, bool cache, bool resolver)
{
    // ...

    if (initialize  &&  !cls->isInitialized()) {
        _class_initialize (_class_getNonMetaClass(cls, inst));
        // If sel == initialize, _class_initialize will send +initialize and 
        // then the messenger will send +initialize again after this 
        // procedure finishes. Of course, if this is not being called 
        // from the messenger then it won't happen. 2778172
    }

	// ...

    return imp;
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

















