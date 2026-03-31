---
aliases:
date: 2022-04-21
update:
author: Will 保哥
language: C#
sourceurl: https://blog.miniasp.com/post/2022/04/21/Loading-Related-Data-in-EF-Core
tags:
  - CSharp
  - 資料庫
  - EF_Core
---

Entity Framework Core 讓你可以透過「導覽屬性」快速的取得「關聯」資料，不過方便的背後可能會犧牲一些效能，早期在 Entity Framework 的年代，預設啟用「延遲載入」機制，這個預設值可能會導致許多意外的效能問題，以致於 "Entity Framework 很慢 " 的臭名一直延續至今，即便 Entity Framework Core 已經沒有這個問題，許多初學者還是不太清楚如何正確的使用 Entity Framework Core 來存取關聯資料。這篇文章我就來說說 Entity Framework Core 提供的三種關聯資料載入策略，以及如何判斷何時該用哪種策略載入資料。

# 不載入關聯資料

在 Entity Framework 6 之前，預設可以透過「導覽屬性」自動載入關聯資料，而且是在「第一次」存取導覽屬性時「延遲載入」資料。不過從 Entity Framework Core 開始，已經不會自動載入關聯資料，任何需要透過「導覽屬性」載入關聯資料的情境，都需要明確指定你要如何載入。

我為這篇文章準備了一個 ASP.NET Core 6 的範例專案，所有程式碼都會從 `master` 分支的最新版開始改起，初始化專案的步驟如下：

1. 取回原始碼

```text
git clone https://github.com/doggy8088/EFCore6Demo.git
cd EFCore6Demo
dotnet build
```

這份原始碼已經包含了一個 `ContosoUniversity.db` 資料庫檔案 (SQLite)，如果你有修改 Entity Framework Core 的實體模型，才需要用以下命令更新資料庫。

```text
dotnet ef migrations add MIGRATION_NAME
dotnet ef database update -v
```

開啟 VS Code

```text
code .
```

啟動專案

```text
dotnet watch run
```

呼叫 API

你可以直接用瀏覽器打開 https://localhost:7295/api/Departments/1 即可看到結果。

也可以透過 `curl` 進行測試：

```text
curl -s 'https://localhost:7295/api/Departments/1'
```

你會得到以下結果：

```json
{
  "departmentId": 1,
  "name": " 教育訓練部 ",
  "budget": 1000,
  "startDate": "2022-04-20T23:38:36.372125",
  "instructorId": null,
  "instructor": null,
  "courses": []
}
```

這裡你可以看到 `courses` 這個屬性回傳一個「空陣列」，這意味著 Entity Framework Core 並不會透過「導覽屬性」載入 `Courses` 的關聯資料。

# 預先載入 (Eager loading) (積極載入) (提前載入)

假設我們想載入一筆 Department 資料所關聯的多筆 Course 資料（一對多關係），我們有幾種可能的實現方式：

## 1. 不透過導覽屬性，直接對 Course 查詢資料

```csharp
[HttpGet("{id}/Courses/v1")]
public ActionResult<IEnumerable<Course>> GetDepartmentCourses(int id)
{
    return this._db.Courses.Where(p => p.DepartmentId == id).ToList();
}
```

存取 https://localhost:7295/api/Departments/5/Courses/v1 應該會得到 3 筆 `Course` 型別的資料：

```text
curl -s 'https://localhost:7295/api/Departments/5/Courses/v1'
```

```json
[
  {
    "courseId": 1,
    "title": "Entity Framework 6 開發實戰 ",
    "credits": 1,
    "departmentId": 5,
    "department": null,
    "enrollments": [],
    "instructors": []
  },
  {
    "courseId": 2,
    "title": "Git 新手入門 ",
    "credits": 1,
    "departmentId": 5,
    "department": null,
    "enrollments": [],
    "instructors": []
  },
  {
    "courseId": 3,
    "title": "Git 進階版控流程 ",
    "credits": 2,
    "departmentId": 5,
    "department": null,
    "enrollments": [],
    "instructors": []
  }
]
```

這種寫法的優點是「效率好」，不用對 `Department` 與 `Course` 進行 JOIN 查詢。

