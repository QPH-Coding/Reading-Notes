# 条款11-令`operator=`中处理“自我赋值”
---
+ “自我赋值”听起来十分愚蠢，但是他是发生的，同时有时候是隐形的
```cpp
a[i] = b[j];
*px = *py;
```

形如上方的代码都是可能产生“自我赋值”的

+ 一般来说，凡是涉及到`heap`的，即要自我管理的部分，是最容易出现问题的，尤其是**内存泄漏**问题
```cpp
class BitMap {...};
class Widget {
	...
private:
	BitMap* pb;
};
```

上方的例子为利用一个`class Widght`去管理指向一块动态分配的位图

+ 下面来看一份不安全的`operator=()`实现代码
```cpp
Widget& Widget::operator=(const Widget& rhs) {
	delete pb;
	pb = new BitMap(*rhs.pb);
	return *this;
}
```

这里的不安全行为十分明显，若`this.pb`与`rhs.pb`指向同一块内存，在此赋值函数中，先`delete`释放了此内存，之后又去访问这块内存，很明显是十分不安全的一个行为

+ 最简单的改进方法即为先进行一次“认同测试”
```cpp
Widget& Widget::operator=(const Widget& rhs) {
	//认同测试
	if(this == &ths) {
		return *this;
	}
	delete pb;
	pb = new BitMap(*rhs.pb);
	return *this;
}
```

但是这并不是最好的版本，当涉及到`heap`的部分时，我们更应该尤为小心，毕竟这是C++完全相信我们的部分，我们更加有理由尽可能设计出一个好的健壮性强的代码片段。

最开始的版本既不具备“**自我赋值安全性**”，也不具备“**异常安全性**”。而这个版本解决了前一个问题，但仍然存在“**异常安全性**”问题。若在`new BitMap()`的过程中异常(分配时内存不足，或`BitMap`的copy构造函数抛出异常)，最终的结果都是`Widget`会持有一个指向被删除的`BitMap`内存的指针，俗称“野指针”。

+ 好消息是，一般具有了“**异常安全性**”，也会自动获得“**自我赋值安全性**”。因此我们只需要将目光放在“**异常安全性**”上。在[[29-为“异常安全”而努力是值得的]]中深度探讨了这一点。很多时候只需要精心安排语句即可。
```cpp
Widget& Widget::operator=(const Widget& rhs) {
	BitMap* pOrg = pb;
	pb = new BitMap(*rhs.pb);
	delete pOrg;
	return *this;
}
```

在这里我们只需要注意在复制`pb`所指东西之前不要删除`pb`。

这里具有“异常安全性”的地方在于，当`new BitMap()`抛出异常，`pb`及`class Widget`会保原状。同时也能够处理自我赋值

+ 除此之外，我们还可以使用`copy and swap`技术([[29-为“异常安全”而努力是值得的]]中有详细的说明)。
```cpp
class Widget {
...
void swap(Widget& rhs); //交换*this和rhs的数据
};
Widget& Widget::operator=(const Widget& rhs) {
	Widget temp(rhs);
	swap(temp);
	return *this;
}
//若operator=接受pass by value，因为此时会自己复制一个副本
Widget& Widget::operator=(Widget rhs) {
	swap(rhs);
	return *this;
}
```

后一个做法实际上牺牲了一定可读性和清晰性，但是将“copying动作”从函数本体转移到“函数参数构造阶段”会让编译器产出更高效的代码

+ 小结：
	+ 确保当对象自我赋值时，`operator=()`有良好行为。可运用的技术有：比较两个对象间的地址、精心设计语句顺序、`copy-and-swap`技术
	+ 确定任何函数操作一个以上的对象，而其中多个对象为同一个对象时，其行为仍然正确