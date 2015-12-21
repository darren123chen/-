# OC中extension的使用浅谈

我们在使用XCode时，创建ObjectiveC file时，会有category、protocol、extension和empty file四个选项，category和protocol大家都很熟悉了，而新建extension想必很少有人用到过吧，我也是在使用的过程发现单独创建的extension貌似没什么用，问周围的朋友也基本没用过单独创建的extension。然而存在即合理，既然可以单独创建extension，那肯定是有它的用处的。于是自己上网搜，也无果，遂自己做实验。


-------------------

## 1.最常见到和用到的extension

 其实extension本身我们已经很熟悉了，大家几乎天天都会和它打交道，新建一个SingleViewApplication，在自动生成的ViewController.m顶部，就会看到一个.m内部的extension，如下：
 

```
@interface ViewController ()

@end
```

这就是我们最常见到的extension，而我们也经常会用这种方式自己给类写一些扩展属性。这也是我上面强调**单独创建**的extension的原因。
我们在使用.m里的extension的时候，形式上来说就是匿名类别，然而事实上却的确是两个不同的东西。这个后面再解释。
而苹果现在推荐给类加extension的方法就是在.m中写，单独创建的extension苹果并不推荐，然而却一直占据着新建Objective-C file的一个选项，我疑惑的就是它存在的意义。


## 2.失败的尝试1---直接给类添加属性

第一个想法是既然category是用来扩展方法的，那么extension是不是就可以用来扩展属性呢？而平常我们在.m中的用法也是用来添加属性，这更增加了这种猜想的可能性。于是首先试着创建一个类ClassA，创建extension ClassA_test,可以看到只有一个.h，类A不写任何属性，在extension中写一个属性strA。

`@property (nonatomic, copy)NSString * strA;`

在main.m中调用ClassA_test.h创建对象a,给a.strA赋值后打印，

```
ClassA *a = [[ClassA alloc]init];
a.strA = @"testA";
NSLog(@"a.strA = %@",a.strA);
```
发生如下报错：

> 2015-11-24 02:26:25.428 extensionTest[6430:409576] -[ClassA setStrA:]: unrecognized selector sent to instance 0x1005039c0
2015-11-24 02:26:25.429 extensionTest[6430:409576] *** Terminating app due to uncaught exception 'NSInvalidArgumentException', reason: '-[ClassA setStrA:]: unrecognized selector sent to instance 0x1005039c0'
*** First throw call stack:
(
	0   CoreFoundation                      0x00007fff92219e32 __exceptionPreprocess + 178
	1   libobjc.A.dylib                     0x00007fff84f384fa objc_exception_throw + 48
	2   CoreFoundation                      0x00007fff9228334d -[NSObject(NSObject) doesNotRecognizeSelector:] + 205
	3   CoreFoundation                      0x00007fff9218a661 ___forwarding___ + 1009
	4   CoreFoundation                      0x00007fff9218a1e8 _CF_forwarding_prep_0 + 120
	5   extensionTest                       0x0000000100000df8 main + 104
	6   libdyld.dylib                       0x00007fff8cee25ad start + 1
)
libc++abi.dylib: terminating with uncaught exception of type NSException

不识别的set方法。
用该方法给官方类添加属性结果相同，实验失败。


## 3.失败的尝试2---搭配category使用，给category增加属性

然后我想到了给category增加属性，毕竟category一般用来增加方法，不能用一般的方法直接给category增加属性，那么，在extension中写上扩展的属性，然后用category调用，结果如何呢？
还是上面扩展了strA的extension，再创建一个category ClassA + test,调用前面的extension，然后main.m中调用category，创建对象a后，给a.strA赋值。


运行结果一样的报错信息，就不再贴图了，实验再次失败。


## 4.成功的尝试1---其实可以增加属性

在第一次尝试的基础上，我想到了平常使用的extension是在.m文件中的，那么我在.m中调用下该extension试试呢？在.m中调用该extension实质便和平常写在.m上面的extension一样了吧，同时还对外暴露了。
于是在.m中调用了该extension，再次按照第一次失败的尝试实验了一下，这时，控制台成功输出了

> 2015-11-24 02:47:31.993 extensionTest[6493:428470] a.strA = testA
Program ended with exit code: 0

于是实验成功，main.m和ClassA.m中分别调用该extension，main.m就可以对该extension声明的属性进行各种愉快的操作了。

## 5.成功的尝试2---还可以修改属性关键字

前面提到网上有帖子extension和匿名category的区别是

