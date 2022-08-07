# 条款17-以独立语句将`newed对象`置入智能指针
---
+ 本条款以一个实际案例去引入
```cpp
int priority();
void processWidget(std::tr1::shared_ptr<Widget> pw, int priority);
```

这里牢记[[13-以对象管理资源]]，`processWidger()`对动态分配来的`Widget`资源运用智能指针进行管理。
```cpp
void f() {
	//考虑调用
	processWidget(new Widget, priority()); //ERROR!!!
	//std::tr1::shared_ptr的构造函数是explicit显式的，因此不能隐式转换
	...
	processWidget(std::tr1::shared_ptr<Widget>(new Widget), priority());
	//正确的调用
}
```

这行代码太长了，干了太多事情，让我们来缕一缕他在进入函数主体之前干了什么：
+ 执行`new Widget`
+ 调用`std::tr1::shared_ptr<Widget>`的构造函数
+ 调用`priority()`

我们可以确定的是：`new Widget`一定在调用`std::tr1::shared_ptr<Widget>`构造函数之前。毕竟其结果需要传递给智能指针的构造函数，但是对于调用`priority()`这一步骤，他是自由的，最终编译器会把他放在哪一步我们不得而知，但是最坏的结果一定是放在`new Widget`和调用`std::tr1::shared_ptr<Widget>`构造函数之间。

还记得我们在[[13-以对象管理资源]]中所说：“**资源取得的时机便是初始化时机(RAII)**”。很明显以上的做法在一定程度上隐性地破坏了这个准则。假如调用`priority()`的顺序在中间，那么若在调用`priority()`的过程中出现异常，之前动态分配的一块内存就注定造成内存泄露问题。

+ 解决的方法也十分简单：分离语句，认为定下调用顺序即可
```cpp
void f() {
	std::tr1::shared_ptr<Widget> pw(new Widget);
	processWidget(pw, priority());
}
```

+ 小结：
	+ 以独立语句将`newed对象`存储于智能指针内。如果不这样做，一旦异常被抛出，有可能导致难以察觉的资源泄漏。