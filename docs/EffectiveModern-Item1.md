---
title: "[翻译]高效现代 C++ 编程 - 条款1：理解模板类型推导 (Understand template type deduction)"
date: 2018-04-06 18:00:59
categories:
- C++
tags:
- modern C++
---
一个设计优秀的系统，用户能够在对系统的复杂度一无所知的情况下愉快的使用它。在这个意义上， C++ 的模板类型推导是一个巨大的成功。数百万的程序员只是将参数传递给`template function`就能得到令人满意的结果，尽管他们中的大多数对于这些函数的类型是如何被推导出来的只有一个朦胧的概念。
如果你也是他们中的一员，你应该对自己的处境喜忧参半。喜的是现代 C++ 最引人入胜的功能——`auto`，正是建立在模板类型推导之上的。如果你对 C++98 推导出的模板类型感到满意，那你可以准备好对 C++11 中`auto`的类型推导结果感到开心了。忧的是使用`auto`的情境与使用`template`的情境相比，有时`auto`的类型推导会显得比较不直观。因此，真正理解模板类型推导的方方面面尤为重要，它是`auto`建立的基石。本条款就覆盖了你所应该知道的一切。

如果你不介意阅读一些伪代码，我们可以假设一个如下的`function template`：
````cpp
template<typename T>
void f(ParamType param);
````
一个对它的调用可能是这样：
````cpp
f(expr);	// 用expr调用f
````
在编译过程中，compiler 会用`expr`推导两个类型——`T`代表的类型和`ParamType`代表的类型。这两个类型通常是不一样的，因为`ParamType`经常包含了一些修饰，比如`const`或者引用限定符(`reference qualifier`)。例如一个如下声明的模板：
````cpp
template<typename T>
void f(const T& param);	// ParamType代表的类型是const T&
````
我们这样调用它
````cpp
int x = 0;
f(x);	// 用int调用f
````
`T`被推导成`int`，但`ParamType`被推导成`const int&`。

期望`T`代表的类型与传入参数的类型相同是很自然的事情，即`T`就是`expr`的类型。上例中就是如此——`x`是一个`int`，`T`被推导成`int`。但其实并不总是如此。推导出的`T`并不只取决于`expr`的类型，还取决于`ParamType`的形式。一共有三种情况：
* `ParamType`是一个指针或者一个引用，但不是一个通用引用(`universal reference`)。(我们将在条款24讨论`universal reference`，此刻你只需知道它的存在，`universal reference`和左值引用(`lvalue reference`)与右值引用(`rvalue reference`)是不同的。)
* `ParamType`是一个`universal reference`。
* `ParamType`既不是一个指针也不是一个引用。
因此我们就有了三种类型推导场景需要讨论。每一种场景都是基于同样形式的 template 和对它的调用：
````cpp
template<typename T>
void f(ParamType param);

f(expr);	// 从expr推导T和ParamType
````

## 场景1：ParamType 是一个指针或者一个引用，但不是一个通用引用(universal reference)

最简单的情况就是`ParamType`是一个引用或者一个指针，但不是一个`universal reference`。在这种情况下，类型推导的规则如下：

1. 如果`expr`是一个引用，忽略引用。
2. 用`ParamType`的形式与`expr`做对比，决定`T`代表的类型。

例如，我们有这样一个 template：
````cpp
template<typename T>
void f(T& param);	// param是一个引用
````
有如下的变量声明：
````cpp
int x = 27;	// x是一个int
const int cx = x;	// cx是一个const int
const int &rx = x;	// rx是一个对x的const int引用
````
在每个调用中推导出的`param`和`T`如下：
````cpp
f(x);	// T是int，param的类型是int&

f(cx);	// T是const int，
	// param的类型是const int&

f(rx);	// T是const int，
	// param的类型是const int&
````
注意到在第二个和第三个调用中，因为`cx`和`rx`是`const`值，`T`被推导成`const int`，因此变量类型成了`const int&`。这对 caller 很重要，当一个`const`对象作为引用参数传递时，当然不希望这个对象被修改。即参数应该是一个`const`的引用。这就是为什么传递一个`const`对象到一个使用`T&`作为参数的 template 是安全的——参数对象的 constness (常量性)成为了推导出的`T`的一部分。

在第三个调用中，注意到即使`rx`是一个引用，推导出的`T`是一个非引用。那是因为`rx`的引用在推导的过程中被忽略了。

这些例子仅展示了形参类型是 lvalue reference 时类型推导是如何工作的，事实上形参类型是 rvalue reference 时类型推导也是一样的规则。当然，仅有 rvalue 实参可以被传递到 rvalue 形参中，但这个限制与类型推导关系不大。

如果我们将f的参数类型从`T&`变为`const T&`，事情会有一些变化，但都在可预料之中。`cx`和`rx`的 constness 依然会被考虑，但因为我们已经假设了`param`是一个对`const`的引用，`const`便不再成为推导出的`T`的一部分：
````cpp
template<typename T>
void f(const T& param);	// param现在是一个对const的引用

int x = 27;		// 同前
const int cx = x;	// 同前
const int& rx = x;	// 同前

f(x);			// T是int，param的类型是const int&

f(cx);			// T是int，param的类型是const int&

