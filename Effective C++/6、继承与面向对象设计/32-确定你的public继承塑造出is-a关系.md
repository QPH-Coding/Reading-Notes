# 条款32-确定你的public继承塑造出is-a关系
---
- `is-a`关系的概念就是指“一种”，即能够用`Base class`的地方，也能够用`Derived class`。

- 若令`class Derived`继承`class Base`，就意味着，你对客户和编译器说，每一个Derived对象也是一个Base对象，反之不成立。
```cpp
class Person {};
class Student: public Person {};
```

以上例子即说明了严格的public继承关系该有的样子。

- 通过以下的一个例子，来看看如何塑造`is-a`的关系
```cpp
class Bird {
public:
	virtual void fly(); //鸟可以飞
	...
};
class Penguin: public Bird {
	...
};
```

众所周知，企鹅是不能飞的，因此此时并未成功的塑造出我们想要的`is-a`关系。
```cpp
class Bird { ... };
class FlyingBird {
public:
	virtual void fly();
	...
};
class Penguin: public Bird { ... };
```

此时塑造出了我们想要的`is-a`关系，然而我们认为原先的双class体系已然令人满意，因此有下面一种做法。

- 为`Derived class`即企鹅重新定义`fly()`函数，令其在运行期间产生一个错误：
```cpp
void error(const std::string& msg); //定义在其他地方
class Penguin: public Bird {
public:
	virtual void fly() {
		error("Attempt to make a penguin fly!");
	}
};
```

以上做法并非最好的做法，因为在[[18-让接口容易被正确使用，不易被误用]]中提到过，好的接口可以防止无效代码通过编译。因此另一种做法则是在`class Bird`和`class Penguin`中都不定义`fly()`，这样可以防止让一些错误的代码通过编译。

- **public继承主张，能够实施在`Base class`上的每件事情都可以实施在`Derived class`上。**

- `is-a`并非是唯一存在class之间的关系。另外两个常见的是`has-a`(有一种)和`is-implemented-in-terms-of`(根据某物实现)。这些将在[[38-通过复合塑造出has-a或“根据某物实现出”]]和[[39-明智而审慎地使用private继承]]中进行讨论。

- 小结：
	- public继承意味着`is-a`。适用于`base class`身上的每件事情一定也适用于`derived class`身上，每个`derived class`也都是一个`base class`对象。