如果你不需要輸出 `Department` 資料，這種寫法其實沒有什麼缺點，唯一的缺點就是 `Course` 型別也有 3 個導覽屬性 (`department`, `enrollments`, `instructors`)，這三個導覽屬性不會有資料，因為 Entity Framework Core 預設並不會「延遲載入」關聯資料。如果你不需要將這三個導覽屬性輸出，可以參考 [How to ignore properties with System.Text.Json](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json-ignore-properties?pivots=dotnet-6-0&WT.mc_id=DT-MVP-4015686) 文章說明，加上 `[JsonIgnore]` 屬性 (Attribute) 到「導覽屬性」上即可。

## 2. 透過導覽屬性，直接對 `Department` 查詢資料就好，關聯資料直接透過「導覽屬性」取得

我們可以透過 `.Include(dept => dept.Courses)` 明確的告訴 EF Core 說要載入關聯資料，這樣的寫法會在查詢資料時自動填滿 dept.Courses 導覽屬性，讓你直接可以取得關聯資料：

```csharp
[HttpGet("{id}/Courses/v2")]
public ActionResult<IEnumerable<Course>> GetDepartmentCoursesByEagerLoading(int id)
{
    var dept = this._db.Departments.Include(dept => dept.Courses).First(p => p.DepartmentId == id);
    return dept.Courses.ToList();
}
```

開啟 https://localhost:7295/api/Departments/5/Courses/v2 你應該會得到以下「錯誤」訊息：

> JsonException: A possible object cycle was detected. This can either be due to a cycle or if the object depth is larger than the maximum allowed depth of 32. Consider using ReferenceHandler.Preserve on JsonSerializerOptions to support cycles. Path: $.Department.Courses.Department.Courses.Department.Courses.Department.Courses.Department.Courses.Department.Courses.Department.Courses.Department.Courses.Department.Courses.Department.Courses.CourseId.
>
> JsonException：檢測到可能存在物件循環。這可能是由於循環造成的，或者物件深度超過了允許的最大深度 32。請考慮在 JsonSerializerOptions 上使用 ReferenceHandler.Preserve 以支援循環。路徑：$.Department.Courses.Department.Courses.Department.Courses.Department.Courses.Department.Courses.Department.Courses.Department.Courses.Department.Courses.Department.Courses.Department.Courses.CourseId。

這算是在 ORM 的領域中常見的問題，因為「循環參考」導致資料無法序列化成 JSON 的狀況。

> 所謂的「循環參考」意味著你在將 `Department` 物件序列化成 JSON 的時候，其中有個「屬性」為「導覽屬性」，導覽屬性會幫你找出關聯的 `Courses` 集合資料，而每一筆 `Course` 資料也包含了一個 `Department` 導覽屬性，又會嘗試序列化這個屬性，如此一來就會形成一個「循環」，這會導致無窮迴圈的問題，因此會出現這個錯誤。

這個問題，我們可以透過 `[JsonIgnore]` 解決「循環參考」問題，你只要將 `Course` 型別上的「導覽屬性」加上 `[JsonIgnore]` 就不會發生「循環參考」問題。不過，你還是要特別小心，預先載入一個集合導覽屬性很有可能對效能帶來巨大衝擊。你可以適時的利用分割查詢 (Split queries) 來優化查詢效率，避免因為 JOIN 的關係從資料庫回傳大量的結果集。

## 3. 連續載入多層導覽屬性

如果你有個「一對多對多」的關聯，可以參考以下語法載入多層關聯的資料：

```csharp
using (var context = new BloggingContext())
{
    var blogs = context.Blogs
        .Include(blog => blog.Posts)
        .ThenInclude(post => post.Author)
        .ToList();
}
```

可以連續載入多層的關聯資料：

```csharp
using (var context = new BloggingContext())
{
    var blogs = context.Blogs
        .Include(blog => blog.Posts)
        .ThenInclude(post => post.Author)
        .ThenInclude(author => author.Photo)
        .ToList();
}
```

也可以載入相對複雜的關聯資料：

```csharp
using (var context = new BloggingContext())
{
    var blogs = context.Blogs
        .Include(blog => blog.Posts)
        .ThenInclude(post => post.Author)
        .ThenInclude(author => author.Photo)
        .Include(blog => blog.Owner)
        .ThenInclude(owner => owner.Photo)
        .ToList();
}
```

注意：==連續載入**多層導覽屬性**會有很大的機率導致 EF Core 查詢效能低落==，請務必小心使用！

