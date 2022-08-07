# 条款14-在资源管理类中小心copying行为
---
+ 在[[13-以对象管理资源]]中，以“**资源获得的最好时机旧时初始化时机**”(RAII)为“资源管理类”概念的脊柱，同时也讨论了`tr1::shared_ptr`和`auto_ptr`在`heap-based`资源上的实现。然而有些时候并非所有需要管理的资源都在`heap`上。

如使用C++中的C API
```c
void lock(Mutex* pm); //锁定或斥锁mutex
void unlock(Mutex* pm); //解除互斥锁mutex
```

这个时候我们需要去精心定制一个属于我们自己的资源管理类
```cpp
class Lock {
public:
	explicit Lock(Mutex* pm): mutexPtr(pm) {
		lock(mutexPtr);
	}
	~Lock() {
		unlock(mutexPtr);
	}
private:
	Mutex* mutexPtr;
};
```

然而当客户想要进行copy时，会发生什么？或者说你想让其发生什么？

+ **禁止复制**
	+ 许多时候复制RAII对象并不是一件合理的事情。同时，我们早已在[[06-若不想使用编译器自动生成的函数，就该明确拒绝]]中便提到过该如何禁用复制。
```cpp
class Lock: private Uncopyable {
public:
	...
};
```

+ **对底层资源使用“引用计数法”**
	+ 有时候我们希望保有资源，直至其最后一个使用者被销毁。这种情况下可以使用引用计数法
	+ 在[[13-以对象管理资源]]中已经进行了讨论，`tr1::shared_ptr`便是使用引用计数法的。理论上只需要在我们设计的资源管理类中使用`tr1::shared_ptr`即可实现。
	+ `tr1::shared_ptr`默认在析构时调用`delete`，然而我们现在要管理的资源并非在`heap`上，幸运的是`tr1::shared_ptr`允许修改其删除器，因此我们可以通过给出其释放动作的函数来实现。
```cpp
class Lock {
public:
	explicit Lock(Mutex* pm): mutexPtr(pm, unlock) {
		lock(mutexPtr.get());
	}
private:
	std::tr1::shared_ptr<Mutex> mutexPtr;
};
```

这里不再声明构造函数，原因是原本在构造函数需要的释放动作，已经在`tr1::shared_ptr`中得到了实现。同时关于`mutexPtr.get()`，可以在[[15-在资源管理类中提供对原始资源的访问]]中了解。

+ **复制底层资源**
	+ 有时候你或者你的客户确实需要有一次彻底的copy行为。
	+ 你需要资源管理类的理由是，当不再需要时能够确保被释放。这两者并不冲突，前提是当你进行copy动作时，不仅要复制资源管理类，还需要复制其所包含的资源(尤其对于heap上的资源更是如此)，就是我们常说的“**深拷贝**”。

+ **转移底部资源的使用权**
	+ 某些特殊场合你可能希望确保只有一个RAII对象指向一个未加工资源，此时资源的拥有权会从被复制物转移到目标物。
	+ 这也是`auto_ptr`的思想，见[[13-以对象管理资源]]

+ 小结：
	+ 复制RAII对象必须一起复制它管理的资源，所以资源的copying行为决定RAII对象的copying行为
	+ 普遍常见的RAII class copying行为是：抑制copying、施行引用计数法。不过其他行为也可能实现