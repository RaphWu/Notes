---
aliases:
created: 
update:
author:
language:
sourceurl:
tags:
date:
---

# 建立 SDK-style Library

# 最終範本

- 要修改的部分
    - 專案資訊內的參數。
    - 確認有 `README.md` (有分大小寫)，且與 `.csproj` 在相同目錄。
    - 若不需建置 NuGet 則拿掉 `NuGet`。
    - 有*自動增加版本號*功能時，要加入 `UpdateVersion.ps1` 及 `Version.txt` 兩個檔案。

```xml title="工控系統 .csproj 基本設定"
<Project Sdk="Microsoft.NET.Sdk">

    <PropertyGroup>
        <TargetFrameworks>net48;net8.0</TargetFrameworks>
        <PlatformTarget>AnyCPU</PlatformTarget>
        <LangVersion>9.0</LangVersion>
        <Nullable>disable</Nullable>
        <Deterministic>true</Deterministic>
        <AutoGenerateBindingRedirects>true</AutoGenerateBindingRedirects>

        <!-- .NET 6+ 及 SDK-Style 專案才有效果 -->
        <!-- 須在非開發機Debug時勿enable -->
        <!-- 工控系統建議關閉或直接不要寫 -->
        <ImplicitUsings>disable</ImplicitUsings>

        <!-- 專案資訊 -->
        <GenerateAssemblyInfo>false</GenerateAssemblyInfo> <!-- 若有自動增加版本號，則不要這一行 -->
        <Authors>佳凌科技股份有限公司 Calin Technology Co.,Ltd.</Authors>
        <Company>佳凌科技股份有限公司 Calin Technology Co.,Ltd.</Company>
        <Copyright>Copyright © 2026 Calin Technology Co.,Ltd.</Copyright>

        <AssemblyTitle>Calin.ProjectName</AssemblyTitle>
        <RootNamespace>Calin.ProjectName</RootNamespace>
        <AssemblyName>Calin.ProjectName</AssemblyName>
        <Description></Description>

        <!-- Library -->
        <OutputType>Library</OutputType>
        <GenerateDocumentationFile>true</GenerateDocumentationFile>
        <PackageReadmeFile>README.md</PackageReadmeFile>
        <Version>0.0.1</Version>
    </PropertyGroup>
    
    <!-- NuGet -->
    <PropertyGroup Condition="'$(Configuration)' == 'Release'">
        <GeneratePackageOnBuild>true</GeneratePackageOnBuild>
        <PackageOutputPath>..\..\nuget\</PackageOutputPath>
    </PropertyGroup>
    <PropertyGroup Condition="'$(Configuration)' == 'Debug'">
        <GeneratePackageOnBuild>false</GeneratePackageOnBuild>
    </PropertyGroup>
    
    <!-- 讓 NuGet 包含 XML -->
    <ItemGroup>
        <None Include="$(OutputPath)\$(AssemblyName).xml"
            Condition="Exists('$(OutputPath)\$(AssemblyName).xml')"
            Pack="true"
            PackagePath="lib\$(TargetFramework)" />
    </ItemGroup>
    
    <!-- 將 README.md 包進 NuGet -->
	<ItemGroup>
		<None Include="README.md" Pack="true" PackagePath="" />
	</ItemGroup>

	<!-- 自動增加版本號 -->
	<Target Name="UpdateAssemblyVersion" BeforeTargets="BeforeBuild">
		<PropertyGroup>
            <VersionFile>$(ProjectDir)version.txt</VersionFile>
            <AssemblyInfoFile>$(ProjectDir)Properties\AssemblyInfo.cs</AssemblyInfoFile>
        </PropertyGroup>
		<ReadLinesFromFile File="$(VersionFile)">
			<Output TaskParameter="Lines" PropertyName="NewVersion" />
		</ReadLinesFromFile>
		<Message Text="Updating AssemblyInfo.cs with version $(NewVersion)" Importance="high" />
		<Exec Command="powershell -ExecutionPolicy Bypass -NoProfile -File &quot;$(ProjectDir)UpdateVersion.ps1&quot; &quot;$(NewVersion)&quot; &quot;$(AssemblyInfoFile)&quot;" />
	</Target>
    
</Project>
```

## 需要 UI 的

