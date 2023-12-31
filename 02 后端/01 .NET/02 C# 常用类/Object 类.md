# Object 类

> 官方介绍：[Object类](https://learn.microsoft.com/zh-cn/dotnet/api/system.object?view=net-7.0)
>
> Object 类是所有 .NET 类的最终基类；是类型层次结构的根。在 C# 的统一类型系统中，所有类型（预定义类型、用户定义类型、引用类型和值类型）都是直接或间接从 Object 类继承的。
>
> 自定义定义类时，自动隐式继承 Object 类。

Object 类常用方法如下：

| 方法                                                         | 说明                                       |
| ------------------------------------------------------------ | ------------------------------------------ |
| [ToString](https://learn.microsoft.com/zh-cn/dotnet/api/system.object.tostring?view=net-7.0) | 制造描述类实例的可读文本字符串。           |
| [Equals](https://learn.microsoft.com/zh-cn/dotnet/api/system.object.equals?view=net-7.0) | 支持对象之间的比较。                       |
| [Finalize](https://learn.microsoft.com/zh-cn/dotnet/api/system.object.finalize?view=net-7.0) | 在自动回收对象之前执行清理操作。           |
| [GetHashCode](https://learn.microsoft.com/zh-cn/dotnet/api/system.object.gethashcode?view=net-7.0) | 生成与对象值对应的数字，以支持使用哈希表。 |
| [GetType()](https://learn.microsoft.com/zh-cn/dotnet/api/system.object.gettype?view=net-7.0#system-object-gettype) | 获取当前实例的 Type，                      |
