---
layout: post
title: "Programming with Objective-C"
date: 2014-11-07 12:39:00 +0800
comments: true
categories: iOS开发
---
Objective-C(以下简称OC)是C语言的超集，OC中添加了面向对象以及动态运行时的特性，它继承了C的语法特性，简单类型，以及控制条件，并添加了定义类和方法的语法。同时也对于对象图的管理和literal syntax上添加了语言级的支持，包括动态绑定以及推迟尽可能多的职责到运行时。  
<!-- more -->
##Defining classes
什么是对象呢？对象一组数据以及与与该组数据行为有关的一个package。  
什么是类呢？类就是对象的一个蓝图或者模版，我们可以按照模版这个生成对象。  
当你自己自定义一个类的时候，你应该考虑到哪些属性是允许被公开访问的，这些属性可以被改变吗？其他的一些对象怎么跟你的对象进行交互。  
类本身也是一种类型为Class的对象。  

##Working with objects
NSLog中的的%@用来表示一个对象，在运行时，该表示符会被该对象调用description方法返回的对象替代，descrption是NSObject中的一个方法，它是用来获取该对象的类以及内存地址的，但是很多Cocoa和Cocoa Touch类会重载该方法，从而返回一些更有用的信息，比如NSString 就会返回该字符串所代表的具体字符。  
 
 self其实是指向某个对象自身的一个指针，super通常用来表示某个对象调用父类中的方法，最常用的使用场景是当你需要在子类中重载某一个函数的时候。  
 
 当你想要创建一个对象的时候通常你第一件事是开辟足够的空间，当然不仅仅是为当前类的属性，同时也要为他所有父类中的属性开辟空间，而＋alloc方法正是做这件事儿的，同时＋alloc还有另外一件非常重要的事情要做，那就是清空它所开辟的内存空间，也就是将这块内存空间全部置为0。而init方法则是确保该对象中的属性在创建的时候都有正确的初始值。  
 
 当你创建一个对象的时候，如果不需要在初始化的时候传递参数，那么你可以使用new方法，这跟［Object alloc］init］是一样的。  
 
 某些类的对象可以才用Literals提供的一种更简洁的对象创建方式：  
 {%codeblock%}NSString *someString = @"Hello, World!";{%endcodeblock%}
 这其实跟：  
  {%codeblock%}NSString *someString = [NSString stringWithCString:"Hello, World!"  
                                              encoding:NSUTF8StringEncoding];{%endcodeblock%}
效果是一样的。  

类似的还有:  
{%codeblock%}
NSNumber *myBOOL = @YES;  
NSNumber *myFloat = @3.14f;  
NSNumber *myInt = @42;  
NSNumber *myLong = @42L;  
{%endcodeblock%}
 
 对于指针来说，＝＝用于测试两个指针指向的是否是同一个地址，如果想要测试两个对象代表的是否是相同的数据，那么就使用isEqual方法，而对于NSNumber,NSDate,NSString对象，在比较的时候不能使用>和<，而应该采用compare方法。  
 
 对于基本变量，在声明的时候最后初始化，因为他们可能包含之前stack中的垃圾。而对于对象指针则没有必要，因为编译器会自动将它们置为nil如果你没有赋初值的话，在OC中对于nil发送消息是安全的，事实上，这什么也不会发生，而如果你对于一个nil变量发送消息来return，那么对于对象类型返回nil，对于数字类型返回0，对于bool型返回NO,而对于结构体则所有的内存空间都为0.  
 
 对于对象变量，如果为nil，那么他的逻辑值则为0（false），反之为true。  

##Encapsulating Data  
面向对象编程中一个主要的原则是封装性，因此我们应该使用一个类暴露给我们的接口来访问该类内部的值，而不是试图通过直接获取类内部的值。  

对于readonly类型的属性，编译器并不会自动生成setter方法。  

默认情况下，一个readwrite属性被一个实例变量所支持的，而该实例变量的名字跟属性的名字是一样的，只不过前面多了一个下划线。

实例变量的内存是在一个类调用alloc方法的时候，是在该对象被dealloc的时候销毁的。  

通常情况下，你应该使用self. 或者[self setXXX:]的方式来访问一个该类的属性。  

当然你也可以给一个属性所对应的变量取一个别名，例如：@synthesize firstName = ivar_firstName;这样的话，该属性的setter和getter使用的名字仍然是firstName，而实例变量采用的名字则是ivar_firstName，而你最好不要使用@synthesize firstName这样的，因为这样的话，实例变量的名字就跟属性的名字是一样的了。  

最好的方式是任何时候当你想追踪一个值或者对象的话，都使用属性，当然你也可以直接声明一个实例变量，最好带上下划线。  


初始化的时候，你应该直接使用实例变量，因为在属性设置的时候，该对象可能还没有完全初始化。  

