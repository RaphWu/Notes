---
aliases:
created: 
update:
author: Mars
language: CSharp
sourceurl: https://ithelp.ithome.com.tw/articles/10258503
tags:
  - CSharp
  - CSharp_Span/Memory
date: 2021-08-24
---

# 使用記憶體快取 MemoryCache 增加回應速度

在應用程式中有許多向資料庫讀取資料的動作，為了增加程式效能， 有 2 個方向可以調整。
第 1 種是直接調效 SQL 的效能，減少 SQL 的不良寫法，讓資料庫查找資料的速度變快。
第 2 種是直接將上次查到的結果，先暫存在記憶體中，如果資料庫的內容沒有變動的話，直接回傳剛剛已查過的資料內容，可以有效增加系統回應速度。

在 C# .Net 4.0 開始，我們可以善用 System.Runtime.Caching 的 MemoryCache 的快取機制來增加程式回應的速度。
記憶體內的存取速度遠遠比向資料庫取資料快很多，而資料庫取得的資料有時會因為網路限制而變更慢，所以在建置應用程式的時候，適時的加入記憶體快取機制在專案中可以有效增加效能。

## 什麼是 MemoryCache

MemoryCache 是實作記憶體內存取快取的方法。快取可以減少產生內容的工作，將已取得的資料暫存在記憶體空間，並指定暫存時間，可依時間到期而回收或是手動回收。
若應用程式重啟時，原有存在快取的資料將會一併自動清除。此概念跟電腦的記憶體是相同的，存放在記憶體內的資料，當電腦重開機後也會一併自動清除。

## 範例 1 指定時間後回收快取

此範例是說明讀寫快取的方法，而快取存活時間為 60 分鐘後即立刻回收

```java
ObjectCache Cache = MemoryCache.Default;

string CacheName = "Cache1";

// 檢查快取是否存在
bool isSet = Cache[CacheName] != null;

// 取得資料庫取得物件(省略動作)
DataTable dt = new DataTable();

// 寫入快取 (指定時間後回收快取)
CacheItemPolicy policy = new CacheItemPolicy();
policy.AbsoluteExpiration = DateTime.Now + TimeSpan.FromMinutes(60); // 60 分鐘後回收
Cache.Add(new CacheItem(CacheName, dt), policy);

// 讀取快取
dt = (DataTable)Cache[CacheName];

// 移除快取
Cache.Remove(CacheName);
```

此範例講解了存取 4 個主要動作，

1. 是否存在
2. 寫入
3. 讀取
4. 移除

向記憶體存取需指定一個名稱，Ex: Cache1，存取都需要用相同的名稱。

範例中的 60 分鐘是可以改變單位的，換成秒也是可以。
若要換成 60 秒的話，寫法改成

```csharp
policy.AbsoluteExpiration = DateTime.Now + TimeSpan.FromSeconds (60); // 60 秒後回收
```

## 範例 2 指定時間未使用時回收快取

此範例是講解資料寫入快取後，如果在 60 分鐘內沒有被讀取時則會回收，若有被讀取時，則會再延長 60 分鐘

```java
ObjectCache Cache = MemoryCache.Default;

string CacheName = "Cache2";

// 檢查快取是否存在
bool isSet = Cache[CacheName] != null;

// 取得資料庫取得物件(省略動作)
DataTable dt = new DataTable();

// 寫入快取 (指定時間未使用時回收快取)
CacheItemPolicy policy = new CacheItemPolicy();
policy.SlidingExpiration = TimeSpan.FromMinutes(60); // 若 60 分鐘內未呼叫此快取就會回收，若有呼叫則會再延長 60 分鐘
Cache.Add(new CacheItem(CacheName, dt), policy);

// 讀取快取
dt = (DataTable)Cache[CacheName];

// 移除快取
Cache.Remove(CacheName);
```

此範例跟第 1 個差異就是存活時間，如果這個快取有被使用則自動延長存活時間。
此兩種範例都可以依實務上的需求而選擇使用。
範例中的 60 分鐘若要換成 60 秒的話，寫法改成

```csharp
policy.SlidingExpiration = TimeSpan. FromSeconds(60); // 60秒
```

## 自定常用方法類別

以上介紹了 2 種讀寫快取的方法，而快取還有另一種存活時間控制是 ChangeMonitors ，當有資料異動時就回收快取，但我沒有實際上的應用，就沒有實作範例了。

接下來我會展示我將快取常用的方法建立在 Class ，提供 4 個主要方法 `IsSet`, `Set`, `Get`, `Remove` 呼叫。

```csharp
public class AppCache
{
	#region 屬性
	private ObjectCache Cache
	{
		get
		{
			return MemoryCache.Default;
		}
	}

	// 因為與其他應用程式共用此記憶體快取，所以建議增加此應用程式的前置名稱
	private string IdNameStart = "MyProjectName_";
	#endregion

	#region 建構子
	public AppCache()
	{

	}
	#endregion

	#region 方法
	/// <summary>
	/// 取得快取
	/// </summary>
	/// <param name="key"></param>
	/// <returns></returns>
	public object Get(string key)
	{
		return Cache[IdNameStart + key];
	}

	/// <summary>
	/// 移除快取
	/// </summary>
	/// <param name="key"></param>
	public void Remove(string key)
	{
		Cache.Remove(IdNameStart + key);
	}

	/// <summary>
	/// 是否存在快取
	/// </summary>
	/// <param name="key"></param>
	/// <returns></returns>
	public bool IsSet(string key)
	{
		return (Cache[IdNameStart + key] != null);
	}

	/// <summary>
	/// 設定快取
	/// </summary>
	/// <param name="key">KEY</param>
	/// <param name="data">資料</param>
	public void Set(string key, object data)
	{
		this.Set(key, data, "Sliding", 60);
	}

	/// <summary>
	/// 設定快取
	/// </summary>
	/// <param name="key">KEY</param>
	/// <param name="data">資料</param>
	/// <param name="Expiration">保留別</param>
	/// <param name="cacheTime">保存時間(分鐘)</param>
	public void Set(string key, object data, string Expiration, int cacheTime)
	{
		CacheItemPolicy policy = new CacheItemPolicy();
		if (Expiration == "Absolute")
		{
			policy.AbsoluteExpiration = DateTime.Now + TimeSpan.FromMinutes(cacheTime);
		}
		else if (Expiration == "Sliding")
		{
			policy.SlidingExpiration = TimeSpan.FromMinutes(cacheTime);
		}
		Cache.Add(new CacheItem(IdNameStart + key, data), policy);
	}
	#endregion
}
```

寫好此類別，就可以方便使用。

使用的範例可以參考這個

```java
AppCache cache = new AppCache();

// 檢查Cache 是否存在
DataTable dt = null;
if (cache.IsSet("Cache1") == false)
{
	// 取得資料庫取得物件(省略動作)
	dt = new DataTable();

	// 設定新 Cache ， 60 分鐘後就回收快取
	cache.Set("Cache1", dt, "Absolute", 60);
}
else
{
	// 取得 Cache 
	dt = (DataTable)cache.Get("Cache1");
}

// 移除 Cache 
cache.Remove("Cache1");
```

將常用的方法寫入類別裡，方便在應用程式中使用。

## 重點整理

1. 使用 MemoryCache 是提高系統效能的方法之一
2. 可在固定時間後回收 MemoryCache
3. 可在未使用時回收 MemoryCache
4. 整理好常用方法提供給各位
