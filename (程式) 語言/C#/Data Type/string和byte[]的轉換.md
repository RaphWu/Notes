---
aliases:
date: 2012-03-20
update:
author: isan
language:
sourceurl: https://isanhsu.blogspot.com/2012/03/stringbyte-c.html
tags:
  - CSharp
  - CSharp_內建類型
---

 `string` 類型轉成 `byte[]`

```csharp
byte[] byteArray = System.Text.Encoding.Default.GetBytes(str);
```

反過來，`byte[]` 轉成 `string`：

```csharp
string str = System.Text.Encoding.Default.GetString(byteArray);
```

其它編碼方式的，如 `System.Text.UTF8Encoding`，`System.Text.UnicodeEncoding` class 等；例如：
`string` 類型轉成 ASCII `byte[]`：（"01" 轉成 `byte[] = new byte[]{ 0x30, 0x31}`）

```csharp
byte[] byteArray = System.Text.Encoding.ASCII.GetBytes(str);
```

ASCII `byte[]` 轉成 `string`：（`byte[] = new byte[]{ 0x30, 0x31}` 轉成 "01"）

```csharp
string str = System.Text.Encoding.ASCII.GetString(byteArray);
```

有時候還有這樣一些需求：
`byte[]` 轉成原 16 進制格式的 `string`，例如 `0xae00cf`, 轉換成 "ae00cf"；`new byte[]{ 0x30, 0x31}` 轉成 "3031":

```csharp
public static string ToHexString(byte[] bytes) // 0xae00cf => "AE00CF "
{
    string hexString = string.Empty;
    if (bytes != null)
    {
        StringBuilder strB = new StringBuilder();

        for (int i = 0; i < bytes.Length; i++)
        {
            strB.Append(bytes[i].ToString("X2"));
        }
        hexString = strB.ToString();
    }
    return hexString;
}
```

反過來，16 進制格式的 `string` 轉成 `byte[]`，例如, "ae00cf" 轉換成 0xae00cf，長度縮減一半；"3031" 轉成 `new byte[]{ 0x30, 0x31}`:

```csharp
public static byte[] GetBytes(string hexString, out int discarded)
{
    discarded = 0;
    string newString = "";
    char c;
    // remove all none A-F, 0-9, characters
    for (int i = 0; i < hexString.Length; i++)
    {
        c = hexString[i];
        if (IsHexDigit(c))
            newString += c;
        else
            discarded++;
    }
    // if odd number of characters, discard last character
    if (newString.Length % 2 != 0)
    {
        discarded++;
        newString = newString.Substring(0, newString.Length - 1);
    }

    int byteLength = newString.Length / 2;
    byte[] bytes = new byte[byteLength];
    string hex;
    int j = 0;
    for (int i = 0; i < bytes.Length; i++)
    {
        hex = new String(new Char[] { newString[j], newString[j + 1] });
        bytes[i] = HexToByte(hex);
        j = j + 2;
    }
    return bytes;
}
```
