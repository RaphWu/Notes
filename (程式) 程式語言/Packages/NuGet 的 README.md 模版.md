# 目錄

- [目錄](#目錄)
- [NuGet 文件分類，存放，展示](#nuget-文件分類存放展示)
  - [目錄結構（標準化）](#目錄結構標準化)
  - [1. NuGet README（發佈層）](#1-nuget-readme發佈層)
    - [規則](#規則)
    - [內容限制（強制精簡）](#內容限制強制精簡)
    - [禁止](#禁止)
    - [原則](#原則)
  - [2. 完整文件（使用層）](#2-完整文件使用層)
    - [存放](#存放)
    - [文件分類](#文件分類)
    - [展示策略](#展示策略)
    - [API 文件策略](#api-文件策略)
  - [3. 開發文件（開發層）](#3-開發文件開發層)
    - [存放](#存放-1)
    - [內容分類](#內容分類)
    - [規則](#規則-1)
    - [ADR（Architecture Decision Record）建議格式](#adrarchitecture-decision-record建議格式)
  - [4. 文件流動關係（關鍵）](#4-文件流動關係關鍵)
    - [規則](#規則-2)
  - [5. NuGet 打包控制](#5-nuget-打包控制)
    - [預設行為](#預設行為)
    - [控制方式](#控制方式)
    - [確保不打包](#確保不打包)
    - [若有例外（明確排除）](#若有例外明確排除)
  - [6. Git 與展示策略](#6-git-與展示策略)
    - [GitHub 顯示優先順序](#github-顯示優先順序)
    - [建議](#建議)
  - [7. 實務準則（強制）](#7-實務準則強制)
  - [8. 常見錯誤](#8-常見錯誤)
  - [9. 最小落地方案](#9-最小落地方案)
- [NuGet 的 README.md 模版](#nuget-的-readmemd-模版)
  - [建議的 NuGet README.md 結構](#建議的-nuget-readmemd-結構)
    - [1️⃣ 套件簡介（1–2 行）](#1️⃣-套件簡介12-行)
    - [2️⃣ 安裝方式](#2️⃣-安裝方式)
    - [5️⃣ 文件 / Repo 連結](#5️⃣-文件--repo-連結)
    - [6️⃣ License（可選）](#6️⃣-license可選)
  - [不建議放在 NuGet README 的內容](#不建議放在-nuget-readme-的內容)
  - [最推薦的 NuGet README 模板（實務版）](#最推薦的-nuget-readme-模板實務版)
    - [Quick Example](#quick-example)
    - [Features](#features)
    - [Documentation](#documentation)
    - [Quick Start](#quick-start)
    - [Features](#features-1)
    - [Configuration (Optional)](#configuration-optional)
    - [Documentation](#documentation-1)
    - [License](#license)
  - [這個模板為什麼適合 NuGet](#這個模板為什麼適合-nuget)
  - [NuGet README 的最佳長度](#nuget-readme-的最佳長度)
  - [很多熱門套件的策略](#很多熱門套件的策略)
  - [進階技巧（強烈推薦）](#進階技巧強烈推薦)
    - [NuGet README](#nuget-readme)
    - [GitHub README](#github-readme)
    - [1️⃣ README（主要說明）](#1️⃣-readme主要說明)
    - [2️⃣ 版本變更紀錄](#2️⃣-版本變更紀錄)
    - [3️⃣ 授權條款](#3️⃣-授權條款)
    - [4️⃣ 貢獻指南](#4️⃣-貢獻指南)
    - [5️⃣ 行為準則](#5️⃣-行為準則)
    - [6️⃣ 安全漏洞回報](#6️⃣-安全漏洞回報)
    - [7️⃣ FAQ（可選）](#7️⃣-faq可選)
    - [8️⃣ 詳細文件（docs 資料夾）](#8️⃣-詳細文件docs-資料夾)
  - [典型 NuGet Library 專案結構](#典型-nuget-library-專案結構)
  - [最常見（最小集合）](#最常見最小集合)
  - [一個成熟 NuGet Library Repo 的文件結構](#一個成熟-nuget-library-repo-的文件結構)
  - [每個文件的角色](#每個文件的角色)
    - [README.md（GitHub 版）](#readmemdgithub-版)
    - [README.NUGET.md（NuGet 版）](#readmenugetmdnuget-版)
  - [docs 資料夾（最重要）](#docs-資料夾最重要)
  - [CHANGELOG.md](#changelogmd)
  - [UPGRADING.md（很多人忽略但很重要）](#upgradingmd很多人忽略但很重要)
  - [CONTRIBUTING.md](#contributingmd)
  - [SECURITY.md](#securitymd)
  - [最推薦的文件策略](#最推薦的文件策略)
  - [成熟專案常見的小技巧](#成熟專案常見的小技巧)
- [建議的內容分層策略](#建議的內容分層策略)
  - [README.md（NuGet 上顯示的）— 保持簡潔](#readmemdnuget-上顯示的-保持簡潔)
    - [完整說明放哪裡？](#完整說明放哪裡)
    - [實務上主流套件怎麼做](#實務上主流套件怎麼做)
    - [推薦的完整文件工具](#推薦的完整文件工具)
- [企業場景！內部 NuGet 不能連外部網站](#企業場景內部-nuget-不能連外部網站)
  - [解決方案：把文件打包進 NuGet 套件內](#解決方案把文件打包進-nuget-套件內)
    - [專案結構建議](#專案結構建議)
    - [`.csproj` 設定：把 docs 打包進去](#csproj-設定把-docs-打包進去)
    - [使用者安裝後在哪裡找到文件？](#使用者安裝後在哪裡找到文件)
    - [README.md 內容改成指向本地路徑](#readmemd-內容改成指向本地路徑)
    - [更進一步：搭配 XML 註解自動文件](#更進一步搭配-xml-註解自動文件)
    - [總結](#總結)

---

# NuGet 文件分類，存放，展示

文件需拆為三層：

- 發佈層（NuGet 顯示）
- 使用層（外部工程師完整文件）
- 開發層（內部設計與重構紀錄）

三者**物理分離、責任單一、來源一致（可由同一份結構生成）**

## 目錄結構（標準化）

```
/ (Repo Root)
├─ README.md                  → NuGet 顯示用（精簡版）
├─ src/
├─ tests/
├─ docs/                      → 對外完整文件
│  ├─ index.md
│  ├─ getting-started.md
│  ├─ api/
│  ├─ design/
│  └─ faq.md
├─ dev-docs/                  → 內部文件（不對外）
│  ├─ refactor/
│  ├─ decisions/
│  ├─ experiments/
│  └─ notes.md
├─ build/
└─ .github/
```

## 1. NuGet README（發佈層）

### 規則

- 檔案：`/README.md`
- 由 `.csproj` 指定：

```
<PackageReadmeFile>README.md</PackageReadmeFile>
```

### 內容限制（強制精簡）

- 專案簡介（1段）
- 安裝方式
- 最小可執行範例（Minimal Example）
- API 快速入口（連到 docs）
- 版本/相依說明（必要才放）

### 禁止

- ✘ 長篇說明
- ✘ 設計細節
- ✘ 內部架構
- ✘ 完整 API 列表

### 原則

- README = **入口索引 + 快速上手**
- 所有詳細內容 → `docs/`

## 2. 完整文件（使用層）

### 存放

- `/docs/`
- 建議搭配：
  - GitHub Pages
  - DocFX / MkDocs / Docusaurus

### 文件分類

```
docs/
├─ index.md              → 文件首頁（導覽）
├─ getting-started.md    → 快速上手
├─ api/                  → API 分類說明（非自動產生）
├─ design/               → 對外可公開設計（架構、流程）
├─ advanced/             → 進階用法
└─ faq.md
```

### 展示策略

- README → 只放入口
- README 中明確導向：

```
完整文件：https://xxx.github.io/your-repo/
```

### API 文件策略

- ✔ XML Doc + DocFX（或同類工具）
- ✔ 手寫「使用導向 API 文件」
- ✘ 不可只靠自動生成（缺語意）

## 3. 開發文件（開發層）

### 存放

- `/dev-docs/`（或 `/internal/`）

### 內容分類

```
dev-docs/
├─ refactor/         → 重構記錄（before/after、原因）
├─ decisions/        → 架構決策（ADR）
├─ experiments/      → 測試性實作
├─ benchmarks/       → 效能測試
└─ notes.md          → 雜項紀錄
```

### 規則

- 不發佈到 NuGet
- 不放 README 連結
- 不保證穩定性（允許草稿）

### ADR（Architecture Decision Record）建議格式

```
- Context
- Decision
- Consequences
```

## 4. 文件流動關係（關鍵）

```
dev-docs →（整理/萃取）→ docs →（摘要）→ README
```

### 規則

- dev-docs = 原始思考（雜）
- docs = 結構化知識（穩定）
- README = 最小入口（極簡）

## 5. NuGet 打包控制

### 預設行為

- 只有指定檔案會進 NuGet

### 控制方式

```
<ItemGroup>
  <None Include="README.md" Pack="true" PackagePath="" />
</ItemGroup>
```

### 確保不打包

```
docs/**        → 預設不會進
dev-docs/**    → 預設不會進
```

### 若有例外（明確排除）

```
<ItemGroup>
  <None Include="docs\**" Pack="false" />
  <None Include="dev-docs\**" Pack="false" />
</ItemGroup>
```

## 6. Git 與展示策略

### GitHub 顯示優先順序

- Repo 首頁 → `/README.md`
- 詳細文件 → `/docs/`

### 建議

- 啟用 GitHub Pages 指向 `/docs/`
- README 僅做導流

## 7. 實務準則（強制）

- README ≤ 200 行
- docs 必須可「不看原始碼就能使用」
- dev-docs 不需對外可讀
- 所有設計結論需從 dev-docs 萃取到 docs
- README 不允許成為文件主體

## 8. 常見錯誤

- ✘ README 過長 → 使用者無法快速定位
- ✘ docs 缺失 → 使用者只能讀原始碼
- ✘ dev-docs 混入 docs → 汙染對外文件
- ✘ 將 docs 打包進 NuGet → 無法被使用者看到
- ✘ API 文件完全自動生成 → 缺語意與案例

## 9. 最小落地方案

若不導入文件系統：

- README.md（精簡）
- docs/getting-started.md
- docs/api.md
- dev-docs/notes.md

即可形成完整三層架構

[🔝](#目錄)

---

# NuGet 的 README.md 模版

在 NuGet 套件的 **README.md**，確實不建議完全照一般 App 或 GitHub 專案的 README 結構來寫，因為在 NuGet Package Manager（例如 Visual Studio 的 NuGet UI）裡會直接渲染整份 README，如果內容太長，使用者在套件管理員裡閱讀會非常痛苦。

實務上 **NuGet README 的最佳做法是「短、快速上手、連到完整文件」**。
官方和社群常用的結構大概是 **精簡版 README**。

## 建議的 NuGet README.md 結構

### 1️⃣ 套件簡介（1–2 行）

說明套件用途，不要長篇介紹。

```md
## MyLibrary

A lightweight library for generating signed API requests in .NET.
```

### 2️⃣ 安裝方式

NuGet UI 的讀者很常只想看這段。

````md
### Install

```bash
dotnet add package MyLibrary
````

or via NuGet Package Manager.

````
#### 3️⃣ 快速使用範例（最重要）
只放 **最短能跑的 example**。

```md
### Quick Example

```csharp
var client = new ApiClient("api-key");
var result = await client.GetDataAsync();
````

````
⚠️ 建議：
- **10–20 行內**
- 只展示核心 API

#### 4️⃣ 主要功能（簡短 bullet）
不要寫太多。

```md
### Features

- Simple API client
- Automatic retry
- Built-in logging support
````

### 5️⃣ 文件 / Repo 連結

避免 README 太長的關鍵。

```md
### Documentation

Full documentation:
https://github.com/your-org/MyLibrary
```

### 6️⃣ License（可選）

```md
### License

MIT
```

## 不建議放在 NuGet README 的內容

這些會讓 NuGet UI 變成「臭又長」：

❌ 大量架構說明
❌ Changelog
❌ 貢獻指南
❌ 開發流程
❌ CI/CD
❌ 版本策略
❌ 複雜 API 文件

這些都應該放在 **GitHub README 或 docs site**。

## 最推薦的 NuGet README 模板（實務版）

````md
## MyLibrary

A lightweight library for XYZ.

### Install

```bash
dotnet add package MyLibrary
````

### Quick Example

```csharp
var client = new MyClient();
client.DoSomething();
```

### Features

- Feature A
- Feature B
- Feature C

### Documentation

[https://github.com/your-org/MyLibrary](https://github.com/your-org/MyLibrary)

````csharp

通常 **30–60 行內最佳**。



💡 **進階技巧（很多熱門 NuGet 套件會用）**

在 NuGet README：

只放 **Quick Start**

在 GitHub README：

完整文件 + 架構 + 教學

例如像
- :contentReference[oaicite:3]{index=3}
- :contentReference[oaicite:4]{index=4}
- :contentReference[oaicite:5]{index=5}

都是 **NuGet 簡版 + GitHub 完整版** 的模式。



下面是一個 **實務上很常見的 .NET / NuGet 套件 README 模板**。
設計目標是 **在 NuGet Package Manager（例如 Visual Studio）裡看起來簡潔、可快速上手**，但又不會太空。

整體控制在 **40–80 行內**。



## NuGet README 最佳實務模板

## MyLibrary

A lightweight .NET library for **[one-sentence description]**.

### Install

Install via .NET CLI:

```bash
dotnet add package MyLibrary
````

Or install from **NuGet Package Manager**.

### Quick Start

```csharp
using MyLibrary;

var client = new MyClient("api-key");

var result = await client.DoSomethingAsync();

Console.WriteLine(result);
```

### Features

- Simple and lightweight API
- Async-first design
- Works with .NET 6+

### Configuration (Optional)

```csharp
var client = new MyClient(new MyClientOptions
{
    Timeout = TimeSpan.FromSeconds(10)
});
```

### Documentation

Full documentation and advanced usage:

[https://github.com/your-org/MyLibrary](https://github.com/your-org/MyLibrary)

### License

MIT

## 這個模板為什麼適合 NuGet

原因是 **NuGet 使用者的閱讀模式很不一樣**：

他們通常只想知道：

1️⃣ 這是什麼
2️⃣ 怎麼裝
3️⃣ 怎麼用

所以 README 只保留：

| 區塊        | 必要性     |
| ----------- | ---------- |
| Description | 必須       |
| Install     | 必須       |
| Quick Start | **最重要** |
| Features    | 建議       |
| Docs Link   | 建議       |
| License     | 可選       |

## NuGet README 的最佳長度

建議：

```csharp
30 – 80 行
```

超過：

```csharp
120 行
```

在 NuGet UI 就會開始 **閱讀體驗很差**。

## 很多熱門套件的策略

像這些 .NET 套件其實都用 **短 README + 完整 GitHub 文件**：

- Dapper
- Serilog
- Polly
- Newtonsoft.Json

NuGet README：

```csharp
Quick Start
```

GitHub README：

```csharp
完整教學
架構
進階 API
FAQ
Changelog
```

## 進階技巧（強烈推薦）

很多成熟套件會這樣做：

### NuGet README

```csharp
README.NUGET.md
```

### GitHub README

```csharp
README.md
```

然後 `.csproj`：

```xml
<PackageReadmeFile>README.NUGET.md</PackageReadmeFile>
```

這樣：

- NuGet 顯示 **精簡版**
- GitHub 顯示 **完整版**

如果你把 **NuGet README 做成精簡版**，那通常會搭配一套 **常見的文件命名規則**，讓專案在 GitHub 或 GitLab 上看起來清楚、專業，也方便使用者找到資料。

下面是 **.NET / NuGet 套件常見的文件命名方式（實務慣例）**。

### 1️⃣ README（主要說明）

```text
README.md
```

用途：

- GitHub 專案首頁
- 完整介紹
- 架構、使用方式、範例

如果 NuGet 要用簡版：

```text
README.NUGET.md
```

`.csproj`

```xml
<PackageReadmeFile>README.NUGET.md</PackageReadmeFile>
```

### 2️⃣ 版本變更紀錄

```text
CHANGELOG.md
```

用途：

- 記錄每個版本修改內容
- NuGet 套件常見

範例：

```md
### 1.3.0

- Added retry support
- Improved logging

### 1.2.0

- Added caching support
```

很多專案遵循
Keep a Changelog 格式。

### 3️⃣ 授權條款

```text
LICENSE
```

或

```text
LICENSE.md
```

例如：

- MIT
- Apache 2.0

### 4️⃣ 貢獻指南

```text
CONTRIBUTING.md
```

用途：

- Pull Request 規範
- Coding style
- Issue 提交流程

### 5️⃣ 行為準則

```text
CODE_OF_CONDUCT.md
```

開源專案常見。

### 6️⃣ 安全漏洞回報

```text
SECURITY.md
```

說明：

- 如何回報漏洞
- 是否支援舊版本

### 7️⃣ FAQ（可選）

```text
FAQ.md
```

例如：

- 常見錯誤
- 常見設定

### 8️⃣ 詳細文件（docs 資料夾）

很多專案會用：

```text
docs/
```

例如：

```csharp
docs/
 ├─ getting-started.md
 ├─ configuration.md
 ├─ advanced-usage.md
 ├─ architecture.md
```

這樣 README 就不會過長。

## 典型 NuGet Library 專案結構

```text
MyLibrary
│
├─ src/
│
├─ README.md
├─ README.NUGET.md
├─ CHANGELOG.md
├─ LICENSE
├─ CONTRIBUTING.md
├─ SECURITY.md
│
└─ docs/
   ├─ getting-started.md
   ├─ configuration.md
   └─ advanced.md
```

## 最常見（最小集合）

如果是 **一般 NuGet 套件**，通常只需要：

```text
README.md
README.NUGET.md
CHANGELOG.md
LICENSE
```

這已經是 **90% 開源 .NET 套件的標準配置**。

💡 **小建議（很多 .NET 套件會忽略）**

如果是 library，最好再加：

```text
UPGRADING.md
```

用來寫 **破壞性變更（breaking changes）**。

例如：

```csharp
UPGRADING.md
```

```md
## Upgrade from 2.x to 3.x

- `ApiClient` constructor changed
- Removed synchronous API
```

這在大型套件非常重要。

下面是一個 **成熟 .NET / NuGet library 常見的完整文件結構**。
這種結構在很多大型專案（例如 Serilog、Dapper、Polly）都能看到類似設計。

目標是：

- **NuGet 使用者 → 快速上手**
- **GitHub 使用者 → 找得到完整文件**
- **維護者 → 文件不混亂**

## 一個成熟 NuGet Library Repo 的文件結構

```text
MyLibrary
│
├─ src/
│   └─ MyLibrary/
│
├─ tests/
│
├─ docs/
│   ├─ getting-started.md
│   ├─ configuration.md
│   ├─ dependency-injection.md
│   ├─ advanced-usage.md
│   └─ architecture.md
│
├─ README.md
├─ README.NUGET.md
├─ CHANGELOG.md
├─ LICENSE
├─ CONTRIBUTING.md
├─ SECURITY.md
├─ UPGRADING.md
│
└─ MyLibrary.sln
```

## 每個文件的角色

### README.md（GitHub 版）

用途：**完整介紹**

內容通常包含：

```text
Project intro
Installation
Quick Start
Features
Basic usage
Links to docs
```

README 不應該包含：

```csharp
大量 API 文件
完整教學
架構細節
```

那些應該放到 `docs/`

### README.NUGET.md（NuGet 版）

用途：

在 NuGet Package Manager 顯示。

內容只保留：

```csharp
簡短介紹
Install
Quick Start
Docs link
```

通常 **30–60 行**。

`.csproj`

```xml
<PackageReadmeFile>README.NUGET.md</PackageReadmeFile>
```

## docs 資料夾（最重要）

成熟專案通常會拆成多份文件。

建議結構：

```text
docs/
 ├─ getting-started.md
 ├─ configuration.md
 ├─ dependency-injection.md
 ├─ advanced-usage.md
 └─ architecture.md
```

說明：

| 文件                    | 用途     |
| ----------------------- | -------- |
| getting-started.md      | 詳細入門 |
| configuration.md        | 設定方式 |
| dependency-injection.md | DI 整合  |
| advanced-usage.md       | 進階用法 |
| architecture.md         | 架構設計 |

## CHANGELOG.md

版本歷史：

```text
### 2.1.0
Added caching support

### 2.0.0
Breaking changes:
- Removed synchronous API
```

通常遵循
Keep a Changelog。

## UPGRADING.md（很多人忽略但很重要）

當 major version 有 **breaking change**：

```text
UPGRADING.md
```

範例：

```md
## Upgrade from 2.x to 3.x

### ApiClient

Old:

ApiClient(string apiKey)

New:

ApiClient(ApiOptions options)
```

大型 library 幾乎都會有。

## CONTRIBUTING.md

說明：

```csharp
如何提交 PR
Coding style
測試規則
```

對開源很重要。

## SECURITY.md

漏洞回報方式，例如：

```csharp
Email security@company.com
```

GitHub 會自動識別。

## 最推薦的文件策略

**三層文件設計**

```csharp
NuGet README
   ↓
GitHub README
   ↓
docs/*
```

概念：

```csharp
NuGet → Quick start
GitHub → Overview
Docs → Full guide
```

## 成熟專案常見的小技巧

README 裡面放 **docs 入口**

```md
### Documentation

- Getting Started
- Configuration
- Dependency Injection
- Advanced Usage
```

連到：

```csharp
docs/*
```

這樣 README **永遠不會爆長**。

💡 **最後給你一個非常實用的建議（很多資深 .NET library maintainer 都這樣做）**

文件命名 **全部小寫 + dash**

```text
getting-started.md
dependency-injection.md
advanced-usage.md
```

不要：

```csharp
GettingStarted.md
DependencyInjection.md
```

原因：

- URL 比較乾淨
- 跨平台一致
- GitHub 習慣

[🔝](#目錄)

---

# 建議的內容分層策略

## README.md（NuGet 上顯示的）— 保持簡潔

專注於「讓人快速上手」，建議控制在 **100～200 行以內**：

```markdown
# YourPackage

一句話說明這個套件做什麼。

## 安裝
dotnet add package YourPackage

## 快速開始
// 最簡單的使用範例，5～10 行內搞定

## 主要功能
- 功能 A
- 功能 B
- 功能 C

## 文件
完整文件請參考：https://yourproject.github.io/docs

## 授權
MIT
```

### 完整說明放哪裡？

| 層級              | 放置位置                                     | 內容                        |
| ----------------- | -------------------------------------------- | --------------------------- |
| **快速預覽**      | `README.md`（NuGet 顯示）                    | 安裝、Quick Start、功能列表 |
| **完整文件**      | GitHub repo 的 `README.md` 或 `docs/` 資料夾 | 所有 API、進階用法、範例    |
| **詳細 API 參考** | GitHub Pages / DocFX / GitBook               | 每個方法的完整說明          |
| **更新紀錄**      | `CHANGELOG.md`                               | 版本異動內容                |

### 實務上主流套件怎麼做

**Serilog、Newtonsoft.Json** 等大型套件的做法：

- NuGet README → 極簡，只有安裝指令 + 一個範例 + 文件連結
- 詳細內容 → 導向 GitHub Wiki 或官網

**關鍵原則：**

> NuGet README 的任務是「讓人決定要不要用」＋「讓人立刻能跑起來」，不是取代文件網站。

### 推薦的完整文件工具

如果你的套件較複雜，可以搭配：

- **[DocFX](https://dotnet.github.io/docfx/)** — .NET 官方文件產生工具，可從 XML 註解自動產生 API 文件
- **GitHub Pages** — 免費託管靜態文件網站
- **GitHub Wiki** — 輕量選擇，適合中小型套件

這樣 NuGet README 只需一行連結，讀者自然會去找完整說明。

[🔝](#目錄)

---

# 企業場景！內部 NuGet 不能連外部網站

## 解決方案：把文件打包進 NuGet 套件內

### 專案結構建議

```
YourPackage/
├── YourPackage.csproj
├── README.md              ← NuGet 顯示的簡短說明
├── docs/
│   ├── FULL_README.md     ← 完整說明
│   ├── CHANGELOG.md       ← 版本異動
│   └── examples/
│       ├── BasicUsage.md
│       └── AdvancedUsage.md
└── src/
    └── ...
```

### `.csproj` 設定：把 docs 打包進去

```xml
<PropertyGroup>
  <PackageId>YourPackage</PackageId>
  <PackageReadmeFile>README.md</PackageReadmeFile>
  <GenerateDocumentationFile>true</GenerateDocumentationFile> <!-- 產生 XML 註解文件 -->
</PropertyGroup>

<ItemGroup>
  <!-- NuGet 顯示的 README -->
  <None Include="README.md" Pack="true" PackagePath="\" />

  <!-- 完整文件，打包進套件的 docs 資料夾 -->
  <None Include="docs\**\*" Pack="true" PackagePath="docs\" />
</ItemGroup>
```

### 使用者安裝後在哪裡找到文件？

安裝後文件會出現在：

```
%USERPROFILE%\.nuget\packages\yourpackage\1.0.0\docs\
```

或專案內：

```
packages/yourpackage.1.0.0/docs/    ← 舊版 packages.config 方式
```

### README.md 內容改成指向本地路徑

```markdown
## 文件

完整說明請參考套件內的 docs 資料夾：
- 完整說明：`docs/FULL_README.md`
- 更新紀錄：`docs/CHANGELOG.md`
- 範例：`docs/examples/`

安裝後可在 NuGet 快取目錄找到：
`%USERPROFILE%\.nuget\packages\yourpackage\{版本}\docs\`
```

### 更進一步：搭配 XML 註解自動文件

在程式碼加上完整 XML 註解，讓 IDE 直接顯示說明，減少查文件的需求：

```csharp
/// <summary>
/// 初始化連線，使用預設逾時 30 秒。
/// </summary>
/// <param name="connectionString">資料庫連線字串，格式為 Server=...;Database=...</param>
/// <param name="timeout">逾時秒數，預設 30，最大 300。</param>
/// <returns>回傳已初始化的 Client 實例。</returns>
/// <example>
/// <code>
/// var client = new YourClient("Server=localhost;Database=test", timeout: 60);
/// </code>
/// </example>
public YourClient Init(string connectionString, int timeout = 30) { ... }
```

啟用後 `.xml` 會一起打包，開發者在 VS 中 hover 方法就能看到說明。

### 總結

| 說明類型 | 存放位置             | 開發者如何取得      |
| -------- | -------------------- | ------------------- |
| 快速摘要 | `README.md`          | VS 管理介面直接顯示 |
| 完整文件 | `docs/` 打包進套件   | NuGet 快取資料夾    |
| API 說明 | XML 註解 → `.xml` 檔 | VS Hover 提示       |
| 版本紀錄 | `docs/CHANGELOG.md`  | NuGet 快取資料夾    |

這樣即使完全離線、不依賴任何外部空間，所有文件都隨套件走。

[🔝](#目錄)