> 在extension中可以定义可写的属性，公有可读、私有可写的属性，而category做不到
就是说类的.h文件中的readonly属性，可以在extension中再次声明为readwrite属性，这样这个属性就对外只读，类的内部可读写了。
该博主用的自然是.m中的extension和创建在外面的category作的比较，
然后我把修改属性关键字的功能搬到了创建在外面的extension中，

```
//原类ClassA.h中
@property (nonatomic, copy, readonly) NSString * strB;
```

```
//extension  ClassA_test.h中
@property (nonatomic, copy, readwrite) NSString * strB;
```

```
//main.m中
 //2.可以通过extension更改property的关键字，达到修改原本为只读的属性，前提是原类也要调用这个extension
a.strB = @"testB";
NSLog(@"a.strB = %@",a.strB);
```
控制台输出结果如下：

> 2015-11-24 03:00:44.803 extensionTest[6538:441027] a.strB = testB
Program ended with exit code: 0

实验成功，不管是.m中的extension还是写在外面创建的extension都可以修改属性的关键字。而.m中的extension是私有的，外面的则可以按需要公开。

## 6.成功的尝试3---声明方法

既然可以通过这种方法添加属性，那么添加方法试试看？虽然只是个单独的.h文件，但毕竟原类的.m文件会调用这个extension，那么继续尝试。


```
//extension  ClassA_test.h中
- (void)log;
```
```
//原类  ClassA.m中，实现该方法
- (void)log{
    NSLog(@"testC");
}
```
```
//main.m中
//        3.可以调用extension中声明的方法，前提是原类要实现该方法；
 [a log];

```
控制台输出结果如下：

> 2015-11-24 03:06:19.802 extensionTest[6570:446715] testC
Program ended with exit code: 0

实验成功，可以在extension中声明方法，在.m中实现，外部类调用。

## 7.结论

那么，我们得出的结论是，在**类的.m文件调用了该extension的前提下**，我们可以给类扩展属性，可以更改属性关键字，可以扩展方法。
那么，有什么用呢?
.m中的extension作用是写私有属性，而在类的.h中声明的属性和方法都是公有的，那么，如果我们某些属性（包括读写性）和方法只对某些外部类公开，对其他类则不公开，我们就可以使用外部创建的extension，如果需要使用这些属性、读写性或方法的话，则调用该extension文件即可，否则就调用原类的.h文件。也许很少会用到，但某些场景一定会有用。
**注意！！使用外部创建的extension前提是一定要在原类的.m中调用！！！！否则方法无法实现！！！**
**也正因为必须在.m中调用，因此无法给官方框架类增加extension！**


## 8.补充---用category扩展属性的方法

前面提到，一般情况下，category只能扩展方法无法扩展属性因为category无存储空间。
但是我们可以通过一些非常规途径给category增加属性的set和get方法达到增加属性的目的，代码如下：

```
//category的.h文件中,声明属性
@property  (nonatomic,copy) NSString *urlStr;
```


```
//category的.m文件中，#import <objc/runtime.h>,并实现以下setter、getter方法
const char *key;
- (void)setUrlStr:(NSString *)urlStr
{
    // runtime 
    // 在运行时，给某个对象绑定一个值
    objc_setAssociatedObject(self, key, urlStr, OBJC_ASSOCIATION_COPY_NONATOMIC);
    
}

- (NSString *)urlStr
{
    // 获取这个值
    return objc_getAssociatedObject(self, key);
}

```
方法原型为：
`objc_setAssociatedObject(<#id object#>, <#const void *key#>, <#id value#>, <#objc_AssociationPolicy policy#>)`  
和 
`objc_getAssociatedObject(<#id object#>, <#const void *key#>)`

其中，object为传的对象，这里为self，key代表这个绑定的识别码，value则是绑定的对象，最后的<#objc_AssociationPolicy policy#>为一组枚举值，我把枚举值写出来大家就知道是什么意思了。

```
typedef OBJC_ENUM(uintptr_t, objc_AssociationPolicy) {
    OBJC_ASSOCIATION_ASSIGN = 0,           /**< Specifies a weak reference to the associated object. */
    OBJC_ASSOCIATION_RETAIN_NONATOMIC = 1, /**< Specifies a strong reference to the associated object. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_COPY_NONATOMIC = 3,   /**< Specifies that the associated object is copied. 
                                            *   The association is not made atomically. */
    OBJC_ASSOCIATION_RETAIN = 01401,       /**< Specifies a strong reference to the associated object.
                                            *   The association is made atomically. */
    OBJC_ASSOCIATION_COPY = 01403          /**< Specifies that the associated object is copied.
                                            *   The association is made atomically. */
};

```



其他相关内容，还在继续探索中，对于extension和category的使用，欢迎更多讨论。


