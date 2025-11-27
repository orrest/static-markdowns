---
tags:
    - WPF
    - Frontend
icon: fontawesome/solid/desktop
---

> https://learn.microsoft.com/en-us/dotnet/desktop/wpf/events/object-lifetime-events

1. Window()
2. Window.InitializeComponent
3. UserControl()
4. UserControl.InitializeComponent
5. UserControl.Initialized
    - 元素的属性已被设置
    - 构造函数执行完毕
    - 子元素已经初始化完成
    - 但是 Initialized 事件并不等待所有元素及其子元素被初始化完毕再发出
6. Window.Initialized
7. UserControl.Loaded
    - When the logical tree that contains the element is complete and connected to a presentation source. The presentation source provides the window handle (HWND) and rendering surface.
    - 数据绑定已经完成
    - 布局系统已经完成所有渲染所需的计算
    - 在最终渲染之前
    - Loaded 事件会等待所有逻辑树上的元素都加载完成，自上而下传播，类似 tunneling
    - 在 Activaed 事件发出后才会发出
8. Window.Loaded
    - 需要加载数据的时候
9. Window.ContentRendered
    - 需要操作组件的时候
10. Window.Unloaded
11. UseControl.Unloaded
    - 当 Unload 事件来到子元素的时候，它的父级可能已经 unset 了，这意味着父级的绑定、资源引用、样式的值已经不正确了
