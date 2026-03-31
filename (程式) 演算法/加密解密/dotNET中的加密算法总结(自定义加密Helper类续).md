---
aliases:
date: 2011-09-24
update:
author: JK_Rush
language: C#
sourceurl: https://www.cnblogs.com/rush/archive/2011/09/24/2189399.html
tags:
  - CSharp
  - 演算法
  - 加密解密
---

# .NET 中的加密算法总结 (打造属于你的加密 Helper 类续)

## 1.1.1 摘要

相信许多人都使用过 .NET 提供的加密算法，而且在使用的过程我们必须了解每种加密算法的特点（对称或非对称，密钥长度和初始化向量等等）。我也看到过很多人写过 .NET 中加密算法总结，但我发现个别存在一些问题，很多人喜欢罗列每种加密算法的具体实现，假设我们要求实现 AES 和 Triple DES 加密算法，的确可以很多地分别给出它们的具体实现。

那我们真的有必要给出每个加密算法的具体实现吗？而且这样的设计不符合 OOP 设计思想，最重要的是我们要维护多个加密算法啊！OK 接下来让我们实行一个可扩展和好维护的加密算法 Helper。

## 1.1.2 正文

![[加密算法总结_01.jpg|图1 Hash 加密算法继承层次]]

从上面的继承层次我们可以知道 .NET 中提供七种 Hash 加密算法，它们都继承于抽象类 `HashAlgorithm`，而且我们经常使用 MD5，SHA1 和 SHA256 等加密算法。下面我们将给出 MD5 和 SHA1 的实现。

![[加密算法总结_02.jpg|图2 对称加密算法继承层次]]

从上面的继承层次我们可以知道 .NET 中提供五种对称加密算法，它们都继承于抽象类 `SymmetricAlgorithm`，下面我们将给出它们的通用实现。

![[加密算法总结_03.jpg|图3 非对称加密算法继承层次]]

从上面的继承层次我们可以知道 .NET 中提供四种非对称加密算法，它们都继承于抽象类 `AsymmetricAlgorithm`，下面我们将给出 RSA 实现。

