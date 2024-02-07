---
title: OC Runtime(2)之Ivar&Property&Method&Protocol
date: 2014-03-12
---

### Runtime系列笔记

* [Runtime之对象和类](/2014/03/07/OC-Runtime-1-Object&Class/)
* [Runtime之属性和方法](/2014/03/12/OC-Runtime-2-Ivar&Property&Method&Protocol/)
* [Runtime之关联对象](/2014/03/18/OC-Runtime-3-Associated%20Objects/)
* [Runtime之消息转发](/2014/03/21/OC-Runtime-4-Message%20Forwarding/)
* [Runtime之方法替换](/2014/04/02/OC-Runtime-5-Method%20Swizzling/)
* [Runtime之Load&Initialize](/2014/04/09/OC-Runtime-6-Load&Initialize/)

![OC-Runtime](/images/OC-Runtime.png)


### Ivar

```
struct old_ivar {
    char *ivar_name;
    char *ivar_type;
    int ivar_offset;
#ifdef __LP64__
    int space;
#endif
};
```


### Property

```
struct old_property {
    const char *name;
    const char *attributes;
};
```

### Method

```
struct old_method {
    SEL method_name;
    char *method_types;
    IMP method_imp;
};
```


### protocol

```
struct old_protocol {
    Class isa;
    const char *protocol_name;
    struct old_protocol_list *protocol_list;
    struct objc_method_description_list *instance_methods;
    struct objc_method_description_list *class_methods;
};
```

```
struct objc_method_description *
lookup_protocol_method(old_protocol *proto, SEL aSel, 
                       bool isRequiredMethod, bool isInstanceMethod, 
                       bool recursive)
{
    struct objc_method_description *m = nil;
    old_protocol_ext *ext;

    if (isRequiredMethod) {
        if (isInstanceMethod) {
            m = lookup_method(proto->instance_methods, aSel);
        } else {
            m = lookup_method(proto->class_methods, aSel);
        }
    } else if ((ext = ext_for_protocol(proto))) {
        if (isInstanceMethod) {
            m = lookup_method(ext->optional_instance_methods, aSel);
        } else {
            m = lookup_method(ext->optional_class_methods, aSel);
        }
    }

    if (!m  &&  recursive  &&  proto->protocol_list) {
        int i;
        for (i = 0; !m  &&  i < proto->protocol_list->count; i++) {
            m = lookup_protocol_method(proto->protocol_list->list[i], aSel, 
                                       isRequiredMethod,isInstanceMethod,true);
        }
    }

    return m;
}
```



### category

```
struct old_category {
    char *category_name;
    char *class_name;
    struct old_method_list *instance_methods;
    struct old_method_list *class_methods;
    struct old_protocol_list *protocols;
    uint32_t size;
    struct old_property_list *instance_properties;
};
```


### 参考

* [understanding-objective-c-runtime](http://cocoasamurai.blogspot.jp/2010/01/understanding-objective-c-runtime.html)
* [ObjCRuntimeGuide](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40008048)
* [runtime源码](http://opensource.apple.com/tarballs/objc4/)
* [Objective-C 的动态提示和技巧](http://blog.jobbole.com/45963/)
* [NSObject的load和initialize方法](http://www.cocoachina.com/ios/20150104/10826.html)
* [associated-objects](http://esoftmobile.com/2014/02/18/associated-objects/)
* [method-swizzling](http://esoftmobile.com/2014/02/19/method-swizzling/)

















