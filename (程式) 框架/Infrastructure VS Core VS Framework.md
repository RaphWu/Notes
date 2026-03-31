---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
---

# Infrastructure VS Core VS Framework

## 三者的定位總覽

- Core
    放「業務核心與抽象」，不依賴任何外部技術
    
- Infrastructure
    放「技術實作與外部系統」，依賴 Core
    
- Framework
    放「可重用的技術框架或基礎能力」，通常與業務無關

## Core

- 主要內容
    Domain Model
    Business Rules
    Use Case / Application Service
    Interfaces（Repository、Service Contract）
    
- 特性
    不依賴資料庫、UI、第三方套件
    可獨立測試
    專案中最穩定、變動最少
    
- 命名常見
    *.Core
    *.Domain
    *.Application

## Infrastructure

- 主要內容
    Repository 實作
    ORM（EF、Dapper）
    File、SerialPort、PLC、Camera SDK
    Logging、Cache、Message Queue
    
- 特性
    強烈依賴外部技術與環境
    可替換、可重構
    只能依賴 Core，不可反向依賴
    
- 命名常見
    *.Infrastructure
    *.Persistence
    *.Adapters

## Framework

- 主要內容
    通用基礎設施
    技術封裝（DI、Logging、Configuration、Exception Handling）
    跨專案可共用套件
    
- 特性
    與單一業務無關
    可 NuGet 化
    可能被 Core 與 Infrastructure 同時使用
    
- 命名常見
    *.Framework
    *.Common
    *.Shared
    *.Abstractions

## 依賴關係示意（邏輯）

- Framework
    ↑
- Infrastructure
    ↑
- Core
- Core 不依賴任何一層
- Infrastructure 依賴 Core（與 Framework）
- Framework 不應依賴業務 Core

## 什麼時候該放哪一層

- 判斷問題一
    「拿掉資料庫或硬體，業務邏輯是否仍成立？」
    是 → Core
    否 → Infrastructure
    
- 判斷問題二
    「是否能被多個不同專案重用？」
    是 → Framework
    否 → Core 或 Infrastructure
    
- 判斷問題三
    「是否描述『做什麼』而非『怎麼做』？」
    做什麼 → Core
    怎麼做 → Infrastructure

## 製造業／設備軟體常見實例對照

- Core
    ScrewFasteningResult
    TorqueSpec
    LensAssemblyRule
    
- Infrastructure
    ScrewdriverController（對接鎖付機）
    Vision SDK Adapter
    PLC Communication
    
- Framework
    Logging Module
    Autofac Module Base
    Task / Retry / Timeout Helper

## 常見誤用提醒

- 把 EF Entity 放在 Core
    實際上是 Infrastructure Concern
    
- 把硬體 SDK Interface 直接寫在 Core
    應該只放抽象介面
    
- 把 Framework 當成「共用垃圾桶」
    Framework 應該是有設計邊界的技術基礎

## 實務命名建議（.NET 專案）

- Xxx.Core
- Xxx.Infrastructure
- Xxx.Framework（或 Xxx.Common）
- Xxx.Presentation（WinForm／WPF／Web）

如果你願意，我可以用你目前的 WinForm＋Autofac＋設備控制架構，幫你畫一個實際分層對照表。