#####atomic VS nonatomic
atomic是默认的属性修饰符。对于atomic修饰的属性，可以保证同一时刻某个属性的setter和getter方法都能够完整的执行而不会被打断，atomic本身就是原子性的意思，对于多线程而言，这意味着如果两个线程都对执行同一个属性执行setter方法，那么同一时间只能等待一个线程中该属性的setter执行完毕后，另一个线程中该属性的setter才能继续执行，当然对于getter也是同样的道理，但是，这并不意味着atomic修饰的属性是线程安全的，比如某一个对象包含firstName和lastName两个属性，然后，我在线程A上对该对象的firstName和lastName执行setter方法，与此同时，我在线程B上对该对象的firstName和lastName同时执行getter方法，这样处理之后得到的fullName就不一定是正确的了，因为有可能出现的情况时，firstName的获取是在线程A中firstName的setter方法执行前，而lastName的获取则是在线程A中lastName的setter执行后，这就有可能导致错误的姓名搭配。 

而对于nonatomic修饰的属性，则不保证setter和getter操作是原子的，它只是简单的set或者返回一个值，并不保证同一个属性在不同的线程中进行访问的话会出现什么情况，换句话说，每个线程在执行setter的时候，都不会关心其他线程正在做什么，它只是简单的获取到当前要进行访问的变量，然后对其进行set或者返回值，比如两个线程都对Person类中的age进行+1操作，那么两者本来应该＋2，但可能最终的结果仍然是＋1。  

由于上述原因，当你在访问nonatomic属性的时候，速度要比atomic属性更快一些。

另外，sythesized accessors的实现是私有的，因此对于atomic类型的属性，你不能自己实现setter，否则你将会得到一个警告。  

#####Memory
当在考虑内存的问题的时候，你应该从引用的角度去考虑，如果一个对象始终被另外一个对象所引用，那么他所占用的内存就永远不会释放。而对于内存问题，两个比较容易出现的问题就是：  

1. 正在使用的内存已经被释放。
2. 内存已经不使用了，但是没有释放，导致内存泄漏。  

同时，你应该避免循环引用的出现，其实就是，一个对象A对另一个对象B保持强引用，同时，该B也对A保持强引用，这样，即便这个group的外界对两者都没有引用关系了，两者因为互相保持引用而没办法释放内存，导致内存泄漏。  

__srong 是默认的变量修饰符，不用显示说明。  

对于无法使用weak的环境，你应该使用__unsafe_unretained来修饰变量，这个修饰符的功能跟weak类似，唯一的区别就是它并不会在该内存已经被释放的时候将该指针设置为nil，因此该指针就变成了悬垂指针，而对一个悬垂指针发送消息有可能会导致程序崩溃，这也就是unsafe的原因。  

##Customing Existing Classes
你可以对任何一个类添加categories，包括自定义类和cocoa touch的类，添加之后所有该类以及该类的子类都可以调用添加的方法，事实上在运行时，在一个类的category中添加的方法和该类本身的方法是没有任何区别的。  

你不仅可以使用categories向已有类中添加方法，你也可以使用categories来将一个复杂类的逻辑分散到各个不同的实现文件中去。  

你可以使用categories来向类中添加方法，但是你最好不要使用categories来添加属性，因为虽然语法上没有什么错误，但是你并不能在categories中声明实例变量，这也就意味着编译器既不会sythesize实例变量，也不会sythesize出getter和setter方法，虽然你可以手动添加getter和setter方法，但你却无法追踪实例变量的值，除非它已经存储在已有类中。而这样的功能可以通过extension来实现。  

在使用categoty的时候，应该尽量避免名字冲突的问题，事实上，对于你自己的定义的类可能会很少出现category中的方法名跟该类中的方法名一样的情况，但是对于cocoa touch中的方法，如果一旦名字发生冲突，那么在运行时具体执行哪个方法是不明确的，这就有可能导致问题的出现，最好的办法是在你添加的方法中添加一个前缀和下划线，类似：  
{%codeblock%}@interface NSSortDescriptor (XYZAdditions)
+ (id)xyz_sortDescriptorWithKey:(NSString *)key ascending:(BOOL)ascending;
@end
{%endcodeblock%}

Extension通常被称作是匿名类，它跟category的作用类似，但不同的是：  

1. 它只能添加到source code中，而它的具体实现则是在@implementation中，这也就意味着你不能对cocoa的类添加extension。  
2. 你可以在其中添加实例变量和属性就如同正常类中添加的一样，编译器会自动合成相应的实例变量，getter以及setter方法。  

一个类的interface中，我们通常声明一些可以共其他类来访问和调用的属性和方法，换句话说，在一个类的interface中，我们暴露的是公开的信息，通常我们可以使用Class Extension来隐藏私有变量和私有方法。  

我们可以在interface中定义一个readonly的属性，然后在extension中将其重新定义为readwrite的，这样的话，在类的内部，我们就可以对其read和write，而在类的外部，也就是其他的对象不能对其进行set，这样就很好的实现了封装性。  

