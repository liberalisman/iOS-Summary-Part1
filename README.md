# 2018

文章大纲，便于浏览

1. [01-iOS程序启动过程](https://github.com/liberalisman/2018-Interview-Preparation#01-ios-app-startup-process)

    i.[完整启动过程 ](https://github.com/liberalisman/2018-Interview-Preparation#一启动完整过程)
    ii.[程序启动原理](https://github.com/liberalisman/2018-Interview-Preparation#二程序启动原理)
    

2. [02-浅拷贝-深拷贝](https://github.com/liberalisman/2018-Interview-Preparation#02-shallowcopy-deepcopy)

   i. [系统对象的 copy/mutableCopy](https://github.com/liberalisman/2018-Interview-Preparation#一系统对象的-copymutablecopy)
   ii.[自定义对象实现 Copy-MutableCopy](https://github.com/liberalisman/2018-Interview-Preparation#二自定义对象实现-copy-mutablecopy)
   iii.[copy 本质](https://github.com/liberalisman/2018-Interview-Preparation#三copy-本质)




## 01-iOS-App-startup-process

###一、启动完整过程

![](http://okhqmtd8q.bkt.clouddn.com/image/jpg/%E7%A8%8B%E5%BA%8F%E5%90%AF%E5%8A%A8%E6%B5%81%E7%A8%8B.png)

1.`main`函数

2.`UIApplicationMain`函数

* 创建`UIApplication`对象
* 创建`UIApplication的delegate`对象

3.`delegate`对象开始处理(监听)系统事件(没有storyboard)

* 程序启动完毕的时候, 就会调用代理的:`didFinishLaunchingWithOptions:`方法
* 在`application:didFinishLaunchingWithOptions`:中创建`UIWindow` 创建和设置`UIWindow`的`rootViewController`
* 显示窗口

4.根据`Info.plist`获得最主要`storyboard`的文件名,加载最主要的`storyboard`(有storyboard)

* 创建`UIWindow`
* 创建和设置`UIWindow`的`rootViewController`
* 显示窗口


###二、程序启动原理


1.`main`函数中执行了一个`UIApplicationMain`这个函数

```objc
int UIApplicationMain(int argc, char *argv[], NSString *principalClassName, NSString *delegateClassName);

argc、argv：直接传递给UIApplicationMain进行相关处理即可
```

2.`principalClassName`：指定应用程序类名（app的象征），该类必须是`UIApplication`(或子类)。如果为`nil`,则用`UIApplication`类作为默认值

3.`delegateClassName`：指定应用程序的代理类，该类必须遵守`UIApplicationDelegate`协议

4.`UIApplicationMain`函数会根据`principalClassName`创建`UIApplication`对象，根据`delegateClassName`创建一个`delegate`对象，并将该`delegate`对象赋值给`UIApplication`对象中的`delegate`属性

5.接着会建立应用程序的`Main Runloop`（事件循环），进行事件的处理(首先会在程序完毕后调用`delegate`对象的`application:didFinishLaunchingWithOptions`:方法)

程序正常退出时`UIApplicationMain`函数才返回

```objc
int main(int argc, char * argv[]){ @autoreleasepool { 

/**
* argc: 系统或者用户传入的参数个数
* argv: 系统或者用户传入的实际参数 
* 1.根据传入的第三个参数创建UIApplication对象 
* 2.根据传入的第四个产生创建UIApplication对象的代理
* 3.设置刚刚创建出来的代理对象为UIApplication的代理 
* 4.开启一个事件循环 
**/ 
return UIApplicationMain(argc, argv, @"UIApplication", @"YYAppDelegate"); }}

```

启动与代理：
![](http://okhqmtd8q.bkt.clouddn.com/image/jpg/%E7%A8%8B%E5%BA%8F%E5%90%AF%E5%8A%A8%E4%B8%8E%E4%BB%A3%E7%90%86.png)

## 02-ShallowCopy-DeepCopy

简要总结一下什么是浅拷贝，什么是深拷贝

> 深拷贝就是内容拷贝

> 浅拷贝就是指针拷贝

###一.系统对象的 copy/mutableCopy
 

```objc
NSString *string = @"LiMing";
    
NSString *copyString = [string copy];
    
NSString *mutableString = [string mutableCopy];
    
NSLog(@"string = %p",string);
    
NSLog(@"copyString = %p",copyString);
    
NSLog(@"mutableString = %p ",mutableString);

结论：
1.string 和 copyString 他们只是二个不同的指针，指向内存中的同一块地址，copy 只是指针复制
2.string 和 mutableString 打印出来的地址不同，是因为两个指针指向的地址本就不同，mutableCopy 是内容复制

注意：其他对象 NSArray 、NSMutableArray 、NSDictionary 、NSMutableDictionary 一样适用
```

规律可以从这张图看出来

![](http://okhqmtd8q.bkt.clouddn.com/image/jpg/%E6%B7%B1%E6%8B%B7%E8%B4%9D-%E6%B5%85%E6%8B%B7%E8%B4%9D-01)

![](http://okhqmtd8q.bkt.clouddn.com/image/jpg/%E6%B7%B1%E6%8B%B7%E8%B4%9D-%E6%B5%85%E6%8B%B7%E8%B4%9D-02)

###二.自定义对象实现 Copy-MutableCopy

* copy

```objc
GZQPerson *person = [[GZQPerson alloc] init];
person.age = 20;
person.name = @"GZQ";
GZQPerson *copyP = [person copy];  // 这里崩溃
```

崩溃：
![](http://okhqmtd8q.bkt.clouddn.com/image/jpg/%E6%B7%B1%E6%8B%B7%E8%B4%9D-%E6%B5%85%E6%8B%B7%E8%B4%9D-03.png)

看崩溃信息GZQPerson应该先实现：

```objc
- (id)copyWithZone:(NSZone *)zone;
```
测试：

```objc
#import "GZQPerson.h"

@interface GZQPerson ()<NSCopying,NSMutableCopying>

@end

@implementation GZQPerson

- (id)copyWithZone:(NSZone *)zone {

    GZQPerson *person = [[[self class] allocWithZone:zone] init];
    person.age = self.age;
    person.name = self.name;
    return person;
}

- (id)mutableCopyWithZone:(NSZone *)zone {

    GZQPerson *person = [[[self class] allocWithZone:zone] init];
    person.age = self.age;
    person.name = self.name;
    return person;
}

@end


```



```objc
#import "ViewController.h"
#import "GZQPerson.h"

@interface ViewController ()

@end

@implementation ViewController

- (void)viewDidLoad {
    
    [super viewDidLoad];

    GZQPerson *person = [[GZQPerson alloc] init];
    person.age = 20;
    person.name = @"GZQ";
    GZQPerson *copyP = [person copy];
    
    NSLog(@"copyP=%p",copyP);
    NSLog(@"person=%p",person);
    NSLog(@"person=%p",copyP.name);
    NSLog(@"person=%p",person.name);
    
}
@end
```

![](http://okhqmtd8q.bkt.clouddn.com/image/jpg/%E6%B7%B1%E6%8B%B7%E8%B4%9D-%E6%B5%85%E6%8B%B7%E8%B4%9D-04.png)

可以看出虽然指针的地址不同，但是存储的地址是一致的。



###三.copy 本质

`property copy` 实际上就对name干了这个：

```objc
#import <Foundation/Foundation.h>

property copy 实际上就对name干了这个：

- (void)setName:(NSString *)name
{
    _name = [name copy];
}
```

`strong`是不执行`Copy`操作的

```objc
@property (nonatomic, strong) NSString *name;

NSMutableString *string = [NSMutableString stringWithFormat:@"深拷贝-浅拷贝"];

GZQPerson *person = [[GZQPerson alloc] init];
person.name = string;

// 可以改变person.name的值，因为其内部没有生成新的对象
[string appendString:@"LALALA"];

NSLog(@"name = %@", person.name);
```


