---
aliases:
date: 2010-05-01
update:
author: Will 保哥
language: ASP.NET Core
sourceurl: https://blog.miniasp.com/post/2010/05/01/ASPNET-4-Cache-API-and-ObjectCache
tags:
  - ASP
---

# ASP.NET 4 快取 API 有兩種：Cache 與 ObjectCache

ASP.NET 從最早期的版本就實做了一套好用的**快取機制** ([System.Web.Caching.Cache](http://msdn.microsoft.com/en-us/library/system.web.caching.cache.aspx?WT.mc_id=DT-MVP-4015686))，一直以來任何非 ASP.NET 的應用程式 (例如 WinForm, WPF, Console, WinService, …) 若要使用快取機制都必須將 System.Web.dll 參考進專案才能使用，但從 .NET 4.0 開始出現了另一個擴充性更強的快取機制，稱為 Object Caching (物件快取) 機制，未來這兩套快取機制將各司其職、相輔相成。

從 .NET 4.0 開始，.Net Framework 參考了 ASP.NET 內建的快取機制，並明確實做了另一個獨立存在的快取組件：System.Runtime.Caching.dll，此組件非常的小，我用 [NDepend](http://www.ndepend.com/) 工具分析了一下此組件，總共 IL 指令集僅有 **9,844** 個而已，而 System.Web.dll 的 IL 指令集總共有 **496,395** 個之多，就算只計算 System.Web.Caching 命名空間的 IL 也有 **14,760** 個，還是比這次新增的 System.Runtime.Caching.dll 來的大。

_備註：原本的 System.Web.Caching.Cache 類別依然存在，功能與之前一樣，還是可以正常使用。_

這個新的 System.Runtime.Caching.dll 組件在 System.Runtime.Caching 命名空間 包含了一組全新的 Cache API，而此 API 包含了兩組核心類別：

- 透過一組抽象類別可讓你自行實做任意物件快取機制，例如: 實做分散式快取機制。
- 透過此抽象類別實做的**記憶體快取機制**類別**:** [System.Runtime.Caching.MemoryCache 類別](http://msdn.microsoft.com/en-us/library/system.runtime.caching.memorycache.aspx?WT.mc_id=DT-MVP-4015686)。

如果你曾經用過 ASP.NET Cache 物件，因為兩者的相似性非常高，所以要上手使用 [MemoryCache](http://msdn.microsoft.com/en-us/library/system.runtime.caching.memorycache.aspx?WT.mc_id=DT-MVP-4015686) 那可是非常容易，以下是 [MemoryCache](http://msdn.microsoft.com/en-us/library/system.runtime.caching.memorycache.aspx?WT.mc_id=DT-MVP-4015686) 的使用範例 ( 以 Windows Form 做範例 )：

```csharp
private void btnGet_Click(object sender, EventArgs e)
{
    //Obtain a reference to the default MemoryCache instance. 
    //Note that you can create multiple MemoryCache(s) inside 
    //of a single application. 
    ObjectCache cache = MemoryCache.Default;

    //In this example the cache is storing the contents of a file string 
    fileContents = cache["filecontents"] as string;

    //If the file contents are not currently in the cache, then 
    //the contents are read from disk and placed in the cache. 
    if (fileContents == null)
    {
        //A CacheItemPolicy object holds all the pieces of cache 
        //dependency and cache expiration metadata related to a single 
        //cache entry. 
        CacheItemPolicy policy = new CacheItemPolicy();

        //Build up the information necessary to create a file dependency. 
        //In this case we just need the file path of the file on disk. 
        List filePaths = new List();
        filePaths.Add("c:\\data.txt");

        //In the new cache API, dependencies are called "change monitors". 
        //For this example we want the cache entry to be automatically expired 
        //if the contents on disk change. A HostFileChangeMonitor provides 
        //this functionality. 
        policy.ChangeMonitors.Add(new HostFileChangeMonitor(filePaths));

        //Fetch the file's contents 
        fileContents = File.ReadAllText("c:\\data.txt");

        //And then store the file's contents in the cache 
        cache.Set("filecontents", fileContents, policy);
    }
    MessageBox.Show(fileContents);
}
```

System.Web.Caching 中內建的 [Cache](http://msdn.microsoft.com/en-us/library/system.web.caching.cache.aspx?WT.mc_id=DT-MVP-4015686) 與 System.Runtime.Caching 的 [MemoryCache](http://msdn.microsoft.com/en-us/library/system.runtime.caching.memorycache.aspx?WT.mc_id=DT-MVP-4015686) 有幾點需注意：

- System.Runtime.Caching.dll 組件下的 [MemoryCache](http://msdn.microsoft.com/en-us/library/system.runtime.caching.memorycache.aspx?WT.mc_id=DT-MVP-4015686) 類別與 System.Web.dll 組件下的 [Cache](http://msdn.microsoft.com/en-us/library/system.web.caching.cache.aspx?WT.mc_id=DT-MVP-4015686) 類別完全沒有相依性，兩者是完全獨立的組件，任何**非 ASP.NET 應用程式**都不應該載入 System.Web.dll 組件來實做快取。
- ASP.NET 的 [Cache](http://msdn.microsoft.com/en-us/library/system.web.caching.cache.aspx?WT.mc_id=DT-MVP-4015686) 與 [MemoryCache](http://msdn.microsoft.com/en-us/library/system.runtime.caching.memorycache.aspx?WT.mc_id=DT-MVP-4015686) 雖然都是**記憶體快取**(In-memory cache)，但在 ASP.NET 中只能使用一份 [Cache](http://msdn.microsoft.com/en-us/library/system.web.caching.cache.aspx?WT.mc_id=DT-MVP-4015686) 物件，而在 [MemoryCache](http://msdn.microsoft.com/en-us/library/system.runtime.caching.memorycache.aspx?WT.mc_id=DT-MVP-4015686) 可在 AppDomain 中建立多份快取物件。

像我們之前在開發大型網站時，由於 ASP.NET 快取資料只能儲存在記憶體中，這對於大型網站來說非常不實用，甚至於**”不能用”**，有了 [ObjectCache](http://msdn.microsoft.com/en-us/library/system.runtime.caching.objectcache.aspx?WT.mc_id=DT-MVP-4015686) 機制這才出現了一絲生機，這時你就可以在 ASP.NET 中透過 [ObjectCache](http://msdn.microsoft.com/en-us/library/system.runtime.caching.objectcache.aspx?WT.mc_id=DT-MVP-4015686) 機制載入自訂的快取機制 (例如: [Velocity](http://msdn.microsoft.com/appfabric?WT.mc_id=DT-MVP-4015686), [Memcached](http://memcached.org/), [ScaleOut](http://www.scaleoutsoftware.com/), … ) 來加強網站的延展性 (Scalability)，相信此功能對擁有大量快取需求的開發人員來說絕對是一大福音。

其他更進階的資訊請參考以下相關連結。

**相關連結**

- [System.Runtime.Caching.MemoryCache 類別](http://msdn.microsoft.com/en-us/library/system.runtime.caching.memorycache.aspx?WT.mc_id=DT-MVP-4015686)
- [System.Web.Caching.Cache 類別](http://msdn.microsoft.com/en-us/library/system.web.caching.cache.aspx?WT.mc_id=DT-MVP-4015686)
- [.NET Framework 4 - ASP.NET: ASP.NET Caching](http://msdn.microsoft.com/en-us/library/xsbfdd8c.aspx?WT.mc_id=DT-MVP-4015686)
- [Microsoft ASP.NET 4 Core Runtime for Web Developers](http://microsoftpdc.com/Sessions/FT57) [ PDC 2009 ]
- [.Net 4.0 system.runtime.caching](http://www.slideshare.net/larrynung/net-40-systemruntimecaching) (Updated: 2013/06/10)
- [.NET 4.0 New Feature - System.Runtime.Caching](http://www.dotblogs.com.tw/larrynung/archive/2010/11/26/19746.aspx) (Updated: 2013/06/12)
- [How to custom .NET 4.0 System.Runtime.Caching.ChangeMonitor](http://www.dotblogs.com.tw/larrynung/archive/2013/06/12/105458.aspx) (Updated: 2013/06/12)
