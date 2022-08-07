# 条款28-避免返回`handles`指向对象内部成分
---
- 本条款所提到的`handles`，实际上代指的是`pointer`、`reference`、`迭代器`等，即“号码牌”，用来取得某对象之物。

- 当你返回的是`handles`时，可能会破坏封装性，使一个`private`的成员最终又变成了`public`的效果
```cpp
//用于表示一个点的class
class Point {
public:
	Point(int x, int y);
	void setX(int val);
	void setY(int val);
	...
};
struct RectData {
	Point ulhc; //upper left-hand corner  左上角
	Point lrhc; //lower right-hand corner 右上角
};
//用于表示一个矩形的class
class Rectangle {
public:
	Point& upperLeft() const {
		return pRD->ulhc;
	}
	Point& lowerRight() const {
		return pRD->lrhc;
	}
private:
	std::tr1::shared_ptr<RectData> pRD;
};
```

以上的设计能够通过编译，然而却是错误的。他是自相矛盾的，`class Rectangle`中的成员函数为`const`，最终返回`reference`实际上是能够对数据进行修改的，而将内部的成员声明为`private`也失去了意义，毕竟能够用另一种方式对其直接访问。
```cpp
void f() {
	Point coord1(0, 0);
	Point coord2(100, 100);
	const Rectangle rec(coord1, coord2);
	rec.upperLeft().setX(50); //coord1 (0, 0) -> (50, 0)
}
```

- 从上述例子能得出两个结论：
	1. 成员变量的封装最多只等于“返回其`reference`”的函数的访问级别
	2. 如果`const`成员函数传出一个`reference`，后者所指数据与对象自身有关联，又被存储在对象之外，那么这个函数的调用者可以修改那笔数据，这也是`bitwise constness`的一个附带结果(见[[03-尽可能使用const]])

+ 同时，不要返回访问级别比自身低的函数指针，这会将其访问界别提至和自身一样高。

+ 实际上，以上遇到的问题只需要在返回值前加`const`就能够解决
```cpp
class Rectangle {
public:
	const Point& upperLeft() const;
	const Point& lowerRight() const;
	...
};
```

然而有个问题仍然存在

- 当你返回一个指向对象内部的`handles`时，有可能导致`dangling handles`(空悬的号码牌)，如果`handle`是指针，就是一个野指针问题，即`handles`所指的东西不复存在。
```cpp
class GUIObject {...};
const Rectangle boundingBox(const GUIObject& obj);
void f() {
	GUIObject* pgo = new GUIObject();
	...
	const Point* pUpperLeft = &(boundingBox(*pgo).upperLeft());
	...
}
```

此时就出现了一个很明确的`dangling handle`，其用函数返回的`Rectangle`，没有名字，在调用完这个函数后，他将会被析构，最终得到的就是一个`dangling handle`。

这里唯一的关键在于，有个`handle`被传出去了，暴露在“handle比其所指对象更加长寿”的风险之中。

- 这并不意味着，你绝不可以让成员函数返回`handle`。有时候是必须的，如`vector`的`operator[]()`，但毕竟是例外，而不是常态。

- 小结：
	- 避免返回`handles`(`指针`、`reference`、`迭代器`)指向对象内部。遵循这个规定可以增加封装性，帮助`const`成员更像`const`,并将发生`dangling handle`的可能性降至最低。