---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# 學習階段

這是一個很棒的問題，因為你已經有 C# 和桌面應用的基礎，轉向 **ASP.NET**（特別是現代的 ASP.NET Core）並不困難，但要「改變思維模式」。桌面程式主要是**事件驅動（event-driven）**、**狀態保留（stateful）**的，而網頁開發是**請求驅動（request-driven）**riven）**、**無狀態（stateless）**的。這是最大的心智轉折。

我幫你劃分成幾個階段，這樣比較有路線感：

## 一、基礎打底（網頁運作觀念）

在動手寫 ASP.NET 之前，必須了解「網頁到底怎麼運作」：

- **HTTP 基礎**：請求（Request）、回應（Response）、狀態碼（200、404、500）。
- **HTML / CSS / JavaScript**：不需精通，但要能理解如何輸出頁面與互動。
- **瀏覽器與伺服器的關係**：每次請求都是全新的，伺服器不會記得前一次狀態。

👉 小重點：網頁不是「即時監聽事件」那種邏輯，而是「伺服器給你資料 → 瀏覽器顯示 → 用戶再發新請求」。

## 二、ASP.NET Core 架構觀念

ASP.NET Core 是模組化的 Web 平台，幾個主要技術面向如下：

1. **瀏覽器與伺服器的關係**
    這是 ASP.NET Core 的管線機制，所有請求都會經過一連串中介處理（像過濾器、認證、記錄等）。
2. **Middleware（中介軟體）**
    將 URL 對應到控制器或 Razor 頁面。要理解 MVC 路由與 Minimal API 路由的差異。
3. **Routing（路由系統）**
    - MVC 模式：分開控制器（C）、模型（M）、視圖（V）
    - Razor Pages：更輕量、類似 WebForms 的後台程式碼概念
    - Blazor：用 C# 寫前端互動（類似 WPF + Web 結合）
4. **Controller / Razor Page / Blazor**
    這點你會喜歡，因為你用過 Autofac。ASP.NET Core 內建 DI 容器，可以整合你熟悉的 Autofac。

## 三、後端常見基礎

要能讓網站「存資料」、「記狀態」、「驗證使用者」，需要掌握：

- **DI（相依性注入）**：ORM 工具，操作資料庫。
- **Entity Framework Core (EF Core)**：使用者登入、權限、狀態保存。
- **Session / Cookie / Authentication**：管理設定與日誌。

## 四、前後端整合思維

ASP.NET 會常常與前端框架互動：

- **Configuration / Logging / Options Pattern**：定義後端端點讓前端（或別的應用）使用。
- **API（Web API / RESTful）**：資料如何在前後端之間傳遞。
- **JSON、AJAX、Fetch API**（非必須但建議了解）：React / Vue / Angular / Blazor。

## 五、部署與運行

最後要懂得讓網站真正「跑起來」：

- **前端框架**：ASP.NET Core 自帶 Kestrel，也可掛在 IIS。
- **IIS / Kestrel / Nginx**
- **Appsettings / 環境設定（Development / Production）**（選擇性進階）

## 🧭 建議學習路線圖（循序漸進）

1. 了解 HTTP、HTML、CSS、JavaScript 基本概念。
2. 學 ASP.NET Core MVC（學 Routing、Controller、View）。
3. 加入資料存取（EF Core）。
4. 實作登入系統（Authentication）。
5. 學習 Web API，讓你的網站能被其他應用呼叫。
6. 了解 Razor Pages 或 Blazor，看你偏好哪種開發模式。
7. 嘗試部署到雲端或本地 IIS。

---

## ASP.NET Core 入門前應具備的基礎知識與學習順序

ASP.NET Core 是一個跨平台、模組化、可擴充的 Web 框架。
對於熟悉 C# 的桌面開發者來說，學習它的關鍵在於「理解 Web 概念」與「轉換開發思維」。
桌面應用著重事件驅動，而 Web 應用以請求與回應為主軸。

### 一、Web 基礎概念

1. **HTTP 協定**
    
    - 理解 Request（請求）與 Response（回應）的循環。
    - 熟悉常見 HTTP 方法：GET、POST、PUT、DELETE。
    - 知道 HTTP 狀態碼（200、404、500）的意義。
        
