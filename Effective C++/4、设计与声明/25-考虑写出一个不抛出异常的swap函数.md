# 条款25-考虑写出一个不抛出异常的`swap`函数
---
+ `swap`函数原来只是STL的一部分，然而后来却成为了异常安全性编程(见[[29-为“异常安全”而努力是值得的]])的脊柱，以及用来处理自我赋值的可能性(见[[11-令operator=中处理“自我赋值”]])。

+ 传统的`swap`函数实际上将两个对象的值彼此赋予对方。
```cpp
namespace std {
	template<typename T>
	void swap(T& a, T& b) {
		T temp(a);
		a = b;
		b = temp;
	}
}
```

只有类型`T`支持`copying函数`(`copy构造函数`和`copy assignment操作符函数`)，以上的实现方式才有作用。然而其效率却是奇慢的。

+ 提高`swap`函数的一个重要方法就是通过指针的方式，“**以指针指向一个对象，内含真正数据**”，这种设计的常见表现形式就是“**pimpl**”手法(`pointer to implementation`，见[[31-将文件间的编译依存关系降至最低]])
```cpp
class WidgetImpl {
public:
	...
private:
	int a, b, c;
	std::vector<double> v;
	...
};
class Widget {
public:
	Widget(const Widget& rhs);
	Widget& operator=(const Widget& rhs) {
		...
		*pImpl = *(rhs.pImpl);
		...
	}
	...
private:
	WidgetImpl* pImpl; //指向Widget数据
};
```

关于`operator=()`的实现细节可以复习[[10-令operator=返回一个reference-to-*this]]，[[11-令operator=中处理“自我赋值”]]，[[12-复制对象时勿忘其中每一成分]]。

此时要置换两个`Widget`的值，只需要置换其中的`pImpl`指针，然而STL中的`swap`却并不知道，因此需要去将`std::swap`只对`Widget`特化。
```cpp
namespace std {
	template<>                                    //std::swap针对
	void swap<Widget>(Widget& a, Widget& b) {     //T是Widget的特化版本
		swap(a.pImpl, b.pImpl);                   //目前还不能通过编译
	}
}
```

通常而言我们不能去改变`namespace std`命名空间中的任何东西，但可以为标准`template`制造特化版本，使他专属于我们自己的`class`。

以上的版本并不能通过编译，他想直接访问`private`的对象，显然是不合理的。遵循着封装的面向对象原则，能不用`friend`就不用`friend`。我们可以在`Widget`的内部声明一个`swap`的`public`成员做置换工作，然后将`std::swap`特化
```cpp
class Widget {
public:
	...
	void swap(Widget& other) {
		using std::swap;
		swap(pImpl, other.pImpl);
	}
};
namespace std {
	template<>
	void swap<Widget>(Widget& a, Widget& b) {
		a.swap(b);
	}
}
```

这个版本不仅能通过编译，还非常优秀地和STL容器有一致性，因为所有STL容器也都供有`public swap`成员和`std::swap`特化版本。

+ 若`Widget`和`WidgetImpl`都是`class template`而非`class`，会有人尝试将`WidgetImpl`内的数据类型加以参数化。在`Widget`内加入一个`swap`成员是简单的，然而想特化`std::swap`却没那么简单
```cpp
template<typename T>
class WidgetImpl {...};
template<typename T>
class Widget {...};
...
namespace std {
	template<typename T>
	void swap<Widget<T>>(Widget& a, Widget& b) {
		a.swap(b);
	}
}
```

看起来也之前的实现方法大致相同，实际上确实不合法的。C++允许我们去特化`class template`，却不允许我们去特化`function template`。即使有些编译器错误的接受了这段代码，却有可能将你和你的程序带入未定义行为的坑里面。

+ 实际上，当你打算去特化一个`function template`，你可以很简单为其添加一个重载版本。
```cpp
namespace std {
	template<typename T>
	void swap(Widget<T>& a, Widget<T>& b) {
		a.swap(b);
	}
}
```

一般而言，重载`function template`没有问题，但是`namespace std`是个特殊的命名空间，其管理规则也比较特殊。客户可以全特化里面的`template`，却不可以添加新的`template`到`namespace std`内。

+ 为了达成目的，我们声明一个`non-member swap`让他调用`member swap`，但是却不能将其声明为`std::swap`的特化版本或重载版本，这也是为什么上述代码的`swap`后面没有`<...>`。同时，为求简化，最好将`Widget`相关的所有功能都放在同一个命名空间内。
```cpp
namespace WidgetStuff {
	...
	template<typename T>
	class Widget {...};
	...
	template<typename T>
	void swap(Widget<T>& a, Widget<T>& b) {
		a.swap(b);
	}
}
```

以上代码已经近乎是最终版本了，若你想你的`swap`函数在尽可能多的地方都能被正确调用，你需要在`class`所在命名空间下同时写下一个`non-member`版本及一个`std::swap`特化版本。若你不使用额外的`namespace`便不需要考虑，但是为什么不呢，为何要在`global`的命名空间内塞下一堆东西。
```cpp
template<typename T>
void doSomething(T& obj1, T& obj2) {
	using std::swap;
	...
	swap(obj1, obj2);
	...
}
```

现在编译器能够去自动寻找最合适的`swap`进行调用了。C++的名称查找法则，确保找到`global`作用域或`T`所在的命名空间内的专属`swap`。若`T`为`Widget`，则会寻找`namespace WidgetStuff`内的`swap`，若找不到其专属的`swap`，则会使用`std::swap`。若在`namespace std`中存在特化版的`swap`，编译器会偏向选择特化版的`swap`进行调用。

+ 现在对`default swap`、`member swap`、`non-member swap`、`std::swap`特化版本、对`swap`的调用做一个总结。
	+ 当你不满`std::swap`的效率时，可以尝试做以下的事情：
		1. 提供一个`public swap`成员，让他高效的置换你的类型
		2. 在你的`class`或`template`的命名空间内提供一个`non-member swap`，并令其调用对象内部的`swap`
		3. 如果你正编写一个`class`，而非`class template`，为你的`class`特化`std::swap`。并令他调用对象内部的`swap`
		4. 当你调用`swap`时，先包含一个`using std::swap`声明式，将`std::swap`在你的函数内部被暴露，然后再不加任何`namespace`修饰符，直接调用`swap`。

+ 需要注意的是，无论如何，成员版`swap`都不该抛出异常，因为`swap`有个重要的作用就是提供异常安全性的保障([[29-为“异常安全”而努力是值得的]])，但这一约束仅限`class`内的成员`swap`。

+ 小结：
	+ 当`std::swap`对你的类型效率不高时，提供一个`swap`成员函数，并确定这个函数不抛出异常。
	+ 若你提供一个`member swap`，也该提供一个`non-member swap`用来调用前者。对`class`(非`class template`)，也请特化`std::swap`。
	+ 调用`swap`时，应针对`std::swap`调用`using std::swap`声明式，之后在调用`swap`并且不带任何命名空间修饰。
	+ 对用户定义类型进行`std template`全特化是好的，但不要尝试在`namespace std`中加入某些对`namespace std`而言是全新的东西。