除了以上加密算法， .NET 还提供了很多其他类型的加密，这里我们主要介绍一些常用的加密算法，如果大家需要了解的话可以查阅 [MSDN](https://learn.microsoft.com/en-us/dotnet/api/system.security.cryptography)。OK 接下来让我们给出 Hash 加密算法的实现吧。

### Hash 加密算法

在给出具体的算法实现之前，首先让我们回忆一下什么是 Hash 加密算法？

Hash 加密是通过使用 hash 函数对要加密的信息进行加密，然后生成相应的哈希值，那么我们可以定义一个 hash() 函数，要加密的信息 m 和加密后的哈希值 h。

$h = hash(m)$

我们对信息 m1 和 m2 进行 hash 加密，就可以获取相应哈希值 hash(m1) 和 hash(m2)。

$hash(m1) = hash(m2)$

如果信息 m1=m2 那么，那么将得到同一的哈希地址，但是信息 m1!=m2 也可能得到同一哈希地址，那么就发生了哈希冲突（collision），在一般的情况下，哈希冲突只能尽可能地减少，而不能完全避免。当发生哈希冲突时，我们要使用冲突解决方法，而主要的冲突解决方法：开放地址法、再哈希法、链地址法和建立一个公共溢出区。

![[加密算法总结_04.jpg|图4 Hash加密过程（图片来源wiki）]]

现在让我们来实现通用的 hash 加密方法。

```csharp
/// <summary>
/// Encrypts the specified hash algorithm.
/// 1. Generates a cryptographic Hash Key for the provided text data.
/// </summary>
/// <param name="hashAlgorithm">The hash algorithm.</param>
/// <param name="dataToHash">The data to hash.</param>
/// <returns></returns>
public static string Encrypt(HashAlgorithm hashAlgorithm, string dataToHash)
{

    var tabStringHex = new string[16];
    var UTF8 = new System.Text.UTF8Encoding();
    byte[] data = UTF8.GetBytes(dataToHash);
    byte[] result = hashAlgorithm.ComputeHash(data);
    var hexResult = new StringBuilder(result.Length);

    for (int i = 0; i < result.Length; i++)
    {
        //// Convert to hexadecimal
        hexResult.Append(result[i].ToString("X2"));
    }
    return hexResult.ToString();
}
```

上面的加密方法包含一个 `HashAlgorithm` 类型的参数，我们可以传递继承于抽象类 `HashAlgorithm` 的具体 hash 算法（MD5，SHA1 和 SHA256 等），通过继承多态性我们使得加密方法更加灵活、简单，最重要的是现在我们只需维护一个通用的加密方法就 OK 了。

接着我们要添加判断加密后哈希值是否相等的方法，判断哈希值是否相等的方法 `IsHashMatch()` 方法。

```csharp
/// <summary>
/// Determines whether [is hash match] [the specified hash algorithm].
/// </summary>
/// <param name="hashAlgorithm">The hash algorithm.</param>
/// <param name="hashedText">The hashed text.</param>
/// <param name="unhashedText">The unhashed text.</param>
/// <returns>
///   <c>true</c> if [is hash match] [the specified hash algorithm]; 
/// otherwise, <c>false</c>.
/// </returns>
public static bool IsHashMatch(HashAlgorithm hashAlgorithm, string hashedText, string unhashedText)
{
    string hashedTextToCompare = Encrypt(hashAlgorithm, unhashedText);
    return (String.Compare(hashedText, hashedTextToCompare, false) == 0);
}
```

### 对称加密算法

现在我们完成了通用的 Hash 加密方法了，接下来我们继续介绍对称和非对称算法（具体 Demo 可以参考 [这里](http://www.cnblogs.com/rush/archive/2011/07/24/2115613.html)）。

在实现对称加密算法之前，先让我们了解一下对称加密的过程，假设我们有一组数据要加密那么我们可以使用一个或一组密钥对数据进行加密解密，但存在一个问题对称加密算法的密钥长度不尽相同，如 DES 的密钥长度为 64 bit，而 AES 的长度可以为 128bit、192bit 或 256 bit，难道要我们 hard code 每种算法的密钥长度吗？能不能动态地产生对应算法的密钥呢？

其实 .NET 已经提供我们根据不同的对称算法生成对应密钥的方法了，并且把这些方法都封装在 `PasswordDeriveBytes` 和 `Rfc2898DeriveBytes` 类中。

首先让我们看一下 `PasswordDeriveBytes` 类包含两个方法 `CryptDeriveKey` 和 `GetBytes` 用来产生对应算法的密钥，现在让我们看一下它们如何产生密钥。

#### CryptDeriveKey

```csharp
// The sample function.
public void Encrypt()
{
    // The size of the IV property must be the same as the BlockSize property.
    // Due to the RC2 block size is 64 bytes, so iv size also is 64 bytes. 
    var iv = new byte[] { 8, 7, 6, 5, 4, 3, 2, 1 };

    var pdb = new PasswordDeriveBytes("pwd", null);

    // Set the encrypted algorithm and export key algorithm.
    // Then get the key base on encrypt algorithm.
    byte[] key = pdb.CryptDeriveKey("RC2", "SHA1", 128, iv);

    Console.WriteLine(key.Length * 8);
    Console.WriteLine(new RC2CryptoServiceProvider().BlockSize);
    // Creates an RC2 object to encrypt with the derived key
    var rc2 = new RC2CryptoServiceProvider
    {
        Key = key,
        IV = new byte[] { 21, 22, 23, 24, 25, 26, 27, 28 }
    };

    // now encrypt with it
    byte[] plaintext = Encoding.UTF8.GetBytes("NeedToEncryptData");
    using (var ms = new MemoryStream())
    {
        var cs = new CryptoStream(ms, rc2.CreateEncryptor(), CryptoStreamMode.Write);

        cs.Write(plaintext, 0, plaintext.Length);
        cs.Close();
        byte[] encrypted = ms.ToArray();
    }
}
```

示意例子一：我们使用 SHA1 哈希算法为 RC2 加密算法生成 128bit 的密钥，这样我们就可以根据不同对称加密算法获取相应长度的密钥了，注意我们并没用动态地生成初始化向量 iv，这是为了简单起见实际中不应该这样获取初始化向量。

接下来让我们看一下通过 PBKDF1 和 PBKDF2s 算法生成密钥的实现。

#### PBKDF1

GetBytes：`PasswordDeriveBytes` 的 `GetBytes()` 方法实现了 [PBKDF1](http://tools.ietf.org/html/rfc2898)（Password Based Key Derivation Function）。

> PBKDF1 算法过程：
>
> 1.拼接密钥和盐：R0 = Pwd + Salt
> 2.哈希加密过程：R1 = Hash(R2-1)
>
> ……..
>
> 3.哈希加密过程：Rn = Hash(Rn - 1)
> 4. n 是迭代的次数 (参考 PBKDF1 规范请点 [这里](http://www.ietf.org/rfc/rfc2898.txt))

现在我们对 PBKDF1 算法的原理有了初步的了解，接下来我们将通过 `GetBytes()` 调用该算法生成密钥。

```csharp
/// <summary>
/// Uses the PBKDF1 to genernate key, 
/// then use it to encrypt plain text.
/// </summary>
public void PBKDF1()
{
    byte[] salt = new byte[] { 8, 7, 6, 5, 4, 3, 2, 1 };

    // Creates an RC2 object to encrypt with the derived key
    var pdb = new PasswordDeriveBytes("pwd", salt) 
    {
        IterationCount = 23,
        HashName = "SHA1"
    };

    // Gets the key and iv.
    byte[] key = pdb.GetBytes(16);
    byte[] iv = pdb.GetBytes(8);

    var rc2 = new RC2CryptoServiceProvider { Key = key, IV = iv };

    byte[] plaintext = Encoding.UTF8.GetBytes("NeedToEncryptData");
    using (var ms = new MemoryStream())
    {
        // Encrypts data.
        var cs = new CryptoStream(ms, rc2.CreateEncryptor(), CryptoStreamMode.Write);

        cs.Write(plaintext, 0, plaintext.Length);
        cs.Close();
        byte[] encrypted = ms.ToArray();
    }
}
```

示意例子二：我们使用 PBKDF1 算法为 RC2 加密算法生成 128 bit 的密钥和 64 bit 的初始化向量，要注意的是 `PasswordDeriveBytes` 的 `GetBytes()` 方法==已经过时==了，而它的替代项就是接下来要介绍的 `Rfc2898DeriveBytes` 的 `GetBytes()` 方法。

#### PBKDF2

GetBytes：由于 `Rfc2898DeriveBytes` 的 `GetBytes()` 方法实现了 [PBKDF2](http://tools.ietf.org/html/rfc2898) 算法，而且它也替代了 PBKDF1 过时的 `GetBytes()` 方法，所以我们推荐使用 `Rfc2898DeriveBytes` 的 `GetBytes()` 方法。

```csharp
/// <summary>
/// Uses the PBKDF2 to genernate key, 
/// then use it to encrypt plain text.
/// </summary>
public void PBKDF2()
{
    byte[] salt = new byte[] { 23, 21, 32, 33, 46, 59, 60, 74 };
    var rfc = new Rfc2898DeriveBytes("pwd", salt, 23);

    // generate key and iv.
    byte[] key = rfc.GetBytes(16);
    byte[] iv = rfc.GetBytes(8);

    // Creates an RC2 object to encrypt with the derived key
    var rc2 = new RC2CryptoServiceProvider { Key = key, IV = iv };

    // Encrypts the data.
    byte[] plaintext = Encoding.UTF8.GetBytes("NeedToEncryptData");
    using (var ms = new MemoryStream())
    {
        var cs = new CryptoStream(ms, rc2.CreateEncryptor(), CryptoStreamMode.Write);

        cs.Write(plaintext, 0, plaintext.Length);
        cs.Close();
        byte[] encrypted = ms.ToArray();
    }
}
```

示意例子三：我们发现 `PBKDF2()` 方法和之前的 `PBKDF1()` 方法没有什么区别，就是无需指定加密密钥的哈希算法（参考 PBKDF2 规范请点 [这里](http://en.wikipedia.org/wiki/PBKDF2)）。

前面通过三种方法来动态的生成加密密钥，而且我们将使用 `Rfc2898DeriveBytes` 的 `GetBytes()` 方法来获取密钥，那么接下来让我们使用该方法实现通用的对称加密算法吧！

![[加密算法总结_05.jpg|图5 对称算法加密过程]]

首先我们对加密的平文进行编码，这里默认使用 UTF8 对平文进行编码，也可以使用其他编码方式，接着使用相应加密算法对编码后的平文进行加密，最后把加密后的 Byte 数组转换为 Base64 格式字符串返回。

```csharp
/// <summary>
/// Encrypts with specified symmetric algorithm.
/// Can be Aes, DES, RC2, Rijndael and TripleDES.
/// </summary>
/// <param name="algorithm">The symmertric algorithm (Aes, DES, RC2, Rijndael and TripleDES).</param>
/// <param name="plainText">The plain text need to be encrypted.</param>
/// <param name="key">The secret key to encrypt plain text.</param>
/// <param name="iv">The iv should be 16 bytes.</param>
/// <param name="salt">Salt to encrypt with.</param>
/// <param name="pwdIterations">The number of iterations for plain text.</param>
/// <param name="keySize">Size of the key.</param>
/// <param name="cipherMode">The cipher mode.</param>
/// <param name="paddingMode">The padding mode.</param>
/// <returns></returns>
public static byte[] Encrypt(SymmetricAlgorithm algorithm, byte[] plainText, string key, string iv,
    string salt, int pwdIterations, int keySize, CipherMode cipherMode, PaddingMode paddingMode)
{

    if (null == plainText)
        throw new ArgumentNullException("plainText");
    if (null == algorithm)
        throw new ArgumentNullException("algorithm");
    if (String.IsNullOrEmpty(key))
        throw new ArgumentNullException("key");
    if (String.IsNullOrEmpty(iv))
        throw new ArgumentNullException("iv");
    if (String.IsNullOrEmpty(salt))
        throw new ArgumentNullException("salt");

    // Note the salt should be equal or greater that 64bit (8 byte).
    var rfc = new Rfc2898DeriveBytes(key, salt.ToByteArray(), pwdIterations);
    using (SymmetricAlgorithm symmAlgo = algorithm)
    {
        symmAlgo.Mode = cipherMode;
        //symmAlgo.Padding = paddingMode;
        byte[] cipherTextBytes = null;
        using (var encryptor = symmAlgo.CreateEncryptor(rfc.GetBytes(keySize / 8), iv.ToByteArray()))
        {
            using (var ms = new MemoryStream())
            {
                using (var cs = new CryptoStream(ms, encryptor, CryptoStreamMode.Write))
                {
                    cs.Write(plainText, 0, plainText.Length);
                    cs.FlushFinalBlock();
                    cipherTextBytes = ms.ToArray();
                    ms.Close();
                    cs.Close();
                }
            }
            symmAlgo.Clear();
            return cipherTextBytes;
        }
    }
}
```

![[加密算法总结_06.jpg|图6 对称算法解密过程]]

通过上图的解密过程，我们发现解密过程恰恰是加密的反向，首先把 Base64 格式的密文转换为 Byte 数组，接着使用对应的解密算法解密密文，最后对解密后的数据进行编码返回平文（默认使用 UTF8）。

```csharp
/// <summary>
/// Decrypts the specified algorithm.
/// Can be Aes, DES, RC2, Rijndael and TripleDES.
/// </summary>
/// <param name="algorithm">The symmertric algorithm (Aes, DES, RC2, Rijndael and TripleDES).</param>
/// <param name="cipherText">The cipher text.</param>
/// <param name="key">The secret key to decrypt plain text.</param>
/// <param name="iv">The iv should be 16 bytes.</param>
/// <param name="salt">Salt to decrypt with.</param>
/// <param name="pwdIterations">The number of iterations for plain text.</param>
/// <param name="keySize">Size of the key.</param>
/// <param name="cipherMode">The cipher mode.</param>
/// <param name="paddingMode">The padding mode.</param>
/// <returns></returns>
public static byte[] Decrypt(SymmetricAlgorithm algorithm, byte[] cipherText,
    string key, string iv, string salt, int pwdIterations, int keySize,
    CipherMode cipherMode, PaddingMode paddingMode)
{
    if (null == cipherText)
        throw new ArgumentNullException("cipherText");
    if (null == algorithm)
        throw new ArgumentNullException("algorithm");
    if (String.IsNullOrEmpty(key))
        throw new ArgumentNullException("key");
    if (String.IsNullOrEmpty(iv))
        throw new ArgumentNullException("iv");
    if (String.IsNullOrEmpty(salt))
        throw new ArgumentNullException("salt");

    // Note the salt should be equal or greater that 64bit (8 byte).
    var rfc = new Rfc2898DeriveBytes(key, salt.ToByteArray(), pwdIterations);

    using (SymmetricAlgorithm symmAlgo = algorithm)
    {
        symmAlgo.Mode = cipherMode;
        //symmAlgo.Padding = paddingMode;
        byte[] plainTextBytes = new byte[cipherText.Length];
        int cnt = -1;
        using (var encryptor = symmAlgo.CreateDecryptor(rfc.GetBytes(keySize / 8), iv.ToByteArray()))
        {
            using (var ms = new MemoryStream(cipherText))
            {
                using (var cs = new CryptoStream(ms, encryptor, CryptoStreamMode.Read))
                {
                    cnt = cs.Read(plainTextBytes, 0, plainTextBytes.Length);
                    ms.Close();
                    cs.Close();
                }
            }
        }
        symmAlgo.Clear();
        Array.Resize(ref plainTextBytes, cnt);
        return plainTextBytes;
    }
}
```

在前面的加密和解密方法，我们通过 `Rfc2898DeriveBytes` 获取密码、salt 值和迭代次数，然后通过调用 [GetBytes](http://msdn.microsoft.com/zh-cn/library/system.security.cryptography.rfc2898derivebytes.getbytes%28v=vs.80%29.aspx) 方法生成密钥。

现在我们已经完成了通用的对称加密算法，我们只需一组加密和解密方法就可以随意的使用任意一种对称加密算法了，而不是为每个加密和解密算法编写相应的加密和解密方法。

### 非对称加密算法

 .NET Framework 中提供四种非对称加密算法（[DSA](http://msdn.microsoft.com/en-us/library/system.security.cryptography.dsa.aspx)，[ECDiffieHellman](http://msdn.microsoft.com/en-us/library/system.security.cryptography.ecdiffiehellman.aspx)， [ECDsa](http://msdn.microsoft.com/en-us/library/system.security.cryptography.ecdsa.aspx) 和 [RSA](http://msdn.microsoft.com/en-us/library/system.security.cryptography.rsa.aspx)），它们都继承于抽象类 `AsymmetricAlgorithm`，接下来我们将提供 RSA 算法的实现。

RSA 加密算法是一种非对称和双钥加密算法，在公钥加密标准和电子商业中 RSA 被广泛使用。

在双钥加密的情况下，密钥有两把，一把是公开的公钥，还有一把是不公开的私钥。

> 双钥加密的原理如下：
> a) 公钥和私钥是一一对应的关系，有一把公钥就必然有一把与之对应的、独一无二的私钥，反之亦成立。
> b) 所有的（公钥, 私钥）对都是不同的。
> c) 用公钥可以解开私钥加密的信息，反之亦成立。
> d) 同时生成公钥和私钥应该相对比较容易，但是从公钥推算出私钥，应该是很困难或者是不可能的。

现在的数字签名加密主要是使用 RSA 算法，什么是数字签名大家请点这里（[中文](http://www.ruanyifeng.com/blog/2011/08/what_is_a_digital_signature.html?1316831854)）和这里（[英文](http://www.youdzone.com/signature.html?1316831458)）。

现在我们知道 RSA 算法是使用公钥和密钥进行加密和解密，所以我们先定义一个方法来生成公钥和密钥。

```csharp
/// <summary>
/// Generates the RSA public and private key.
/// </summary>
/// <param name="algorithm">The algorithm to creates key.</param>
/// <returns></returns>
public static void GenerateRSAKey(RSACryptoServiceProvider algorithm)
{
    // Contains public and private key.
    RSAPrivateKey = algorithm.ToXmlString(true);

    using (var streamWriter = new StreamWriter("PublicPrivateKey.xml"))
    {
        streamWriter.Write(RSAPrivateKey);
    }

    // Only contains public key.
    RSAPubicKey = algorithm.ToXmlString(false);
    using (var streamWriter = new StreamWriter("PublicOnlyKey.xml"))
    {
        streamWriter.Write(RSAPubicKey);
    }
}
```

通过 `RSACryptoServiceProvider` 的 `ToXmlString()` 方法我们生成了一对公钥和密钥，当参数为 `true` 表示同时包含 RSA 公钥和私钥，反之表示仅包含公钥。

```csharp
/// <summary>
/// Encrypts with the specified RSA algorithm.
/// </summary>
/// <param name="rsa">A RSA object.</param>
/// <param name="plainText">The plain text to decrypt.</param>
/// <param name="key">The key.</param>
/// <param name="encoding">The encoding.</param>
/// <returns></returns>
public static string Encrypt(RSACryptoServiceProvider rsa, string plainText, string key, Encoding encoding)
{
    if (null == rsa)
        throw new ArgumentNullException("rsa");
    if (String.IsNullOrEmpty(plainText))
        throw new ArgumentNullException("plainText");
    if (String.IsNullOrEmpty(key))
        throw new ArgumentNullException("key");
    if (null == encoding)
        throw new ArgumentNullException("encoding");

    string publicKey;

    // Reads public key.
    using (var streamReader = new StreamReader("PublicOnlyKey.xml"))
    {
        publicKey = streamReader.ReadToEnd();
    }
    rsa.FromXmlString(publicKey);
    byte[] cipherBytes = rsa.Encrypt(plainText.ToBytesEncoding(encoding), true);
    rsa.Clear();
    return cipherBytes.ToBase64String();
}
```

接着我们定义 RSA 的加密方法，首先我们从流中读取密钥和公钥，然后传递给 `FromXmlString()` 方法，最后对平文进行加密。

```csharp
/// <summary>
/// Decrypts with the specified RSA algorithm.
/// </summary>
/// <param name="rsa">a RSA object.</param>
/// <param name="cipherText">The cipher text to encrypt.</param>
/// <param name="key">The key.</param>
/// <param name="encoding">The encoding.</param>
/// <returns></returns>
public static string Decrypt(RSACryptoServiceProvider rsa, string cipherText, string key, Encoding encoding)
{
    string privateKey;
    // Reads the private key.
    using (var streamReader = new StreamReader("PublicPrivateKey.xml"))
    {
        privateKey = streamReader.ReadToEnd();
    }

    rsa.FromXmlString(privateKey);
    byte[] plainBytes = rsa.Decrypt(cipherText.FromBase64String(), true);
    rsa.Clear();
    return plainBytes.FromByteToString(encoding);
}
```

参照加密方法我们很快的实现 RSA 的解密方法，同样我们从流中读取密钥，然后传递给 `FromXmlString()` 方法，最后对密文进行解密，注意调用的是 RSA 算法的 `Decrypt()` 方法，在使用胶水代码时千万别忘记修改了。

现在我们终于完成了通用的加密解密方法，接下来肯定是要测试一下这些方法的效果如何，这次我使用单元测试，如果大家要参考应用程序效果可以点 [这里](http://www.cnblogs.com/rush/archive/2011/07/24/2115613.html)。

```csharp
[TestMethod]
public void TestStart()
{
    try
    {
        string cipherText;
        string plainText;

        #region Hash Algo

        cipherText = CryptographyUtils.Encrypt(
            CryptographyUtils.CreateHashAlgoMd5(), @"您们好（Hello everyone）.");
        Assert.IsTrue(CryptographyUtils.IsHashMatch(
            CryptographyUtils.CreateHashAlgoMd5(),
            cipherText,
            @"您们好（Hello everyone）."));

        cipherText = CryptographyUtils.Encrypt(
            CryptographyUtils.CreateHashAlgoSHA1(),
            @"您们好（Hello everyone）.");
        Assert.IsTrue(CryptographyUtils.IsHashMatch(
            CryptographyUtils.CreateHashAlgoSHA1(),
            cipherText,
            @"您们好（Hello everyone）."));

        #endregion

        #region Asymm Algo

        CryptographyUtils.GenerateRSAKey(CryptographyUtils.CreateAsymmAlgoRSA());
        cipherText = CryptographyUtils.Encrypt(
            CryptographyUtils.CreateAsymmAlgoRSA(),
            @"%dk>JK.RusH",
            @"c579D-E>?$)_");
        plainText = CryptographyUtils.Decrypt(
            CryptographyUtils.CreateAsymmAlgoRSA(),
            cipherText,
            @"c579D-E>?$)_");
        Assert.AreEqual<string>(@"%dk>JK.RusH", plainText);

        #endregion

        #region Symm Algo

        cipherText = CryptographyUtils.Encrypt(
            CryptographyUtils.CreateSymmAlgoAes(),
            "JK_huangJK_huangJK_huang黄钧航",
            "JK_huangJK_huang",
            256);

        plainText = CryptographyUtils.Decrypt(
            CryptographyUtils.CreateSymmAlgoAes(),
            cipherText,
            "JK_huangJK_huang",
            256);

        Assert.AreEqual<string>("JK_huangJK_huangJK_huang黄钧航",
            plainText);

        cipherText = CryptographyUtils.Encrypt(
            CryptographyUtils.CreateSymmAlgoDES(),
            "JK_huangJK_huangJK_huang黄钧航",
            "JK_huangJK_huang",
            64);

        plainText = CryptographyUtils.Decrypt(
            CryptographyUtils.CreateSymmAlgoDES(),
            cipherText,
            "JK_huangJK_huang",
            64);

        Assert.AreEqual<string>("JK_huangJK_huangJK_huang黄钧航", plainText);

        cipherText = CryptographyUtils.Encrypt(
            CryptographyUtils.CreateSymmAlgoRC2(),
            "JK_huangJK_huangJK_huang黄钧航",
            "JK_huangJK_huang",
            128);

        plainText = CryptographyUtils.Decrypt(
            CryptographyUtils.CreateSymmAlgoRC2(),
            cipherText,
            "JK_huangJK_huang",
            128);

        Assert.AreEqual<string>("JK_huangJK_huangJK_huang黄钧航", plainText);

        cipherText = CryptographyUtils.Encrypt(
            CryptographyUtils.CreateSymmAlgoRijndael(),
            "JK_huangJK_huangJK_huang黄钧航",
            "JK_huangJK_huang",
            256);

        plainText = CryptographyUtils.Decrypt(
            CryptographyUtils.CreateSymmAlgoRijndael(),
            cipherText,
            "JK_huangJK_huang",
            256);

        Assert.AreEqual<string>("JK_huangJK_huangJK_huang黄钧航", plainText);

        cipherText = CryptographyUtils.Encrypt(
            CryptographyUtils.CreateSymmAlgoTripleDes(),
            "JK_huangJK_huangJK_huang黄钧航",
            "JK_huangJK_huang",
            192);

        plainText = CryptographyUtils.Decrypt(
            CryptographyUtils.CreateSymmAlgoTripleDes(),
            cipherText,
            "JK_huangJK_huang",
            192);

        Assert.AreEqual<string>("JK_huangJK_huangJK_huang黄钧航", plainText);

        #endregion
    }
    catch (Exception ex)
    {
        Debug.Assert(false, ex.Message);
    }
}
```

![[加密算法总结_07.jpg|图7 单元测试结果]]

# 1.1.3 总结

本文给出了 .NET 中的一些加密算法的通用实现（哈希加密，对称加密和非对称加密算法），由于加密和解密的方法都是比较固定，而且每中算法有具有其特性，所以这里我们给出了它们的实现，而且我们可以把上的方法封装在一个加密 Helper 类，这样可以大大提高我们开发效率，而且它充分的利用多态性使得加密算法灵活性和维护性大大提高。
