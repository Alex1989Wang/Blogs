# Blogs
For more posts, visit my [personal blog site](http://awesomejiang.cn/)

See my [profile](https://github.com/Alex1989Wang/alex1989wang.github.io/blob/master/about.markdown)

# Contents

## [Infinite Scrolling and the Tiling Logic](http://www.awsomejiang.com/2018/03/24/Infinite-Scrolling-and-the-Tiling-Logic/)
Sometimes, your app’s UX designer wants a infinite scroll for one of your collection views when the data displayed is limited. When a user scrolls to the very end of the data set, the first piece of data reappears on screen; if the use scrolls the other way, the last piece of data reappears.Traditionally, the solution for an infinite UICollectionView is to have a large duplicated data set (for example, 1000 * original data set) to trick the user into believing the collection view is infinite. This post explains how tiling can be used to make a infinitely-scrolling scroll view and a similar approach can be applied to collection view to achieve the same result. 

## [Design Waving Effect Of High Performance](http://www.awsomejiang.com/2018/03/20/Highly-perfomant-Waving-Effect/)
Some comparisons and measurements carried out between timer-driven waving animation and CAReplicatorLayer-based animation are laid out.

## [Core Animation动画的不同阶段](http://www.awsomejiang.com/2018/03/06/about-core-animtion-animation-stages/)
了解动画的不同阶段，更加深入地理解runloop和CoreAnimation的关系；另外，动画的不同阶段cpu usage能够直到动画性能的调优。

## [iOS应用的内存管理（二)](http://www.awsomejiang.com/2018/01/15/Memory-Management-For-iOS-Apps-No-2/)
iOS平台上的`virtual memory`机制和基本概念。

## [iOS应用的内存管理（一)](http://www.awsomejiang.com/2018/01/14/Memory-Management-for-iOS-Apps/)
主要介绍了`retain count`的基础和`MRR`下的内容管理规则和实践。

## [视图控制器转换(二) View Controller Transistion - No.2](http://www.awsomejiang.com/2018/01/01/custmize-navigation-controller-transition-animations/)
介绍了自定义交互类型转场动画的原理，同时用demo展示了实现自定义`navigation transition`效果的实现过程。

## [视图控制器转换(一) View Controller Transistion - No.1](http://www.awsomejiang.com/2017/12/25/View-Controller-Transistion-customizing-presentation-md/)
介绍了自定义转场动画的原理，同时用demo展示了实现自定义`presentation`效果的实现过程。

## [Design Alert Views](contents/2017-09-27-design-roubust-alert-views.md)
如何给`App`设计一套弹窗系统。

## [LLDB命令补充](contents/about-lldb-what-else-do-you-know-two.md)
对上篇[LLDB命令基本使用](contents/about-lldb-what-else-do-you-know.md)常用命令的分类补充。

## [LLDB命令基本使用](contents/about-lldb-what-else-do-you-know.md)
除了在`XCode`的调试控制台（`debugging console`）使用`po`，对于`lldb`还需要知道些什么？

## [Objective-C Runtime 基础](contents/runtime-programming-guide-reading-notes-one.md)
什么是`OC`的动态特性？动态绑定？消息发送机制？

## [使用XCode测试工程代码（二）](contents/testing-with-xcode-two.md)
性能测试和对异步功能模块的测试。

## [使用XCode测试工程代码（一）](contents/testing-with-xcode-one.md)
简单地介绍了使用`XCtest`框架编写单元测试和运行。

## [Hit Test和响应者链条](contents/hittest-and-responder-chain.md)
`Hit-Test`和`Responder chain`是一个每个开发者都应该知道的基本知识。`hit-test`是如何确定`first responder`的？`responder chain`是如何动态地确定的？

## [Core Animation文档阅读笔记](contents/about-core-animation.md)
`Core Animation`能够让你制作复杂的动画，你是否理解`UIView block animation`和`Core Animation`的关系？什么是`explicit animation`，什么是`implicit animation`？

## [XCode常用快捷键](contents/xcode-keyborad-shortcuts.md)
常用的`XCode`快捷键，让你摆脱鼠标。

## [Category VS. Class Extension](contents/category-vs.-class-extension.md)
什么是`OC`中的类别（`category`）和类扩展（`class extension`），在类别中**申明**属性（`property`）合法的吗？有什么作用？

## [变量extern vs. static 修饰关键字](contents/static-vs.-extern-keywords.md)
`extern`关键字和`static`关键字既有联系又有很大区别。另外由这两个关键字引出的另外两个概念：`静态变量`和`外部变量`和关键字又是什么关系？如何正确使用`static`和`extern`？

## [How I Learned iOS Programming](contents/how-i-learned-iOS-programming.md)
How I learned iOS programming all by myself.

## [Hello World](./contents/hello-world.md)
The story about how I began programming.

