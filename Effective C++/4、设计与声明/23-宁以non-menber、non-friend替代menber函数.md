# 条款23-宁以`non-menber函数`、`non-friend函数`代替`menber函数`
---
+ 在具体探讨本条款先讨论以下的例子：
一个网页浏览器类，其中有清理缓存、清理历史记录、清理Cookies等功能。
```cpp
class WebBrowser {
public:
	void clearCache();
	void clearHistory();
	void removeCookies();
	...
};
```

我们想要将这三个整合成一个函数`clearEverything()`。现在有两种实现方法：

第一种，`menber函数`：
```cpp
class WebBrowser {
public:
	void clearEverything();
	...
};
```

第二种，`non-menber函数`：
```cpp
void clearEverything(WebBrowser& wb) {
	wb.clearCache();
	wb.clearHistory();
	wb.removeCookies();
}
```

大部分人的第一直觉是：应该使用第一种`menber函数`，毕竟其与`class WebBrowser`紧密相连。然而恰恰相反，这是对面向对象守则中数据尽可能封装的一种误解。`menber函数`比`non-menber函数`的封装性更低。同时`non-member函数`对`WebBrowser`的相关功能有较大的**包裹弹性**，从而使最终的**编译依赖度降低**，增加`WebBrowser`的可延伸性。

+ 首先我们需要了解封装的原因：**他能使我们能够改变事务而只影响有限的客户**。

考虑对象内部的数据，实际上越少的函数能够直接看见他，其封装性就越高。[[22-将成员变量声明为private]]中提到，成员变量应该是`private`，只有他才提供封装性。那么能看见`private`的函数只有`member函数`和`friend函数`，实际上这里说的是`non-member & non-friend函数`和`member函数`的比较。如果你能够通过`non-member & non-firend函数`实现和`member函数`一样的功能，那就以前者代替他。

+ 值得注意的是，本条款并不是指这里的`clearEverything()`不能为其他`class`的`member函数`，对一些必须把东西都写在`class`的语言(Java、C#等)，你可以通过写一个工具类同时在里面写一个`static`方法。那么对于C++而言，更常见的是将方法写在`namespace`中。
```cpp
namespace WebBrowserStuff {
	class WebBrowser {...};
	void clearEverything();
	...
}
```

这样看起来更加自然，同时有一个显而易见的好处：**`namespace`能够跨越文件，而`class`则不能**。

当`WebBrowser`有大量的功能函数时，我们更多会这样做：
```cpp
//webbrower.h --- 包含着class WebBrowser, 及一些常用的核心功能
namespace WebBrowser {
	class WebBrowser {...};
	...
}
//webbrowserbookmarks.h --- 包含着与书签相关的功能
namespace WebBrowser {
	...
}
//webbrowsercookies.h --- 包含着与cookies相关的功能
namespace WebBrowser {
	...
}
```

实际上**这正是C++标准程序库的组织方式**。将所有功能函数放在多个头文件，却隶属同一个命名空间，意味着能够轻松扩展这一组函数。

+ 小结：
	+ 宁以`non-member & non-friend函数`替换`member函数`。这样做可以增加封装性、包裹弹性和功能扩充性。