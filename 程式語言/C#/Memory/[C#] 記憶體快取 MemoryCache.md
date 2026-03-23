---
aliases:
date: 2016-09-23
update:
author: Brian
language: CSharp
sourceurl: https://dotblogs.com.tw/LazyCodeStyle/2016/07/24/155838
tags:
  - CSharp
  - CSharp_Span/Memory
---

# 記憶體快取 MemoryCache

最近開發一個 Web Api 程式，它每次被呼叫都必須先讀取一份 XML 檔案，然後將內容反序列化成一個清單變數。但由於 Web Api 物件的生命週期都只維持一個 Request，所以每次 Request 都必須要重新反序列化，這樣便會造成系統效率下降。因此為避免這樣的情況發生，便很適合導入快取的機制到此系統

## 快取注意事項

使用「快取」這種機制前，最該要注意的是「資料一致」的問題，也就是在「資料一致」的前提下，如何設計一種機制讓「快取」可以使用最久。

在 MemoryCache 中，清除「快取」資料的機制可分為「時間」及「變動」兩大類:

1. 時間
    1. 絕對時間：當現在時間超過某個設定時間點時，此快取資料就會被清除
    2. 使用時間間隔：當此快取資料超過多久沒被使用到的話，就會被清除掉
2. 變動
    1. 檔案：當某檔案內容有更動時，快取資料就會被清除
    2. 資料庫內容：當 SQL 語法的執行結果有不同時，快取資料會被清除

## 使用方式

MemoryCache 的使用方式非常單純，當快取中找不到呼叫的 key 值時，會回傳 object 的預設值。

CacheItemPolicy 就是先前所說的快取清除機制，像下述範例程式中所使用的 AbsoluteExpiration 就是「絕對時間」的設定，當時間超過現在時間的一天之後，此快取資料就會被清除。

```cs
static int GetGrade(string name)
{
    if(MemoryCache.Default[name] == default(object))
    {
        CacheItemPolicy policy = new CacheItemPolicy
        {
            AbsoluteExpiration = DateTimeOffset.Now.AddDays(1),
        };
        
        MemoryCache.Default.Add(name, new Random().Next(0, 100), policy);
    }

    return Convert.ToInt32(MemoryCache.Default[name]);
}
```

更改成「使用時間間隔」機制，當此快取資料超過一分鐘沒被讀取到時，這快取資料就會重新被讀取，其餘程式碼跟上述範例一樣。但要注意一點，「絕對時間」跟「使用時間間隔」只能二選一使用。

```cs
CacheItemPolicy policy = new CacheItemPolicy
{
    SlidingExpiration = new TimeSpan(0, 1, 0)
};
```

如果快取的內容是來自於檔案內容時，這時就是要改用「檔案變動」的清除機制了。「檔案變動」的機制可以與「時間」一起做搭配，且可同時監控多個檔案是否有變動，但監控檔案的路徑只能使用絕對路徑，不能使用相對路徑。

```cs
CacheItemPolicy policy = new CacheItemPolicy
{
    SlidingExpiration = new TimeSpan(0, 1, 1)
};

var filePaths = new List<string>() { "D://config.xml", "D://name.xml" };
policy.ChangeMonitors.Add(new HostFileChangeMonitor(monitorFilePaths));
```

## 參考資料

[保哥 - ASP.NET 4 快取 API 有兩種：Cache 與 ObjectCache](http://blog.miniasp.com/post/2010/05/01/ASPNET-4-Cache-API-and-ObjectCache.aspx)
[搞搞就懂 - [C#] 初探 MemoryCache 及使用方式介紹](https://dotblogs.com.tw/wasichris/2015/11/14/153922)
