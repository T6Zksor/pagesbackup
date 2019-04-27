# 销毁 window 时 child window 和 owned window 收到消息的区别

date: 2019-02-22

## a story

最近遇到了一个问题。创建一个 win32 window 时，有时需要 window 是 child window，有时是 owned window。这二者在它们的 owner/parent window 销毁时，收到消息的表现略有不同。

## delve into

通常的 win32 application 而言，常见的一个 windows message loop 形式如下，

```cpp
MSG msg = {};

while( (bRet = GetMessage( &msg, NULL, 0, 0 )) != 0)
{
    if (bRet == -1)
    {
        // handle the error and possibly exit
    }
    else
    {
        TranslateMessage(&msg); 
        DispatchMessage(&msg); 
    }
}
```
一个 window 可以选择在收到 `WM_CLOSE` 销毁窗口。在 wndproc 中使用
```cpp
    case WM_CLOSE:
        DestroyWindow(hwnd);
        return 0;
```
`DestroyWindow` 会通知 OS 销毁当前 window。有时需要 window 知道自己将要被销毁，从而进行一些 cleanup。通常而言的做法是以收到 `WM_DESTROY` 为标志，在收到 `WM_DESTROY` 后在 window 对应的 object 中进行 cleanup。  
然而在 debug 的过程中，发现如果这个 window 有一个 child window，或者 owned window，它们收到 `WM_DESTROY` 的顺序是不一样的。

| window type | parent/child | owner/owned |
| ------------- | ------------- | ----- |
| first receive WM_DESTROY | parent window | owned window |
| second receive WM_DESTROY | child window | owner window |

这就令人好奇，结构上十分相似的两种 window 关系，在一个关键消息的接受上顺序却截然相反。难道不是一个不好的设计吗？如果假设了两个窗口收到 `WM_DESTROY` 的顺序并在代码中依赖这种顺序，甚至可能埋下一些 bug。

查阅了一下文档，根据 `DestroyWindow` 中的描述，
> If the specified window is a parent or owner window, DestroyWindow automatically destroys the associated child or owned windows when it destroys the parent or owner window. The function first destroys child or owned windows, and then it destroys the parent or owner window.

明确表述了，child/owned window 会先被销毁，parent/owner window 后被销毁。这就令人费解，明明实际观察到的现象是 child/owned window 收到 `WM_DESTROY` 顺序相反。看来是自己对这个过程有什么误解？

参考了网上的一些资料和文档，最终找到了答案，这里总结一下。

## summary
- 实际的窗口销毁顺序，是 child/owned window 先， parent/owner 后
- 而 child/parent 和 owned/owner 收到 `WM_DESTROY` 的顺序的确不一样
- 收到 `WM_DESTROY` 不代表销毁完毕，文档只保证 window 已经从屏幕上移除，处于销毁的一系列过程中
- 收到 `WM_NCDESTROY` 才是代表 window 已经被销毁，有些 `delete this` 的用法就是在此时执行
- 阅读文档很重要，切忌望文生义

--------
## references
- https://docs.microsoft.com/en-us/windows/desktop/winmsg/using-messages-and-message-queues
- https://docs.microsoft.com/en-us/windows/desktop/api/winuser/nf-winuser-destroywindow
- https://stackoverflow.com/questions/3155782/what-is-the-difference-between-wm-quit-wm-close-and-wm-destroy-in-a-windows-pr/3155879#3155879
- https://blogs.msdn.microsoft.com/oldnewthing/20050726-00/?p=34803