---
tags:
    - WPF
    - FrontEnd
icon: fontawesome/solid/desktop
---

- UserControl：适用于粒度较大，需要自己的 ViewModel 的组件；
- Custom Control：适用于粒度小，无额外状态或状态较少的组件；

## UserControl

- 形式上是 .xaml.cs + .xaml；
- 多个组件的组合，一般用于页面内容（或者用 Page）；
- 一般有自己对应的 viewModel；

总的来说适用于粒度比较大，其中引用多个无状态组件，但是自己本身有状态（viewModel）的组件情况。

实现步骤：

1. 创建 UserControl（.xaml.cs, .xaml 文件）；
2. 创建对应的 ViewModel；
3. 在 ViewModel 中声明用于绑定的属性；
4. 在 .xaml 中进行绑定；

如果情况比较简单，不使用 ViewModel，也可以在 .xaml.cs 中声明依赖属性，将 DataContext 设为控件自己。

> 使用依赖属性时，如果仅有一个“主属性”，其它属性都是由主属性计算得来，那么在声明依赖属性时，利用 PropertyMetadata 中的 onPropertyChanged 回调进行其它依赖于它的属性的计算即可。

## Custom control

1. 可以直接手动创建 .cs 文件，继承 System.Windows.Controls.Control（不需要 .xaml 文件了）;
2. 声明组件的 Style（对于控件样式使用 TemplateBinding，对于自定义的依赖属性，使用 Binding）；
    ``` XML
    <Style TargetType="{x:Type local:MyCustomControl}">
        <Setter Property="Template">
            <Setter.Value>
                <ControlTemplate TargetType="{x:Type local:MyCustomControl}">
                    <Border Padding="10" Background="LightGray">
                        <TextBlock Text="{TemplateBinding Title}" FontSize="16"/>
                    </Border>
                </ControlTemplate>
            </Setter.Value>
        </Setter>
    </Style>
    ```

3. 在 App.xaml 中（或者所需控件中）添加相关控件所在的资源字典；