```xml title="WinForm"
<Project Sdk="Microsoft.NET.Sdk.WindowsDesktop">

    <PropertyGroup>
        <!-- 多目標 Framework -->
        <TargetFrameworks>net48;net8.0-windows</TargetFrameworks>
        <LangVersion>latest</LangVersion>
        <OutputType>WinExe</OutputType>
        <UseWindowsForms>true</UseWindowsForms>
        <AutoGenerateBindingRedirects>true</AutoGenerateBindingRedirects>
        <GenerateBindingRedirectsOutputType>true</GenerateBindingRedirectsOutputType>
    </PropertyGroup>

</Project>
```

```xml title="WPF (建議最低版本 .NET Framework 4.8)"
<Project Sdk="Microsoft.NET.Sdk.WindowsDesktop">

    <PropertyGroup>
        <!-- 多目標 Framework -->
        <TargetFrameworks>net48;net8.0-windows</TargetFrameworks>
        <LangVersion>latest</LangVersion>
        <OutputType>WinExe</OutputType>
        <UseWPF>true</UseWPF>
        <AutoGenerateBindingRedirects>true</AutoGenerateBindingRedirects>
        <GenerateBindingRedirectsOutputType>true</GenerateBindingRedirectsOutputType>
    </PropertyGroup>

</Project>
```

## **編譯不正常的時候**

1. 清除方案 -> 離開 Visual Studio。
2. 刪除 `.vs` 資料夾。
3. 進入 Visual Studio，重建方案。

---

# 如何選擇 SDK 格式的.csproj 檔?

這個問題其實很多人會卡到 👍
**重點先講結論：**

> **在 VS2022 / VS2026（預覽）中，「新建方案」時並沒有一個叫「SDK 格式 .csproj」的選項可以直接勾選**
> 👉 **是否為 SDK-style，是由「專案範本（Template）」決定的**

## ✅ 一句話判斷法（最重要）

👉 **只要你建立的是「.NET Core / .NET / .NET 6+」的專案**
👉 **產生的一定是 SDK-style `.csproj`**

## 🔍 哪些範本「一定是 SDK-style」

在「**建立新專案**」視窗中，選這些就對了：

### ✔ 現代（SDK-style）

| 專案類型                         | 說明        |
| ---------------------------- | --------- |
| **Console App (.NET)**       | SDK-style |
| **Class Library (.NET)**     | SDK-style |
| **Windows Forms App (.NET)** | SDK-style |
| **WPF App (.NET)**           | SDK-style |
| **ASP.NET Core**             | SDK-style |

🔑 關鍵字是 **「(.NET)」**

## ❌ 哪些是舊式（非 SDK-style）

| 專案類型                                   | 產生結果       |
| -------------------------------------- | ---------- |
| Console App **(.NET Framework)**       | ❌ 舊 csproj |
| Windows Forms App **(.NET Framework)** | ❌ 舊 csproj |
| WPF App **(.NET Framework)**           | ❌ 舊 csproj |

這些會產生：

```xml
<Project ToolsVersion="15.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
```

## 🧪 如何快速確認是不是 SDK-style？

打開 `.csproj`，看第一行：

### ✅ SDK-style

```xml
<Project Sdk="Microsoft.NET.Sdk">
```

或 WinForms：

```xml
<Project Sdk="Microsoft.NET.Sdk.WindowsDesktop">
```

### ❌ 舊格式

```xml
<Project ToolsVersion="4.0" xmlns="http://schemas.microsoft.com/developer/msbuild/2003">
```

## 🛠 VS 中正確建立 SDK-style 的「實際步驟」

1. **檔案 → 新增 → 專案**
2. 搜尋你要的類型（例如：WinForms）
3. **選「Windows Forms App (.NET)」**

   * ❌ 不要選 `.NET Framework`
4. 下一步
5. Framework 選：

   * `.NET 6`
   * `.NET 7`
   * `.NET 8`
6. 建立完成 ✅

## 🔄 已有舊專案，能不能轉成 SDK-style？

✔ **可以，但不是「點一下」就好**
✔ WinForms / WPF / Class Library 都可轉
⚠ 大型專案要小心

如果你需要，我可以：

* 幫你 **舊 `.csproj` → SDK-style** 範例
* 或幫你判斷 **你目前專案適不適合轉**

## 🧭 總整理（記住這個就好）

> **選「(.NET)」 = SDK-style**
> **選「(.NET Framework)」 = 舊格式**

---

# .NET Framework 的 Class Library 範本

## ✅ 結論先講（重點）

> **Visual Studio 內建範本中，`.NET Framework` 的 Class Library「預設都是舊式（非 SDK-style）」**

也就是說：

* ✔ `.NET Framework 4.x` **可以**用 SDK-style
* ❌ **但 VS「新建專案」時不會直接給你 SDK-style 的 `.NET Framework Class Library`**

