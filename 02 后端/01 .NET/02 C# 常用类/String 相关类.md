# String 相关类

## 1. String 字符串

### 1.1 介绍

官方介绍：[字符串类型](https://learn.microsoft.com/zh-cn/dotnet/api/system.string?view=net-7.0)

> String 字符串类又称：
>
> * 字符串常量
> * 不可变字符数组

### 1.2 字符串不可变含义

> ①假设我们定义了一个字符串变量str，并定义了初始值hello，系统会在堆中开辟一个空间，存储hello字符串，并把存储的内存地址赋值给str；
>
> ②当我们修改 str 重新赋值为 world时，实际是在堆中新开辟了一个空间来存储world字符串，然后把这个world字符串的地址赋值给字符串变量str。
>
> 所以，原先堆中的hello字符串，并没有被改变，只是不再被引用了而已。

例1：

````c#
string str = "hello";
str = "world";
````

例2：

> 实参name，传入方法时，是传入字符串“张三”在堆空间的地址值。
>
> 当 str 修改时，因为字符串是不可变的，修改 str 变量时实际是让 str 指向了一个新的堆空间地址。
>
> str 的指向改变了，然而 name 的指向从始至终并没有做修改，所以仍然为 张三。

````C#
internal class Program
{
    static void Main(string[] args)
    {
        string name = "张三";
        changeStr(name);
        Console.WriteLine(name);//输出仍然是张三
    }
    static void changeStr(string str)
    {
        str = "222";
    }
}
````

### 1.3 字符串常量池

> * 用字面量定义字符串 str，系统会先在常量池中寻找是否之存有该字符串，
>
>     如果有，则直接把常量池中的该字符串的内存地址赋值给 str；
>
>     如果没有，则在常量池中新开辟一个空间，存储该字符串，并把内存地址赋值给 str;
>
> * 用变量运算定义的字符串，则是每次直接在堆中开辟新空间。

````c#
static void Main(string[] args)
{
	string str1 = "hello";  //字面量创建，存储在常量池中。
	string str2 = "world";  //字面量创建，存储在常量池中。
	string str3 = "helloworld";//字面量创建，存储在常量池中。
	string str4 = str1 + str2;//【变量运算】创建的字符串，存储在堆中。
    
    //false，虽然值都是 helloworld，但是一个在常量池中，一个在堆中，不是同一个东西。 
	Console.WriteLine(Object.ReferenceEquals(str3, str4));//False
    
    //str5 和 str4 引用的堆中同一个东西
	string str5 = str4;
	Console.WriteLine(Object.ReferenceEquals(str4, str5));//True
    
    // 对str4的修改,实际是在堆创建了一个新的空间存储新字符串，所以str4和str5将不再是同一个东西。
	str4 += str3; 
    Console.WriteLine(Object.ReferenceEquals(str4, str5));//False
    
	//str6 字面量创建，创建前在常量池寻找到已存有 helloworld，所以会直接和 str3 引用同一个地址。
	string str6 = "helloworld";
    Console.WriteLine(Object.ReferenceEquals(str6, str3));//True,
    
    //字面量的拼接，也是会从常量池中寻找
    string str7 = "hello" + "world";
    Console.WriteLine(Object.ReferenceEquals(str7,str3));//True
	Console.WriteLine(Object.ReferenceEquals(str7, str6));
    
    //经过变量运算的结果，都会存在堆里，不会存在常量池中。
    string str8 = str1 + "world";
	Console.WriteLine(Object.ReferenceEquals(str8, str3));//False
}
````

### 1.4 字符串模板

> 在字符串前使用 $ 符号，可直接引用作用域中的成员

````C#
string name = "张三";
int age = 10;
string introduce = $"姓名:{name},年龄：{age}";
Console.WriteLine(introduce);
````

### 1.5 格式字符串

> 使用 String 类的 format() 或者 toString() 方法，将数据按指定格式输出。
>
> 官方文档：[设置数字、日期以及其他类型数据格式](https://learn.microsoft.com/zh-cn/dotnet/standard/base-types/formatting-types)

````c#
decimal value = 123.456m;
Console.WriteLine(value.ToString("C2"));//￥123.46
Console.WriteLine("Your account balance is {0:C2}.", value);

string format = string.Format("{0},{1}",3,4);
Console.WriteLine(format);
````

### 1.6 String 常用Api

#### 1.6.1 汇总

| 方法                      | 作用                                                         |
| ------------------------- | ------------------------------------------------------------ |
| Length                    | 获取字符串的长度，即字符串中字符的个数                       |
| Empty                     | public static readonly string                                |
| IndexOf                   | 返回整数，得到指定的字符串在原字符串中第一次出现的位置       |
| LastlndexOf               | 返回整数，得到指定的字符串在原字符串中最后一次出现的位置     |
| StartsWith                | 返回布尔型的值，判断某个字符串是否以指定的字符串开头         |
| EndsWith                  | 返回布尔型的值，判断某个字符串是否以指定的字符串结尾         |
| ToLower                   | 返回一个新的字符串，将字符串中的大写字母转换成小写字母       |
| ToUpper                   | 返回一个新的字符串，将字符串中的小写字母转换成大写字母       |
| Trim                      | 返回一个新的字符串，不带任何参数时表示将原字符串中前后的空格删除。 参数为字符数组时表示将原字符串中含有的字符数组中的字符删除 |
| TrimStart                 | 返回一个新的字符串，将字符串中左侧的空格或指定字符数组中的字符删除 |
| TrimEnd                   | 返回一个新的字符串，将字符串中右侧的空格或指定字符数组中的字符删除 |
| Remove                    | 返回一个新的字符串，将字符串中指定位置的字符串移除           |
| PadLeft                   | 返回一个新的字符串，从字符串的左侧填充空格（或指定字符串）达到指定的字符串长度 |
| PadRight                  | 返回一个新的字符串，从字符串的右侧填充空格（或指定字符串）达到指定的字符串长度 |
| Split                     | 返回一个字符串类型的数组，根据指定的字符数组或者字符串数组中的字 符 或字符串作为条件拆分字符串 |
| Replace                   | 返回一个新的字符串，用于将指定字符串替换给原字符串中指定的字符串 |
| Substring                 | 返回一个新的字符串，用于截取指定的字符串                     |
| Insert                    | 返回一个新的字符串，将一个字符串插入到另一个字符串中指定索引的位置 |
| Concat                    | 返回一个新的字符串，将多个字符串合并成一个字符串             |
| Compare                   |                                                              |
| CompareOrdinal            |                                                              |
| CompareTo                 |                                                              |
| EndsWith                  |                                                              |
| Equals                    |                                                              |
| StartsWith                |                                                              |
| join                      |                                                              |
| char[] ToCharArray()      |                                                              |
| Format                    |                                                              |
| static bool IsNullOrEmpty |                                                              |
| IsNullOrWhiteSpace        |                                                              |
| GetPinnableReference      |                                                              |
| IsNormalized              |                                                              |

#### 1.6.2 构造器

````c#
//利用字面量创建 
string s1 = "张三abc123";

//利用 char 字符创建
string s2 = new string('c', 1);//c
string s3 = new string('c', 5);//ccccc

//利用 char 字符数组创建
char[] charArr = new char[] { 'a', 'b', 'c', };
string s4 = new string(charArr);//abc
string s5 = new string(charArr,1,2);//bc
````

#### 1.6.3 this[] 和 Length

> 字符串可以看成是一个字符数组，具有一些和数组类似的属性：
>
> * 可以和数组一样通过[] 获取指定下标位置的字符
> * 具有 Length 属性，获取字符串的长度，即字符串中字符个数。

