# 条款22-将成员变量声明为`private`
---
+ 当我们提出这个条款时，很显然会想到一个很重要的概念：封装。我们更多会在本条款告诉你封装到底带来了什么好处。
```cpp
class SpeedDataCollection {
public:
	void addValue(int speed);
	double averageSoFar() const;
};
```

当我们思考`averageSoFar()`的具体实现时，其有两种思路：第一种，`class`内设计一个成员变量用于记录至今所有速度平均值，当函数被调用时，只需要返回即可；第二种，每次调用`averageSoFar()`时，重新计算所有速度的平均值，此函数能够调取收集器中的每一个`speed`。

第一种做法：会是`class SpeedDataCollection`的体积变大，然而其`averageSoFar()`的效率却十分高，他可以只是一个返回平均值的`inline`函数(见[[30-透彻了解inlining的里里外外]])。

第二种做法：和第一种方法相比，`class SpeedDataCollection`的体积明显小很多，但是其函数的执行效率显然没那么高效。

+ 仅通过上述的分析，你很难判断应该采用哪种方式实现。然而当你将其封装起来之后，由于封装带来的好处之一就是语法的一致性(见[[18-让接口容易被正确使用，不易被误用]])，你可以随时替换不同的实现方式，最多只需要重新编译，甚至当遵循[[31-将文件间的编译依存关系降至最低]]时，可以消除重新编译的不便性。

+ 封装除了能给`class`作者灵活的实现方式，还能够提供细微的权限控制
```cpp
class AccessLevels {
public:
	int getReadOnly() const {
		return readOnly;
	}
	void setReadWrite(int value) {
		readWrite = value;
	} 
	int getReadWrite() {
		return readWrite;
	}
	void setWriteOnly(int value) {
		writeOnly = value;
	}
private:
	int noAccess, readOnly, writeOnly, readWrite;
};
```

+ 封装的重要性比你初见时想象的更加重要。当你对客户隐藏成员变量时，你可以确保`class`的约束条件是起效果的，同时还保留了日后变更实现的权力。越是广泛使用的`class`，其对封装的诉求就越大。

+ 以上的论述基本都在说`public`，现在我们来讨论一下`protected`。

+ 对于**语法一致性**和**细微划分的访问控制**，`protected`能够做到，但是在封装方面`protected`未必就高于`public`成员。[[23-宁以non-menber、non-friend替代menber函数]]中，某些东西的封装性与其内容改变时可能造成的代码破坏量成反比，这里改变实际上可以看作是从`class`中移除他。

当你把一个`class`中的`public`对象直接移除时，可能大多数客户端的代码都被破坏了；假设把`class`中的`protected`对象移除，那么所有`derived class`中使用他的代码都被破坏了，其破坏的代码量是“未知”的大量。**因此`protected`对象和`public`对象一样不具有封装性。**

+ 一旦你将一个成员变量声明为`public`或者`protected`，就很难改变那个成员变量涉及的一切。太多代码需要重写、重新测试、重新编写文档、重新编译。**从封装的角度看，只有两种访问权限：`private`(封装)和其他(不封装)。**

+ 小结：
	+ 切记将成员变量声明为`private`，这赋予客户访问数据的一致性、可细微划分访问控制、允许约束条件获得保证，并提供`class`作者充分的实现弹性。
	+ `protected`并不比`public`更具有封装性。