# Objective-C Runtime 基础

`Objective-C`语言的`runtime`环境就好比是该语言的操作系统；它是让`OC`拥有面向对象和动态特性的基础。那`runtime`能够提供什么呢？

- 在运行时动态加载类
- 将消息转发给其他对象（此处的`消息`指`OC`的消息发送机制）
- 在运行时获取一个对象的信息（实例变量、实例方法，等）
- ...

## 理解动态特性

`OC`的动态特性使得该语言能够将代码执行的许多决定从编译时或者链接时推迟到运行时。这是[runtime 官方文档](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html)对`OC`动态属性的解释。

> The Objective‐C language defers as many decisions as it can from compile time and link time to runtime. Whenever possible, it does things dynamically.

对比静态语言，比如`C`语言就可以发现这点不同。一个`C`语言的程序，完成编译、链接生成二进制可执行文件，该可执行文件在运行时是没有任何动态特性的：从`main`函数开始，调用其他函数，到整个程序运行结束，都是一个自上而下的执行过程。并没有说在调用某个函数时，在运行时才动态地确定这个函数的实现是什么。

而对于`OC`语言来讲，在对一个`receiver`发送一个消息时（`[receiver messgae]`），该消息的执行可以是`receiver`对象；也可以是`receiver`对象将该消息转发出去，另外一个对象执行该消息对应的方法实现；甚至，可以动态地对该消息增加方法实现。

这就是`OC`这个动态语言和类似于`C`语言这样的静态语言在动态特性上的差别。

所以，这就前面说到的：***`OC`的动态特性使得该语言能够将代码执行的许多决定从编译时或者链接时推迟到运行时***。简单地讲，也就是`OC`在运行时才决定为某个发送的消息**动态地绑定**一个方法实现。

## 理解`messaging`

上面已经讲到了在`OC`中，直到运行时，某个消息才和特定的方法实现绑定。（也就是所谓的**动态绑定**）。

> In Objective‐C, messages aren’t bound to method implementations until runtime. 

那么，编译器会将：

```objc
[receiver message]
```
这样一个消息表达式，转换为：

```c
bjc_msgSend(receiver, selector)
```
对消息发送函数的调用。该函数接受`消息接受者`和`消息对应的selector`作为两个主要的参数。如果，该消息本身还有其他参数，则跟在`selector`之后：

```c
objc_msgSend(receiver, selector, arg1, arg2, ...)
```

对于一个`OC`的对象而言，当其被创建时，其内存布局就已经确定。该对象的`isa`指针指向该对象的类；而该对象的类的`isa`指针指向其父类。因此，可以一直回溯整个继承层级（`inheritance hierachy`）获得该对象的所有信息（如，所有的实例方法、实例变量等）。

消息的发送也利用了这一点。

当一个消息发送给了一个对象之后，该对象会查看其类的`dispatch table`中是不是有对应的方法。如果没有对应的方法就去查看父类的`dispatch table`中是否存在该方法。直到回溯到`NSObject`。

如果找不到该方法的话，`OC`的运行时会先尝试寻找是否为该消息动态地提供了实现。也就是调用：

```objc
+ (BOOL)resolveClassMethod:(SEL)sel OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0);
+ (BOOL)resolveInstanceMethod:(SEL)sel OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0);
```
如果并没有在这两个方法中找到实现，`OC`就会尝试进行消息的转发机制，而不是立即报`unrecognized selector sent to target ...`的错误。

消息转发分为快速消息转发（`fast message forwarding`）和普通消息转发机制（`regular message forwarding`）。快速转发机制发生在普通转发机制之前。

### 快速消息转发机制（`fast message forwarding`）

快速消息转发之所以叫做快速消息转发，是因为其效率较普通的消息转发机制要高。其实质就是将对象不识别，同时也没有提供动态方法实现的消息转发给其他的对象。需要不识别该消息的对象实现消息转发的方法：

```objc
- (id)forwardingTargetForSelector:(SEL)aSelector OBJC_AVAILABLE(10.5, 2.0, 9.0, 1.0);
```

在该方法中返回一个消息需要转发给的`target`对象。对于对于不需要处理的消息的话，返回对于`super`的调用。

当消息被转发给另外对象之后，前面提到的查询`dispatch table`的过程就会通过这个新的`receiver`的继承层级重新启动。

官方文档对该方法的解释：

> Returns the object to which unrecognized messages should first be directed. 
>
> If an object implements (or inherits) this method, and returns a non-nil (and non-self) result, that returned object is used as the new receiver object and the message dispatch resumes to that new object. (Obviously if you return self from this method, the code would just fall into an infinite loop.)
>
> If you implement this method in a non-root class, if your class has nothing to return for the given selector then you should return the result of invoking super’s implementation.
>
> This method gives an object a chance to redirect an unknown message sent to it before the much more expensive forwardInvocation: machinery takes over.

### 普通消息转发机制（`regular message forwarding`）

普通转发机制更加的`expensive`。也是最后一个处理不识别消息的机会。其调用的方法是：

```objc
- (void)forwardInvocation:(NSInvocation *)anInvocation OBJC_SWIFT_UNAVAILABLE("");
- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector OBJC_SWIFT_UNAVAILABLE("");
```

在```- (void)forwardInvocation:```方法中判断需要拦截的`selector`，将该消息转发给另外一个识别该消息的对象。

但是，需要注意的是在`runtime`调用```- (void)forwardInvocation:```之前，它会调用```- (NSMethodSignature *)methodSignatureForSelector:```方法，需要返回正确的`method signature`后，消息转发才能正常使用。

## 代码示例

[示例项目](https://github.com/Alex1989Wang/Demos/tree/master/DemoProjects/OCRuntimeDemo)简单地实践了动态提供方法实现（`+ (BOOL)resolveInstanceMethod:`）、快速消息转发和普通消息转发。

`Gift`类本身并没有实现任何的`priceInUSD`这个方法。

```objc
@implementation Gift

+ (BOOL)resolveInstanceMethod:(SEL)sel {
    NSLog(@"resolve instance method called.");
    return [super resolveInstanceMethod:sel];
}

- (id)forwardingTargetForSelector:(SEL)aSelector {
    NSLog(@"forward target for selector.");
    if (aSelector == @selector(priceInUSD)) {
	    return self.price;
    }
    else {
	    return [super forwardingTargetForSelector:aSelector];
    }
}

- (void)forwardInvocation:(NSInvocation *)anInvocation {
    NSLog(@"forward invacation called.");
    if (anInvocation.selector == @selector(priceInUSD)) {
	    [anInvocation invokeWithTarget:self.price];
    }
}

- (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector {
    NSLog(@"method signature for selector.");
    if (aSelector == @selector(priceInUSD)) {
	    NSMethodSignature *mthdSig =
	    [self.price methodSignatureForSelector:@selector(priceInUSD)];
	    return mthdSig;
    }
    else {
	    return [super methodSignatureForSelector:aSelector];
    }
}
@end
```

`Gift`类的任何对象本身没有实现任何的`priceInUSD`的方法，因此将这个消息通过快速和普通消息转发的方式转发给`price`对象（当然在真是编程实践中，只需要直接调用`price`的方法就好了。这里只是用来说明问题。）

通过前面的讨论知道，快速转发发生在普通转发之前。因此如果需要普通转发正常工作，需要注释掉`- (id)forwardingTargetForSelector:`。

## 参考资料

- [Runtime Programming Guide](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Introduction/Introduction.html)
- [Method Reference](https://developer.apple.com/documentation/objectivec/nsobject/1571955-forwardinvocation)
