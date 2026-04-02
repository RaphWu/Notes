---
aliases:
date: 2015-11-15
update: 2015-12-29
author: CHRIS CHEN
language: CSharp
sourceurl: https://dotblogs.com.tw/wasichris/2015/11/14/153922
tags:
  - CSharp
  - CSharp_Span/Memory
---

# C# 初探 MemoryCache 及使用方式介紹

初探 MemoryCache 及使用方式介紹

## 前言

一般而言系統資料多來自遠端資料庫及 WebAPI 等，若頻繁地從資料源撈取資料，有可能造成資料提供者的負擔，且系統亦需承擔因頻繁撈取資料所造成的等待時間成本；這時我們可以考慮使用快取 (Cache) 來增加資料讀取效率，避免上述衝擊發生。簡單來說，快取就是以資料時效性及記憶體空間來換取資料獲得效能，所以可能會面臨資料不同步 (資料源與快取) 的問題，因此決定何時要收回快取來向資料源重新撈取資料，是程式開發者需要考量的一個重要議題，以下介紹幾種不同快取回收時機，可以依照自身需求選擇適用之條件。

![[MemoryCache及使用方式介紹_01.png]]

## 快取使用方式

從 .NET 4.0 開始，我們可以載入 `System.Runtime.Caching` 組件來實現快取機制，透過 `MemoryCache.Default` 來取得預設記憶體快取實體，使用方式就類似操作 `Session` 資料，但可依照需求來自定快取回收時機。

### 1. 加入快取 (Set, Add, AddOrGetExisting)

- `Set` (快取已存在時，直接覆寫)
- `Cache[key]=value`  (快取已存在時，直接覆寫，但無法設定 `CacheItemPolicy`)
- `Add` (快取已存在時，不會覆寫原有設定，會回傳 `false` 結果告知新增失敗)

![[MemoryCache及使用方式介紹_02.png]]

### 2. 設定快取回收時機 ( [CacheItemPolicy](https://learn.microsoft.com/en-us/dotnet/api/system.runtime.caching.cacheitempolicy) )

讓系統可以在適當的時機點再次向資料源請求資料，作為新快取值供系統取用，可用邏輯如下。

- 指定時間是否到期 (Expiration)
    - `AbsoluteExpiration` (ex. 設定快取後 10s 回收快取)
    - `SlidingExpiration` (ex. 10s 內沒人取用就回收快取)
- 監控資料源是否改變 (ChangeMonitor)
    - `HostFileChangeMonitor` (實體檔案異動時回收快取)
    - `SqlChangeMonitor` (DB 檔案異動時回收快取)
    - `CustomizedChangeMonitor` ( 可以繼承實作 ChangeMonitor 類別來建立獨有的監控邏輯)

### 3. 接收快取異動通知 (Callback)

- `UpdateCallback` (回收快取 - 前)
- `RemovedCallback` (回收快取 - 後)

## 應用層面

舉個簡單範例來比較各種快取回收機制。需求是這樣，筆者有許多資料會放置在 TXT 檔案中，這些資料都是系統會經常使用到的資訊，但我們不希望頻繁地去讀取該檔案，以避免頻繁讀取磁碟而造成效能的消耗，因此希望加入快取來達成我們的目標；由於此範例重點在比較各快取收回機制效果，因此就不著墨適用性的問題了。

首先定義 `DataSourceProvide` 做為資料來源類別，其中 `FileContents` 屬性會從實體檔案中取得資料，接著會將資料存放於快取中，由於筆者想要驗證不同快取回收機制所產生的效果，因此提供三種快取機制供切換測試使用；由於資料來自於實體檔案，因此除了可在時間上的操作快取回收時機外，還可以使用 `HostFileChangeMonitor` 做為資料異動監視器，如有異動隨即回收快取。測試程式代碼如下所示。

```csharp
public class DataSourceProvider
{
    // fields
    private ObjectCache _cache = MemoryCache.Default; 

    // proterties
    public CacheEntryRemovedCallback OnFileContentsCacheRemove;
    public string PolicyType { get; set; }
    public string FileContents
    {
        get
        {
            string cacheKey = "FileContents";
            string fileContents = _cache[cacheKey] as string;
            if (string.IsNullOrWhiteSpace(fileContents))
            {
                // load file
                string filePath = @"D:\Holidays.txt";
                fileContents = File.ReadAllText(filePath, Encoding.Default); 

                // set policy (when to remove cache)
                var policy = new CacheItemPolicy();
                policy.RemovedCallback = OnFileContentsCacheRemove;

                // dynamic change policy for testing 
                switch (PolicyType)
                {
                    case "1":
                        // 距離設定快取時間超過2秒後，回收快取
                        policy.AbsoluteExpiration = DateTimeOffset.Now.AddSeconds(2);
                        break;
                    case "2":
                        // 3秒期限內未使用快取時，回收快取
                        policy.SlidingExpiration = TimeSpan.FromSeconds(3);
                        break;
                    default:
                        // 資料異動時，回收快取
                        policy.ChangeMonitors.Add(new HostFileChangeMonitor(new List<string>() { filePath }));
                        break;
                }

                // set cache
                _cache.Set(cacheKey, fileContents, policy);
            }

            return fileContents;
        }
    }
}
```

