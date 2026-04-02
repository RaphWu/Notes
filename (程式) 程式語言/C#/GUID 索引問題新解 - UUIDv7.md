---
date: 2025-06-24
tags:
  - GUID/UUID
  - 分散式系統
  - 資料庫
author: Jeffrey,黑暗執行緒
sourceurl: https://blog.darkthread.net/blog/uuidv7/
---

在系統中產生唯一鍵值，GUID(UUID) 始終是我的首選，老讀者們也都知道我屬於 GUID 陣營。
(註：GUID 是微軟針對 [UUID 開放標準](https://zh.wikipedia.org/zh-tw/%E9%80%9A%E7%94%A8%E5%94%AF%E4%B8%80%E8%AF%86%E5%88%AB%E7%A0%81)的實作，幾十年來我說 GUID 說習慣了，故本文會使用 GUID 這個名稱，但 GUID/UUID 可視為相同的東西。)

用 GUID 當唯一值有很多好處，像是不仰賴資料庫就能取得唯一鍵值，在設計分散式系統或離線作業情境時，能跟資料庫脫鉤猶如甩掉手銬腳鐐，讓你飛簷走壁。而 GUID 的不可預測性，當成參數時自帶防護效果，能避免被駭客修改參數嘗試存取非授權範圍資料的風險。

但是當 GUID 唯一值轉成資料庫索引時，會伴隨一些缺點：像是不利於人工查詢或偵錯、增加儲存空間，其中較嚴重的問題，是會導致索引破碎(Index Fragement)，降低系統效能。

關於 GUID 作為唯一識別碼及資料庫索引的議題，過去己探討蠻多了，這裡幫大家複習一下：

- [好問題：GUID 真的不會重複嗎？](https://blog.darkthread.net/blog/is-guid-really-unique/)
- [GUID Primary Key 資料庫避雷守則](https://blog.darkthread.net/blog/guid-as-pk-on-db/)
- [Guid Primary Key 問題新解法 - SequentialGuidValueGenerator](https://learn.microsoft.com/zh-tw/sql/t-sql/functions/newsequentialid-transact-sql?view=sql-server-ver15&WT.mc_id=DOP-MVP-37580)
- [GUID 叢集索引測試 1 - 偶發寫入逾時之三千萬筆實測](https://blog.darkthread.net/blog/how-guid-clust-index-bad-is/)
- [GUID 叢集索引測試 2：索引碎片化分析與資料表虛胖問題](https://blog.darkthread.net/blog/guid-clust-idx-frag-n-space/)
- [GUID 叢集索引測試 3 - 觀察索引重組與索引重建對寫入動作的影響](https://blog.darkthread.net/blog/index-reorg-n-index-rebuild/)

簡單總結一下：GUID 是優秀唯一值選項，適用分散式系統情境且具天然的防猜防暴力破解效果，但其非連續性是做為資料庫索引時一大致命傷。

針對此一問題，有不少人提出解決方案，例如：

- 另設自動跳號叢集索引，避開索引碎片化問題。(此為目前我較常採用的策略)
- EF Core 提出的 [SequentialGuidValueGenerator](https://blog.darkthread.net/blog/sequentialguidvaluegenerator/)，部分位元改用 DateTime.UtcNow.Ticks 初始值加每次 +1 跳號確保順序，缺點是隨機部分剩下 58 位元。
- SQL [NEWSEQUENTIALID](https://learn.microsoft.com/zh-tw/sql/t-sql/functions/newsequentialid-transact-sql?WT.mc_id=DOP-MVP-37580)，SQL Server 提供，可確保由小到大順序，缺點是可預測及依賴資料庫。
- [Snowflake ID](https://zh.wikipedia.org/zh-tw/%E9%9B%AA%E8%8A%B1%E7%AE%97%E6%B3%95)，由 Twitter 推出之 64 位元唯一碼，由 41 位元 Timestamp、10 位元機器 ID、12 位元各機器流水號組成。因與 UUID 不相容，較少應用於資料庫、且有可預測性。
- 開源社群也嘗試提出一些解決方案，例如：1) ULID (Universally Unique Lexicographically Sortable Identifier)：以時間戳+隨機數組成，字串可排序，適合分散式系統。2) KSUID (K-Sortable Unique Identifier)：類似 ULID，但時間範圍更廣，適合大規模分散式應用。

隨著雲端與分散式資料庫的普及，開發者需要一種兼具唯一性、順序性(晚產生者排序在後)，且能避免索引效能問題的識別碼規格已是鐵一般的事實。

終於，IETF(Internet Engineering Task Force) 針對這些需求與現有實踐進行討論，於 2024 年 5 月發佈 [RFC 9562](https://www.rfc-editor.org/rfc/rfc9562.html)，納入 UUIDv6、v7、v8 三個新版本。其中 UUIDv7 以毫秒(ms)級 Unix 時間戳為主體，結合隨機位元，兼顧唯一性與有序性，特別適合資料庫索引與分散式系統。

![[UUIDv7_Fig1.png]]

目前預計今年發佈的 PostgreSQL 18 預設會內建支援 UUIDv7、DuckDB 13.0+ 開始提供 uuidv7() 函數。其他 DB 雖還未內建支援，但要透過程式語言產生 UUIDv7 不是難事，每年定期更新 .NET 自然也要與時俱進，去年發布的 .NET 9 已開始支援 [UUIDv7](https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-9/libraries?WT.mc_id=DOP-MVP-37580#systemguid)，提供 [Guid.CreateVersion7()](https://learn.microsoft.com/en-us/dotnet/api/system.guid.createversion7?view=net-9.0&WT.mc_id=DOP-MVP-37580#system-guid-createversion7) 及 [Guid.CreateVersion7(DateTimeOffset)](https://learn.microsoft.com/en-us/dotnet/api/system.guid.createversion7?view=net-9.0&WT.mc_id=DOP-MVP-37580#system-guid-createversion7(system-datetimeoffset)) 可產生 UUIDv7 規格的 GUID。

```CSharp
var list = Enumerable.Range(1, 10).Select(i =>
{
    Thread.Sleep(1);
    return new Entry
    {
        Id = i,
        GuidV4 = Guid.NewGuid(),
        UuidV7 = Guid.CreateVersion7()
    };
}).ToList();
Action<string> printTitle = (title) =>
{
    Console.WriteLine();
    Console.ForegroundColor = ConsoleColor.Cyan;
    Console.WriteLine(title);
    Console.WriteLine(new string('=', 100));
    Console.ResetColor();
};

printTitle("Ordered by GUIDv4");
foreach (var entry in list.OrderBy(e => e.GuidV4))
{
    Console.WriteLine($"Id: {entry.Id:00}, GUIDv4: {entry.GuidV4}, UUIDv7: {entry.UuidV7}");
}
printTitle("Ordered by UUIDv7");
foreach (var entry in list.OrderBy(e => e.UuidV7))
{
    Console.WriteLine($"Id: {entry.Id:00}, GUIDv4: {entry.GuidV4}, UUIDv7: {entry.UuidV7}");
}
class Entry
{
    public int Id { get; set; }
    public Guid GuidV4 { get; set; }
    public Guid UuidV7 { get; set; }

}
```

上面的程式會產生 10 個 GUID 及 UUIDv7 形成物件清單，產生過程刻意間隔 1ms，以確保每次產生的 UUIDv7 `unix_time_ts` 一定比前一筆大(該時間戳之精確度到 ms)。實測以 GUID (UUIDv4) 排序時與 Id 順序不同，而 UUIDv7 版本 GUID 的前 11 位 16 進位數字具連續性 0197a24638c ~ 0197a246395 (接近 JavaScript new Date().getTime().toString(16) 得到的數字) ，排序後其 Id 也如其建立順序由 01 ~ 10。

![[UUIDv7_Fig2.png]]

原本 UUIDv4 有 122 位元隨機數字，UUIDv7 用了 48 位元放時間戳，隨機位元剩下 74 個，會不會造成隨機性不足、碰撞率上升呢？

簡單推算，UUIDv7 每個毫秒內，隨機性降為 74 位元（約 1.8 × 1022 種可能)，只要單一毫秒內產生的 UUID 數量遠小於 237 個 （約 1370 億），碰撞機率仍然極低。以生日悖論近似公式，若每毫秒產生 1 百萬個 UUID，碰撞機率約為 1.05 × 10-10，機率低到幾可忽略。除非是極端高併發情境（每毫秒產生數十億個 UUID），才需進一步評估是否需要額外設計。[參考](https://www.perplexity.ai/search/yuan-ben-uuidv4-you-122-wei-yu-254Y.5iBTHewPL5hBIC1uA)

類似的構想過去也有人提過，原理很單純也不難實踐，但差在一直缺乏公認的官方標準，未來若需要「依產生時間順序排序的 GUID」，多了一個強而有力的選項。
