---
date: 2017-05-11
author: scott.cgi
sourceurl: https://blog.csdn.net/tom_221x/article/details/71643015
tags:
  - CSharp
  - CSharp_內建類型
---

# C# byte[]数组和string的互相转化 (四种方法)

## 第一种

```csharp
string  str    = System.Text.Encoding.UTF8.GetString(bytes);
byte[] decBytes = System.Text.Encoding.UTF8.GetBytes(str);
```

同样的，`System.Text.Encoding.Default`，`System.Text.Encoding.ASCII`也是可以的。还可以使用`System.Text.Encoding.UTF8.GetString(bytes).TrimEnd('\0')`给字符串加上结束标识。

## 第二种

```csharp
string    str    = BitConverter.ToString(bytes);
String[] tempArr = str.Split('-');
byte[]   decBytes = new byte[tempArr.Length];
for (int i = 0; i < tempArr.Length; i++)
{
    decBytes[i] = Convert.ToByte(tempArr[i], 16);
}
```

这种方法会给字符串加上 `'-'` 连字符，并且没有函数转换回去。所以需要手动转换为`bytes`。

## 第三种

```csharp
string str      = Convert.ToBase64String(bytes);
byte[] decBytes = Convert.FromBase64String(str);
```

这种方法简单明了，完美无问题。需要注意的是，转换出来的`string`可能会包含 '+'，'/' ， '=' 所以如果作为url地址的话，需要进行encode。

## 第四种

```csharp
string  str    = HttpServerUtility.UrlTokenEncode(bytes);
byte[] decBytes = HttpServerUtility.UrlTokenDecode(str);
```

这种方法会自动编码url地址的特殊字符，可以直接当做url地址使用。但需要依赖`System.Web`库才能使用。