2. **URL 與路由**
    
    - 了解 URL 結構與 Query String。
    - 理解如何將 URL 對應到後端控制器。
        
3. **HTML / CSS / JavaScript**
    
    - HTML：用於結構化網頁內容。
    - CSS：用於控制樣式與版面。
    - JavaScript：用於前端互動與 AJAX 呼叫。
        
4. **前後端互動機制**
    
    - 瀏覽器如何送出表單或 AJAX 請求。
    - 伺服器如何回傳 HTML 或 JSON。

### 二、ASP.NET Core 的核心概念

1. **Middleware（中介軟體）**
    
    - 每個 HTTP 請求會經過一連串中介軟體處理。
    - 可插入驗證、日誌、錯誤處理等邏輯。
        
2. **Routing（路由）**
    
    - 定義 URL 如何對應到控制器或 Razor Page。
        
3. **Controller 與 Action**
    
    - Controller：處理請求。
    - Action：具體的處理方法（例如：GetUser、CreateOrder）。
        
4. **依賴注入（Dependency Injection, DI）**
    
    - ASP.NET Core 內建 DI 容器。
    - 讓類別之間不直接依賴彼此，方便測試與維護。
        
5. **Configuration 與 Logging**
    
    - 用設定檔（appsettings.json）管理環境變數與連線資訊。
    - 使用 Logging 紀錄執行狀況與錯誤。

### 三、C# 層面的延伸重點

1. **非同步程式設計（async / await）**
    
    - ASP.NET Core 預設以非同步方式處理請求。
    - 理解非同步方法可避免伺服器阻塞。
        
2. **物件導向設計（OOP）與介面設計**
    
    - 控制反轉（IoC）與依賴注入息息相關。
    - Interface 可讓服務抽象化。
        
3. **LINQ 與 Entity Framework Core**
    
    - 使用 LINQ 查詢資料。
    - 透過 EF Core 以 C# 操作資料庫。

### 四、資料交換與資料庫基礎

1. **JSON 格式**
    
    - 理解伺服器與前端資料交換的標準格式。
        
2. **RESTful API 概念**
    
    - 以資源導向設計 API（例如 /api/products）。
    - 使用 HTTP 方法表示操作意圖。
        
3. **資料庫基礎**
    
    - 熟悉 SQL、資料表設計與關聯。
    - 了解資料庫連線與遷移（Migration）。

### 五、ASP.NET Core 專案的運作架構

1. **Program.cs / Startup.cs（或 builder 模式）**
    
    - 定義應用程式啟動流程與服務註冊。
        
2. **Controllers / Models / Views 結構**
    
    - Controller：邏輯層。
    - Model：資料與商業邏輯。
    - View：呈現層（HTML + Razor）。
        
3. **範例架構邏輯**

```csharp
// Program.cs
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
var app = builder.Build();
app.MapControllers();
app.Run();
```

```csharp
// WeatherController.cs
[ApiController]
[Route("api/[controller]")]
public class WeatherController : ControllerBase
{
    [HttpGet]
    public IEnumerable<string> Get() => new[] { "Sunny", "Cloudy", "Rainy" };
}
```

請求流程示意：

1. 瀏覽器送出 `GET /api/weather`
2. ASP.NET Core 路由找到 `WeatherController.Get()`
3. 回傳 JSON 結果給瀏覽器

### 六、建議學習順序

1. 熟悉 **HTTP 與 Web 概念**
2. 學習 **HTML / CSS / JavaScript 基礎**
3. 練習 **建立 ASP.NET Core 專案與路由配置**
4. 了解 **Controller 與 API 輸出邏輯**
5. 進一步學習 **Entity Framework Core 與資料庫操作**
6. 練習 **非同步處理與中介軟體設計**

### 七、延伸方向

1. **Razor Pages**：適合從桌面轉入 Web 的初學者，開發模式較接近傳統 WinForms。
2. **Blazor**：以 C# 開發互動式前端（類似桌面開發思維）。
3. **Web API + 前端框架**：進階階段可整合 Vue、React、Angular。

ASP.NET Core 的精髓在於「組合」：組合中介軟體、控制器、資料存取與設定，使伺服器能穩定且彈性地回應每一個請求。
學會它後，將能把 C# 能力延伸到 Web 世界，打造真正跨平台的應用程式。
