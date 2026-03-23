---
aliases:
date: 2011-07-24
update:
author: JK_Rush
language: CSharp
sourceurl: https://www.cnblogs.com/rush/archive/2011/07/24/2115613.html
tags:
  - CSharp
  - 演算法
  - 加密解密
---

# 打造属于你的加密 Helper 类

## 摘要

在我们软件系统设计中，数据的安全性是我们考虑的重中之重，特别像银行系统的设计账户和密码都需进行加密处理。这时我们可以使用加密算法对数据进行加密处理，这就是我们今天要介绍的主题。

首先让我们了解加密算法分为：对称、非对称加密算法和 Hash 加密。

对称加密算法：首先需要发送方和接收方协定一个密钥 K。K 可以是一个密钥对，但是必须要求加密密钥和解密密钥之间能够互相推算出来。在最简单也是最常用的对称算法中，加密和解密共享一个密钥。

非对称加密算法：首先得有一个密钥对，这个密钥对含有两部分内容，分别称作公钥（PK）和私钥（SK），公钥通常用来加密，私钥则用来解密。在对称算法中，也讲到了可以有两个密钥（分为加密和解密密钥）。但是，对称算法中的加解密密钥可以互相转换，而在非对称算法中，则不能从公钥推算出私钥，所以我们完全可以将公钥公开到任何地方。

## 正面

![[打造属于你的加密Helper类_01.png|图1 .NET中对称加密算法]]

![[打造属于你的加密Helper类_02.jpg|图2 .NET中非对称加密算法]]

通过上面 .NET 的对称和非对称加密算法类结构图，我们可以发现在 .NET Framework 中通过提供者模式实现加密算法动态扩展，SymmetricAlgorithm 和 AsymmetricAlgorithm 类分别为对称和非对称算法的基类，接着通过扩展不同的算法类型（如：DES，TripleDES 等），来动态地扩展不同类型的算法，最后是每种算法的具体实现 Provider（如：DESCryptoServiceProvider，TripleDESCryptoServiceProvider 等）。

现在我们对 .NET Framework 中的加密算法有了初步了解，接下来让我们通过打造属于我们的加密 Helper 类，来加深加密算法的使用。

### 对称加密算法

首先让我们实现对称算法的加密和解密方法，先定义对称算法加密方法 `Encypt()`，它提供所有对称算法加密算法的加密实现。

```csharp
/// <summary>
/// Encrypts the specified algorithm.
/// The type of algorithm is SymmetricAlgorithm, so the Encrypt
/// supports all kind of Symmetric Algorithm (cf: DES, TripleDes etc).
/// </summary>
/// 
/// <param name="algorithm">The algorithm.</param>
/// <param name="plainText">The plain text.</param>
/// <param name="key">The key.</param>
/// <param name="cipherMode">The cipher mode.</param>
/// <param name="paddingMode">The padding mode.</param>
/// <returns> The string base on base64. </returns>
public static string Encrypt(SymmetricAlgorithm algorithm, string plainText, byte[] key,
    CipherMode cipherMode, PaddingMode paddingMode)
{
    byte[] plainBytes;
    byte[] cipherBytes;
    algorithm.Key = key;
    algorithm.Mode = cipherMode;
    algorithm.Padding = paddingMode;

    BinaryFormatter bf = new BinaryFormatter();

    using (MemoryStream stream = new MemoryStream())
    {
        bf.Serialize(stream, plainText);
        plainBytes = stream.ToArray();
    }

    using (MemoryStream ms = new MemoryStream())
    {
        // Defines a stream for cryptographic transformations
        CryptoStream cs = new CryptoStream(ms, algorithm.CreateEncryptor(), CryptoStreamMode.Write);

        // Writes a sequence of bytes for encrption
        cs.Write(plainBytes, 0, plainBytes.Length);

        // Closes the current stream and releases any resources 
        cs.Close();
        // Save the ciphered message into one byte array
        cipherBytes = ms.ToArray();
        // Closes the memorystream object
        ms.Close();
    }
    string base64Text = Convert.ToBase64String(cipherBytes);
   
    return base64Text;
}
```

