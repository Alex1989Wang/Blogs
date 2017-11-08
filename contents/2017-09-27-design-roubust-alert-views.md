---
title: How To Design an Alert View Mechanism For Your App
---

UI consistency seems to be a big requirement when designing your App - all the UI elements should have a consistent style throughout the entire App. This is especially true for Apps with different kinds of `alert views` or `alert controllers`. Well, `UIKit` itself comes with two different styles of `alert controllers`: `UIAlertControllerStyleActionSheet` and `UIAlertControllerStyleAlert`. Using them is absolutely convenient, but the drawback is that they don't allow you to have much customization. 

For the App which we are currently developing, I have been doing some experiments with the alert view mechanism of the App - making this module as independent as possible and, at the same time, as convenient to use as possible. 

When designing alert views for the entire App, a few things have to be considered before you actually start to get your hands dirty:

- How will the public interfaces be like so that an object of `UIViewController (or its subclass)` can easily present an alert view to the user
- How to keep the presented alert views (or alert controllers) separated from the rest of your view hierarchy?
- How the user experience be defined when there are multiple instances of your designed alert view presented?
- How the presentation and dismissal animations be like?

## Question One

Well, the first question would be easy to answer. To alleviate the pain when using your customized alert view (or alert controller) in one of your view controllers, you could have a category of `UIViewController`, for example, `UIViewController+CustomizedAlerViewPresentation.h`. And in this category, public interfaces can be implemented to present your customized alert views (or alert controllers). 

However, this solution might be somewhat not so convenient, sometimes. To be frank, this inconvenience is trivial and if this happens, your project might not strictly follow the `MVC` pattern strongly recommended by Apple. But, anyhow, it's an inconvenience that can be discussed here, which is what if you want to also present alert views (or alert controllers) from one of your customized views or some of your manger objects, especially the latter? For the former, delegating the presentation action to a view controller is a much elegant solution.

So, this brings us to another solution: using a singleton to manage all your alert views' (or alert controllers') presentation. Although, too many unnecessary singletons is always something I oppose to. This is why I insist to use the first solution in my App, but using singleton in this case is able to solve the problem. 

```
//[Demo Code Link](https://github.com/Alex1989Wang/Demos/blob/master/DemoProjects/JWAlertController/JWAlertController/Alert%20Module/Controllers/UIViewController%2BJWAlertPresentation.m)

@class JWAlertViewController;
@interface UIViewController (JWAlertPresentation)
/**
弹出一个提示控制器

@param alertController JWAlertViewController类型的自定义弹窗
@param animated 是否需要弹出的时候动画
*/
- (void)presentAlertController:(JWAlertViewController *)alertController
		      animated:(BOOL)animated;

/**
取消弹窗控制器

@param alertController JWAlertViewController类型的自定义弹窗
@param animated 是否需要淡出动画
@note 点击按钮时已经自动调用
*/
- (void)dismissAlertController:(JWAlertViewController *)alertController
 		      animated:(BOOL)animated;

@end
```

## Question Two

You definitely don't want to instantiate an object of your `alert views` or `alert controllers` and then refer to it in your views or view controllers. Like the behavior of `UIKit`'s `UIAlertController`, it always floats above any of your views and when you click any of its buttons it dismisses from the window. So, the idea here is that your `alert views` or `alert controllers` should behave very much like Apple's default `UIAlertController` and keep themselves away from the rest of your view hierarchy. So, how can we achieve this? The recipe is `using a customized window`. 

Your App can have one `key window` and another `alert window` to hold your `alert views` or `alert controllers`. Because the `key window` which has all your visible contents and `alert window` are siblings, the `alert window` is guaranteed to be above all your view elements. 

Demo code snippets:

```
- (void)presentAlertController:(JWAlertViewController *)alertController
	animated:(BOOL)animated {

	if (!alertController) {
		return;
	}

	static dispatch_once_t onceToken;
	dispatch_once(&onceToken, ^{
		CGRect windowBounds = [UIScreen mainScreen].bounds;
		alertRootWindow = [[UIWindow alloc] initWithFrame:windowBounds];
		alertRootWindow.windowLevel = UIWindowLevelNormal;
	});

	//缓存第一个弹窗为传入弹窗
	if ([self unpackFirstAlertController] == alertController) {
		if (!alertRootWindow.superview) {
			JWAppDelegate *appDelegate = (JWAppDelegate *)[UIApplication sharedApplication].delegate;
			[appDelegate.window addSubview:alertRootWindow];
		}
		alertRootWindow.rootViewController = alertController;
		alertRootWindow.hidden = NO;
		[alertController displayAlertViewAnimated:animated completed:nil];
		return;
	}

	//是否已经有弹窗
	NSMutableArray *alertControllers =
	objc_getAssociatedObject(alertRootWindow, &kAlertControllersArray);
	if (alertControllers.count) {
		//只缓存
		NSDictionary *alertInfo = [self infoDictionaryWithController:alertController
		willAnimateAppearance:animated];
		if (alertInfo) {
			[alertControllers addObject:alertInfo];
		}
		return;
	}

	//没有弹窗
	JWAppDelegate *appDelegate = (JWAppDelegate *)[UIApplication sharedApplication].delegate;
	[appDelegate.window addSubview:alertRootWindow];
	alertRootWindow.rootViewController = alertController;
	alertRootWindow.hidden = NO;
	[alertController displayAlertViewAnimated:animated completed:nil];

	NSDictionary *alertInfo = [self infoDictionaryWithController:alertController
	willAnimateAppearance:YES];
	NSAssert(alertInfo, @"alert info should exist");
	NSMutableArray *alertArray = [NSMutableArray arrayWithObject:alertInfo];
	objc_setAssociatedObject(alertRootWindow, &kAlertControllersArray, alertArray, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
```
## Question Three

This question should be considered in terms of how you want to customize queued behavior of multiple `alert views`. So, it will be queued, that's for sure. But, will the second alert view replace the already-displayed alert view and when this second alert view disappears the former-displayed alert view can reappear again? Or, the second will be cached and begin to be presented until the already-displayed one disappears?

The first kind of behavior is actually the apples' default way of presentation. But, in my App, we want the second. 

The queuing logic is simple. When the first alert view is dismissed, you check whether there are more to be presented. If so, present the first one in the queue. 

## Question Four

This is very much up to the taste of your designer. You could have a alert view appearing with size of zero then animated to its actual size. Or, you could make the alert view flying from the top of screen to the middle. 

So, choose whatever you like. But, there is one thing worthy of mentioning: the disappearing animation should be better a reversal of its appearing animation for the sake of consistency. 

## The Demo

The github [demo link](https://github.com/Alex1989Wang/Demos/tree/master/DemoProjects/JWAlertController)
