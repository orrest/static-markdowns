---
tags:
    - WPF
    - Frontend
    - Navigation
icon: fontawesome/solid/desktop
---

## 问题

从 DI 容器中获得实例的时候，需要泛型或者 Type 实例，如果直接通过类型或者 `typeof` 关键字获取，都会使得当前项目依赖于所需要导航到的视图所属的项目（同一个项目内是无所谓，但一旦开始模块化这就会造成耦合）。

在 `简单导航` 一文中，通过使用中介的枚举变量来解耦这种导航间的依赖关系，但还是需要手动维护 `switch case`。

在 Prism 中，可以直接通过字符串来进行视图的定位，从而导航：

```C#
_regionManager.RequestNavigate(RegionNames.ContentRegion, "DashboardView");
```

那么它是如何实现从字符串到视图之间的映射的？

## 从字符串到视图

在 Prism 中，定义了 `ViewRegistration` 类来保存视图的名字以及视图相关的信息。

```C#
namespace Prism.Mvvm;

 /// <summary>
/// Represents information about a registered view.
/// </summary>
public record ViewRegistration
{
    /// <summary>
    /// Gets the type of view this registration represents (Page, Region, or Dialog).
    /// </summary>
    public ViewType Type { get; init; }

    /// <summary>
    /// Gets the type of the view class associated with this registration.
    /// </summary>
    public Type View { get; init; }

    /// <summary>
    /// Gets the type of the view model associated with this registration, if any.
    /// </summary>
    public Type ViewModel { get; init; }

    /// <summary>
    /// Gets the unique name used to identify this view registration.
    /// </summary>
    public string Name { get; init; }
}
```

```C#
namespace Prism.Mvvm;

/// <summary>
/// Enumerates the different types of views supported by the framework.
/// </summary>
public enum ViewType
{
    /// <summary>
    /// Unknown view type.
    /// </summary>
    Unknown,

    /// <summary>
    /// Represents a full-screen page or window.
    /// </summary>
    Page,

    /// <summary>
    /// Represents a reusable region within a view.
    /// </summary>
    Region,

    /// <summary>
    /// Represents a modal dialog or popup window.
    /// </summary>
    Dialog,
}
```

## RegisterForNavigation

注册视图的同时也注册了这个字符串-视图关系。

```C#
22    public static IServiceCollection RegisterForNavigation(this IServiceCollection services, Type view, Type viewModel, string name = null)
23    {
24        if (view is null)
25            throw new ArgumentNullException(nameof(view));
26
27        if (!view.IsAssignableTo(PageType))
28            throw new InvalidOperationException($"The view type '{view.FullName}' is not a type of Page.");
29
30        if (string.IsNullOrEmpty(name))
31            name = view.Name;
32
33        services.AddSingleton(new ViewRegistration
34            {
35                Type = ViewType.Page,
36                Name = name,
37                View = view,
38                ViewModel = viewModel
39            })
40            .AddTransient(view);
41
42        if (viewModel != null)
43            services.AddTransient(viewModel);
44
45        return services;
46    }
```

## 根据字符串创建视图

`ViewRegistryBase` 是用于注册和创建视图的。

DI 容器会自动收集所有相同类型的实例作为一个集合，注入 `ViewRegistration` 集合：

```C#
protected ViewRegistryBase(ViewType registryType, IEnumerable<ViewRegistration> registrations)
{
    _registrations = registrations;
    _registryType = registryType;
}
```

然后就可以根据 name 筛选出所需 ViewRegistration 的实例，并通过其中的 type 等从容器中取得对应的视图实例

```C#
public object? CreateView(IContainerProvider container, string name)
{
    try
    {
        var registration = GetRegistration(name) ?? throw new KeyNotFoundException($"No view with the name '{name}' has been registered");
        var view = container.Resolve(registration.View) as TBaseView;
        // ...
```

```C#
/// <summary>
/// Gets the registration information for a view with the specified name, or null if not found.
/// </summary>
/// <param name="name">The name of the view to look up.</param>
/// <returns>The view registration object, or null if not found.</returns>
protected ViewRegistration? GetRegistration(string name) =>
    Registrations.LastOrDefault(viewRegistration => viewRegistration.Name == name);
```

## 总结

Prism 中主要通过向 DI 容器中注册 `ViewRegistration` 实例，达到能够在 DI 容器中获得字符串到类型映射的目的，同时 DI 容器又能够通过类型获得注册的视图实例。

仅出于导航的目的的话可以直接将视图注册为 KeyedService。