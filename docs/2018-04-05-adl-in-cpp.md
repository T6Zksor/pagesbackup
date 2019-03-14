# C++中的 Argument-dependent lookup

date: 2018-4-5

C++中有一项很少被提及，也不怎么被主动使用的特性，就是`Argument-dependent lookup`(ADL),亦即参数依赖的查找。

所谓`Argument-dependent lookup`，又称为`Koenig lookup`，是指在函数调用的过程中，对于 unqualified 的函数名称，有一组规则来找寻这个函数名称所对应的实际函数。除去查找了当前的 scope 和 namespace 后，还会查找实参所在的 namespace 来进一步确认将被调用的是哪个函数。

## Example

ADL最常见的应用之一就是`std::cout`了。参考如下代码：
``` cpp
#include <iostream>
int main()
{
    std::cout << "hello world!"
    return 0;
}
```
作为C++难得的语法糖之一，`<<`运算符实际调用的是`operator <<(arg1, arg2)`，因此上方的语句等价于
``` cpp
operator <<(std::cout, "hello world!");
```
而此处就产生了一个疑问，`operator<<`并未存在于当前 scope 或者 global namespace 中，如果按照严格的语法规则，应该调用的语句是
``` cpp
std::operator <<(std::cout, "hello world!");
```
事实上，如上语句不但编译通过了，还工作得非常好，成为了无数 C++ 学习者第一次写下的代码。这其中起到助力的就是`ADL` 。通过`ADL`的规则，C++ 在查找当前 scope 和 global namespace 无果后，又查找了`std::cout`所在的`namespace std`，在其中找到并成功调用了`std::operator <<`。

## Back Fire

C++ 规定了`ADL`的应用场景，仅限 usual unqualified lookup 没有查找成功时才会使用`ADL`。看起来似乎只要在当前的 scope 或 namespace 能够找到对应的函数，`ADL`就不会启用，也不会造成我们调用其它 namespace 的函数，如此我们就可以高枕无忧了？其实不然，考虑下面的代码：
``` cpp
namespace aa {
	struct a_t {};
	void trap(a_t) {
		std::cout << "in namespace aa" << std::endl;
	}
}

void trap(...) {
	std::cout << "in namespace bb" << std::endl;
}

int main()
{
	trap(42);
	trap(aa::a_t());

	return 0;
}
```
输出的结果为：
``` shell
in namespace bb
in namespace aa
```
显然，如上的例子看起来像是`ADL`在某些情况下被触发了，造成我们的代码调用了非预期的函数。

在 global namespace 中，我们有函数`trap`，参数不定。在`namespace aa`中，我们有同名函数`trap`，接受类型为`a_t`的参数。当传入实参满足了`ADL`的条件，`namespace aa`中的`trap`函数被“意外”调用了。当然，这样的场景条件比较严苛。首先需要两个`namespace`中有同名函数，其次需要这两个同名函数的参数是可“兼容”的，即同样的参数对两个函数都合法。实际上能满足这样条件的参数定义也只有可变参数了。

问题是，上述的例子是否真的是“非预期”的？

事实上 C++ 标准对于上面的样例有着明确的定义。例中是场景实际上是两个函数名称相同参数不同的重载情况，编译器在此需要判断的是，对于这两个重载函数，应该选用哪一个作为实际被调用的函数。

这两个候选函数，一个的参数是有明确类型的，另一个是`variadic arguments`。对于 C 时代沿袭下来的`variadic arguments`，文档中已有明确说明参数为`variadic arguments`的候选函数在重载选择中优先级是最低的。
> variadic parameters have the lowest rank for the purpose of overload resolution

可见，在上述的例子中，并非`ADL`被非预期的调用了。而是 C++ 认为这两个同名函数是一种重载，因为`variadic arguments`的重载优先级最低，所以优先调用了另一版本的重载，给我们造成了一种非预期调用的错觉。

## References
- http://en.cppreference.com/w/cpp/language/adl
- http://en.cppreference.com/w/cpp/language/variadic_arguments
- https://en.wikipedia.org/wiki/Argument-dependent_name_lookup
