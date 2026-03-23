---
aliases:
date: 2019-09-22
update:
author: Jeffrey,黑暗執行緒
language: C#
sourceurl: https://blog.darkthread.net/blog/ef-core-notes-5/
tags:
  - CSharp
  - 資料庫
---

# EF Core 筆記 5 - 淺談正式環境資料庫建立與換版

目前為止，筆記裡示範 EF Core Code First 都是先 dotnet ef migrations add InitialCreate 自動產生對映 Model 所需的資料表結構類別， 再透過 dotnet ef database update 帥氣地直接連上資料庫建立或更新資料表、Index... 。 有在江湖走跳的老鳥看到這些肯定搖頭，開發測試用的資料庫就算了，正式營運環境想這樣搞根本痴人說夢。 正式台資料庫的管制比 51 區還嚴 (但或許可考慮 [火影跑](https://news.ltn.com.tw/news/world/breakingnews/2922341))，豈容你想連就連，更甭提上線時每一個更動細節都需經層層審核， 放任程式連上資料庫搞黑箱作業，天地不容呀~

退一步想，就算組織規章沒訂到這麼嚴格，將系統核心資料庫的生殺大權交給 EF Core 的 Migration 機制全權處理你安心嗎？ 若是帶賽踩中罕見 Bug，一個指令弄壞幾百萬筆資料，你很快就能享受在保鏢簇擁下抱著紙箱走在辦公室，接受眾人祝福的無上尊榮。 所以我說，**在正式營運環境應用 Code First 的正確姿勢，應該是先產生 SQL Script，經過人為覆核確認後，再依組織上線流程部署上線。**

借用 [EF Core 筆記 4](https://blog.darkthread.net/blog/ef-core-notes-4/) 範例專案 (本篇以 SQL Server 示範)，我們來演練如何透過 SQL Script 建立及更新資料表。

使用 dotnet ef migrations add InitialCreate 建立 Model 快照及升級動作後，執行 dotnet ef migrations script 可得到建立資料表用的 SQL Script：

![[淺談正式環境資料庫建立與換版_01.png]]

如此便能在實際執行前人工覆核將進行的操作，必要時回頭修改 Model，確保資料表結構符合我們的期望。

假設資料表建好並運轉了一陣子，資料表已有資料，之後系統改版修改程式，在 Post Title 加了 `[MaxLength(64)]` 指定 `NVARCHAR` 長度，並增加一個 `public string Author` 屬性：

![[淺談正式環境資料庫建立與換版_02.png]]

重新編譯後，聰明的 EF Core 可以幫我們調整資料表結構。做法是先 dotnet ef migrations add ModelUpdate1，新增加一個移轉作業。 EF Core 比較修改前後變化轉換成升級動作，但警告修改動作可能造成資料遺失，建議我們人工檢視以保安全。 dotnet ef migrations list 列出我們現在共有 InitialCreate 及 ModelUpdate1 兩個移轉項目。 此時若執行 dotnet ef migrations script 將包含 InitialCreate 的 CREATE TABLE 及 ModleUpdate1 更新， 資料庫已建好資料表，要換版只需要 ModelUpdate1 的修改動作，故可執行 dotnet ef migrations script InitialCreate， 指定只列出 IntialCreate 後發生的所有更新 (在本案中僅有 ModelUpdate1)， 如下圖所示，ModelUpdate1 更新內容包含移除 Title 相關 `CONSTRAINT`、`ALTER COLUME Title nvarchar(64)`、`ADD Author nvarchar(max)`， 實際應用時可視狀況手動調整優化過再部署。

![[淺談正式環境資料庫建立與換版_03.png]]

先看過 EF Core Migration 將進行的操作再實地執行，讓人安心許多，也更符合一般上線流程要求。

但要注意，不是所有 Model 修改動作都能自動產生升級 Script，而各家 Data Provider 的支援程度也不盡相同 (像是 SQLite 支援的範圍就 [很有限](https://docs.microsoft.com/zh-tw/ef/core/providers/sqlite/limitations?WT.mc_id=DOP-MVP-37580)，SQL Server 則是最齊全的)。

我另做了一個範例，在 Update1 時新增 `Post.Author`、Update2 時移除 `Post.Author`，使用 SQLite，Update1 可以產生 Script，但 Update2 會出現錯誤：

![[淺談正式環境資料庫建立與換版_04.png]]

若改用 SQL Server，則 Update2 沒問題，端看各家 Data Provider 支援度及實作方式。

![[淺談正式環境資料庫建立與換版_05.png]]

有人可能會懷疑：EF、LINQ 不就是標榜不懂 SQL 也能寫資料庫程式，但在這篇又一再強調人工覆核 EF Core 產生的 SQL 指令，豈不走回頭路？

既然提到這點，我是這麼看的：現在的開發者比較幸福，有許多強大工具，只需要前輩一半的知識就能寫出系統來。 但有個讓菜鳥警覺老鳥釋懷的事實 - ==工具不斷進步的同時，成為專業開發者的門檻卻沒有降低太多，進階工作 (如本例中的 Script 覆核工作) 仍需要大量累積知識與經驗才能勝任。==

如果不想被貼上「只會靠工具的小屁孩」標籤，你得了解工具背後的運作機制與原理，才能提升自己成為專業開發者。 故正確的心態是「==要求自己具備不靠工具也能完成系統的能力，工具只能節省重複動作少花點力氣，遇到疑難雜症仍得靠自己==」， 有了這層體認你的競爭力才會上升到另一個等級。
