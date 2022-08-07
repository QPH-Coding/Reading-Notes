# 条款20-宁以`pass-by-reference-to-const`替代`pass-by-value`
---
+ 缺省情况下，C++以`pass-by-value`的方式传递对象至函数。除非你另外指定，这样的函数参数都是以实参的副本为初值的。这些副本由对象的copy构造函数产出，这会使得`pass-by-value`成为一种费时的操作
```cpp
class Person {
public:
	Person();
	virtual ~Person();
private:
	std::string name;
	std::string address;
};
class Student: public Person {
public:
	Student();
	~Student();
private:
	std::string schoolName;
	std::string schoolAddress;
};
...
bool validateStudent(Student s);
void f() {
	Student Tom;
	bool TomIsOk = validateStudent(Tom);
}
```

仔细观察上述例子，我们很高兴看到的是，在`base class`中的析构函数用了`virtual`(见[[07-为多态基类声明virtual析构函数]])。现在，我们再次把目光放在本条款相关内容上，在本例中一次调用`validateStudent()`，最终经历了`class Student`的构造过程、`class Person`的构造过程，别忘了还有四个`std::string`的构造过程，最终在退出函数的栈时，又调用了以上6个对象的析构函数，也就是说，在本次调用中，额外付出了6次构造和析构函数的成本。

+ 如何避开这6次成本，很简单，就是用`pass-by-reference-to-const`的方式传递参数。
```cpp
bool validateStudent(const Student& s);
```

这样的传递方式效率高出不少：没有任何对象被创建，因而不用额外付出任何调用构造和析构函数的成本。同时将其声明为`const`也是必要的，因为不这样做调用者会忧虑是否会改变传入的对象。

+ `pass-byreference-to-const`可以避免一个“**slicing**(对象切割)问题”。当一个`derived class`以 `pass-by-value`的方式传递并被视为一个`base class`对象，那么最终传递到函数内仅剩一个copy的`base class`。这种情况大概率不是你想要见到的。
```cpp
class Window {
public:
	std::string name() const;
	virtual void display() const;
};
class WindowWithScrollBars: public Window {
public:
	...
	virtual void display() const;
};
void printNameAndDisplay(Window w) {
	std::cout << w.name() << std::endl;
	w.display();
}
void f() {
	WindowWithScrollBars wwsb;
	printNameAndDisplay(wwsb);
}
```

本例中，所有WIndow对象都一个名称，可通过`name()`去获取。所有窗口都可以显示，通过`display()`去完成。`display()`是个`virtual`函数，意味着`base class Window`的显示方式和`derived class WindowWithScrollBars`的显示方式不同(见[[34-区分接口继承和实现继承]]和[[36-绝不重新定义继承而来的non-virtual函数]])。
在`f()`中，参数`wwsb`在传递进去后，会构造出一个`base class Window`对象出来。因此调用的总是`Window::display()`而非`WindowWithScrollBars::display()`。

我们可以通过`pass-by-reference-to-const`很好的解决这个问题。
```cpp
void printNameAndDisplay(const Window& w) {
	std::cout << w.name() << std::endl;
	w.display();
}
```

+ 从C++底层的角度出发，`reference`是以指针实现的，因此`pass-by-reference`意味着传递指针。因此你有个对象属于内置类型(如`int`)，`pass-by-value`往往比`pass-by-reference`效率高一些。这个道理同样适用于STL的迭代器和函数对象，在[[01-视C++为语言联邦]]中早已有讨论。

+ 一般而言，可以合理假设`pass-by-value`不昂贵的对象就只有内置类型和STL的迭代器和函数对象。至于其他东西，都尽量遵循本条款，以`pass-by-reference-to-const`替代`pass-by-value`。

+ 小结：
	+ 尽量以`pass-by-reference-to-const`替代`pass-by-value`。前者通常更加高效，并且可以解决**Slicing对象切割问题**
	+ 以上规则并不适用于内置类型，以及STL的迭代器和函数图像。对他们而言，`pass-by-value`往往比较恰当。