## 🧠 為什麼會這樣？

### 1️⃣ SDK-style 是為 **.NET Core / .NET (5+)** 設計的主流格式

* 微軟的方向是：
  * **新專案 → .NET**
  * **舊系統相容 → .NET Framework**
* 所以 VS 範本刻意不「鼓勵」用 SDK-style + .NET Framework

### 2️⃣ 技術上「支援」，但不是預設

實際上 **MSBuild 早就支援**：

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net48</TargetFramework>
  </PropertyGroup>
</Project>
```

👉 **完全合法、能編、能跑**

## ✅ 正確做法一（最推薦）

### 🎯 用 SDK-style 建立「.NET Framework Class Library」

#### 步驟

1. **新建專案**
2. 選：

   > **Class Library (.NET)**  ← 重點！
3. 建立後
4. 打開 `.csproj`
5. 把：

```xml
<TargetFramework>net8.0</TargetFramework>
```

改成：

```xml
<TargetFramework>net462</TargetFramework>
```

✔ 專案立刻變成：

* SDK-style
* .NET Framework 4.8
* 可被舊 WinForms / WPF 使用

## ✅ 正確做法二（從舊專案轉）

如果你已經有：

* `Class Library (.NET Framework)`（舊 csproj）

可以 **手動轉成 SDK-style**

### 最小可用範例

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net462</TargetFramework>
  </PropertyGroup>
</Project>
```

然後：

* 刪掉：
  * `<Compile Include=...>`
  * `<Reference Include=...>`（GAC 的除外）
* NuGet 改用 `<PackageReference>`

⚠ **大型專案要小心測**

## ❌ 哪些情況「不適合」SDK-style + .NET Framework？

| 情況              | 建議     |
| --------------- | ------ |
| 大量 COM、老第三方 DLL | 保留舊格式  |
| WebForms        | 不建議轉   |
| 超舊 Build Script | 不建議轉   |
| 新共用 Library     | ✔ 非常適合 |

## 🧭 判斷準則（很實用）

> 🔹 **新寫的 Library → SDK-style + net48 / netstandard**
> 🔹 **維護中的老專案 → 不動最安全**

## 📌 總整理（記住這句）

> **.NET Framework「不是不能用 SDK-style」
> 而是「VS 不主動給你」**

---

# 共用 DLL/WINFORM/.NET462/新專案

很好，這個條件其實**非常明確，也是一個「很適合用 SDK-style」的組合** 👍
我直接幫你下結論，順便給你「**最穩定、最少踩雷**」的做法。

## ✅ 結論（先看這段就好）

**你的條件：**

* ✔ 共用 DLL
* ✔ WinForms 使用
* ✔ .NET Framework **4.6.2**
* ✔ 新專案

👉 **強烈建議：使用「SDK-style `.csproj`」**

這個組合在 VS2022 / VS2026 都是 **穩定可用、官方支援、實務常見**。

## 🛠 最推薦建立方式（一步到位）

### 🎯 方法一（最乾淨、最少問題）【推薦】

1. **新建專案**
2. 選擇：

   > **Class Library (.NET)**  ← 一定要這個
3. 下一步
4. Framework 先隨便選（例如 .NET 8）
5. 建立完成
6. 打開 `.csproj`，改成這樣👇

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net462</TargetFramework>
  </PropertyGroup>
</Project>
```

✅ 完成後你得到的是：

* SDK-style 專案
* .NET Framework 4.6.2
* 可被 WinForms (.NET 4.6.2) 直接引用

## 🔧 WinForms 共用 DLL 常用設定（建議加）

### 📌 建議加這些屬性（穩定又實用）

```xml
<PropertyGroup>
  <TargetFramework>net462</TargetFramework>
  <LangVersion>latest</LangVersion>
  <Nullable>disable</Nullable>
</PropertyGroup>
```

（`.NET Framework` 專案通常不開 Nullable，避免大量警告）

## 📦 NuGet / DLL 參考（SDK-style 正確方式）

### ✔ NuGet（推薦）

```xml
<ItemGroup>
  <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
</ItemGroup>
```

### ✔ 引用其他 DLL

```xml
<ItemGroup>
  <Reference Include="SomeLegacyDll">
    <HintPath>libs\SomeLegacyDll.dll</HintPath>
  </Reference>
