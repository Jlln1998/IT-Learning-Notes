# C# 特殊关键字用法

| 关键字       | 用法                                                         |
| ------------ | ------------------------------------------------------------ |
| as           | 引用类型转换，如果不能转换则会转换成 null                    |
| is           | 用来判断对象是不是某种类型                                   |
| static       | 定义静态成员                                                 |
| dynamic      | 推迟校验类型                                                 |
| Checked      | 如果代码区域中存在栈溢出，则会抛出报异常                     |
| **abstract** | 定义抽象方法和抽象类                                         |
| **sealed**   | 定义密封方法，表示该类不能被继承                             |
| readonly     | 定义字段或者属性，表示只读，类似与只定义了get访问器的字段或属性。 |
| partial      | 局部类型。允许我们将一个类、结构或接口分成几个部分，分别实现在几个不同的.cs文件中。C#编译器在编译的时候仍会将各个部分的局部类型合并成一个完整的类。 |



## Checked 案例

> 进行进行栈溢出检查。如果代码区域中存在栈溢出，则会抛出报异常。

````C#
sbyte b = 127;
b += 55;
Console.WriteLine(b);//会出现栈溢出丢失数据情况，会得到错误数据。
````

````c#
sbyte b = 127;
checked //当出现栈溢出情况时，会抛出异常。
{
    b += 55;
}
Console.WriteLine(b);
````

## dynamic 案例

> 推迟校验类型，动态类型。

````c#
class Tool
{
    //泛型方法
    public T add<T>(T a,T b)
    {
        dynamic? da = a;
        dynamic? db = b;
        return da + db;
    }
}

Tool tool = new Tool();
Console.WriteLine(tool.add<int>(1, 2));
Console.WriteLine(tool.add<double>(1.11, 2.22));
````