f(rx);			// T是int，param的类型是const int&
````
与之前一样，`rx`的引用在类型推导中被忽略了。

如果`param`不是一个引用，而是一个指针(或者一个指向`const`的指针)，推导的规则基本还是一样。
````cpp
template<typename T>
void f(T* param);	// param现在是一个指针

int x = 27;		// 同前
const int *px = &x;	// px是一个指向const int类型变量x的指针

f(&x);			// T是int，param的类型是int *

f(px);			// T是const int
			// param的类型是const int*
````
现在，你可能正在摇头晃脑地打哈欠，C++ 的类型推导在引用和指针类型的参数上工作得太显然了，还要看大段的文字来描述它实在太无趣了。所有的一切都那么明显！正和你想象的类型推导系统一模一样。

## 场景2：ParamType 是一个 universal reference

对于接受 universal reference 作为参数的 template 来说，事情就没有那么直观了。这些参数在声明处看起来像一个 rvalue reference (即在接受类型参数`T`的 function template 中，对应的 universal reference 就声明为`T&&`)，但如果传入了 lvalue 实参，表现得就不一样了。详细的表述见条款24，此处是一个概要版本。
* 如果`expr`是一个 lvalue，`T`和`ParamType`都会被推导为 lvalue reference。这很不同寻常。首先，在模板类型推导中，这是唯一一种T会被推导为引用(reference)的情况。其次，尽管`ParamType`在以 rvalue reference 的语法形式被声明了，却被推导成了一个 lvalue reference。
* 如果`expr`是一个 rvalue，使用场景1的推导规则。

例如：
````cpp
template<typename T>
void f(T&& param);	// param现在是一个universal reference

int x = 27;		// 同前
const int cx = x;	// 同前
const int& rx = x;	// 同前

f(x);			// x是lvalue，所以T是int&，
			// param的类型也是int&

f(cx);			// cx是lvalue，所以T是const int&,
			// param的类型也是const int&

f(rx);			// rx是lvalue，所以T是const int&,
			// param的类型也是const int&

f(27);			// 27是rvalue，所以T是int，
			// 因此param的类型是T&&

````
条款24解释了为什么这些例子是这样的原因。最关键的原因是对于 universal reference 参数的类型推导规则与参数是 lvalue reference 或者 rvalue reference 的时候不同。特别是当使用 universal reference 时，类型推导会区分 lvalue 实参和 rvalue 实参。这在 non-universal reference 上是永远不会发生的。

## 场景3：ParamType 既不是一个指针也不是一个引用

当`ParamType`既不是一个指针也不是一个引用时，我们实际上通过传值的方式在进行调用：
````cpp
template<typename T>
void f(T param);	// param现在是一个值传递(pass-by-value)
````
这意味着`param`是实参的一份拷贝(copy)——一个完全全新的对象。`param`会是一个全新对象这一事实也影响了`T`将如何从`expr`中被推导出来：

1. 同前，如果`expr`是一个引用，忽略引用。
2. 忽略引用后，如果`expr`是一个 const，也忽略 const。如果`expr`是 volatile，也忽略 volatile。(volatile 对象不怎么常见，它们基本上只在实现设备驱动的时候使用，详细的描述参考条款40.)

因此：
````cpp
int x = 27;		// 同前
const int cx = x;	// 同前
const int& rx = x;	// 同前

f(x);			// T和param的类型都是int

f(cx);			// T和param的类型又都是int

f(rx);			// T和param的类型依然都是int
````
注意到尽管`cx`和`rx`是 const 值，`param`却不是 const。这其实并不矛盾。`param`是一个完全独立于`cx`和`rx`的对象——一个`cx`或`rx`的拷贝。`cx`和`rx`的不可修改并不代表`param`也是如此。这也是`expr`的 constness (常量性)和 volatileness (易失性)在类型推导过程中被忽略的原因——`expr`不能被修改并不代表它的拷贝也不能被修改。

认识到 const 和 volatile 仅在参数是传值进入的时候才被忽略是很重要的。我们可以看到，对于那些传引用进入，或者传指向 const 的指针进入的参数，`expr`的 constness 在类型推导中被保留了。但考虑这样一个情况，`expr`是一个指向 const 对象的 const 指针，并且通过值传递(pass-by-value)给`param`：
````cpp
template<typename T>
void f(T param);		// param依然是值传入

const char* const ptr =		// ptr是指向const对象的const指针
	"Fun with pointers";

f(ptr);				// 实参类型是const char * const
````
此处，星号右侧的 const 声明了`ptr`是一个 const 指针——`ptr`不能再指向别处，也不能置空。(星号左侧的 const 表明`ptr`指向的对象，也就是那个字符串，是 const 的，因此该字符串不能被修改。)当`ptr`被传递给`f`，指针的内容原样拷贝给`param`。因此，指针本身(`ptr`)通过值传递给参数。根据值传递参数的类型推导规则，`ptr`的 constness 将被忽略，推导出的`param`类型会是`const char*`，也就是一个可以指向别处的指针指向一个不能被修改的字符串对象。`ptr`指向对象的 constness 在类型推导的过程中被保留了，`ptr`本身的 onstness 在创建新指针`param`的拷贝过程中的却被忽略了。

## Array Arguments 数组实参

(Unfinished yet)
