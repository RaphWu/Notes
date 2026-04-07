| 情境                 | 是否使用 NullLoggerFactory | 正確做法                             |
| -------------------- | -------------------------- | ------------------------------------ |
| NuGet Library/Driver | ❌ 否                      | 建構子注入 ILogger<T>，不提供預設值  |
| App Composition Root | ✅ 可以（作為容錯）        | 註冊 ILoggerFactory，讓 DI 自動處理  |
| 靜態工具類別         | ❌ 否                      | 改為非靜態類別並注入                 |
| 舊版非 DI 專案       | ⚠️ 有限度                  | 提供 Initialize(ILoggerFactory) 方法 |