同时你也可以考虑其他的一些替代方法用于Class Customization:  

1. 设计一个子类。  
2. 使用delegate，这个其实是将决策权放到运行时，比如UITableView，为了实现重用就使用了delegate。  
3. 直接使用OC的runtime，OC可以在runtime system中使用动态的行为，例如我们可以使用 Associative References，但是跟使用extension不同的是，使用这种方法并不会影响任何的原始类的定义和实现，这也就意味着对于那些framework中你没有权限访问的代码你可以使用这种方法， Associative References会将一个对象跟另一个对象关联起来，有点像属性或者实例变量。  

##Working with Protocols
与类相反，protocol是用来声明独立于任何特定的class的属性和方法的，其中既可以包含类方法，也可以包含实例方法，同时也能包含属性。  

和类一样，你的protocol也可以继承另外一个protocol，这样的话，意味着所有adopt你的protocol的类，也同样adopt该protocol中的方法。

当你发现你的某一个类adopting非常多的协议，那可能意味着将这个类按照责任去重构成比较小的类了。   

我们常见的datasource和delegate本质上是一样的，只不过分工不同。  

protocols可以用作匿名，比如，有些时候你并不知道一个对象的类型或者说这个类型需要保持隐藏状态。

这样的话，使用protocols，即使你不知道某个对象是什么类型的，你也能明确的知道该类能响应某个方法，从而去调用该方法。

##Values and Collections
在使用Cocoa编程过程中，尽量使用NSInteger,NSUInteger,CGFloat这样的类型，因为这样可以使得代码的兼容性更好，因为这样的变量在定义的时候会根据具体的环境是32位还是64位来定，而对于局部变量，例如for循环中的counter，在你知道counter的值在限制范围内的时候，你可以使用C中的基本类型。  

NSNumber 也可以使用 Literal syntax的方式来创建：  
{%codeblock%} 
 NSNumber *magicNumber = @42;
    NSNumber *unsignedNumber = @42u;
    NSNumber *longNumber = @42l;
    NSNumber *boolNumber = @YES;
    NSNumber *simpleFloat = @3.14f;
    NSNumber *betterDouble = @3.1415926535;
    NSNumber *someChar = @'T';
 {%endcodeblock%}  
 
 NSNumber是NSValue的子类，NSValue可以用来存放指针和结构体，例如：  
 {%codeblock%} 
  NSString *mainString = @"This is a long string";
    NSRange substringRange = [mainString rangeOfString:@"long"];
    NSValue *rangeValue = [NSValue valueWithRange:substringRange];
 {%endcodeblock%} 

所有的集合类，例如NSArray都会对其内容保持强引用。  

NSArray中的元素，不是必须都是同样的类型。  

当你想要在集合存放一个空指针的时候，你应该使用NSNull类型，而不是nil，事实上，使用nil，就会导致异常，NSNull是一个单例类。  

NSSet和NSArray类似，唯一的区别是NSSet是无序集合，所以在测试membership的时候，NSSet会有性能提升。  

NSDictionary的键也可以使用除了NSString类型之外的其他类型，但是由于NSDictionary会对每一个key做一份copy，这样你就得保证你使用的key必须支持NSCopying协议。  

对于NSArray和NSDictionary你可以使用writeToURL将其持久化到本地磁盘中，同时如果其中的每一个元素都是property list types（例如，NSArray,NSDictionay,NSString,NSNumber,NSDate,NSData,NSNumber），你可以通过arrayWithContentsOfURL方法重新创建。  

而如果你想持久化其他类型而不是property list types，那么你可以使用NSKeyedArchiver来创建archive来collected objects，但你要集合中的每一个元素都支持NSCoding协议，这也就意味着每一个元素都必须知道如何encode itself（通过实现encodeWithCoder:），并且知道如何从已知的archive中decode itself（通过实现initWithCoder:）。  

事实上，如果你使用Interface Builder的来布局windows和views的话，那么最终的nib文件只不过是一个你使用视觉化的创建方式创建的层级结构的archive，而在运行时，这个nib文件就会使用相关类unarchive成层级结构。  

虽然你可以使用for循环的方式来枚举collection中的每一个元素，但最好的方式是通过使用：  
 {%codeblock%} for (<Type> <variable> in <collection>) {
        ...
    }{%endcodeblock%}

多数的集合也支持Enumerator Objects,使用方法如下： 
 
{%codeblock%} for (id eachObject in [array reverseObjectEnumerator]) {
        ...
    }{%endcodeblock%}

{%codeblock%} id eachObject;
    while ( (eachObject = [enumerator nextObject]) ) {
        NSLog(@"Current object is: %@", eachObject);
    }{%endcodeblock%}  
    
多数的集合也支持Block-Based Enumeration。 
