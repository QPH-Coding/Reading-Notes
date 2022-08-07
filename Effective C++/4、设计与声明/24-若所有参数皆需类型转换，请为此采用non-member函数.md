# 条款24-若所有参数皆需类型转换，请为此采用`non-member函数`
---
+ 通常情况下，我们建议把构造函数设置为`explicit`，即不允许隐式转换。然而有时候隐式转换会看起来比较合理，同时也更加方便人们使用
```cpp
class Rational {
public:
	Rational(int numerator = 0, int denominator = 0);
	int numerator();
	int denominator();
private:
	...
};
```

这是一个有理数类，因此你会想支持一些计算运算符，但是实现操作符重载时，可以由`member函数`，`non-member friend函数`来实现，这时应该选什么函数实现呢。

先来考虑`member函数`的写法
```cpp
class Rational {
public:
	const Raional operator* (const Rational& rhs) const;
	...
};
void f() {
	Rational oneHalf(1, 2);
	Rational oneEighth(1, 8);
	Rational result = oneHalf * oneEighth; //GOOD!
	result = result * oneEighth; //GOOD!
	result = oneHalf * 2; //GOOD!
	result = 2 * oneHalf; //ERROR!!!
}
```

可以看到前面使用时是正常的，然而当进行混合式运算时，却出现了很奇怪的错误，一半是通过，另一半却是错误的，很显然此处一个有理数不支持乘法交换律是很不正常的。那么我们可以看看最后是怎么回事。
```cpp
void f() {
	result = oneHalf.operator*(2); //GOOD!
	result = 2.operator*(oneHalf); //ERROR!!!
}
```

现在答案很明显了。`class Rational`中的`operator*()`函数支持将2隐式转换为`Rational`，然而整数2并没有一个对应的`class`，也就没有对应的`operator*()`函数。此时编译器会去尝试查找有没有`non-member函数`名为`operator*()`。

+ 只有当参数被列于参数列内，这个参数才能进行隐式类型转换。当你令`operator*()`成为一个`non-member函数`时，便允许编译器在每一个实参上执行隐式转换。接下来看看`non-member函数`的实现。
```cpp
class Rational {...};
const Rational operator*(const Rational& lhs, const Rational& rhs) {
	return Rational(lhs.numerator() * rhs.numerator(), 
					lhs.denominator() * rhs.demominator());
}
void f() {
	Rational oneFourth(1, 4);
	Rational result;
	result = 2 * oneFourth; //GOOD!
	result = oneFourth * 2; //GOOD!
}
```

现在他可以正常的工作了。

+ 本条款内含真理，但却不是所有的真理，当你从`Object-Oriented C++`踏入`Template C++`(见[[01-视C++为语言联邦]])并让`class Rational`成为一个`class template`时，又有一些新的东西需要考虑，将在[[46-需要类型转换时请为模板定义非成员函数]]中进行讨论。

+ 小结：
	+ 如果你需要为某个函数的所有参数进行类型转换，那么这个函数必须是个`non-member`。