測試主程式如下，可選擇不同快取回收機制 (`CacheItemPolicy`) 來比較各自成效。其中設定快取被回收後 `Callback` 方法，讓它直接印出 cache removed 訊息告知用戶，方便後續測試時於畫面顯示快取被回收的時機點。

```csharp
static void Main(string[] args)
{
    // data source provider
    var dataSource = new DataSourceProvider();

    // set cache removed callback
    dataSource.OnFileContentsCacheRemove = (arguments) =>
    {
        // cache key: arguments.CacheItem.Key
        // cache value: arguments.CacheItem.Value as string

        Console.WriteLine("==============> cache removed "); 
    };

    // choose cache policy
    Console.WriteLine("[1] AbsoluteExpiration(2s) ");
    Console.WriteLine("[2] SlidingExpiration(3s) ");
    Console.WriteLine("[3] ChangeMonitors ");
    Console.Write("Please choose cache policy: ");
    dataSource.PolicyType = Console.ReadLine();

    // show data for testing cache
    while (true)
    {
        Console.WriteLine("[time] " + DateTime.Now.ToString("yyyy/MM/dd hh:mm:ss.fff"));
        Console.WriteLine("[cache value] " + dataSource.FileContents);
        
        string cmd = Console.ReadLine();
        if (cmd == "exit") { break; }
    }
}
```

程式啟動後，首先就是選擇所需使用的 `CacheItemPolicy` 機制

![[MemoryCache及使用方式介紹_03.png]]

當選擇 AbsoluteExpiration 作為 CacheItemPolicy 時，表示設定快取 2 秒後會回收快取； 從以下測試發現當設定快取後，時間超過 2 秒確實會回收快取，且在下一次取資料時，再次讀取實體檔案來載入至快取。

![[MemoryCache及使用方式介紹_04.png]]

若使用 SlidingExpiration 作為快取 Policy 時，表示快取超過 3 秒沒被取用的情況下，快取會自動被收回，結果如下圖所示；所以換句話說，如果此快取密集地不斷被取用 (3 秒內至少一次)，此快取將永不被回收。

![[MemoryCache及使用方式介紹_05.png]]

最後，若使用 ChangeMonitors 作為快取 Policy 時，則表示當實體檔案資料來源只要一有異動，快取就隨即被收回；因此在這個 CacheItemPolicy 中，快取回收機制將與時間沒有任何關係了。

![[MemoryCache及使用方式介紹_06.png]]

## 結論

綜觀上述三種方式，可能會有人覺得當然是使用 `ChangeMonitors` 作為 `CacheItemPolicy` 最好，在資料源被異動時馬上收回快取，讓資料不會有不同步的情況發生，又兼具快取特性；但是試想一個情況，當資料來源每秒都在不斷地變動時 (也許是匯率)，而我所需的資料其實不需要這麼即時，因為每次取得的匯率資料是可以被保留使用 10 秒鐘，因此在這情境中是否可以使用 `AbsoluteExpiration` 作為快取回收機制，讓快取經過 10 秒後過期被收回才會比較合適呢? 其實筆者想表達的意思是，==快取的使用不一定適用所有情境，需要搭配不同快取回收機制才能達到提升效能的效果，因此請不要一昧地加註快取機制於系統中，否則可能會造成反效果。==

## 參考資訊

[http://blog.miniasp.com/post/2010/05/01/ASPNET-4-Cache-API-and-ObjectCache.aspx](http://blog.miniasp.com/post/2010/05/01/ASPNET-4-Cache-API-and-ObjectCache.aspx "http://blog.miniasp.com/post/2010/05/01/ASPNET-4-Cache-API-and-ObjectCache.aspx")

[http://blog.darkthread.net/blogs/darkthreadtw/archive/2008/06/23/kb-cache-add-vs-cache-insert.aspx](http://blog.darkthread.net/blogs/darkthreadtw/archive/2008/06/23/kb-cache-add-vs-cache-insert.aspx "http://blog.darkthread.net/blogs/darkthreadtw/archive/2008/06/23/kb-cache-add-vs-cache-insert.aspx")
