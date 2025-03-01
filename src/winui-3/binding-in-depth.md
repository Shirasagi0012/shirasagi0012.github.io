# 深入`{x:Bind}`

在使用XAML描述界面的时候，我们使用数据绑定来实现View层和ViewModel层的解耦合。XAML提供了一些标记扩展（Markup Extension）,比如`{x:Bind}`、`{Binding}`、`{TemplatedBinding}`，借助它们，我们可以直接在XAML中轻松的描述一个数据到界面元素依赖属性的绑定关系。在WinUI 3/UWP中，我们可以同时使用以上三种绑定标记扩展，对于初学者而言很可能会引起不少疑惑。如果你接触过WPF，可能在初次使用`{x:Bind}`时会感觉熟悉而又陌生。

`{x:Bind}`是伴随着WinRT XAML（也就是UWP、WinUI 3所使用的XAML）而诞生的。它旨在解决`{Binding}`的一些问题。`{x:Bind}`也拥有比`{Binding}`更丰富的特性，并且语法更简洁。

- `{x:Bind}`支持使用一个函数作为绑定路径
- `{x:Bind}`支持绑定事件
- `{x:Bind}`是强类型的、静态的。
- `{x:Bind}`是编译期的绑定，在编译时生成对应的绑定代码。

在使用`{Binding}`时，我们需要和数据上下文（`DataContext`）打交道。如果不设置正确的数据上下文，就会找不到对应的绑定元素和路径，导致绑定失败。

与`{Binding}`不同，`{x:Bind}`不需要、也不能设置数据上下文，因为`{x:Bind}`并不具有`{Bidning}`的动态性，无法在运行时判断某个数据上下文（是`Object`类型）里是否有需要绑定的元素。`{x:Bind}`将XAML根元素作为其查找元素和属性的根。

