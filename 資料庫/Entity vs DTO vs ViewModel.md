---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# 三層架構劃分

- **拆分原實體屬性與邏輯：**
    - 將原始的 Employee 實體（EF Entity）中只與資料庫結構相關的欄位保留在 Entity 層，並移除與顯示或組合邏輯相關的欄位。
    - 如原本在 Entity 中的 `[NotMapped] FullName`、`DepartmentName`、`StatusChangeAtString` 等，只是顯示用或格式化的屬性，應該移到專門負責 UI 或應用層展示的 ViewModel 層（或在傳輸需要時放在 DTO 層）中。
    - Entity 層只保留對應資料表的欄位與必要的關聯屬性，確保實體類別只承擔「映射資料庫欄位」的職責。
- **Entity、DTO、ViewModel 的角色界定：** 具體來說，
    - Entity 層只做資料對映，不含任何 UI 或服務邏輯；
    - DTO（Data Transfer Object）層則用於封裝跨層或跨服務傳輸的純資料（通常是序列化用契約），不含 UI 相關內容；
    - ViewModel 層專注於 UI 展示需求，包含所有視圖所需資料（可能來自一個或多個 DTO/實體）及輔助 UI 的屬性或行為。
    - 整體的依賴方向是：**ViewModel 依賴 DTO（或 Domain），DTO 不依賴 ViewModel，Entity 與 UI 無關**，保持單向流動。例如，應用層可從 Entity 讀取資料填充 DTO，再由 UI 層轉換 DTO 生成 ViewModel。

## 1. Entity 層職責與設計要點

- **純粹資料對映：**
    - Entity 類別應以 ==POCO (Plain CLR Object)== 形式呈現，對應資料表的結構（欄位與關聯）。不應包含任何與 UI 顯示或服務層邏輯有關的屬性/方法。
    - 以 Employee 為例，應只含 `Id`、`FirstName`、`LastName`、`DepartmentId` 等資料欄位，不應包含 `FullName`（顯示用）或 `StatusChangeAtString`（格式化字串）等屬性。
    - 實體若需額外欄位（例如計算屬性），應考慮移到適當層級實現。
- **單一職責：**
    - Entity 層專注於「持久化資料」責任，符合單一職責原則。不要將驗證規則、UI 呈現設定、或業務流程硬編碼在實體類別中，以避免層級混淆。
    - 實體可以包含基本的領域邏輯（Domain Logic），但若這邏輯與服務流程或顯示邏輯無關，否則亦應拆分到業務服務層或其他領域模型。
    - 簡單來說，實體僅與資料儲存相關，不涉及表示層。
