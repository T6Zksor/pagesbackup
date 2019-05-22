# C1002: Compiler is out of heap space in pass 2

## a story

今天使用VS编译一个大型项目的时候，遇到了一个从来没有见过的报错。

> C1002: compiler is out of heap space in pass 2

一开始还以为是源码中哪里出现了错误，根据 IDE 提示的位置跳转到出错的 target file，单独编译没有遇到语法错误。并且，报错发生的时机是所有源文件都编译完毕后，已经进入 link 阶段了。

经过一番查询，发现了一个比较接近的错误码，

> C1060: compiler is out of heap space

MSDN 提供的解决方案是这样的，

> To fix this error try the following possible solutions
> 1. If the compiler also issues errors C1076 and C3859, use the /Zm compiler option to lower the memory allocation limit. More heap space is available to your application if you lower the remaining memory allocation.
>
>    If the /Zm option is already set, try removing it. Heap space might be exhausted because the memory allocation limit specified in the option is too high. The compiler uses a default limit if you remove the /Zm option.
>
> 1. If you are compiling on a 64-bit platform, use the 64-bit compiler toolset. For information, see How to: Enable a 64-Bit Visual C++ Toolset on the Command Line.
>
> 1. On 32-bit Windows, try using the /3GB boot.ini switch.
>
> 1. Increase the size of the Windows swap-file.
>
> 1. Close other running programs.
>
> 1. Eliminate unnecessary include files.
>
> 1. Eliminate unnecessary global variables, for example, by allocating memory dynamically instead of declaring a large array.
>
> 1. Eliminate unused declarations.
>
> 1. Split the current file into smaller files.

简单来说就是，out of memory 了。

又观察了一下在 VS 中 link 出错时任务管理器中大概的进程资源占用，出错时 link.exe 内存占用刚刚超过 3G 多一点，离整台机器的物理内存上限还有比较远。看来是因为 devenv.exe 本身是 32 位造成了进程内存使用量的限制。

参考 MSDN 给出的第二点，需要使用 64bit 的 link.exe 进行 link。

使用方法也很简单，在 cmd 里跑一下 vcvars64.bat 初始化 vs command prompt 环境，然后运行 msbuild 命令进行编译即可。

``` cmd
call "path/to/vcvars64.bat"
msbuild project_name.vcxproj -t:build -p:Configuration=Release;Platform=x64

pause
```

在命令行中直接跑 devenv 是行不通的，因为 devenv 只有 32bit 的，在 link 阶段依然是调用 32bit 的 link.exe。所以还是要用 msbuild。

另外，也有其他方法绕过，就是设置项目 link 属性中的 `/GL` 和 `/LTCG`，来减少 link 阶段的工作量，降低内存占用。不过会影响生成的 binary 大小。

## summary

- 报错的原因是 link.exe 的 heap 内存耗尽了
- 可以设置 `/GL` 和 `/LTCG` 减少 link.exe 工作量来减少内存使用
- 可以使用 VS x64 Native Command Prompt 来调用 64bit 的 link.exe，充分使用硬件内存

--------
## references
- https://docs.microsoft.com/en-us/cpp/error-messages/compiler-errors-1/fatal-error-c1060
- https://docs.microsoft.com/en-us/cpp/build/how-to-enable-a-64-bit-visual-cpp-toolset-on-the-command-line
- https://docs.microsoft.com/en-us/cpp/build/reference/gl-whole-program-optimization
- https://docs.microsoft.com/en-us/cpp/build/reference/ltcg-link-time-code-generation