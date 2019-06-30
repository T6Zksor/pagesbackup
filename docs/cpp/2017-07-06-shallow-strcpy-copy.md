# strcpy 到 ZString 的奇怪现象
2017-07-06

项目中有一个自己实现的 string 类，这里把它叫做 ZString 吧。

struct Type 有一 ZString 成员变量 a，getdata 得到指针后传入函数，函数内使用了 strcpy。之后所有 Type 变量初始化都会导致原先的 Type 变量内 a 内容丢失。

原因，Type 构造函数中对成员 a 初始化，a=""，用 strcpy 实际将数据拷贝到了空字符串的内存区，导致新建 Type 变量时，新的 a 获取了拷贝到此处的数据，并在随后的操作中将其清除，导致原始 a 看不到数据。

结论，浅拷贝深拷贝的变种问题。