要解決「循環參考」導致資料無法序列化成 JSON 的狀況，也有多種解決方案，不同的解決方案對效能的影響巨大，不可不知！

# 以下我分享三種解決方案

## 1. 調整 Controller 的 Json 序列化設定 (JsonSerializerOptions)

==這是效能最差最差的解法，拜託不要這樣用==，寫給你看不是要你學，是提醒你不要這樣寫！

```csharp
builder.Services.AddControllers().AddJsonOptions(options =>
{
    options.JsonSerializerOptions.ReferenceHandler = ReferenceHandler.IgnoreCycles;
});
```

這種寫法的優點是「不會出現例外」，而且你在不特別加入 [JsonIgnore] 的情況下，你的 API 就可以順利輸出 JSON 結果。你知道的，有些人要求不多，只求程式能動就好，那這招你可以參考，快，又有效！XD

這種寫法的缺點是「效能極差」，而且可能會浪費大量頻寬，產生許多無意義的關聯資料。

你直接看執行結果就知道了，許多無意義的循環參考資料都被加入，非常沒有效率：

```json
[
  {
    "courseId": 1,
    "title": "Entity Framework 6 開發實戰 ",
    "credits": 1,
    "departmentId": 5,
    "department": {
      "departmentId": 5,
      "name": " 專案開發部 ",
      "budget": 1000,
      "startDate": "2022-04-20T23:38:36.3721319",
      "instructorId": null,
      "instructor": null,
      "courses": [
        null,
        {
          "courseId": 2,
          "title": "Git 新手入門 ",
          "credits": 1,
          "departmentId": 5,
          "department": null,
          "enrollments": [],
          "instructors": []
        },
        {
          "courseId": 3,
          "title": "Git 進階版控流程 ",
          "credits": 2,
          "departmentId": 5,
          "department": null,
          "enrollments": [],
          "instructors": []
        }
      ]
    },
    "enrollments": [],
    "instructors": []
  },
  {
    "courseId": 2,
    "title": "Git 新手入門 ",
    "credits": 1,
    "departmentId": 5,
    "department": {
      "departmentId": 5,
      "name": " 專案開發部 ",
      "budget": 1000,
      "startDate": "2022-04-20T23:38:36.3721319",
      "instructorId": null,
      "instructor": null,
      "courses": [
        {
          "courseId": 1,
          "title": "Entity Framework 6 開發實戰 ",
          "credits": 1,
          "departmentId": 5,
          "department": null,
          "enrollments": [],
          "instructors": []
        },
        null,
        {
          "courseId": 3,
          "title": "Git 進階版控流程 ",
          "credits": 2,
          "departmentId": 5,
          "department": null,
          "enrollments": [],
          "instructors": []
        }
      ]
    },
    "enrollments": [],
    "instructors": []
  },
  {
    "courseId": 3,
    "title": "Git 進階版控流程 ",
    "credits": 2,
    "departmentId": 5,
    "department": {
      "departmentId": 5,
      "name": " 專案開發部 ",
      "budget": 1000,
      "startDate": "2022-04-20T23:38:36.3721319",
      "instructorId": null,
      "instructor": null,
      "courses": [
        {
          "courseId": 1,
          "title": "Entity Framework 6 開發實戰 ",
          "credits": 1,
          "departmentId": 5,
          "department": null,
          "enrollments": [],
          "instructors": []
        },
        {
          "courseId": 2,
          "title": "Git 新手入門 ",
          "credits": 1,
          "departmentId": 5,
          "department": null,
          "enrollments": [],
          "instructors": []
        },
        null
      ]
    },
    "enrollments": [],
    "instructors": []
  }
]
```

## 2. 調整實體模型的導覽屬性，特別加上 `[JsonIgnore]` 屬性 (Attribute)

```csharp
public partial class Course
{
    public Course()
    {
        Enrollments = new HashSet<Enrollment>();
        Instructors = new HashSet<Person>();
    }

    public int CourseId { get; set; }
    public string? Title { get; set; }
    public int Credits { get; set; }
    public int DepartmentId { get; set; }

    [JsonIgnore] // <-- 加上這行
    public virtual Department Department { get; set; } = null!;
    [JsonIgnore] // <-- 加上這行
    public virtual ICollection<Enrollment> Enrollments { get; set; }
    [JsonIgnore] // <-- 加上這行
    public virtual ICollection<Person> Instructors { get; set; }
}
```

