---
aliases:
date: 2016-04-12
update:
author: Jeffrey,黑暗執行緒
language: CSharp
sourceurl: https://blog.darkthread.net/blog/improved-getcachabledata
tags:
  - CSharp
  - CSharp_Span/Memory
---

# 改良式 GetCachableData 可快取查詢函式

多年前發展過一種 [可快取查詢](http://blog.darkthread.net/post-2010-06-04-cachable-data-object.aspx)：呼叫 `GetCachableData` 函式時傳入 Cache Key、查詢或產生資料 `Callback` 函式、Cache 保留期限（或指定閒置未用多久自動清除）三個參數，`GetCachableData` 會依「若 Cache 有資料就直接沿用；若 Cache 無資料則當場產生並存入 Cache」原則聰明處理，從此不需操心何時該查資料何時用 Cache，應用起來挺方便的。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Runtime.Caching;
using System.Text;
using System.Threading;
using System.Threading.Tasks;
 
namespace GetCachable
{
    public static class CacheManager
    {
        /// <summary>
        /// 取得可以被Cache的資料(注意：非Thread-Safe)
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key">Cache保存號碼牌</param>
        /// <param name="callback">傳回查詢資料的函數</param>
        /// <param name="cacheMins"></param>
        /// <param name="forceRefresh">是否清除Cache，重新查詢</param>
        /// <returns></returns>
        public static T GetCachableData<T>(string key, Func<T> callback, 
            int cacheMins, bool forceRefresh = false) where T : class
        {
            ObjectCache cache = MemoryCache.Default;
            string cacheKey = key;
 
            T res = cache[cacheKey] as T;
            //是否清除Cache，強制重查
            if (res != null && forceRefresh)
            {
                cache.Remove(cacheKey);
                res = null;
            }
            if (res == null)
            {
                res = callback();
                cache.Add(cacheKey, res, 
                    new CacheItemPolicy() {
                        SlidingExpiration = new TimeSpan(0, cacheMins, 0)
                    });
            }
            return res;
        }
 
        /// <summary>
        /// 取得可以被Cache的資料(注意：非Thread-Safe)
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key">Cache保存號碼牌</param>
        /// <param name="callback">傳回查詢資料的函數</param>
        /// <param name="absExpire">有效期限</param>
        /// <param name="forceRefresh">是否清除Cache，重新查詢</param>
        /// <returns></returns>
        public static T GetCachableData<T>(string key, Func<T> callback, 
            DateTimeOffset absExpire, bool forceRefresh = false) where T : class
        {
            ObjectCache cache = MemoryCache.Default;
            string cacheKey = key;
            //取得每個Key專屬的鎖定對象
            T res = cache[cacheKey] as T;
            //是否清除Cache，強制重查
            if (res != null && forceRefresh)
            {
                cache.Remove(cacheKey);
                res = null;
            }
            if (res == null)
            {
                res = callback();
                cache.Add(cacheKey, res, new CacheItemPolicy()
                {
                    AbsoluteExpiration = absExpire
                });
            }
            return res;
        }
    }
}
```

不過原本的設計有個小問題，例如：有個網站透過 `GetCachableData` 由資料庫讀取五千筆員工資料並 Cache 住一小時，以便後續能快速地用員編查姓名。想像一個場景，尖峰時刻 Cache 逾時被清除（或是網站因故重啟），線上一百名使用者同時瀏覽某一網頁使用到員工姓名查詢，於是 `GetCachableData` 同時被 100 條 Thread 呼叫，`MemoryCache` 本身為 Thread-Safe 多執行緒讀寫不致出錯，但 Cache 不存在觸發 100 個資料庫查詢，對形成一波完美的 DDoS 攻擊！接著資料庫忙碌、網頁卡住、使用者無助、老闆暴怒、開發者想哭…

以下範例可展示此問題，同時開啟三條 Thread 呼叫 `GetCachableData`，則 Callback 動作也會同時三份（Callback 執行時會印出 Thread n Start/Stop Job 訊息以利觀察）。這三次查詢動作只有一次是必要的，其餘兩次將取得相同結果覆寫同一 Cache，平白消耗資源，在極端案例中甚至可能讓系統崩潰。

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;
using System.Threading;
using System.Threading.Tasks;
 
namespace GetCachable
{
    class Program
    {
        static void Main(string[] args)
        {
            var tasks = new List<Task>();
            for (var i = 0; i < 3; i++)
            {
                tasks.Add(Task.Factory.StartNew(() =>
                {
                    var data = CacheManager.GetCachableData<string>("KEY", () =>
                    {
                        Console.WriteLine("Thread {0} Start Job",
                            Thread.CurrentThread.ManagedThreadId);
                        Thread.Sleep(3000);
                        Console.WriteLine("Thread {0} Stop Job",
                            Thread.CurrentThread.ManagedThreadId);
                        return "OK";
                    }, 10);
                    Console.WriteLine("Data:" + data);
                }));
            }
            tasks.ForEach(t => t.Wait());
 
            Console.WriteLine("Done");
            Console.ReadLine();
        }
    }
}
```

執行結果如下，可觀察到三條 Thread 同時執行 Callback：

> Thread 13 Start Job
> Thread 11 Start Job
> Thread 12 Start Job
> Thread 13 Stop Job
> Data:OK
> Thread 11 Stop Job
> Thread 12 Stop Job
> Data:OK
> Data:OK
> Done

要改良此一缺點，可在多執行緒查詢時加入 `Lock` 機制，相同 `Key` 值的查詢單一時間只允許一組 Callback 執行，執行完成後其餘等待的 Thread 可直接取用 Cache 結果，省下無效益的 Callback 動作。程式範例如下，==依 Key 值建立 Object 作為鎖定對象==，即能實現一 Key 值不會有兩份以上 Callback 同時執行：

```csharp
using System;
using System.Collections.Generic;
using System.Linq;
using System.Runtime.Caching;
using System.Text;
using System.Threading.Tasks;
 
namespace GetCachable
{
    public static class BetterCacheManager
    {
        //加入Lock機制限定同一Key同一時間只有一個Callback執行
        const string AsyncLockPrefix = "$$CacheAsyncLock#";
        
        /// <summary>
        /// 取得每個Key專屬的鎖定對象
        /// </summary>
        /// <param name="key">Cache保存號碼牌</param>
        /// <returns></returns>
        static object GetAsyncLock(string key)
        {
            ObjectCache cache = MemoryCache.Default;
            //取得每個Key專屬的鎖定對象（object）
            string asyncLockKey = AsyncLockPrefix + key;
            lock (cache)
            {
                if (cache[asyncLockKey] == null) cache.Add(asyncLockKey,
                    new object(),
                    new CacheItemPolicy() {
                        SlidingExpiration = new TimeSpan(0, 10, 0)
                    });
            }
            return cache[asyncLockKey];
        }
 
        /// <summary>
        /// 取得可以被Cache的資料(注意：非Thread-Safe)
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key">Cache保存號碼牌</param>
        /// <param name="callback">傳回查詢資料的函數</param>
        /// <param name="cacheMins"></param>
        /// <param name="forceRefresh">是否清除Cache，重新查詢</param>
        /// <returns></returns>
        public static T GetCachableData<T>(string key, Func<T> callback, 
            int cacheMins, bool forceRefresh = false) where T : class
        {
            ObjectCache cache = MemoryCache.Default;
            string cacheKey = key;
 
            //取得每個Key專屬的鎖定對象
            lock (GetAsyncLock(key))
            {
                T res = cache[cacheKey] as T;
                //是否清除Cache，強制重查
                if (res != null && forceRefresh)
                {
                    cache.Remove(cacheKey);
                    res = null;
                }
                if (res == null)
                {
                    res = callback();
                    cache.Add(cacheKey, res, 
                        new CacheItemPolicy() {
                            SlidingExpiration = new TimeSpan(0, cacheMins, 0)
                        });
                }
                return res;
            }
        }
 
        /// <summary>
        /// 取得可以被Cache的資料(注意：非Thread-Safe)
        /// </summary>
        /// <typeparam name="T"></typeparam>
        /// <param name="key">Cache保存號碼牌</param>
        /// <param name="callback">傳回查詢資料的函數</param>
        /// <param name="absExpire">有效期限</param>
        /// <param name="forceRefresh">是否清除Cache，重新查詢</param>
        /// <returns></returns>
        public static T GetCachableData<T>(string key, Func<T> callback, 
            DateTimeOffset absExpire, bool forceRefresh = false) where T : class
        {
            ObjectCache cache = MemoryCache.Default;
            string cacheKey = key;
            //取得每個Key專屬的鎖定對象
            lock (GetAsyncLock(key))
            {
                T res = cache[cacheKey] as T;
                //是否清除Cache，強制重查
                if (res != null && forceRefresh)
                {
                    cache.Remove(cacheKey);
                    res = null;
                }
                if (res == null)
                {
                    res = callback();
                    cache.Add(cacheKey, res, new CacheItemPolicy()
                    {
                        AbsoluteExpiration = absExpire
                    });
                }
                return res;
            }
        }
    }
}
```

改用 `BetterCacheManager` 後，同時三條 Thread 呼叫 `GetCachableData()` 只會觸發一次 Callback，可減少高承載系統產生重複查詢的壓力：

> Thread 9 Start Job
> Thread 9 Stop Job
> Data:OK
> Data:OK
> Data:OK
> Done

以上私房做法，提供大家參考。
