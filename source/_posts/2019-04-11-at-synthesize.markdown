---
layout: post
title: "@synthesize"
date: 2019-04-11 14:00:18 +0800
comments: true
categories: 
---
@synthesize做了什么，写和不写的区别是什么，现在还有哪些使用场景<!--more-->
直接上代码，首先是正常的属性声明。

```javascript

@interface ViewController : UIViewController

@property (nonatomic, strong) NSNumber *num;
@end

@implementation ViewController

- (void)viewDidLoad {
[super viewDidLoad];
// Do any additional setup after loading the view.
}
```
查看其C++实现，查看方法[之前的文章有提到过](http://devyang.space/blog/2019/04/09/self-super/)
```javascript


extern "C" unsigned long OBJC_IVAR_$_ViewController$_num;
struct ViewController_IMPL {
struct UIViewController_IMPL UIViewController_IVARS;
NSNumber *_num;
};

// @property (nonatomic, strong) NSNumber *num;
/* @end */
// @interface ViewController ()

/* @end */

// @implementation ViewController

static void _I_ViewController_viewDidLoad(ViewController * self, SEL _cmd) {
((void (*)(__rw_objc_super *, SEL))(void *)objc_msgSendSuper)((__rw_objc_super){(id)self, (id)class_getSuperclass(objc_getClass("ViewController"))}, sel_registerName("viewDidLoad"));
}

static NSNumber * _I_ViewController_num(ViewController * self, SEL _cmd) { return (*(NSNumber **)((char *)self + OBJC_IVAR_$_ViewController$_num)); }
static void _I_ViewController_setNum_(ViewController * self, SEL _cmd, NSNumber *num) { (*(NSNumber **)((char *)self + OBJC_IVAR_$_ViewController$_num)) = num; }
```
由此可以看出，我们平时声明的属性，在编译期间会由编译器自动添加上set和get方法的实现。同时也可以发现属性在这里面被转换成了
set+get+成员变量的形式。如果此时声明一个成员变量，其名为_num，其他不变，再执行clang，可以发现其实现没有任何改变。

下面我们加入@synthesize，OC代码如下并指定合成变量名为_num
#### : 示例a

```javascript
@interface ViewController : UIViewController
@property (nonatomic, strong) NSNumber *num;
@end

@implementation ViewController
@synthesize num=_num;
- (void)viewDidLoad {
[super viewDidLoad];
}
```
C++代码如下
```javascript

extern "C" unsigned long OBJC_IVAR_$_ViewController$_num;
struct ViewController_IMPL {
struct UIViewController_IMPL UIViewController_IVARS;
NSNumber *_num;
};

// @property (nonatomic, strong) NSNumber *num;
/* @end */


// @interface ViewController ()

/* @end */

// @implementation ViewController

// @synthesize num=_num;
static NSNumber * _I_ViewController_num(ViewController * self, SEL _cmd) { return (*(NSNumber **)((char *)self + OBJC_IVAR_$_ViewController$_num)); }
static void _I_ViewController_setNum_(ViewController * self, SEL _cmd, NSNumber *num) { (*(NSNumber **)((char *)self + OBJC_IVAR_$_ViewController$_num)) = num; }


static void _I_ViewController_viewDidLoad(ViewController * self, SEL _cmd) {
((void (*)(__rw_objc_super *, SEL))(void *)objc_msgSendSuper)((__rw_objc_super){(id)self, (id)class_getSuperclass(objc_getClass("ViewController"))}, sel_registerName("viewDidLoad"));
}
```
此时我们发现，@synthesize执行的操作其实就是上面例子中添加set和get的过程。那么@synthesize的合成规则是什么样的呢，继续来看代码
#### : 示例b(改变成员变量的变量名)

```javascript

//这行代码我们改变下
//@synthesize num=_num;

//变成这样，然后再次clang
@synthesize num=_num1;

```
clang之后的代码，为了便于查看只展示变化部分
```javascript
extern "C" unsigned long OBJC_IVAR_$_ViewController$_num1;
struct ViewController_IMPL {
struct UIViewController_IMPL UIViewController_IVARS;
NSNumber *_num1;
};

// @synthesize num=_num1;
static NSNumber * _I_ViewController_num(ViewController * self, SEL _cmd) { return (*(NSNumber **)((char *)self + OBJC_IVAR_$_ViewController$_num1)); }
static void _I_ViewController_setNum_(ViewController * self, SEL _cmd, NSNumber *num) { (*(NSNumber **)((char *)self + OBJC_IVAR_$_ViewController$_num1)) = num; }

```
此时合成的成员变量名也变成_num1。我们现在通过_num已经不能访问属性num了。同时，在合成的set和get 方法内部也变成了成员变量_num1。、
此时访问属性的访问为set+get+_num1。

那么如果不写名成员变的名字 又变成什么样子呢。
#### : 示例c(不写成员变量)
```javascript

//这行代码我们改变下
//@synthesize num=_num1;

//变成这样，然后再次clang
@synthesize num

```
clang之后的代码
```javascript

extern "C" unsigned long OBJC_IVAR_$_ViewController$num;
struct ViewController_IMPL {
struct UIViewController_IMPL UIViewController_IVARS;
NSNumber *num;
};

// @synthesize num;
static NSNumber * _I_ViewController_num(ViewController * self, SEL _cmd) { return (*(NSNumber **)((char *)self + OBJC_IVAR_$_ViewController$num)); }
static void _I_ViewController_setNum_(ViewController * self, SEL _cmd, NSNumber *num) { (*(NSNumber **)((char *)self + OBJC_IVAR_$_ViewController$num)) = num; }

```
我们发现如果不指明成员变量的名称，则会自动生成一个跟属性名同名的成员变量，此时访问属性的访问为set+get+num。注意是num（不带下划线）。
### @synthesize的合成规则结论
* 如果是 @synthesize num（示例c); 会生成一个名称为 num 的成员变量，也就是说：如果没有指定成员变量的名称会自动生成一个属性同名的成员变量，
* 如果指定了成员变量的名称，会生成一个指定的名称的成员变量 @synthesize num = _num1(示例b);。如果这个成员已经存在了就不再生成了。
### 需要使用@synthesize的场景
* 同时重写了 setter 和 getter 时
* 重写了只读属性的 getter 时
* 使用了 @dynamic 时
* 在 @protocol 中定义的所有属性
* 重载的属性