在執行一次，就可以得到相當乾淨的 JSON 結果：

```json
[
  {
    "courseId": 1,
    "title": "Entity Framework 6 開發實戰 ",
    "credits": 1,
    "departmentId": 5
  },
  {
    "courseId": 2,
    "title": "Git 新手入門 ",
    "credits": 1,
    "departmentId": 5
  },
  {
    "courseId": 3,
    "title": "Git 進階版控流程 ",
    "credits": 2,
    "departmentId": 5
  }
]
```

這種寫法的==優點是「輸出的 JSON 結果很乾淨，也節省網路頻寬，下載速度快」==，而且你的 API 也可以順利輸出 JSON 結果！

這種寫法的==缺點是「需要手動調整實體模型」，而且你若用 DB First 的方式自動產生實體模型，下次在重新產生實體模型時可能會蓋掉你自己手動加上的 `[JsonIgnore]` 屬性 (Attribute)==，這個層次的程式**維護性**是需要考慮的！

==要解決這個缺點，可以考慮用使用自訂 ViewModel 的方式==，將 API 回應 JSON 部分避開「循環參考」等問題。

## 3. 透過 `Microsoft.AspNetCore.Mvc.NewtonsoftJson` 套件搭配 `MetadataType` 在另一個型別套用 `[JsonIgnore]` 屬性 (Attribute)

先加入 `Microsoft.AspNetCore.Mvc.NewtonsoftJson` 套件

```text
dotnet add package Microsoft.AspNetCore.Mvc.NewtonsoftJson
```

調整 `Program.cs` 的 `builder.Services.AddControllers()` 內容：

```text
builder.Services.AddControllers().AddNewtonsoftJson();
```

加入一個 `Course.Partial.cs` 檔案，內容如下，但重點是你不能用 ASP.NET Core MVC 的 `ModelMetadataType` 屬性 (Attribute)，必須改用 `System.ComponentModel.DataAnnotations` 下的 `MetadataType` 屬性 (Attribute)，而且 `[JsonIgnore]` 屬性 (Attribute) 要用 `Newtonsoft.Json` 命名空間下的才行：

```csharp
using System.ComponentModel.DataAnnotations;

namespace EFCore6Demo.Models;

[MetadataType(typeof(CourseMetadata))]
public partial class Course
{
}

internal class CourseMetadata
{
    public int CourseId { get; set; }
    public string? Title { get; set; }
    public int Credits { get; set; }
    public int DepartmentId { get; set; }

    [Newtonsoft.Json.JsonIgnore]
    public virtual Department Department { get; set; } = null!;
    [Newtonsoft.Json.JsonIgnore]
    public virtual ICollection<Enrollment> Enrollments { get; set; }
    [Newtonsoft.Json.JsonIgnore]
    public virtual ICollection<Person> Instructors { get; set; }
}
```

這種寫法的==優點是輸出的 JSON 結果很乾淨==，你的 API 也可以順利輸出 JSON 結果，而且==就算你用 DB First 的方式自動產生實體模型，下次在重新產生實體模型時也不會蓋掉你自己手動加上的 [JsonIgnore] 屬性 (Attribute)==，可以==兼具效率與程式維護性==！

這種寫法的==缺點==是必須繼續使用 `Newtonsoft.Json` (JSON.NET) 來序列化回應的資料，==無法享受 System.Text.Json 命名空間下的類別帶來的效能優勢！==

# 延遲載入 (Lazy loading) (消極載入)

