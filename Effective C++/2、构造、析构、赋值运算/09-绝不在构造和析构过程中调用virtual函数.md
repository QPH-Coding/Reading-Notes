# 条款09-绝不在构造和析构过程中调用virtual函数
---
+ 不要在构造函数中调用virtual函数，那不会得到你想要的结果
```cpp
//编写一个交易类，同时内部有一个virtual函数logTransaction
//logTransaction用于记录交易日记，根据买/卖交易类型实现有所不同
class Transaction {
public:
	Transaction();
	virtual void logTransaction() const = 0;
};
Transaction::Transaction() {
	...
	logTransaction();
}
class BuyTransaction: public Transaction {
public:
	virtual void logTransaction() const;
	...
};
class SellTransaction: public Transaction {
public:
	virtual void logTransaction() cosnt;
	...
};
...
BuyTransaction bt;
```

当最下面那行代码运行时，会首先调用`base class`的构造函数，再调用自身即`derived class`的构造函数，那么在这个时候`base class`调用的virtual函数`logTransaction()`实际上不会时`derived class`的版本，而是`base class`的版本，然而在`base class`阶段，其仍是一个`pure virtual`函数。有这么一句话十分传神：**`base class`构造期间，virtual函数不是virtual函数**。

为什么在进行`derived class`的构造期间，先构造`base class`为何不可去使用`derived class`版本的virtual函数，原因在于，当进行`base class`的构造期间，一切`derived class`的成员对象都将其视为未初始化的未知对象，因此不去使用`derived class`的virtual函数版本是合理的，毕竟其难免可能会用到`derived class`中的成员对象。因此C++不会让你走上一条十分危险的道路。

+ 同理，对析构函数而言，当进入`base class`之后，`derived class`中一切对象都被视为未知，因此一样会调用`base class`版本的virtual函数

+ 实际上，更多时候去检测构造及析构函数并不是一件非常容易的事情
```cpp
class Transaction {
public:
	Transaction() {
		init();
	}
	virtual void logTransaction() const = 0;
	...
private:
	void init() {
		...
		logTransaction();
	}
};
```

通常可能会有几个构造函数，同时其内部大部分逻辑是相通的，因此我们更乐意去用一个`init()`函数将大部分重复的逻辑包裹起来去节省代码量。那么如果这里的`logTransaction()`不是一个`pure virtual`函数，那将会更加糟糕，其能够通过编译，通过连接，最后得到一个“正常运行”的，在`derived class`调用了`base class`版本`logTransaction()`

+ 对于如何在合适的版本调用合适的函数，一种较好的改进方法是，将`virtual`转化为`non-virtual`
```cpp
class Transaction {
public:
	explicit Transaction(const std::string& logIndo);
	void logTransaction(const std::string& logInfo) const;
	...
};
Transaction::Transaction(const std::string& logInfo) {
	...
	logTransaction(logInfo);
}
class BuyTransaction: public Transaction {
public:
	BuyTransaction(parameters)
		: Transaction(createLogString(parameters));//将log信息传给base class构造函数
private:
	static std::string createLogString(parameters);
};
```

要注意的是`private static`函数`createLogString()`的运用。比起在成员初值列，给予`base class`所需要的数据，利用一个辅助函数创建一个值传给`base class`更加方便，也更加具有可读性。同时返回的是`static`对象，也就不太可能有指向`BuyTransaction`内未定义对象的问题

+ 小结
	+ 在构造和析构期间，不要调用virtual函数，因为这类调用不会下降至`derived class`，往往得不到你想要的结果