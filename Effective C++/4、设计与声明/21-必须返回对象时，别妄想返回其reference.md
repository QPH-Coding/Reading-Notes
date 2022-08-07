# 条款21-必须返回对象时，别妄想返回其`reference`
---
+ 在[[20-宁以pass-by-reference-to-const替代pass-by-value]]中了解到了`pass-by-reference-to-const`的好处后，十分容易落入一个陷阱：万物都以用`reference`，尤其返回也使用`reference`。
```cpp
class Rational {
public:
	Rational(int numerator = 0, int denominator = 1);
private:
	int n, d;
	friend const Rational operator* (const Rational& lhs, const Rational& rhs);
};
```

上面是一个返回一个对象副本的例子，毫无疑问，他付出了构造和析构函数的代价。
然而，当我们要使用`reference`时，我们一定要问自己一个问题，他的另一个名字是什么？或者说他的原始对象在哪里？本例中若将函数的返回值换成`reference&`，那么他需要指向哪个对象呢？

在函数中创建对象的途径有两种：在`stack`上创建和在`heap`上创建。

我们先来思考第一种：在`stack`上创建
```cpp
const Rational& operator* (const Rational& lhs, const Rational& rhs) {
	Rational result(lhs.n * rhs.n, lhs.d * rhs.d);
	return result;
}
```

很明显，这是一段很糟糕的代码，因为在退出函数调用时，会调用`result`的析构函数，那么此时返回的引用就是一个空的值。总的来说，返回一个`reference`指向一个`local`对象都将一败涂地。

接下来考虑第二种：在`heap`上创建
```cpp
const Rational& operator* (const Rational& lhs, const Rational& rhs) {
	Rational* result = new Rational(lhs.n * rhs.n, lhs.d * rhs.d);
	return result;
}
```

这是一段比之前更糟糕的代码，当你在函数内部用`new`分配了动态内存后，在外界应该由谁去使用`delete`释放呢。同时再观察以下代码：
```cpp
Rational x, y, z;
w = x * y * z;
```

在第二行的调用中，使用了两次`new`，但是此时只能使用`delete`，因为此时有一个`new`出来的对象被隐藏起来了，这是无法挽回的百分百发生的内存泄露。

也许有人会想到利用`static`的性质，让其脱离以上的限制。
```cpp
const Rational& operator* (const Rational& lhs, const Rational& rhs) {
	static Rational result;
	result = ...;
	return result;
}

void f() {
	Rational a, b, c, d;
	if((a *b) == (c * d)) {
		...
	}
}
```

猜猜会发生什么？上面的判等永远会是`true`！因为他们返回的`reference`都指向同一个对象，其值永远是相等的。

+ 综上，有时候我们并不是不可以承受构造和析构函数的成本，长远来看，这是获得正确行为的一个小小代价。

+ 小结：
	+ 绝不要返回`pointer`或`reference`指向一个`local stack`对象，或返回`reference`指向一个`heap-allocated`对象，或返回`pointer`或`reference`指向一个`local static`对象。