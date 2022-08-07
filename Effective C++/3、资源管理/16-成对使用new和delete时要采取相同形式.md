# 条款16-成对使用`new`和`delete`时要采取相同形式
---
+ 当你使用`new`时，会有两件事发生(见[[49-了解new-handler的行为]]和[[51-编写new和delete时需固守常规]])：一、内存被分配出来；二、针对这块内存会有一个或多个构造函数被调用；当你使用`delete`时，也会有两件事发生(见[[51-编写new和delete时需固守常规]])：一、针对这块内存会有一个或多个析构函数被调用；二、这块内存最终得到释放。其中`delete`最大的问题就在于：“**即将被删除的内存之内究竟存有多少对象？**”，这个问题决定了会有多少个析构函数被调用。

+ 当我们简化上述问题，可以发现这个问题实际上就是：“**即将被删除的指针，所指的是单一对象还是对象数组？**”。

+ 一般而言，对象在内存中如下：
![[Pasted image 20220707210330.png]]
	而对象数组在内存中如下：
![[Pasted image 20220707210139.png]]
大多编译器都是这样实现的。

+ 当你对着一个指针使用`delete`时，唯一能够让编译器知道内存中是否存在一个“数组大小记录”的办法就是：你去告诉他。当你使用`delete[]`时，他便认为指针指向一个数组；当你使用`delete`时，他便认为指针指向一个对象。
```cpp
void f() {
	std::string* stringPtr1 = new std::string;
	std::string* stringPtr2 = new std::string[100];
	...
	delete stringPtr1;
	delete[] stringPtr2;
}
```

+ 若对`stringPtr1`使用`delete[]`会发生什么事暂且是未定义的(我记录笔记时顺带一试，LSP-Language-Support-Server会给出“WARMING”，编译正常通过，然而实际运行时会报出“**invalid pointer**”的错误导致程序提前结束)。此书的作者给出的猜测是会将此内存的数据读取为数组大小，并多次调用析构函数。

+ 若对`stringPtr2`使用`delete`会发生什么事也是未有定义的。此书作者也猜想可能导致太少析构函数被调用。

+ 实际上概括起来十分简单：**当你调用时用了`new[]`，你必须对应的调用`delete[]`；当你调用时用了`new`，你必须对应的调用`delete`。**

+ 当你设计的类中涉及动态分配内存，并提供多个构造函数时，上述规则尤为重要。你需要在每个构造函数中给出`new`或者`new[]`，在析构函数中给出对应的`delete`和`delete[]`。

+ 上述规则对于喜欢是用`typedef`的人也很重要，因为它意味者`typedef`的作用必须说清楚，当程序员以`new`创建该种`typedef`对象时，该以那一种`delete`形式删除之。
```cpp
typedef std::string AddressLines[4]; //每个人地址有4行
void f() {
	std::string* pal = new AddressLines; //这里等同于new string[4]
	...
	delete pal; //ERROR!!!
	delete[] pal; //GOOD!!!
}
```

为了避免此类错误，最好不要对`typedef`使用`new`。毕竟C++的STL部分含有`vector`，`string`等`template`，此例中可用`vector<string>`替代。

+ 小结：
	+ 当你调用时用了`new[]`，你必须对应的调用`delete[]`；当你调用时用了`new`，你必须对应的调用`delete`。