##9.补充2---protocol也可以声明属性
之前一直觉得协议都是一组方法的声明，今天试了下声明属性，发现也是可以的，当然，也得在调用的类的.m中写上@synthesize。

``` 
//protocol test.h
#import <Foundation/Foundation.h>

@protocol test <NSObject>
@property (nonatomic,copy)NSString * str;
@end

```


```
//调用test协议的类的.m文件

#import "classA.h"

@implementation classA
@synthesize str = _str;
- (void)log{
    
    NSLog(@"%@",self.str);
}
@end

```

在main.m中调用该类给.str赋值并打印，成功。
protocol可以声明一个类可以声明的属性和方法，完成一个作为父类需要完成的事。所以OC中虽然不支持直接多继承，但可以通过协议实现多继承。


##10.补充3---千万不要用category重写方法！！！（重要的事加3个感叹号）
在很多博客都看到过一篇关于面试的文章，第一题就是『重写一个方法用类别好还是用继承好』，那个贴子给出的答案是用类别好，可能是博主疏忽打错了吧，但这里还是要强调下，千万不要用类别重写方法！这是件很危险的事！！！
先看看这几句简单的代码：

```
//main.m
#import <Foundation/Foundation.h>

int main(int argc, const char * argv[]) {
    @autoreleasepool {
        NSString * str = @"test";
        NSLog(@"%@",str);
    }
    return 0;
}
```
控制台打印结果如下：

> 2015-12-03 21:21:43.161 CategoryTest[28514:3291623] This is a trick!
Program ended with exit code: 0

main.m中我并没有写其他代码，也并没有导入任何头文件，然而结果却这么诡异。为什么呢？
是的，我的确没有导入任何其他文件，但是我写了一个NSString的类别，类别中重写了-description方法，返回了『 This is a trick!』这个字符串。

```
#import "NSString+CJTest.h"

@implementation NSString (CJTest)
- (NSString *)description
{
    return @"This is a trick!";
}
@end

```

然后，在整个工程中，只要我打印NSString的对象，输出的都只有这句话了，无论有没有导入这个类别的头文件。
所以，结论1：**不要用类别改写本来存在的方法！**

然后，我又做了个实验，给NSString添加两个不同的类别，里面写两个同名不同实现的方法，在main.m中import两个类别的头文件，并调用这个方法。。。。。结果是不可预知的（按编译的顺序）。。。
于是，结论2：**不同类别中不要存在同名方法！**

那么，为什么呢？
category本身是不带存储空间的（所以不能直接添加属性，因为set方法是需要存储空间的），category的实现机制，是在运行时，直接把方法添加在原来的类上的，如果存在同名类，添加就成了覆盖。所以category中的方法在运行时是全局的，只要调用了这个类，不管有没有import这个类别，都是调用的改写后的方法！！！
同理，如果同一个类的两个类别存在两个同名的方法（不论是新增还是改写），外部调用时实际也是调用这个类的方法，而这个类的方法是通过哪个类别实现的我们不知道，所以也会造成混乱！

另外，如果通过category改写了父类的方法，子类又改写了同样的方法，那么子类对象调用改方法时是按照哪个实现呢？
实验结果为按照子类的方法实现，这个也好理解。本身子类就是重写的父类的方法，无论是原本的方法还是通过category改写（或新增）的方法都是如此了。


**12.6补充原理**：前天看官方文档发现本篇讨论的内容其实官方文档都有解释，顺便也解释了原因，并从实现上区分了category和extension，category是运行时决议，而extension是编译时决议，因此extension必须在原类中实现而category不必。也因此category在运行时方法会覆盖掉原类中的同名方法。以下是官方文档相关原文：

>  At runtime, there’s no difference between a method added by a category and one that is implemented by the original class.

在运行时，通过分类添加的方法和原类中实现的方法没有任何区别。

> If the name of a method declared in a category is the same as a method in the original class, or a method in another category on the same class (or even a superclass), the behavior is undefined as to which method implementation is used at runtime.

如果一个分类中声明的方法和原类中的方法或同一个类的另一个分类（甚至父类中的分类）中的方法同名，在运行时会按照哪个方法实现是不确定的（实际上应该是有分类就按照分类，多个分类则按照编译顺序，以最后编译的为准，编译顺序可以在Build Phases-Compile Sources中查看和修改）。

> A class extension bears some similarity to a category, but it can only
> be added to a class for which you have the source code at compile time
> (the class is compiled at the same time as the class extension).
类扩展和分类有点像，但是类扩展只能在编译时加到你有源代码的类里（类和类扩展同时进行编译）。

证实了我的结论。