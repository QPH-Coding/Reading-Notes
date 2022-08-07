# 条款07-为多态基类声明`virtual析构函数`
---
+ 若一个`pointer-base-to-class`指向了一个`derived class`，同时此对象在`heap`上，当调用`delete base_class_ptr`时，若`base class`的析构函数不为`virtual`，会发生仅调用`base class`的析构函数，不掉用`derived class`的析构函数，从而产生一种局部释放内存的怪现象，这种怪现象，有一个响当当的名字--**内存泄漏**
```cpp
//计时器有许多种，因而为其编写一个base class，同时为其编写几个derived class
//base class
class TimeKeeper {
public:
	TimeKeeper() {...};
	~TimeKeeper() {...};
	...
};
//derived classes
class AtomTimeKeeper: public TimeKeeper {...};//原子钟
class WaterTimeKeeper: public TimeKeeper {...};//水钟
class WristTimeKeeper: public TimeKeeper {...};//腕表，

//为了向外界的调用者隐藏其细节，可以利用factory模式
TimeKeeper* tk_ptr = getTimeKeeper();
...
//同时这里的factory模式要求tk_ptr指向一块heap上的内存，因此需要释放它
delete tk_ptr;//ERROR!!!
```

将其变为正确的做法很简单，仅需要将其`base class`的析构函数前加上`virtual`关键词即可
```cpp
class TimeKeeper {
public:
	TimeKeeper() {...};
	virtual ~TimeKeeper() {...};
	...
};
class AtomTimeKeeper: public TimeKeeper {...};
...
TimeKeeper* tk_ptr = getTimeKeeper();
...
delete tk_ptr;//It's OK.
```

+ 如果一个`class`不被设计为`base class`就无需为其设计`virtual析构函数`，`virtual函数`的工作原理是有一张`virtual table(虚表)`，而在`base class`存有一个`pointer`指向`virtual table`的函数地址，一个显然的问题就在于，当你无端地将析构函数设置为virtual时，会增加类的体量，增加多少取决于多少位的操作系统

+ 许多人的有效经验为：**只有类中至少有一个virtual函数时，才将析构函数声明为virtual**

+ 拒绝继承`non-virtual`析构的`class`，包括STL的`string`，`vector`等所有容器，它们都是`non-virtual`析构的`class`

+ 当你编写一个`base class`时，没有`pure virtual`的函数，由于`base class`的特性又不想有人将其实例化，即你需要有一个`pure virtual`函数。最好的方法就是将其析构函数设为`pure virtual`
```cpp
class AWOV {//abstract w/o virtual
public:
	...
	virtual ~AWOV() {} = 0; //将析构函数设为pure virtual
};
//实现 ～AWOV() 否则会有链接错误
AWOV::~AWOV() { }
```

+ 正如标题所说，给`base class`设置一个`virtual析构函数`是为多态服务的，即不必未所有`base class`都赋予`virtual析构函数`

+ 小结：
	+ `polymorphic(多态) base class`应该声明一个`virtual析构函数`。如果`class`带有任何`virtual函数`，就应该拥有一个`virtual析构函数`
	+ `class`设计的目的不是作为`base class`使用，或不是为了多态，就不该声明`virtual析构函数`