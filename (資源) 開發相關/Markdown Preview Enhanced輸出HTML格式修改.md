# 寬度變更

寬度控制的關鍵在這段 CSS（壓縮在一行的 `<style>` 裡）：

```css
@media screen and (min-width:914px){
  html body[for=html-export]:not([data-presentation-mode]) .markdown-preview{
    padding:2em calc(50% - 457px + 2em)
  }
}
```

`calc(50% - 457px + 2em)` 的意思是：用左右 padding 把內容夾住，固定在 **914px**（457 × 2）寬度居中。

**要讓它隨螢幕縮放，有兩種方式：**

### 方式一：加大固定寬度（最簡單）

把 `457px` 改大，例如改成 `600px`（最大寬度 1200px）或 `640px`（最大寬度 1280px）。

### 方式二：改用百分比 padding（完全流體）

把該段改成：

```css
@media screen and (min-width:914px){
  html body[for=html-export]:not([data-presentation-mode]) .markdown-preview{
    padding:2em 5%
  }
}
```

這樣左右各保留 5% 的邊距，內容會隨螢幕寬度自動縮放。

---

因為這段 CSS 是 Markdown Preview Enhanced 匯出時自動生成的壓縮碼，你可以在 `</style>` 標籤之前新增一段覆蓋樣式來修改，不需要改動壓縮碼：

```css
@media screen and (min-width:914px){
  html body[for=html-export]:not([data-presentation-mode]) .markdown-preview{
    padding: 2em 5% !important;
  }
}
```

---

# MERMAID無法顯示

已搜尋文字 `mermaid`，20 個結果

問題原因很明確：第 14-16 行的 Mermaid JS 是從本機 路徑載入的：

```html
src="file:///c:\Users\rapha\.vscode\extensions\shd101wyy.markdown-preview-enhanced-0.8.22\crossnote\dependencies\mermaid\mermaid.min.js"
```

**當這個 HTML 被嵌在 index.html 的 `<iframe>` 裡顯示時，瀏覽器的安全政策會阻擋跨來源的 資源載入，導致 Mermaid JS 無法執行。**

---

**解決方式：將 路徑改為 CDN 連結**

把這段：

```html
<script
  type="text/javascript"
  src="file:///c:\Users\rapha\.vscode\extensions\shd101wyy.markdown-preview-enhanced-0.8.22\crossnote\dependencies\mermaid\mermaid.min.js"
  charset="UTF-8"
></script>
```

改為：

```html
<script src="https://cdn.jsdelivr.net/npm/mermaid@10/dist/mermaid.min.js"></script>
```

---

> **注意**：如果未來還有其他 Markdown Preview Enhanced 匯出的 HTML 也有同樣問題，每個檔案都需要做相同替換。KaTeX CSS 也有同樣的 問題（第 10 行），建議一併改為 CDN：
>
> ````html
> <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16/dist/katex.min.css">
> ```> <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.16/dist/katex.min.css">
> ````