我可以說「延遲載入」(Lazy loading) 是 Entity Framework Core 或任何 ORM 框架的初學者最愛的功能，但是越方便的功能越可能帶來致命的效能傷害，[經典的 N+1 問題](https://medium.com/doctolib/understanding-and-fixing-n-1-query-30623109fe89) 就是因為「延遲載入」所造成的，不得不謹慎看待，==沒事千萬不要啟用這個功能！==

要在 Entity Framework Core 啟用「延遲載入」功能還挺麻煩的，步驟如下：

1. 安裝 `Microsoft.EntityFrameworkCore.Proxies` 套件
   當透過導覽屬性讀取關聯資料時，會自動載入關聯資料 (需額外設定)

```text
dotnet add package Microsoft.EntityFrameworkCore.Proxies
```

2. 調整載入 DbContext 的設定

```csharp
builder.Services.AddDbContext<ContosoUniversityContext>(options =>
  options.UseLazyLoadingProxies()
    .UseSqlite(builder.Configuration.GetConnectionString("DefaultConnection")));
```

需特別注意 Related data and serialization 的問題！

3. 接著你的 `DepartmentController` 就可以加入以下 Action

```csharp
[HttpGet("{id}/Courses/v3")]
public ActionResult<IEnumerable<Course>> GetDepartmentCoursesByLazyLoading(int id)
{
    var dept = this._db.Departments.Find(id);
    return dept.Courses.ToList();
}
```

若不想要啟用 Proxy 實體模型的話，那就麻煩很多，並不適用 DB First 的用戶，有興趣可瞭解的朋友可以參考官網的 [Lazy loading without proxies](https://learn.microsoft.com/en-us/ef/core/querying/related-data/lazy?WT.mc_id=DT-MVP-4015686#lazy-loading-without-proxies) 文件。

# 明確載入 (Explicit loading)

當你想精準的控制「關聯資料」的載入時機，就可以透過明確載入的 API 來自行載入導覽屬性的關聯資料。

- 明確載入集合導覽屬性 (載入一對多的導覽屬性內容)

```csharp
[HttpGet("{id}/Courses/v4")]
public ActionResult<IEnumerable<Course>> GetDepartmentCoursesByExplicitLoading(int id)
{
    var dept = this._db.Departments.Find(id);
    this._db.Entry(dept).Collection(p => p.Courses).Load();
    return dept.Courses.ToList();
}
```

- 明確載入參考導覽屬性 (載入多對一的導覽屬性內容)

```csharp
[HttpGet("~/api/Courses/{id}/Department")]
public ActionResult<Department> GetDepartmentFromCourseByExplicitLoading(int id)
{
    var course = this._db.Courses.Find(id);
    this._db.Entry(course).Reference(p => p.Department).Load();
    return course.Department;
}
```

若不想 JSON 輸出導覽屬性，請記得要加上正確的 `[JsonIgnore]` 屬性 (Attribute)

- 設定明確載入但僅定義 LINQ 查詢
  從官網的 [Querying related entities](https://learn.microsoft.com/en-us/ef/core/querying/related-data/explicit?WT.mc_id=DT-MVP-4015686#querying-related-entities) 可以看到一個很棒的用法，你在「明確載入集合導覽屬性」或「明確載入參考導覽屬性」的時候不要執行 `.Load()` 方法，而是改執行 `.Query()` 方法。這段語法將不會立即載入資料，而是產生 LINQ 語法而已，你可以搭配「匯總函式」來優化查詢效率！👍

```csharp
using (var context = new BloggingContext())
{
    var blog = context.Blogs
        .Single(b => b.BlogId == 1);

    var postCount = context.Entry(blog)
        .Collection(b => b.Posts)
        .Query()
        .Count();
}
```

# 相關連結

- [Loading Related Data - EF Core | Microsoft Docs](https://learn.microsoft.com/en-us/ef/core/querying/related-data/?WT.mc_id=DT-MVP-4015686)
	- [Eager Loading of Related Data](https://learn.microsoft.com/en-us/ef/core/querying/related-data/eager?WT.mc_id=DT-MVP-4015686)
	- [Lazy Loading of Related Data](https://learn.microsoft.com/en-us/ef/core/querying/related-data/lazy?WT.mc_id=DT-MVP-4015686)
	- [Explicit Loading of Related Data](https://learn.microsoft.com/en-us/ef/core/querying/related-data/explicit?WT.mc_id=DT-MVP-4015686)
- [Use Newtonsoft's Json.NET instead of System.Text.Json in ASP.NET Core 3+ MVC projects](https://www.ryadel.com/en/use-json-net-instead-of-system-text-json-in-asp-net-core-3-mvc-projects/)
- [How to set json serializer settings in asp.net core 3?](https://stackoverflow.com/a/69404465/910074)
- [System.Text.Json add JsonIgnore attribute at runtime](https://stackoverflow.com/a/65768580/910074)
- [System.Text.Json.Serialization.JsonIgnoreAttribute not working in asp.net Core 3.1 partial ModelMetadataType class #17891](https://github.com/dotnet/aspnetcore/issues/17891)
- .[Net Core 3.0 possible object cycle was detected which is not supported](https://stackoverflow.com/a/70843023/910074)
