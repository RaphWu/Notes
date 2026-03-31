---
aliases:
date: 2024-10-01
update:
author: Will 保哥
language:
sourceurl: https://blog.miniasp.com/post/2024/10/01/Writing-C-Sharp-asynchronous-methods-without-async-await-and-simplifying-code-using-the-Unwrap-method
tags:
  - 非同步程式設計
  - CSharp
---

經 [BenchmarkDotNet](https://github.com/dotnet/BenchmarkDotNet) 實測驗證 [發現](https://share.linqpad.net/iruj6t.linq)，只要在現有的非同步方法加上 `async`/`await` 關鍵字，其方法的執行效能就有可能會降低 2,700 倍之多。這是因為 `async`/`await` 關鍵字會讓 C# 編譯器在編譯程式時，會將非同步方法轉換成一種**狀態機**的實作，這樣的轉換會增加程式的執行時間。因此，我們在大多數「函式庫」中的程式碼，幾乎都不會使用 `async`/`await` 關鍵字來撰寫程式。然而我們在函式庫中實作程式時，是有可能遇到多次非同步等待的情況，此時我們就可以使用 `TaskExtensions.Unwrap` 擴充方法來簡化寫法，今天這篇文章我就來說說這個開發技巧。

首先，我先來撰寫一個使用 `async`/`await` 實作的非同步方法，而在程式碼主體中你會發現有多次非同步等待的情況。

```csharp
async Task<string> FetchUrlContentAsync(string url)
{
    using HttpClient client = new HttpClient();
    HttpResponseMessage response = await client.GetAsync(url);
    response.EnsureSuccessStatusCode();
    return await response.Content.ReadAsStringAsync();
}
```

若要將這段程式碼轉換成不使用 `async`/`await` 關鍵字的非同步方法，我們可以利用 `ContinueWith` 方法來接續上一個 `Task` 的執行。

```csharp
Task<string> FetchUrlContentAsync2(string url)
{
    HttpClient client = new HttpClient();
    return client.GetAsync(url)
        .ContinueWith(responseTask =>
        {
            HttpResponseMessage response = responseTask.Result;
            response.EnsureSuccessStatusCode();
            return response.Content.ReadAsStringAsync();
        }).Unwrap();
}
```

接著我來抽絲剝繭上述程式碼的每一個細節：

1. 第一行 `HttpClient client = new HttpClient();` 不能用 `using` 關鍵字來宣告
   因為 `using` 關鍵字會讓 `client` 物件在離開 `using` 區塊時自動釋放資源。由於我們這份非同步方法並沒有使用 `async`/`await` 關鍵字的關係，所以這個非同步方法會在 `client.GetAsync` 方法開始執行之前，就已經結束並回傳 `Task<string>` 型別。所以當 `client.GetAsync` 方法開始執行時，`client` 物件已經被釋放，這樣的寫法會導致程式出現錯誤。
   一般來說 `HttpClient` 實例通常都會在程式中重複使用，而不是每次呼叫非同步方法都建立一個新的 `HttpClient` 物件，所以將 `HttpClient` 透過宣告為一個 `static` 物件，或是使用 `HttpClientFactory` 才是最佳實務的寫法。

2. 使用 `ContinueWith` 方法來接續上一個 `Task` 的執行，並回傳最後一個 `Task`
   由於我們的程式碼有「等待」的需求，但又不能用 `await` 的關係，所以我們可以使用 `ContinueWith` 方法來接續上一個 `Task` 的執行。這個 `ContinueWith` 方法中的第一個參數 `responseTask` 將會是一個「已經取得結果」的 `Task` 物件，所以執行 `responseTask.Result` 並不會封鎖執行緒 (Blocking thread)。
   但是我們又需要在這裡執行另一個 `response.Content.ReadAsStringAsync()` 非同步方法以取得結果，如果你寫成 `response.Content.ReadAsStringAsync().Result` 的話，就會有「封鎖執行緒」的情況發生，所以這並不是一個漂亮的寫法。
   但是，我們依然不想在 `ContinueWith` 方法中使用 `await` 關鍵字，所以我們勢必會回傳 `response.Content.ReadAsStringAsync()` 方法的回傳值，也就是一個 `Task` 物件。這樣的寫法會導致 `ContinueWith` 會回傳一個 `Task<Task<string>>` 型別，這種嵌套式 (Nested) 的 `Task` 有點討厭，如果要處理這種狀況，會讓程式變的有點複雜。
   還好微軟已經幫我們想到了解決方案，我們可以使用 `TaskExtensions.Unwrap` 擴充方法來簡化程式。`Unwrap` 方法會將 `Task<Task<string>>` 型別轉換成 `Task<string>` 型別，同時又不會造成執行緒的封鎖。 👍

> 你可以到 [source.dot.net](https://source.dot.net/) 網站查閱 [TaskExtensions.Unwrap](https://source.dot.net/#System.Private.CoreLib/src/libraries/System.Private.CoreLib/src/System/Threading/Tasks/TaskExtensions.cs,d9050196a8ec97d0) 的原始碼。

相關連結

- [TaskExtensions.Unwrap](https://source.dot.net/#System.Private.CoreLib/src/libraries/System.Private.CoreLib/src/System/Threading/Tasks/TaskExtensions.cs,d9050196a8ec97d0) 原始碼
- [dotnet/BenchmarkDotNet: Powerful .NET library for benchmarking](https://github.com/dotnet/BenchmarkDotNet)