- **不可做 UI 聯結：**
    - 不可將實體當作 ViewModel 使用。
    - 正如有經驗開發者指出的：「將 EF 實體（Entity）當作表單/ViewModel 使用是錯誤的做法，不應將實體類別當作 DTO 傳遞，否則會暴露許多不必要或敏感欄位」[stackoverflow.com](https://stackoverflow.com/questions/79171906/updating-an-entity-record-not-found#:~:text=)。
    - Entity 層不應引用任何 UI 專屬類別或包含與顯示相關的標籤註釋。

## 2. DTO 層用途與設計原則

- **資料交換契約：**
    - DTO 全名「Data Transfer Object 資料傳輸物件」，負責在不同層或子系統之間==攜帶資料== [c-sharpcorner.com](https://www.c-sharpcorner.com/article/data-transfer-objects-dtos-in-c-sharp/#:~:text=A%20Data%20Transfer%20Object%20,exposing%20the%20underlying%20implementation%20details)。
    - 它只包含應用程式不同部分間需要傳遞的欄位，不含任何行為或 UI 資訊。
    - 使用場景例如：服務層（Web API、WCF 或跨進程）返回資料給客戶端，或服務層之間交換資料時，都可使用 DTO 作為契約。
    - DTO 層在架構中常與資料層隔離，用來「**扁平化**」複雜的領域物件圖形或隱藏不必要欄位 [learn.microsoft.com](https://learn.microsoft.com/zh-tw/aspnet/web-api/overview/data/using-web-api-with-entity-framework/part-5#:~:text=%E7%8F%BE%E5%9C%A8%EF%BC%8C%E6%88%91%E5%80%91%E7%9A%84%20Web%20API%20%E6%9C%83%E5%B0%87%E8%B3%87%E6%96%99%E5%BA%AB%E5%AF%A6%E9%AB%94%E5%85%AC%E9%96%8B%E7%B5%A6%E7%94%A8%E6%88%B6%E7%AB%AF%E3%80%82%20%E7%94%A8%E6%88%B6%E7%AB%AF%E6%9C%83%E6%8E%A5%E6%94%B6%E7%9B%B4%E6%8E%A5%E5%B0%8D%E6%87%89%E8%87%B3%E8%B3%87%E6%96%99%E5%BA%AB%E8%B3%87%E6%96%99%E8%A1%A8%E7%9A%84%E8%B3%87%E6%96%99%E3%80%82,%E7%84%B6%E8%80%8C%EF%BC%8C%E9%80%99%E4%B8%A6%E4%B8%8D%E7%B8%BD%E6%98%AF%E5%80%8B%E5%A5%BD%E4%B8%BB%E6%84%8F%E3%80%82%20%E6%9C%89%E6%99%82%E5%80%99%E6%82%A8%E6%83%B3%E8%A6%81%E8%AE%8A%E6%9B%B4%E5%82%B3%E9%80%81%E8%87%B3%E5%AE%A2%E6%88%B6%E7%AB%AF%E7%9A%84%E8%B3%87%E6%96%99%E5%9C%96%E5%BD%A2%E3%80%82%20%E4%BE%8B%E5%A6%82%EF%BC%8C%E6%82%A8%E5%8F%AF%E8%83%BD%E8%A6%81%EF%BC%9A)[c-sharpcorner.com](https://www.c-sharpcorner.com/article/data-transfer-objects-dtos-in-c-sharp/#:~:text=,by%20preventing%20sensitive%20information%20from)。
- **減少傳輸負擔：**
    - DTO 允許只傳輸「所需資料」，減少網路或層間傳遞的負荷。例如只送出客戶端要用到的欄位而跳過其他大型欄位，以優化效能 [c-sharpcorner.com](https://www.c-sharpcorner.com/article/data-transfer-objects-dtos-in-c-sharp/#:~:text=,associated%20with%20unnecessary%20data%20transfer)。
    - 此設計也能提高安全性：只暴露必要資料可避免敏感欄位洩漏 [c-sharpcorner.com](https://www.c-sharpcorner.com/article/data-transfer-objects-dtos-in-c-sharp/#:~:text=one%20format%20to%20another%2C%20which,by%20preventing%20sensitive%20information%20from)。
- **扁平化與聚合：**
    - DTO 可將多個實體或複雜結構==扁平化成更簡單的資料結構==，方便序列化與客戶端使用。它可以選擇性的整合或省略原本實體的欄位（例如範例中 `BookDto` 只包含作者名稱而不包含整個作者物件 [learn.microsoft.com](https://learn.microsoft.com/zh-tw/aspnet/web-api/overview/data/using-web-api-with-entity-framework/part-5#:~:text=%2F%2F%20GET%20api%2FBooks%20public%20IQueryable,Name)）。
    - DTO 也可依需求分級（如 `EmployeeSummaryDto` vs `EmployeeDetailDto`）來滿足不同 API 呼叫的要求。
    - 建議每個實體對應一個或多個 DTO 類，並使用明確後綴（如 `EmployeeDto`）[stackoverflow.com](https://stackoverflow.com/questions/18834565/dto-naming-conventions-modeling-and-inheritance#:~:text=Recommendation%20is%20that%20you%20should,as%20per%20choice%20and%20requirements)。
- **獨立於實體：**
    - DTO 應與實體解耦，不要直接繼承自實體類別，也不應該包含實體對映的屬性以外的任何行為。
    - 它完全不依賴於 UI 或資料庫框架，只是個純資料容器 [c-sharpcorner.com](https://www.c-sharpcorner.com/article/data-transfer-objects-dtos-in-c-sharp/#:~:text=A%20Data%20Transfer%20Object%20,exposing%20the%20underlying%20implementation%20details)[c-sharpcorner.com](https://www.c-sharpcorner.com/article/data-transfer-objects-dtos-in-c-sharp/#:~:text=,associated%20with%20unnecessary%20data%20transfer)。
    - 在服務層中使用 DTO 可以將服務層與資料層隔離，例如在 Web API 中，只回傳 DTO 而非 EF 實體，藉此避免過度發佈漏洞（Over-Posting）或循環參考問題 [learn.microsoft.com](https://learn.microsoft.com/zh-tw/aspnet/web-api/overview/data/using-web-api-with-entity-framework/part-5#:~:text=%E7%8F%BE%E5%9C%A8%EF%BC%8C%E6%88%91%E5%80%91%E7%9A%84%20Web%20API%20%E6%9C%83%E5%B0%87%E8%B3%87%E6%96%99%E5%BA%AB%E5%AF%A6%E9%AB%94%E5%85%AC%E9%96%8B%E7%B5%A6%E7%94%A8%E6%88%B6%E7%AB%AF%E3%80%82%20%E7%94%A8%E6%88%B6%E7%AB%AF%E6%9C%83%E6%8E%A5%E6%94%B6%E7%9B%B4%E6%8E%A5%E5%B0%8D%E6%87%89%E8%87%B3%E8%B3%87%E6%96%99%E5%BA%AB%E8%B3%87%E6%96%99%E8%A1%A8%E7%9A%84%E8%B3%87%E6%96%99%E3%80%82,%E7%84%B6%E8%80%8C%EF%BC%8C%E9%80%99%E4%B8%A6%E4%B8%8D%E7%B8%BD%E6%98%AF%E5%80%8B%E5%A5%BD%E4%B8%BB%E6%84%8F%E3%80%82%20%E6%9C%89%E6%99%82%E5%80%99%E6%82%A8%E6%83%B3%E8%A6%81%E8%AE%8A%E6%9B%B4%E5%82%B3%E9%80%81%E8%87%B3%E5%AE%A2%E6%88%B6%E7%AB%AF%E7%9A%84%E8%B3%87%E6%96%99%E5%9C%96%E5%BD%A2%E3%80%82%20%E4%BE%8B%E5%A6%82%EF%BC%8C%E6%82%A8%E5%8F%AF%E8%83%BD%E8%A6%81%EF%BC%9A)。

## 3. ViewModel 層用途與差異

- **UI 專用模型：**
    - ViewModel （或稱畫面模型）專注於==呈現層==，對應單一視圖（View）的資料需求。
    - 它通常只含有畫面需要顯示和交互的欄位，例如 `FullName`、`IsActive`（切換開關）、或用於下拉選單的清單。
    - ViewModel 可以彙整自一個或多個 DTO/實體的資料，並加入特定格式化或輔助屬性，以簡化資料綁定。
    - 正如某開發者建議：「==ViewModel 是 UI 關注的內容，EF 實體是領域關注的內容==；ViewModel 只包含視圖所需的欄位 [stackoverflow.com](https://stackoverflow.com/questions/35513414/asp-net-mvc-entity-framework-how-to-deal-with-dtos-and-viewmodels#:~:text=Always%20create%20a%20ViewModel,the%20View%20that%20uses%20them)。」。
- **包含行為或狀態：**
    - 與 DTO 不同，ViewModel ==可以包含簡單的行為==（例如計算屬性、按鈕命令、狀態標誌等），協助 UI 互動 [stackoverflow.com](https://stackoverflow.com/questions/1982042/dto-viewmodel#:~:text=The%20canonical%20definition%20of%20a,an%20object%20without%20any%20behavior)。
    - 在 MVVM 架構下，ViewModel 甚至可能實作事件或通知，提供雙向綁定和動態更新。但在 MVC 模式中，ViewModel 通常只作為單向資料結構，仍可以包含一些 UI 相關的輔助資料。
- **與 DTO 的差異：**
    - 總結來說，DTO 注重「資料傳輸」，與 UI 無關；而 ViewModel 注重「界面呈現」，與視圖直接互動。
    - ViewModel 不應直接用於跨層傳輸（通常由控制器或服務層填充或回寫），它在概念上不與資料庫或服務層綁定，僅對應展示層 [stackoverflow.com](https://stackoverflow.com/questions/1982042/dto-viewmodel#:~:text=The%20canonical%20definition%20of%20a,an%20object%20without%20any%20behavior)[stackoverflow.com](https://stackoverflow.com/questions/35513414/asp-net-mvc-entity-framework-how-to-deal-with-dtos-and-viewmodels#:~:text=Always%20create%20a%20ViewModel,the%20View%20that%20uses%20them)。例如，EmployeeViewModel 可能包含一個 `FullName` 來顯示 `FirstName + LastName`，也可能包含如 `StatusChangeAtString` 這種以資料格式化為字串的欄位，這些都是純粹為 UI 提供的，不屬於 DTO 或 Entity。

## 4. 層間轉換實作

- **手動對映：**
    - 最直接的方式是手寫程式碼或 LINQ Select 進行轉換。可以在服務或控制器層，用類似 `select new EmployeeDto { ... }` 的方式從 Entity 投影到 DTO；或用靜態 Mapper 類別/extension 方法將 Entity 轉成 DTO，再由 Controller 將 DTO 轉成 ViewModel[learn.microsoft.com](https://learn.microsoft.com/zh-tw/aspnet/web-api/overview/data/using-web-api-with-entity-framework/part-5#:~:text=%E6%8E%A5%E4%B8%8B%E4%BE%86%EF%BC%8C%E5%B0%87%20,Select%20%E9%99%B3%E8%BF%B0%E5%BC%8F%EF%BC%8C%E5%BE%9E%20Book%20%E5%AF%A6%E9%AB%94%E8%BD%89%E6%8F%9B%E6%88%90%20DTO%E3%80%82)。
    - 例如 Web API 範例示範對 `Book` 實體使用 LINQ `Select` 產生對應的 DTO[learn.microsoft.com](https://learn.microsoft.com/zh-tw/aspnet/web-api/overview/data/using-web-api-with-entity-framework/part-5#:~:text=%E6%8E%A5%E4%B8%8B%E4%BE%86%EF%BC%8C%E5%B0%87%20,Select%20%E9%99%B3%E8%BF%B0%E5%BC%8F%EF%BC%8C%E5%BE%9E%20Book%20%E5%AF%A6%E9%AB%94%E8%BD%89%E6%8F%9B%E6%88%90%20DTO%E3%80%82)，以及在新增後手動將 `Book` 實體轉成 `BookDto` 回傳 [learn.microsoft.com](https://learn.microsoft.com/zh-tw/aspnet/web-api/overview/data/using-web-api-with-entity-framework/part-5#:~:text=%2F%2F%20New%20code%3A%20%2F%2F%20Load,x.Author%29.Load)[learn.microsoft.com](https://learn.microsoft.com/zh-tw/aspnet/web-api/overview/data/using-web-api-with-entity-framework/part-5#:~:text=%E6%B3%A8%E6%84%8F)。
- **AutoMapper 等自動對映工具：**
    - 使用 AutoMapper 或類似函式庫可自動映射屬性相同的類別，減少大量重複程式碼。只需配置來源與目標類別之間的映射規則，之後呼叫 `Mapper.Map<Destination>(source)` 即可批次轉換 [learn.microsoft.com](https://learn.microsoft.com/zh-tw/aspnet/web-api/overview/data/using-web-api-with-entity-framework/part-5#:~:text=%E6%B3%A8%E6%84%8F)。
    - 此外，可在 Entity/DTO/ViewModel 類別上撰寫擴充方法（extension methods），或使用工具（如 Mapster）均可達成自動對映，簡化轉換實作。
- **層級映射工具：**
    - 也可考慮在資料存取層的 Repository 或在應用層引入 Mapper 類別，統一管理 Entity↔DTO↔ViewModel 之間的邏輯。
    - 例如 Matt 的回答指出，適當的基底類別（如 DTOBase）和映射引擎能加速各層間轉換 [stackoverflow.com](https://stackoverflow.com/questions/35513414/asp-net-mvc-entity-framework-how-to-deal-with-dtos-and-viewmodels#:~:text=I%20know%20it%20is%20a,models%20and%20your%20dto%20objects)[stackoverflow.com](https://stackoverflow.com/questions/35513414/asp-net-mvc-entity-framework-how-to-deal-with-dtos-and-viewmodels#:~:text=To%20prevent%20all%20this%20it,it%20etc%20without%20any%20worry)。需要注意的是，轉換邏輯本身不應放在 Entity 類別中，而是放在服務層或 Mapper 類別內，以避免污染實體。

## 5. 常見錯誤與反模式

- **混用 Entity 和 UI 屬性：**
    - 直接在 Entity 類別中加入顯示格式或 NotMapped 屬性是常見反模式。
    - 例如將姓名組合、日期格式化字串等寫在 Entity，而非拆到 ViewModel，會導致實體層依賴 UI、混淆層級。
    - 正如前述，切勿將實體當做表單模型或 ViewModel 使用 [stackoverflow.com](https://stackoverflow.com/questions/79171906/updating-an-entity-record-not-found#:~:text=)。
- **不進行分層映射：**
    - 一些小型專案可能直接將 EF 實體回傳給前端，但在大型系統常會出問題，如快取與延遲加載衝突、暴露過多欄位、測試困難等 [stackoverflow.com](https://stackoverflow.com/questions/35513414/asp-net-mvc-entity-framework-how-to-deal-with-dtos-and-viewmodels#:~:text=You%20cannot%20cache%20the%20EF,are%20hard%20to%20trace%20down)[stackoverflow.com](https://stackoverflow.com/questions/35513414/asp-net-mvc-entity-framework-how-to-deal-with-dtos-and-viewmodels#:~:text=To%20prevent%20all%20this%20it,it%20etc%20without%20any%20worry)。
    - 不對應 DTO 也讓服務層綁死在資料庫實體上，違反「服務層與資料層分離」的設計。相反地，推薦在服務邊界使用 DTO，以避免上述問題 [stackoverflow.com](https://stackoverflow.com/questions/35513414/asp-net-mvc-entity-framework-how-to-deal-with-dtos-and-viewmodels#:~:text=You%20cannot%20cache%20the%20EF,are%20hard%20to%20trace%20down)[stackoverflow.com](https://stackoverflow.com/questions/35513414/asp-net-mvc-entity-framework-how-to-deal-with-dtos-and-viewmodels#:~:text=To%20prevent%20all%20this%20it,it%20etc%20without%20any%20worry)。
- **ViewModel 過度多且冗餘：**
    - 雖然要建立一對一符合各視圖需求的 ViewModel，但如果每個小結果都做一個新類別，可能造成過多類別。在這種情況下可考慮更抽象的資料結構（如 Key-Value、泛型模型）或重構視圖。
    - 但總原則是「ViewModel 只包含視圖需要的部分」，避免單一 ViewModel 泡大成包含所有場景的「萬用模型」。
- **命名不一致：**
    - 常見錯誤是給予不清楚的類別名稱，或不同層的同名類別混淆。
    - 例如在多層中有同時名為 `Employee` 的實體、DTO、ViewModel，就容易引起混淆與誤用。應統一命名後綴以區隔不同角色，如 `EmployeeEntity`、`EmployeeDto`、`EmployeeViewModel`。同時避免在 Entity 類別中與 DB 未對應的欄位命名（以免 EF 自動推斷欄位而出錯）。

## 6. 命名規範與資料夾結構

- **命名後綴：**
    - 慣例上，實體類別可直接用代表資料表的名稱（或加上 `Entity` 後綴），DTO 類別加上 `Dto`（或 `DTO`）後綴，如 `EmployeeDto`，以強調其角色 [stackoverflow.com](https://stackoverflow.com/questions/18834565/dto-naming-conventions-modeling-and-inheritance#:~:text=Recommendation%20is%20that%20you%20should,as%20per%20choice%20and%20requirements)；ViewModel 類別則加上 `ViewModel` 後綴，如 `EmployeeViewModel`。
    - 服務接口、回傳型別等也應明確命名，讓各層職責易於辨識。
- **資料夾/命名空間：**
    - 建議按層級分開資料夾：例如在資料庫或 Domain 專案下放 `Entities` 或 `Models` 資料夾存放實體；在應用/服務層放 `DTOs` 資料夾；在 UI 專案（MVC、WPF、Blazor 等）放 `ViewModels` 資料夾；視需要可再細分子目錄。例如：`Project.Data.Entities`、`Project.Service.DTOs`、`Project.Web.ViewModels`。如此結構清晰，能避免不同層類別混用。
- **附註註解：**
    - 若使用像 Entity Framework 的 Code First，可在實體類別上使用資料註解（如 `[Table]`、`[Column]`）或在 DTO/VM 上使用序列化屬性（如 `[JsonIgnore]`）以控制序列化行為和過度發佈。命名一致性也利於使用 AutoMapper 等工具自動對映。

## 7. 範例：EmployeeEntity、EmployeeDto、EmployeeViewModel

以下是簡化的對應範例（僅示意欄位）：

```csharp
// Entity 層：僅映射資料表字段，無顯示邏輯
public class EmployeeEntity {
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public int DepartmentId { get; set; }
    public DateTime StatusChangedAt { get; set; }
    // (不包含 FullName、DepartmentName、StatusChangeAtString 等 UI 屬性)
}

// DTO 層：純資料傳輸結構，不含 UI 特殊需求
public class EmployeeDto {
    public int Id { get; set; }
    public string FullName { get; set; }          // 將 FirstName + LastName 合併
    public string DepartmentName { get; set; }    // 對應部門名稱
    public DateTime StatusChangedAt { get; set; } // 僅時間型別
}

// ViewModel 層：專用於 UI 的模型，可含格式化字串或狀態欄位
public class EmployeeViewModel {
    public int Id { get; set; }
    public string FullName { get; set; }             // 同DTO
    public string DepartmentName { get; set; }       // 同DTO
    public string StatusChangeAtString { get; set; } // 已格式化的日期字串，UI使用
    public bool IsStatusRecent { get; set; }         // UI特定欄位（範例）
    // 可加入其它用於View互動的屬性或指令
}
```

轉換關係示意：程式中可由 `EmployeeEntity` 資料庫實體撈取後，使用手動對映或 AutoMapper 建立 `EmployeeDto`，再由 Controller/服務將 DTO 的資料塞給 `EmployeeViewModel` [learn.microsoft.com](https://learn.microsoft.com/zh-tw/aspnet/web-api/overview/data/using-web-api-with-entity-framework/part-5#:~:text=%E6%8E%A5%E4%B8%8B%E4%BE%86%EF%BC%8C%E5%B0%87%20,Select%20%E9%99%B3%E8%BF%B0%E5%BC%8F%EF%BC%8C%E5%BE%9E%20Book%20%E5%AF%A6%E9%AB%94%E8%BD%89%E6%8F%9B%E6%88%90%20DTO%E3%80%82)[learn.microsoft.com](https://learn.microsoft.com/zh-tw/aspnet/web-api/overview/data/using-web-api-with-entity-framework/part-5#:~:text=%E6%B3%A8%E6%84%8F)。如此各層分工明確，ViewModel 只關心顯示、DTO 只關心傳輸格式、Entity 只關心資料持久化 [stackoverflow.com](https://stackoverflow.com/questions/1982042/dto-viewmodel#:~:text=The%20canonical%20definition%20of%20a,an%20object%20without%20any%20behavior)[stackoverflow.com](https://stackoverflow.com/questions/35513414/asp-net-mvc-entity-framework-how-to-deal-with-dtos-and-viewmodels#:~:text=Always%20create%20a%20ViewModel,the%20View%20that%20uses%20them)。

**參考資料：** 分層架構與 DTO/VM 模式已被廣泛討論。
- DTO 模式強調「只運送所需欄位，避免洩漏內部結構」[learn.microsoft.com](https://learn.microsoft.com/zh-tw/aspnet/web-api/overview/data/using-web-api-with-entity-framework/part-5#:~:text=%E7%8F%BE%E5%9C%A8%EF%BC%8C%E6%88%91%E5%80%91%E7%9A%84%20Web%20API%20%E6%9C%83%E5%B0%87%E8%B3%87%E6%96%99%E5%BA%AB%E5%AF%A6%E9%AB%94%E5%85%AC%E9%96%8B%E7%B5%A6%E7%94%A8%E6%88%B6%E7%AB%AF%E3%80%82%20%E7%94%A8%E6%88%B6%E7%AB%AF%E6%9C%83%E6%8E%A5%E6%94%B6%E7%9B%B4%E6%8E%A5%E5%B0%8D%E6%87%89%E8%87%B3%E8%B3%87%E6%96%99%E5%BA%AB%E8%B3%87%E6%96%99%E8%A1%A8%E7%9A%84%E8%B3%87%E6%96%99%E3%80%82,%E7%84%B6%E8%80%8C%EF%BC%8C%E9%80%99%E4%B8%A6%E4%B8%8D%E7%B8%BD%E6%98%AF%E5%80%8B%E5%A5%BD%E4%B8%BB%E6%84%8F%E3%80%82%20%E6%9C%89%E6%99%82%E5%80%99%E6%82%A8%E6%83%B3%E8%A6%81%E8%AE%8A%E6%9B%B4%E5%82%B3%E9%80%81%E8%87%B3%E5%AE%A2%E6%88%B6%E7%AB%AF%E7%9A%84%E8%B3%87%E6%96%99%E5%9C%96%E5%BD%A2%E3%80%82%20%E4%BE%8B%E5%A6%82%EF%BC%8C%E6%82%A8%E5%8F%AF%E8%83%BD%E8%A6%81%EF%BC%9A)[c-sharpcorner.com](https://www.c-sharpcorner.com/article/data-transfer-objects-dtos-in-c-sharp/#:~:text=,by%20preventing%20sensitive%20information%20from)；
- ViewModel 則專注在畫面所需的資料和行為 [stackoverflow.com](https://stackoverflow.com/questions/1982042/dto-viewmodel#:~:text=The%20canonical%20definition%20of%20a,an%20object%20without%20any%20behavior)[stackoverflow.com](https://stackoverflow.com/questions/35513414/asp-net-mvc-entity-framework-how-to-deal-with-dtos-and-viewmodels#:~:text=Always%20create%20a%20ViewModel,the%20View%20that%20uses%20them)。
- 實務經驗指出，大型系統應避免直接將實體傳遞至 UI 或快取，而應使用 DTO 及映射工具如 AutoMapper[stackoverflow.com](https://stackoverflow.com/questions/35513414/asp-net-mvc-entity-framework-how-to-deal-with-dtos-and-viewmodels#:~:text=I%20know%20it%20is%20a,models%20and%20your%20dto%20objects)[stackoverflow.com](https://stackoverflow.com/questions/35513414/asp-net-mvc-entity-framework-how-to-deal-with-dtos-and-viewmodels#:~:text=To%20prevent%20all%20this%20it,it%20etc%20without%20any%20worry) 來維持清晰的層級界線並確保可測試性與擴充性。
