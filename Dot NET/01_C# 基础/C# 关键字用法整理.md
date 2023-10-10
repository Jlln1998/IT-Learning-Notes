# C# 特殊关键字用法

## Checked

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

## as

> 引用类型转换，如果不能转换则会转换成 null

## is

> 用来判断对象是不是某种类型

## static

> 定义静态成员

## dynamic

> 推迟校验类型

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

