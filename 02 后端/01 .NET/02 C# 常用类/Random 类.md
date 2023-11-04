# Random

## 1. API 方法

> Random 类用于产生随机数

| 方法                   | 作用                                                         |
| ---------------------- | ------------------------------------------------------------ |
| **Next()**             | 返回一个非负随机整数，范围为0~int.MaxValue。                 |
| Next(Int32)            | 返回一个小于所指定最大值的非负随机整数（int取值范围）。      |
| Next(Int32,Int32)      | 返回在指定范围内的任意整数（int取值范围）。                  |
| **NextInt64()**        | 返回一个非负随机整数，范围为0~long.MaxValue。                |
| NextInt64(Int32)       | 返回一个小于所指定最大值的非负随机整数（long取值范围）。     |
| NextInt64(Int32,Int32) | 返回在指定范围内的任意整数（long取值范围）。                 |
| **NextSingle()**       | 返回一个大于或等于0.0且小于1.0的随机浮点数（float类型）。    |
| **NextDouble()**       | 返回一个大于或等于 0.0 且小于 1.0 的随机浮点数（double类型）。 |
| protected Sample()     | 返回一个介于 0.0 和 1.0 之间的随机浮点数（double类型）。     |

例：

````C#
Random random = new Random();
int number = random.Next(10);
int number2 = random.Next(10,20);
````

