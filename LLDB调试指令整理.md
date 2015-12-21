lldb是Xcode自带的一个功能强大的调试器，然而很多初学者并没有很好地利用这一工具，可能就用到几个常用的指令。这里我为大家总结了一些常用指令和调试技巧，欢迎补充。
##一、基本指令
###1.帮助
#### help 可以列出所有指令及用法说明。
#### help + 指令名，可以列出指定指令的说明，如 help image

###2.打印
#### p 打印非对象变量
	
 ![p a:打印a的值](http://img.blog.csdn.net/20151208114201633)
		 
	其中$0为变量在lldb的引用名，后面在lldb中调用这个变量，可以直接用变量名a,也可以用a的地址,也可以用a的引用名。p也可以换成print.

#### p/(格式) 按格式打印非对象变量
	
	包括p/x(按十六进制打印),p/o(按八进制打印),p/t(按二进制打印),p/c (char) (打印对应的c字符)，p/d(按int格式打印),p/f (float) (按float打印),
	
####po 按照对象的description打印对象

	po指令是用得最多的指令，按照对象的description的格式打印对象
	


###3.操作
####expr 修改变量的值或声明新的变量：
![expr的操作](http://img.blog.csdn.net/20151208131453235)

	注意：expr在lldb中声明新的变量时，需要加上$符号，表示引用。

####call 调用方法，(不立刻生效，程序往下执行才生效)

```
(lldb) call [(UIView *)self.view setBackgroundColor:[UIColor redColor]]
```
###4.流程控制（依次对应下方四个流程控制按钮）
####c (continue)  继续
####n (next) 下一步
####s (step in) 进入
####f (finish) 跳出

###5.断点（breakpoint）
Xcode本身也有加断点功能，也很方便，但lldb加断点功能比Xcode直接加断点强大很多。
####给某一行加断点

```
(lldb) breakpoint set --file ViewController.m --line 26
```
####给某个方法加断点

```
(lldb) breakpoint set --selector viewDidLoad
```
####给某个库加断点

```
(lldb) breakpoint set --shlib libc++.dylib --name libc++
```
####查看所有断点

```
(lldb) breakpoint list
```

##二、高级指令
###1.image
####寻址

```
(lldb) image lookup --address
```

在程序发生崩溃时，查看指定内存地址，如果为发生崩溃所在位置则打印所在文件和行数：如：

>异常如下：
> *** First throw call stack:
(
	0   CoreFoundation                      0x000000010d2baf45 __exceptionPreprocess + 165
	1   libobjc.A.dylib                     0x000000010cd34deb objc_exception_throw + 48
	2   CoreFoundation                      0x000000010d1a9b14 -[__NSArrayI objectAtIndex:] + 164
	3   WechatShareSDK                      0x000000010c49ca58 -[ViewController shareBtClicked:] + 680
	4   UIKit                               0x000000010d72ae91 -[UIApplication sendAction:to:from:forEvent:] + 92
	5   UIKit                               0x000000010d8964d8 -[UIControl sendAction:to:forEvent:] + 67
	6   UIKit                               0x000000010d8967a4 -[UIControl _sendActionsForEvents:withEvent:] + 311
	7   UIKit                               0x000000010d8958d4 -[UIControl touchesEnded:withEvent:] + 601
	8   UIKit                               0x000000010d798ed1 -[UIWindow _sendTouchesForEvent:] + 835
	9   UIKit                               0x000000010d799c06 -[UIWindow sendEvent:] + 865
	10  UIKit                               0x000000010d7492fa -[UIApplication sendEvent:] + 263
	11  UIKit                               0x000000010d723abf _UIApplicationHandleEventQueue + 6844
	12  CoreFoundation                      0x000000010d1e7011 __CFRUNLOOP_IS_CALLING_OUT_TO_A_SOURCE0_PERFORM_FUNCTION__ + 17
	13  CoreFoundation                      0x000000010d1dcf3c __CFRunLoopDoSources0 + 556
	14  CoreFoundation                      0x000000010d1dc3f3 __CFRunLoopRun + 867
	15  CoreFoundation                      0x000000010d1dbe08 CFRunLoopRunSpecific + 488
	16  GraphicsServices                    0x000000010ff87ad2 GSEventRunModal + 161
	17  UIKit                               0x000000010d72930d UIApplicationMain + 171
	18  WechatShareSDK                      0x000000010c4a049f main + 111
	19  libdyld.dylib                       0x000000010ed8892d start + 1
	20  ???                                 0x0000000000000001 0x0 + 1
)

	因为之前是在ViewController的 shareBtClicked方法里崩溃的，因此尝试用`image lookup --address  0x000000010c49ca58`指令查看，结果如下：
	

> (lldb) image lookup --address  0x000000010c49ca58
      Address: WechatShareSDK[0x0000000100001a58] (WechatShareSDK.__TEXT.__text + 776)
      Summary: WechatShareSDK`-[ViewController shareBtClicked:] + 680 at ViewController.m:49

按照返回的结果，在ViewController.m的第49行，果然找到了异常的原因：

```
48    NSArray *arr=@[@(1),@(2)];
49    NSLog(@"%@",arr[2]);
```
####查看指定类的详情：

```
image lookup --type 
```

####列出工程中用到的所有库

```
image list
```

###2.添加观察点watchpoint：
	相当于在lldb中添加观察者，被观察值变化时则中断。
```
watchpoint set var
```
比如我在一个循环中对循环控制变量i加个观察点：

```
(lldb) watchpoint set var i
```
这样，循环继续进行，控制台输出如下：

```
Watchpoint created: Watchpoint 1: addr = 0x7fff5db36c3c size = 4 state = enabled type = w
    declare @ '/Users/chenjia/代码/WechatShareSDK/WechatShareSDK/ViewController.m:33'
    watchpoint spec = 'i'
    new value: 0

Watchpoint 1 hit:
old value: 0
new value: 1
```


以上是lldb自带的一些调试指令，灵活运用将会大大方便我们的调试。但是系统自带指令并没有关于UI调试的，实际开发中我们整天与UI打交道，目前经常做的就是用Xcode自带的Debug View Hierarchy观察图层，然而，利用chisel插件，我们可以用lldb做更多事情。

##二、chisel
###1.安装chisel
chisel是facebook用python写的一个lldb插件，github地址为：https://github.com/facebook/chisel，安装方法如下：
####（1）打开终端，安装brew：

```
curl -LsSf http://github.com/mxcl/homebrew/tarball/master | sudo tar xvz -C/usr/local --strip 1
```
#### （2）更新brew，然后下载chisel

```
brew update
brew install chisel
```
#### （3）安装 Xcode 的 Command Line Tools:

```
xcode-select --install
```
#### （4）注入脚本

```
cd ~
vim .lldbinit  
```

在里面加如下一行，然后保存退出：

```
command script import /usr/local/opt/chisel/libexec/fblldb.py

```
重启Xcode。

###2.常用指令：
	chisel的指令都很简单很好用，就不一个个举例了。
####pviews  按层级打印所有视图信息
####pvc 打印当前视图控制器和视图信息
####visualize 显示指定视图控件或图片，可以指定地址或变量名
####fv 打印指定类型视图信息：如`fv button`
####fvc 打印指定类型视图控制器
####hide/show 隐藏/显示指定视图
####border/unborder 显示/隐藏边框 如`border -c red -w 2`
####pinternals 打印指定视图内部结构
####presponder 按层级打印消息链:(消息链顺序为由内到外，响应链顺序为由外到内)
####pclass 打印一个对象的继承关系
####taplog 模拟敲击屏幕，并打印接收tap动作的对象
