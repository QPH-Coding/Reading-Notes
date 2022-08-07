# 条款10-令`operator=`返回一个`reference *this`
---
+ 为何要令`operator=()`返回一个`reference *this`，最关键的地方在于一个名为“链式编程”的思想
```cpp
int x, y, z;
x = y = z;//这个就叫链式编程
```

+ 为了实现“链式编程”，通常会让赋值运算符返回一个`renference *this`
```cpp
class Widget {
public:
	...
	Widget& operator=(const Widget& rhs) {
		...
		return *this;
	}
	...
};
```

以上的操作不仅仅适用于赋值运算符，同时适用于所有赋值相关的运算
```cpp
class Widget {
public:
	...
	Widget& operator=(const Widget& rhs) {
		...
		return *this;
	}
	Widget& operator+=(const Widget& rhs) {
		...
		return *this;
	}
};
```

+ 实际上，这只是个大家通用且习惯的协议，并没有什么强制性，若不遵循它，代码一样能够正常的通过编译。然而，当你有机会将一个类当作内置类型设计时，不要放过它(见[[19-设计class犹如设计type]])。

+ 小结：
	+ 令赋值运算符返回一个`reference *this`