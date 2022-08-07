# 条款03-尽可能使用`const`
---
- 对指针来说，使用`const`主要看在**之前还是之后
```cpp
const char *p;//non-const pointer, const data
char const *p;//non-const pointer, const data
char* const p;//const pointer, non-const data
```

- 对于STL的`迭代器iterator`，如下
```cpp
const std::vector<int>::iterator it = v.begin();
*it = 10;//正确的
++it;//错误的，iterator为const
---
std::vector<int>::const_iterator it = v.begin();
*it = 10;//错误的，*it为const
++it;//正确的
```

- 对于某些情况，可以使返回值为`const`，防止在使用此函数对其进行修改
```cpp
class Rational{...};
const Rational operator*(const Rational& lhs, const Rational& rhs){};
---
Rational a, b, c;
if((a * b) = c){...}
//很明显，实际上是一个输入错误（少输入了一个=）
//若数据类型为内置类型将不合法
//我们也应该保持和内置数据类型的一致性
```

- 对于成员函数，也可以使用`const`关键词对于`const`对象的特殊操作，并且`const`和`non-const`之间可组成重载关系
```cpp
class TextBlock {
public:
    const char& operator[](std::size_t index) const {
        return text[index];
    }
    char& operator[](std::size_t index) {
        return text[index];
    }
priavte:
    std::string text;
};
```

- 使用`mutable`关键词可以使变量在`const`成员函数中更改
```cpp
class TextBlock {
public:
    std::size_t getLength() const {
        if(!isLengthValid) {
            textLength = std::strlen(text);
            isLengthValid = true;
        }
        return textLength;
    }
private:
    std::string text;
    mutable std::size_t textLength;
    mutable bool isLengthValid;
};
```

- 为了避免代码的重复，我们可以将`const`成员函数做的东西塞到`non-const`成员函数中，只需要进行2次转型

```cpp
class TextBlock {
public:
    const char& operator[](std::size_t index) const {
        ...//边界检查 bounds checking
        ...//记录数据访问 log access data
        ...//检验数据完整性 verify data integrity
        return text[index];
    }
    char& oprator[](std::size_t index) {
        return
            const_cast<char&>( //去除返回值的const
                static_cast<const TextBlock&>(*this)[index]);
                //为*this加上const关键词
    }
private:
    std::string text;
}
```

- 理论上也可以利用转型的动作，在`const`成员函数中调用`non-const`去避免代码重复，但最好不这么做，这么做违反了`const`成员函数的初衷，然而`non-const`成员函数并没有此种限制，因而我们采取以上的做法
    
- 小结：
    - 将某些东西声明为`const`有利于编译器对其错误进行检查，`const`可声明到作用域的一切对象、函数参数、函数返回类型、成员函数本体
    - 当`const`成员函数和`non-const`成员函数在实质性的实现上相同时，使`non-const`成员函数调用`const`成员函数可避免代码重复