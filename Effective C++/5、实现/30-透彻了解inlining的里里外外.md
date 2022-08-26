# 条款30-透彻了解inlineing的里里外外
---
- `inline`函数，看起来像函数，动作像函数，比宏好很多(见[[02-尽量以const，enum，inline代替define]])，同时不必带来调用函数所招致的额外开销。

- `inline`的原理是对每一个此函数的调用都以其函数本体代替。很显然，这将会增加最终目标码的大小，同时这会使程序的体积变大，额外的换页行为并且降低高速缓存的击中率。

- 然而根据上述两点，倘若`inline`的函数体积很小，小到其生成的目标码比函数调用产生的目标码还要小，那它就可以兼具以上的所有优点。

- 同时，`inline`仅是一个对编译器的申请，不是一个强制的命令。这项申请能够隐喻提出，也可以明确提出。即他可以隐式定义，也可以显式定义。
```cpp
class Person {
public:
	int age() { 
		return theAge;
	}
	...
private:
	int theAge;
};
```
以上例子即是`inline`的隐式声明。

- `inline`函数的显式定义就是在函数的声明处加上`inline`关键词，如标准的`max template`就是这样的
```cpp
template <typename T>
inline const T& std::max(const T& a, const T& b) {
	return a < b ? b : a;
}
```

- 然而并非所有`template`都应该是`inline`，大部分的inlining行为都发生在编译阶段，极少数发生在运行阶段，因此根据inlining的行为而言，对于template其也必须在编译阶段清楚其具体实现。对于大部分带有递归或循环的函数，及所有的`virtual`函数，编译器基本上都拒绝将他们inlining。

- 有时候即使编译器决定将某个函数inlining，仍然会为其创建outline函数。如：需要取该函数的地址时，其必须是个outline函数。

- 看起来一些空的构造和析构函数是`inline`的绝佳人选，然而事实往往相反。构造函数和析构函数看起来是空的，然而编译器往往会安插一些代码，毕竟构造和析构的行为不可能真的什么都不做。
```cpp
class Base {
public:
	...
private:
	std::string bm1, bm2;
};
class Derived: public Base {
public:
	...
private:
	std::string dm1, dm2, dm3;
}

Derived::Derived() {
	Base::Base();
	try {
		dm1.std::string::string();
	} catch {
		Base::~Base();
		throw;
	}

	try {
		dm2.std::string::string();
	} catch {
		dm1.std::string::~string();
		Base::~Base();
		throw;
	}
	...
}
```
以上代码并非是编译器实际产生的代码，编译器会以更加复杂精致的做法来处理异常。

- 程序的设计者必须评估是否将函数声明为`inline`。`inline`函数没法随着程序库的升级而升级：若一个函数f是inline函数，则每次对f进行修改后，都必须将所有用到了f的客户端程序都重新进行编译，若f不是inline函数，则只需要重新连接，倘若程序库采用动态连接的方式，甚至能够不知不觉被应用程序吸纳。

- 最好的做法是：一开始不要讲任何函数声明为`inline`，或至少将其范围圈定在那些“一定要用`inline`”(见[[46-需要类型转换时请为模板定义非成员函数]])或那些“比较平淡无奇”(如本条款中的`Person::age()`)。

- 谨记80-20经验法则：平均一个程序80%的执行时间话费在20%的代码上面。作为一个软件开发者，你的目标就是找出这个可以有效增进程序整体效率的20%代码，然后用inline或竭尽所能的将其瘦身。

- 小结：
	- 将大多数inlining限制在小型、被频繁调用的函数上。可使日后的调试过程和二进制升级更容易，也可使潜在的代码膨胀问题最小化，使程序的提升速度最大化。
	- 不要只因为`template function`和`inline function`都出现在头文件，就将`template`都用`inline`声明。