</ItemGroup>
```

## ❌ 不要做的事（很重要）

| 行為                                 | 原因             |
| ---------------------------------- | -------------- |
| 建 `Class Library (.NET Framework)` | 會是舊 csproj     |
| 混用 `packages.config`               | SDK-style 不建議  |
| 手動列出 `<Compile Include>`           | SDK-style 會自動抓 |
| TargetFramework 設太低                | net462 已是安全下限  |

---

## 🧪 如何確認你是 SDK-style？

打開 `.csproj`，第一行應該是：

```xml
<Project Sdk="Microsoft.NET.Sdk">
```

不是 `ToolsVersion`。

## 🧭 延伸建議（實務經驗）

如果這個 DLL **只給 WinForms 用**：

✔ `net462` 很適合
如果未來可能給：

* Console
* Service
* 其他工具

👉 可以考慮：

```xml
<TargetFrameworks>net462;net48</TargetFrameworks>
```

（SDK-style 才能這樣玩）

## 🎯 最後一句總結

> **「共用 DLL + WinForms + .NET 4.6.2 + 新專案」
> 👉 SDK-style 是最佳解**

---

# 含 UI 的 LIB 要選那個?

我直接依你的條件（**WinForms / .NET Framework 4.6.2 / 新專案 / SDK-style**）給你**明確答案**。

## ✅ 一句話結論（先記這個）

> **「含 UI 的 WinForms 共用 DLL」
> 👉 選 `Class Library (.NET)`，然後在 `.csproj` 開啟 WinForms**

**不是**選一般的 WinForms App。

## 🏆 正確選擇（最推薦）

### 🎯 建立方式

1. **新建專案**
2. 選：

   > **Class Library (.NET)**  ← 一定是這個
3. 建立後，修改 `.csproj`：

```xml
<Project Sdk="Microsoft.NET.Sdk.WindowsDesktop">

  <PropertyGroup>
    <TargetFramework>net462</TargetFramework>
    <UseWindowsForms>true</UseWindowsForms>
  </PropertyGroup>

</Project>
```

✅ 這樣你就得到：

* ✔ SDK-style
* ✔ .NET Framework 4.6.2
* ✔ 可放 `Form / UserControl`
* ✔ 可被 WinForms 主程式引用
* ✔ **真正的「含 UI 共用 DLL」**

## 🧩 為什麼不是選 WinForms App？

| 類型                             | 問題               |
| ------------------------------ | ---------------- |
| WinForms App (.NET Framework)  | ❌ 會產生 exe        |
| WinForms App (.NET)            | ❌ 目的是應用程式，不是 DLL |
| Class Library (.NET Framework) | ❌ 舊 csproj       |
| **Class Library (.NET)**       | ✅ 正確             |

👉 **UI ≠ App**
👉 **共用 UI 元件 = Library**

## 🧱 你可以在這個 DLL 放什麼？

✔ `Form`（對話框、子視窗）
✔ `UserControl`
✔ 共用 UI 邏輯
✔ 自訂控制項

### 範例

```csharp
public class MyDialog : Form
{
    public MyDialog()
    {
        InitializeComponent();
    }
}
```

主程式：

```csharp
var dlg = new MyDialog();
dlg.ShowDialog();
```

## 🛠 常見補充設定（建議）

```xml
<PropertyGroup>
  <TargetFramework>net462</TargetFramework>
  <UseWindowsForms>true</UseWindowsForms>
  <LangVersion>latest</LangVersion>
  <Nullable>disable</Nullable>
</PropertyGroup>
```

## ⚠ 常見踩雷提醒（重要）

### ❌ 忘了 `<UseWindowsForms>true</UseWindowsForms>`

➡️ 會出現：

* 找不到 `Form`
* 找不到 `System.Windows.Forms`

### ❌ 用 `Microsoft.NET.Sdk`

➡️ WinForms 設計器可能不能用

👉 **一定要用：**

```xml
Sdk="Microsoft.NET.Sdk.WindowsDesktop"
```

## 🧭 延伸建議（架構層面）

實務上最穩定的拆法是：

```csharp
MyApp.UI        → WinForms EXE
MyApp.UI.Shared → 含 Form / UserControl 的 DLL（你現在這個）
MyApp.Core      → 純邏輯 DLL（不含 UI）
```

## ✅ 最後總結（記住這段）

> **含 UI 的 WinForms Library =
> `Class Library (.NET)` +
> `Microsoft.NET.Sdk.WindowsDesktop` +
> `<UseWindowsForms>true</UseWindowsForms>` +
> `net462`**

---

# 「可直接用的 WinForms UI 共用 DLL（SDK-style / .NET 4.6.2）」標準範本

## ✅ WinForms UI Library（SDK-style）範本

### 📄 `MyCompany.MyApp.UI.Shared.csproj`

```xml
<Project Sdk="Microsoft.NET.Sdk.WindowsDesktop">

  <PropertyGroup>
    <TargetFramework>net462</TargetFramework>
    <UseWindowsForms>true</UseWindowsForms>

    <!-- 建議設定 -->
    <LangVersion>latest</LangVersion>
    <Nullable>disable</Nullable>
    <GenerateAssemblyInfo>true</GenerateAssemblyInfo>
  </PropertyGroup>

