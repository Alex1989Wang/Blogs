# Category VS. Class Extension

分类和类的扩展在`Objective-C`中是紧密相连的话题；但是，他们之间存在着不少区别。能够合理地使用`分类`和`类扩展`不断能够使得一个类的代码结构更加清晰，还能将私有方法、属性和公开方法、属性合理的区分开。

## `Category`分类

分类提供了为某个类增加其他行为的途径；该类可以是自己写的能够获得其实现源代码的类，也可以是已经封装好的不能够看到实现源代码（如`framework`的类)。

分类的语法：

```objc
#import "DemoChatViewController.h"

@interface DemoChatViewController (DemoGroupChat)

/**
 首次加载群聊消息
 */
- (void)loadGroupChatMessages;

/* 
 other methods
 ...
 */
@end
```
```objc
#import "DemoChatViewController+DemoGroupChat.h"

@implementation DemoChatViewController (DemoGroupChat)
- (void)loadGroupChatMessages {
   /* implementation */
}
@end
```

上面的代码示例展示的是为类`DemoChatViewController`写一个名为`DemoGroupChat`的分类。该分类既可以看做为原来类添加了群聊的相关功能，也可以认为是对原来类的方法按照`功能模块`来划分不同类别。

在为某个类使用分类的方法进行扩展其功能的时候，需要注意的一点就是**分类中的方法是不能够和该类的主要实现中某方法、该类其他分类中的某方法以及该类父类中的方法同名。如果出现同名，那么在运行时，那个方法被调用是未知的。**

关于分类，有一点非常重要。***分类中尽管是可以申明`property`，但是该`property`是不会自动合成实例变量，也不会有`accessor methods`***。

- 分类中是可以申明不同的`property`的；
- 这些`property`不是`instance-variable-backed property`；

所以，不能简单理解为分类中不能够申明`property`。后面的[案例](#案例)中可以看到，利用分类中可以申明`property`的特性，可以将一个类的属性合理地暴露给其他类或分类。而在不需要知道这些属性的情形下，只是暴露该类的`primary header file`主头文件。

同样延续上面代码案例的使用场景，可以设计一个除了`DemoChatViewController.h`这个主要的头文件（`primary header file`）以外的另一个头文件`DemoChatViewController+Private.h`。在该头文件中存放一些不需要被其他类知道，但是需要在`DemoChatViewController`的多个分类中需要使用的属性。

例如：

```objc
#import "DemoChatViewController.h"
@interface DemoChatViewController (Private)
/* 视图 */
@property (nonatomic, readonly, weak) UITableView *chatTable; //消息列表
@property (nonatomic, strong) NSMutableArray *cellData; //消息数据
```

这样，在前面的案例代码`DemoChatViewController+DemoGroupChat.m`可以导入该名为`private`的分类，那么在从服务器获取完群消息之后，就可以直接在方法`- (void)loadGroupChatMessages`中处理完消息数据，加入到`cellData`数组中，在调用`self.chatTable`的`reloadData`方法刷新界面的消息。

那么，刚才已经说过了。分类中声明的`property`是不会有`instance variable`被自动合成的，也就是说在运行时，`DemoChatViewController`的对象是没有`chatTable`和`cellData`实例变量的。那上面的写法难道没有很大的问题？

当然没有，因为这些`property`在类扩展中被重新申明了。

## `Class Extension`类扩展

类扩展实际上就是匿名的分类（`anonymous category`)。它实际上就是在`@implementation`语句上的未命名分类。

例如，

```objc
@interface DemoChatViewController ()
/* 视图 */
@property (nonatomic, weak) UITableView *chatTable; //消息列表
@property (nonatomic, strong) NSMutableArray *cellData; //消息数据
@end

@implementation DemoChatViewController
@end
```
所以，尽管类扩展是一种特殊的分类。但是其中定义的属性会被自动合成对象的实例变量，也就是对于属性`chatTable`来说，`DemoChatViewController`对象会存在一个`_chatTable`的实例变量。同理，对于`cellData`也是一样。

如果仔细观察分类`DemoChatViewController+Private.h`中属性`chatTable`和类扩展中的该属性，它们在属性的修饰关键字上还有一点不同。

```objc
@property (nonatomic, readonly, weak) UITableView *chatTable; //来自分类

@property (nonatomic, weak) UITableView *chatTable; //来自类扩展
```

也就是分类中的该属性是`readonly`而在类扩展中是默认`readwrite`的。这样写的好处是在于，在外部尽管可以使用`chatTable`；但是，如果视图`set`该`chatTable`的话，编译器就会警告。而在该类的主要实现文件（`primary implementation`）中，是完全可以使用`self.chatTable = ...`来设置该`chatTable`。

另外，需要注意的是类扩展（`class extension`）中声明的属性或者方法都和该类的实现文件有很强的绑定关系。简单的说就是，被自动合成的实例变量和`accessor methods`是放到`primary implementation`文件中的；那扩展中申明的方法也是必须要在`primary implementation`文件中实现。

## 示例整合

整合上面的实例代码，分别获得了五个文件：

- `DemoChatViewController.h`为主头文件（`primary header`）

  ```objc
  #import <UIKit/UIKit.h>

  @interface DemoChatViewController : UIViewController
  @property (nonatomic, assign, readonly) DemoChatType type;

  /**
   利用聊天类型来创建新的聊天控制器

   @param type 聊天ID类型
   @return 新创建的聊天控制器
   */
   - (instancetype)initWithChatType:(DemoChatType)type NS_DESIGNATED_INITIALIZER;
   @end
  ```
  在其他类导入该头文件时，只能看到一个公开的属性和方法。这样使得这个控制器的实现被很好地隐藏了起来。

- `DemoChatViewController.m`为主实现文件（`primary implementation`)

  ```objc
  @interface DemoChatViewController ()
  /* 视图 */
  @property (nonatomic, weak) UITableView *chatTable; //消息列表
  @property (nonatomic, strong) NSMutableArray *cellData; //消息数据
  /* 其他 */
  @property (nonatomic, assign) DemoChatType type;
  @end

  @implementation DemoChatViewController
   - (instancetype)initWithChatType:(DemoChatType)type {
   /* implementation ... */
   }
  @end
  ```
  在该类的实现文件上部，包括了该类的扩展。扩展中重写了`Private`分类头文件以及主头文件中的属性。

- `DemoChatViewController+Private.h`为私有头文件

  ```objc
  #import "DemoChatViewController.h"
  @interface DemoChatViewController (Private)
  /* 视图 */
  @property (nonatomic, readonly, weak) UITableView *chatTable; //消息列表
  @property (nonatomic, strong) NSMutableArray *cellData; //消息数据
  ```
  方便该类的各个分类之间使用该类定义的属性。

- `DemoChatViewController+GroupChat.h`为群聊分类头文件

  ```objc
  #import "DemoChatViewController.h"

  @interface DemoChatViewController (DemoGroupChat)

  /**
  首次加载群聊消息
  */
  - (void)loadGroupChatMessages;

  /* 
  other methods
  ...
  */
  @end
  ```
  定义群聊功能分类的头文件。

- `DemoChatViewController+GroupChat.h`为群聊分类实现文件

  ```objc
  #import "DemoChatViewController+DemoGroupChat.h"

  @implementation DemoChatViewController (DemoGroupChat)
  - (void)loadGroupChatMessages {
  /* implementation */
  }
  ```
  实现群聊的功能。
