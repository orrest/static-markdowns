---
tags:
    - C#
    - Double
icon: fontawesome/solid/note-sticky
---

## 问题

```C#
private const double doubleVal =  10.789738975897389458937485738947857349;
private const string doubleStr = "10.789738975897389458937485738947857349";

[TestMethod]
public void DoubleToString()
{
    var s = doubleVal.ToString();

    // NOT EQUAL!
    Assert.AreNotEqual(doubleStr, s);
}
```

在 C# 中 `double` 类型的数值在执行 `ToString` 之后损失精度。

## 原因

当通过 `ToString` 方法进行数值转换的时候，通常意味着要把这个值显示在屏幕上，因此会截断一些精度。

由“精度”引申出一个问题，在计算机中，数值本来就不精确。因为二进制的数值无法模拟连续的实数。

再回到 “double.ToString” 的问题，不考虑浮点数本身的精度损失，仅考虑传输和转换的情况，可以通过 `BitConverter` 将其转换为字节数组避免 `ToString` 的精度损失：

```C#
[TestMethod]
public void BitConverterToBytes()
{
    byte[] doubleBytes = BitConverter.GetBytes(doubleVal);

    double convertedDouble = BitConverter.ToDouble(doubleBytes, 0);

    Assert.AreEqual(doubleVal, convertedDouble);
}
```

## Ref

[Unity: dont use double tostring](https://discussions.unity.com/t/dont-use-double-tostring-if-you-dont-want-to-lose-precision/750928/13)

[0.1 + 0.2 returns 0.30000000000000004](https://qntm.org/notpointthree)

[Microsoft: BitConverter](https://learn.microsoft.com/en-us/dotnet/api/system.bitconverter?view=net-10.0)

[Github: orrest/Tests](https://github.com/orrest/Tests)