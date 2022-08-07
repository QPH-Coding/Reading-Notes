# 条款02-尽量以`const`，`enum`，`inline`替代`#define`
---
- 条款的意思即为：宁可以编译器替换预处理器。`#define`并不可视为语言的一部分
- 更多时候会以`常量变量`替换`#define`
    - 当使用字符串时，若用C部分，需要用两次`const`，如：`const char* const authorName = "QPH";`；实际上，用C++中的`string`对象更加合适
    - 当此常量为class的专属常量时，应该将其作为class的一个成员变量，同时又必须确保其至多只有一个实体，因此又应该将其变为static成员

```cpp
class MyClass {
private:
    static const int num = 5;
    int array[num];
    ...
};
//实际上，更好的是类内定义，类外赋值，为更多编译器所接受
class MyClass {
private:
    static const int num;
    int array[num];
    ...
};
const int MyClass::num = 5;
```

- 当编译器不允许`static整型class变量`完成`in class初值设定`，可用`the enum hack`的方法补偿；原理为 “一个属于枚举类型的数值可权充int被使用”
```cpp
class MyClass {
private:
    enum { num = 5 };
    int array[num];
	...
};
```
    
使用`enum hack`的理由有：1. `enum hack`的行为更像`#define`，如不可以取`enum`的地址，也不可以取`#define`的地址，可以限制其他人获得一个`pointer`或`reference`指向其地址，编译器也不会为其分配不必要的内存；2. 实用主义，`enum hack`是`template metaprogramming-模板元编程`的基础技术
	
- `#define`实现的宏有时更像一个函数，但实际上并未产生调用的过程，导致出现一些未知的错误，此时应以`template inline函数`去替代

```cpp
template<typename T>
inline void callWithMax(const T& a, const T& b) {
    f(a > b ? a : b);
}
```

- 小结：对于单纯的变量，应以`const对象`或`enums`取代替`#define`；对于形似函数的宏，改以`inline函数`替换`#define`