现在我们完成了对称加密算法的加密方法 `Encrypt()`，它的签名包含一个 `SymmetricAlgorithm` 类参数，前面提到对称加密算法都是继承于 `SymmetricAlgorithm` 抽象类，所用我们通过多态性降低了 `Encrypt()` 方法和不同的加密算法类之间的耦合度。

接着传递的参数是要加密的明文 `plainText`、密钥 `key`，使用的加密模式 `cipherMode` 和加密数据的填充方式 `paddingMode`。最后返回一组由 [base64格式](http://zh.wikipedia.org/wiki/Base64) 组成的加密数据。

上面我们已经完成了对称加密算法的加密方法 `Encrypt()`，接着让我们实现对称算法的解密方法，首先定义对称算法解密方法 `Decrypt()`，它提供所有对称算法解密算法的解密实现。

```csharp
/// <summary>
/// Decrypts the specified algorithm.
/// </summary>
/// <param name="algorithm">The algorithm.</param>
/// <param name="base64Text">The base64 text.</param>
/// <param name="key">The key.</param>
/// <param name="cipherMode">The cipher mode.</param>
/// <param name="paddingMode">The padding mode.</param>
/// <returns> The recovered text. </returns>
public static string Decrypt(SymmetricAlgorithm algorithm, string base64Text, byte[] key,
    CipherMode cipherMode, PaddingMode paddingMode)
{
    byte[] plainBytes;

    //// Convert the base64 string to byte array. 
    byte[] cipherBytes = Convert.FromBase64String(base64Text);
    algorithm.Key = key;
    algorithm.Mode = cipherMode;
    algorithm.Padding = paddingMode;

    using (MemoryStream memoryStream = new MemoryStream(cipherBytes))
    {
        using (CryptoStream cs = new CryptoStream(
            memoryStream,
            algorithm.CreateDecryptor(),
            CryptoStreamMode.Read))
        {
            plainBytes = new byte[cipherBytes.Length];
            cs.Read(plainBytes, 0, cipherBytes.Length);
        }
    }

    string recoveredMessage;
    using (MemoryStream stream = new MemoryStream(plainBytes, false))
    {
        BinaryFormatter bf = new BinaryFormatter();
        recoveredMessage = bf.Deserialize(stream).ToString();
    }

    return recoveredMessage;
}
```

我们已经完成了对称加密算法的解密方法 `Decrypt()`，它的签名形式和加密方法 `Encrypt()` 基本一致，只是我们传递要解密的密文进行解密就 OK 了。

现在让我们回忆一下对称加密算法的定义，其中有几点我们要注意的是，==加密和解密密钥可以相同，或通过一定算法由加密密钥推演出解密密钥==，所以我们为了确保数据的安全性最好设置对称算法初始化向量 IV，其目的是在初始加密数据头部添加其他数据，从而降低密钥被破译的几率。

初始化向量 IV 在加密算法中起到的也是增强破解难度的作用。在加密过程中，如果遇到相同的数据块，其加密出来的结果也一致，相对就会容易破解。加密算法在加密数据块的时候，往往会同时使用密码和上一个数据块的加密结果。因为要加密的第一个数据块显然不存在上一个数据块，所以这个初始化向量就是被设计用来当作初始数据块的加密结果。

现在我们已经完成了对称加密算法的加密和解密方法，接下来我们继续完成非对称加密算法的加密和解密方法吧！

### 非对称加密算法

在 .NET Framework 中提供四种非对称加密算法（[DSA](http://msdn.microsoft.com/en-us/library/system.security.cryptography.dsa.aspx)，[ECDiffieHellman](http://msdn.microsoft.com/en-us/library/system.security.cryptography.ecdiffiehellman.aspx)， [ECDsa](http://msdn.microsoft.com/en-us/library/system.security.cryptography.ecdsa.aspx) 和 [RSA](http://msdn.microsoft.com/en-us/library/system.security.cryptography.rsa.aspx)），它们都继承于抽象类 `AsymmetricAlgorithm`。但这里我们只提供 RSA 的加密和解密方法的实现。

首先我们定义一个方法 `GenerateRSAKey()` 来生成 RSA 的公钥和密钥，它的签名包含一个 `RSACryptoServiceProvider` 类型的参数，细心的大家肯定会发现为什么我们没有把参数类型定义为 `AsymmetricAlgorithm`，这不是可以充分的利用抽象类的特性吗？当我们查看抽象类 `AsymmetricAlgorithm` 发现，它并没有定义公钥和密钥的生成方法——`ToXmlString()` 方法，这个方法是在 RSA 类中定义的，所以这里使用 `RSACryptoServiceProvider` 类型。

```csharp
/// <summary>
/// Generates the RSA key.
/// </summary>
/// <param name="algorithm">The algorithm.</param>
/// <returns></returns>
public static void GenerateRSAKey(RSACryptoServiceProvider algorithm)
{
    RSAPrivateKey = algorithm.ToXmlString(true);

    using (StreamWriter streamWriter = new StreamWriter("PublicPrivateKey.xml"))
    {
        streamWriter.Write(RSAPrivateKey);
    }

    RSAPubicKey = algorithm.ToXmlString(false);
    using (StreamWriter streamWriter = new StreamWriter("PublicOnlyKey.xml"))
    {
        streamWriter.Write(RSAPubicKey);
    }
}
```

我们通过 `ToXmlString()` 方法生成公钥和密钥，然后把公钥和密钥写到流文件中，当我们要使用公钥和密钥时，可以通过读流文件把公钥和密钥读取出来。

OK 接下来让我们完成 RSA 的加密和解密方法吧！

```csharp
/// <summary>
/// Encrypts the specified algorithm.
/// The Asymmetric Algorithm includes DSA，ECDiffieHellman， ECDsa and RSA.
/// Code Logic:
/// 1. Input encrypt algorithm and plain text.
/// 2. Read the public key from stream.
/// 3. Serialize plian text to byte array.
/// 4. Encrypt the plian text array by public key.
/// 5. Return ciphered string.
/// </summary>
/// <param name="algorithm">The algorithm.</param>
/// <param name="plainText">The plain text.</param>
/// <returns></returns>
public static string Encrypt(RSACryptoServiceProvider algorithm, string plainText)
{
    string publicKey;
    List<Byte[]> cipherArray = new List<byte[]>();
    
    //// read the public key.
    using (StreamReader streamReader = new StreamReader("PublicOnlyKey.xml"))
    {
        publicKey = streamReader.ReadToEnd();
    }
    algorithm.FromXmlString(publicKey);

    BinaryFormatter binaryFormatter = new BinaryFormatter();
    byte[] plainBytes = null;

    //// Use BinaryFormatter to serialize plain text.
    using (MemoryStream memoryStream = new MemoryStream())
    {
        binaryFormatter.Serialize(memoryStream, plainText);
        plainBytes = memoryStream.ToArray();
    }

    int totLength = 0;
    int index = 0;

    //// Encrypt plain text by public key.
    if (plainBytes.Length > 80)
    {
        byte[] partPlainBytes;
        byte[] cipherBytes;
        myArray = new List<byte[]>();
        while (plainBytes.Length - index > 0)
        {
            partPlainBytes = plainBytes.Length - index > 80 ? new byte[80] : new byte[plainBytes.Length - index];

            for (int i = 0; i < 80 && (i + index) < plainBytes.Length; i++)
                partPlainBytes[i] = plainBytes[i + index];
            myArray.Add(partPlainBytes);

            cipherBytes = algorithm.Encrypt(partPlainBytes, false);
            totLength += cipherBytes.Length;
            index += 80;

            cipherArray.Add(cipherBytes);
        }
    }

    //// Convert to byte array.
    byte[] cipheredPlaintText = new byte[totLength];
    index = 0;
    foreach (byte[] item in cipherArray)
    {
        for (int i = 0; i < item.Length; i++)
        {
            cipheredPlaintText[i + index] = item[i];
        }

        index += item.Length;
    }
    return Convert.ToBase64String(cipheredPlaintText);
}
```

上面给出了 RSA 的加密方法实现，首先该方法传递一个 `RSACryptoServiceProvider` 类型的参数和明文，然后我们从流中读取预先已经创建的公钥，接着把明文序列化为 Byte 数组，而且使用公钥对明文进行加密处理，最后把加密的 Byte 数组转换为 base64 格式的字符串返回。

OK 现在让我们完成 RSA 的解密方法 `Decrypt()`，其实原理和前面的加密方法基本一致，只是我们要使用密钥对密文进行解密。

```csharp
/// <summary>
/// Decrypts the specified algorithm.
/// </summary>
/// <param name="algorithm">The algorithm.</param>
/// <param name="base64Text">The base64 text.</param>
/// <returns></returns>
public static string Decrypt(RSACryptoServiceProvider algorithm, string base64Text)
{
    byte[] cipherBytes = Convert.FromBase64String(base64Text);
    List<byte[]> plainArray = new List<byte[]>();
    string privateKey = string.Empty;

    //// Read the private key.
    using (StreamReader streamReader = new StreamReader("PublicPrivateKey.xml"))
    {
        privateKey = streamReader.ReadToEnd();
    }

    algorithm.FromXmlString(privateKey);

    int index = 0;
    int totLength = 0;
    byte[] partPlainText = null;
    byte[] plainBytes;
    int length = cipherBytes.Length / 2;
    //int j = 0;
    //// Decrypt the ciphered text through private key.
    while (cipherBytes.Length - index > 0)
    {
        partPlainText = cipherBytes.Length - index > 128 ? new byte[128] : new byte[cipherBytes.Length - index];

        for (int i = 0; i < 128 && (i + index) < cipherBytes.Length; i++)
            partPlainText[i] = cipherBytes[i + index];

        plainBytes = algorithm.Decrypt(partPlainText, false);
  
        totLength += plainBytes.Length;
        index += 128;
        plainArray.Add(plainBytes);

    }

    byte[] recoveredPlaintext = new byte[length];
    List<byte[]> recoveredArray;
    index = 0;
    for (int i = 0; i < plainArray.Count; i++)
    {
        for (int j = 0; j < plainArray[i].Length; j++)
        {
            recoveredPlaintext[index + j] = plainArray[i][j];
        }
        index += plainArray[i].Length;
    }

    BinaryFormatter bf = new BinaryFormatter();
    using (MemoryStream stream = new MemoryStream())
    {
        stream.Write(recoveredPlaintext, 0, recoveredPlaintext.Length);
        stream.Position = 0;
        string msgobj = (string)bf.Deserialize(stream);
        return msgobj;
    }
}
```

我们很快就实现 RSA 的解密方法实现，代码逻辑和加密算法基本一致，我们只需注意把明文换成密文，公钥换成密钥就基本不会出错了。

### Hash 加密

Hash 算法特别的地方在于它是一种单向算法，用户可以通过 Hash 算法对目标信息生成一段特定长度的唯一的 Hash 值，却不能通过这个 Hash 值重新获得目标信息，因此 Hash 算法常用在不可还原的密码存储、信息完整性校验等，所用我们代码逻辑就没有解密方法，而是判断加密的密文是否匹配。

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

    string[] tabStringHex = new string[16];
    UTF8Encoding UTF8 = new System.Text.UTF8Encoding();
    byte[] data = UTF8.GetBytes(dataToHash);
    byte[] result = hashAlgorithm.ComputeHash(data);
    StringBuilder hexResult = new StringBuilder(result.Length);

    for (int i = 0; i < result.Length; i++)
    {
        //// Convert to hexadecimal
        hexResult.Append(result[i].ToString("X2"));
    }
    return hexResult.ToString();
}
```

接着我们定义 `IsHashMatch()` 方法判断加密的密文是否一致。

```csharp
/// <summary>
/// Determines whether [is hash match] [the specified hash algorithm].
/// </summary>
/// <param name="hashAlgorithm">The hash algorithm.</param>
/// <param name="hashedText">The hashed text.</param>
/// <param name="unhashedText">The unhashed text.</param>
/// <returns>
///   <c>true</c> if [is hash match] [the specified hash algorithm]; otherwise, <c>false</c>.
/// </returns>
public static bool IsHashMatch(HashAlgorithm hashAlgorithm, string hashedText, string unhashedText)
{
    string hashedTextToCompare = Encrypt(hashAlgorithm, unhashedText);
    return (String.Compare(hashedText, hashedTextToCompare, false) == 0);
}
```

现在我们已经实现了属于我们的加密 Helper 类了，现在让我们测试一下自定义 Helper 类是否好用，我们可以通过 Unit Test 或一个界面程序来测试我们的 Helper 类的功能是否完善，这里我们选择使用一个界面程序来说话。

首先我们创建一个应用程序界面，然后我们添加两个 Tabpage 分别对应对称和非对称加密程序。

![[打造属于你的加密Helper类_03.jpg|图3 加密程序界面]]

![[打造属于你的加密Helper类_04.jpg|图4 非对称加密程序界面]]

![[打造属于你的加密Helper类_05.jpg|图5 对称加密程序界面]]

现在我们已经把界面设计好了，接下来让我们完成界面逻辑的设计。

```csharp
/// <summary>
/// Handles the Click event of the btnEncrypt control.
/// Encypt plain text by symmetric algorithm.
/// </summary>
/// <param name="sender">The source of the event.</param>
/// <param name="e">The <see cref="System.EventArgs"/> instance containing the event data.</param>
private void btnEncrypt_Click(object sender, EventArgs e)
{

    if (String.IsNullOrEmpty(txtInputMsg.Text) ||
        String.IsNullOrEmpty(txtRandomInit.Text) ||
        String.IsNullOrEmpty(txtRandomKey.Text))
    {
        MessageBox.Show("Invalid parameters!");
        return;
    }

    if (_symmetricAlgorithm != null)
    {
        try
        {
        //// Specified IV.
        _symmetricAlgorithm.IV = _initVector;

        //// Invoke Cryptography helper.
        /// Get ciphered text.
        txtCipherMsg.Text = CryptographyUtils.Encrypt(
            _symmetricAlgorithm,
            txtInputMsg.Text.Trim(),
            _key,
            _cipherMode,
            _paddingMode);

            //// Convert string to byte array.
            _cipherBytes = Convert.FromBase64String(txtCipherMsg.Text);

            //// Initialize text box.
            InitTextBox(_cipherBytes, ref txtCipherByte);
        }
        catch (Exception ex)
        {
            MessageBox.Show(ex.Message);
        }
    }
}
```

接着我们同样通过调用 Helper 类的 `Decrypt()` 方法，使用对称算法对数据进行解密处理就 OK 了。

```csharp
/// <summary>
/// Handles the Click event of the btnDecrypt control.
/// Decrypt plain text by symmetric algorithm
/// </summary>
/// <param name="sender">The source of the event.</param>
/// <param name="e">The <see cref="System.EventArgs"/> instance containing the event data.</param>
private void btnDecrypt_Click(object sender, EventArgs e)
{
    if (_symmetricAlgorithm != null)
    {
        try
        {
            //// Specified IV.
            _symmetricAlgorithm.IV = _initVector;

            //// Decrypt ciphered text.
            txtOutputMsg.Text = CryptographyUtils.Decrypt(
                _symmetricAlgorithm,
                txtCipherMsg.Text.Trim(),
                _key,
                _cipherMode,
                _paddingMode);
        }
        catch (Exception ex)
        {
            MessageBox.Show(ex.Message);
        }
    }
}
```

接着我们同样通过调用 Helper 类的 `Decrypt()` 方法，使用对称算法对数据进行解密处理就 OK 了。

![[打造属于你的加密Helper类_06.jpg|图6 对称加密程序效果]]

![[打造属于你的加密Helper类_07.jpg|图7 非对称加密程序效果]]

## 总结

通过本文的介绍相信大家对 .NET Framework 中提供的加密算法有了初步的了解。

最后，我们在实际应用中对于大量数据进行加密时应该采用对称加密算法，对称加密算法的优点在于加解密的高速度和使用长密钥时的难破解性。而非对称加密的缺点是加解密速度要远远慢于对称加密，非对称加密算法适合于数据量少的情况，应该始终考虑使用对称加密的方式进行文件的加解密工作。当然，如果文件加密后要传给网络中的其它接收者，而接收者始终要对文件进行解密的，这意味着密钥也是始终要传送给接收者的。这个时候，非对称加密就可以派上用场了，它可以用于字符串的加解密及安全传输场景。

[加密Helper源代码](https://files.cnblogs.com/rush/%E5%8A%A0%E5%AF%86Helper.zip)
