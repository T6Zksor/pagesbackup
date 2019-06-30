# Preprocessor line number error
2017-05-12

项目有个自己实现的 C-like 方言，配套有一个 IDE，这里叫做 Code Creator 吧。

在使用 Code Creator debug 时候出现了一个 bug：当函数 body 中某些行使用了 backslash 进行换行 escape 之后，body 中剩余部分的 code 便无法在 step debug 中跳转到正确的行号，相差的行数正好是上方 body 中 escape 过的行数。

尽管此时程序的运行和逻辑都没有问题，甚至 call stack 和 watch 窗口的变量更新也都正确，但这种无法准确得知运行到何处的情况依然令人不爽。

尝试找出问题所在，

1. 首先确定出问题的 code 大致在什么范围，既然运行时行号错乱了也不影响逻辑，说明是在编译阶段记录语句行号时记录了错误的行号。在 Code Creator 中设置 break point 进入 step debug，然后在 Visual Studio 中 break，很容易发现确实是在 compiler 的 getlinenumber 函数中获得了错误的 line number，而 line number 是在 compile 阶段 parsing code 后保存在一个记录 code 内容与 line number 的 map 中得到的，问题初步确认。

2. 往上继续寻找 map 的构建，找到 map 是在 preprocessor 处理完毕后，获得预处理后的 code 并用于进行 init。

3. 写了一个 test.c 并编译，进入 init 中查看预处理后的 code，发现果然对 escape line 的处理存在缺陷，escape 的部分在预处理后变成了一行，而后面却没有考虑已经 escape 了数行，行号继续从下一行开始计算，出现错误。

4. 考虑在 escape 的行后补充 line feed，使预处理后的行能和原始的 code 保持对齐。这里有一个问题，就是 #define 的宏在使用时是占用了一行，有些宏命令会通过 backslash escape 来定义一些 readable 的宏，如果在调用处也对它进行 line feed，也会导致行号的错误。

5. 在 preprosessor 中找到了 define 相关的 code，发现在调用处替换时已经处理成一行，不会有 backslash escape，而定义处的 line feed 也不在一个 body 中，不会影响到。

6. 在 preprosessor 处理 backslash escape 的 code 后面添加补充 line feed 的内容，纪录下 escape 了多少行，并在 getstringnocomment 最终补充了 CRLF，由于原先的 code 也有添加 CRLF 的部分，这里没有考虑其它平台，我也就照做暂时只针对 Windows 的 CRLF 进行添加。

7. 编译后work了。

8. 随后又发现 bug，在运行完当前 body 跳转到 caller 时，行号又错乱了，这是原来没有的情况，说明我们的改动并不完美。继续用同样的方法跟踪，发现问题出在 /**/ 类型的多行注释上，原来的 code 已经针对这种多行注释进行了 line feed，而在我的修改过后其实已经对所有在 preprocessor 中这种多行文本当作一行的情况进行了 line feed。

9. 去掉原先的line feed，完成。

自此花费一个下午 Code Creator 的这个老 bug 便处理完成，难度不是很大，但需要很多耐心在跟踪 preprocessor 的处理上，也对公司的 compiler 有了初步的了解。