</Project>
```

## 🧱 專案裡你可以放什麼？

### ✔ UI 類型

* `Form`
* `UserControl`
* 自訂控制項（繼承 `Control`）

### ✔ 共用 UI 工具

* 樣式（字型 / 顏色）
* UI Helper
* 共用對話框

## 🛠 新增 UI 檔案的正確方式（很重要）

在專案上：

```csharp
右鍵 → 新增 → 新項目
```

然後選：

* **Windows Form**
* **User Control**

👉 VS 會自動幫你產 `.Designer.cs`
👉 設計器可正常使用

## 🔗 主 WinForms 專案如何引用？

### 在主專案

```csharp
右鍵 → 參考 → 新增專案參考
```

選你的 UI DLL 專案即可。

## ⚠ 常見錯誤快速檢查表

| 問題       | 檢查                                        |
| -------- | ----------------------------------------- |
| 找不到 Form | `<UseWindowsForms>true</UseWindowsForms>` |
| 設計器不能開   | `Microsoft.NET.Sdk.WindowsDesktop`        |
| 不能編譯     | 主程式 TargetFramework 必須 ≥ net462           |
| 參考失敗     | DLL 與主專案 Framework 不一致                    |

## 🧭 建議的專案結構（實務）

```csharp
Solution
 ├─ MyApp.WinForms        (EXE)
 ├─ MyApp.UI.Shared      (UI DLL ← 你現在這個)
 └─ MyApp.Core           (純邏輯 DLL)
```

---

# Deterministic

在 SDK-style 專案裡，`<Deterministic>` 是 MSBuild 的一個屬性，用來控制 **可重現編譯（Deterministic Builds）**。意思是：

- 當 `Deterministic` 設為 `true` 時，編譯相同原始碼、相同編譯器版本、相同依賴項，生成的 **DLL/EXE 的二進位檔內容會完全一樣**（除了檔案系統時間等少數例外）。
- 設為 `false`（預設值）時，每次編譯即使程式碼沒改，也可能生成不同的二進位檔（例如時間戳不同）。

## 用途

1. **版本控制友好**：確保 CI/CD 生成的檔案與本地編譯一致。
2. **NuGet 發佈**：Deterministic Builds 能保證相同版本的套件內容一致。
3. **安全/驗證**：方便做 checksum 或簽章驗證。

## 範例（SDK-style）

```xml
<PropertyGroup>
    <TargetFramework>net472</TargetFramework>
    <Deterministic>true</Deterministic>
</PropertyGroup>
```

## 注意

- 對 .NET Framework 專案可用，但對某些舊式工具鏈（如 VS2015/2017 非 SDK-style）可能不支援。
- 對 exe 專案尤其有用，如果想保證重現性，建議保留 `true`。

---

# ImplicitUsings

在 SDK-style 專案裡，`<ImplicitUsings>` 是 .NET 6+ 引入的新功能，目的是 **自動引入常用的 `using` 指令**，讓程式碼更簡潔，不需要每個檔案都手動寫 `using System;` 或 `using System.Collections.Generic;` 之類的。

## 說明

- `<ImplicitUsings>enable</ImplicitUsings>`
    啟用後，專案會自動生成一個隱含的檔案（類似 `_Imports.razor` 或自動 usings）來引入常用命名空間。

- `<ImplicitUsings>disable</ImplicitUsings>`
    不自動加入，需手動在每個檔案加入 `using`。

## 支援情況

- 主要針對 **.NET 6+ / SDK-style 專案**
- 對 **.NET Framework 專案 (例如 net472)** 其實沒有作用，SDK 會忽略這個設定。

## 範例

```xml
<PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
</PropertyGroup>
```

啟用後，你的程式碼可以省略很多常用 `using`，例如：

```csharp
// 不需要手動 using System;
// 不需要手動 using System.Collections.Generic;
Console.WriteLine("Hello World");
```

如果你還在用 **.NET Framework**，`<ImplicitUsings>` 寫了也不會有影響。
只有改成 **.NET 6+** 才會真的幫你自動引用命名空間。
