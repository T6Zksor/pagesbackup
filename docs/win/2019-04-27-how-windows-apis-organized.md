# How Windows APIs organized

date: 2019-04-27

## story

一名开发者，不论是进行应用层面的开发，还是内核层面的开发。频繁或偶尔的都需要使用到系统环境提供的一些功能，最常见的如发起一次网络连接、处理硬件传来的中断请求等。而这些本质上是向作为软硬件资源管理角色的操作系统请求资源的调用，是脱离于语言本身存在的。是通过一组接口向操作系统发起和接收，这组接口就是操作系统 API。

为了方便程序语言调用，Windows API 封装成了 C interface 的形式。任意一个 API，都可以在 Windows SDK 提供的 include 文件中找到对应的 C 形式声明，以及对应的 lib 供 loader 在 link 阶段进行 symbol 查找。

## DLLs linked before

早期的 Windows API 分工明确，不同的功能划分至不同的模块，接口封装于不同的 DLL 中，十分清晰。
比如常见的几个 library，kernel32.dll/user32.dll/gdi32.dll，几乎在所有桌面应用中都需要使用到这三个 DLL 提供的 API，进行基本的系统调用。

## API Sets

~~~从 Windows 7 开始，Microsoft 提供了一种新的 API 组织方式，可以令开发者在大部分情况下更方便的处理 API 依赖的问题，而不需要再去关心这个 API 在哪个 library 里这样琐碎的问题，同时也方便了 old codebase 迁移到新项目中依赖的处理。~~~

~~~查看编译到 Windows 7 之后平台的程序，会在它们的依赖项中找到命名规则形如~~~


从 Windows 7 开始，Microsoft 改变了以往组织 API 的方式。如果在 Windows 7 之后的 OS 查看一个 executable 的 dependency，可以发现命名格式如下的 DLL。

```
...
api-ms-win-core-com-l1-1-1.dll
api-ms-win-core-com-midlproxystub-l1-1-0.dll
api-ms-win-core-comm-l1-1-0.dll
api-ms-win-core-console-ansi-l2-1-0.dll
api-ms-win-core-console-l1-1-0.dll
api-ms-win-core-console-l2-1-0.dll
api-ms-win-core-datetime-l1-1-1.dll
api-ms-win-core-datetime-l1-1-2.dll
api-ms-win-core-debug-l1-1-1.dll
api-ms-win-core-debug-l1-1-2.dll
api-ms-win-core-delayload-l1-1-1.dll
api-ms-win-core-errorhandling-l1-1-1.dll
api-ms-win-core-errorhandling-l1-1-3.dll
api-ms-win-core-fibers-l1-1-1.dll
api-ms-win-core-fibers-l2-1-1.dll
api-ms-win-core-file-ansi-l1-1-0.dll
api-ms-win-core-file-ansi-l2-1-0.dll
api-ms-win-core-file-l1-2-1.dll
api-ms-win-core-file-l1-2-2.dll
api-ms-win-core-file-l2-1-1.dll
api-ms-win-core-file-l2-1-2.dll
...
```

按照 Microsoft 自己的说法是为了对 host DLL 和 API 解耦，这样才能让 Microsoft 在 ***Windows as a Service*** 的口号里大展拳脚。

如果反汇编这组 DLL，会发现它们其实什么都没做。API Sets 实际上并不是真正的 DLL，只是一个 `strong name` ，Windows 在 load DLL 的时候会将 DLL 中的入口替换为 kernel 中实际的 API 地址。

## Umbrella Library

然而这样细化的分类，如果依然要求显式的指定 link 到哪个 DLL，显然是更加烦琐。Microsoft 提供了一个叫做 Umbrella library 的机制，其实就是一个文件名为 `OneCore.lib` 的 library。在 project settings 中只需要 link OneCore.lib 一个，它会自动处理需要 link 到哪些 DLL，一劳永逸。

当然，这也造成了一些兼容性问题，一个 link 到 OneCore.lib 的 app，显然没办法继续在 Windows XP 上使用了——并不见得是一件坏事 :)

## References

- https://www.nirsoft.net/articles/windows_7_kernel_architecture_changes.html
- https://docs.microsoft.com/en-us/windows/desktop/apiindex/windows-apisets
- https://blog.quarkslab.com/runtime-dll-name-resolution-apisetschema-part-i.html
- https://www.geoffchappell.com/studies/windows/win32/apisetschema/index.htm?tx=2