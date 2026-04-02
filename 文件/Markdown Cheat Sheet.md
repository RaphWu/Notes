---
aliases:
date:
update:
author:
language:
sourceurl:
tags:
  - Markdown
---

# Markdown Cheat Sheet

參考來源：

- [Markdown Guide](https://www.markdownguide.org/)
	- [Basic Syntax](https://www.markdownguide.org/basic-syntax/)
	- [Extended Syntax](https://www.markdownguide.org/extended-syntax/)
	- [Hacks](https://www.markdownguide.org/hacks/)
- [CommonMark Spec](https://spec.commonmark.org/)
- [Markdown Preview Enhanced](https://github.com/shd101wyy/vscode-markdown-preview-enhanced)

# Basic Syntax

## Overview  概述

Nearly all Markdown applications support the basic syntax outlined in the original Markdown design document. There are minor variations and discrepancies between Markdown processors — those are noted inline wherever possible.
幾乎所有 Markdown 應用程式都支援 Markdown 原始設計文件中概述的基本語法。不同 Markdown 處理器之間存在一些細微差別和差異——這些差別和差異會在可能的情況下以程式碼內文的形式註明。

## Headings  標題

To create a heading, add number signs (`#`) in front of a word or phrase. The number of number signs you use should correspond to the heading level. For example, to create a heading level three (`<h3>`), use three number signs (e.g., `### My Header`).
若要建立標題，請在單字或短語前面加上井號 ( `#` )。使用的井號數量應與標題等級相對應。例如，要建立三級標題 ( `<h3>` )，請使用三個井號（例如， `### My Header` ）。

# Heading level 1

| Markdown            | HTML                       |
| ------------------- | -------------------------- |
| `# Heading level 1` | `<h1>Heading level 1</h1>` |

## Heading level 2

| Markdown             | HTML                       |
| -------------------- | -------------------------- |
| `## Heading level 2` | `<h2>Heading level 2</h2>` |

### Heading level 3

| Markdown              | HTML                       |
| --------------------- | -------------------------- |
| `### Heading level 3` | `<h3>Heading level 3</h3>` |

#### Heading level 4

| Markdown               | HTML                       |
| ---------------------- | -------------------------- |
| `#### Heading level 4` | `<h4>Heading level 4</h4>` |

##### Heading level 5

| Markdown                | HTML                       |
| ----------------------- | -------------------------- |
| `##### Heading level 5` | `<h5>Heading level 5</h5>` |

###### Heading level 6

| Markdown                 | HTML                       |
| ------------------------ | -------------------------- |
| `###### Heading level 6` | `<h6>Heading level 6</h6>` |

## Alternate Syntax 替代語法

Alternatively, on the line below the text, add any number of `==` characters for heading level 1 or `--` characters for heading level 2.
或者，在文字下方的一行中，添加任意數量的 `==` 字元表示一級標題，添加任意數量的 `--` 字元表示二級標題。

H1
==========

| Markdown                                 | HTML                       |
| ---------------------------------------- | -------------------------- |
| `Heading level 1`<br>`=‌=‌=‌=‌=‌=‌=‌=‌=` | `<h1>Heading level 1</h1>` |

H2
----------

| Markdown                               | HTML                       |
| -------------------------------------- | -------------------------- |
| `Heading level 2`<br>`---------------` | `<h2>Heading level 2</h2>` |

## Heading Best Practices  標題 最佳實踐 [](https://www.markdownguide.org/basic-syntax/#heading-best-practices)

Markdown applications don’t agree on how to handle a missing space between the number signs (`#`) and the heading name. For compatibility, always put a space between the number signs and the heading name.
Markdown 應用程式對於在數字符號 (`#`) 與標題名稱之間是否應該有空白空間的處理方式並不一致。為了兼容性，始終在數字符號與標題名稱之間放置一個空白。

---

# Paragraphs 段落

要建立段落，請使用一個空白行來分開一或多行文字。

I really like using Markdown.

I think I'll use it to format all of my documents from now on.

## Paragraph Best Practices 段落最佳實踐

Unless the [paragraph is in a list](#paragraphs), don’t indent paragraphs with spaces or tabs.
除非段落在清單中，否則不要使用空格或制表符縮進段落。

>  **Note:** If you need to indent paragraphs in the output, see the section on how to [indent (tab)](#indent-tab).
> 注意：如果您需要在輸出中縮進段落，請參考如何縮進（制表符）的相關部分。

---

# Line Breaks 換行

若要建立換行符或新行（`\<br\>`），請在行尾新增兩個或更多空格，然後按回車鍵。
This is the first line.`<br\>`And this is the second line.

| Markdown                                                                                                                                              | HTML                                                                                                                                           |
| ----------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| `This is the first line.`<ins>　</ins> <ins>　</ins><br>`And this is the second line.`<br><br>This is the first line.  <br>And this is the second line. | \<p\>This is the first line.\<br\><br>And this is the second line.\</p\><br><br><p>This is the first line.<br>And this is the second line.</p> |

## Line Break Best Practices 斷行最佳實踐

You can use two or more spaces (commonly referred to as “trailing whitespace”) for line breaks in nearly every Markdown application, but it’s controversial. It’s hard to see trailing whitespace in an editor, and many people accidentally or intentionally put two spaces after every sentence. For this reason, you may want to use something other than trailing whitespace for line breaks. If your Markdown application [supports HTML](https://www.markdownguide.org/basic-syntax/#html), you can use the `<br>` HTML tag.
您可以在幾乎所有的 Markdown 應用程序中使用兩個或更多空格（常稱為“尾隨空白”）來進行斷行，但這是具有爭議性的。在編輯器中難以看到尾隨空白，許多人不經意或故意在每個句子後面加上兩個空格。因此，您可能希望使用除尾隨空白以外的其他方法來進行斷行。如果您的 Markdown 應用程序支援 HTML，您可以使用 `<br>` HTML 標籤。

For compatibility, use trailing white space or the `<br>` HTML tag at the end of the line.
為了兼容性，您可以在行的最後使用尾隨空白或 `<br>` HTML 標籤。

There are two other options I don’t recommend using. CommonMark and a few other lightweight markup languages let you type a backslash (`\`) at the end of the line, but not all Markdown applications support this, so it isn’t a great option from a compatibility perspective. And at least a couple lightweight markup languages don’t require anything at the end of the line — just type return and they’ll create a line break.
還有兩種我不推薦使用的選項。CommonMark 和一些其他輕量級標記語言允許您在行的最後打上反斜杠（ `\` ），但並非所有的 Markdown 應用程序都支援這一功能，所以從兼容性的角度來看，這不是一個很好的選項。而且至少有兩種輕量級標記語言不需要在行的最後加上任何內容——只需按回車鍵，它們就會創建斷行。

---

# Emphasis  強調

You can add emphasis by making text bold or italic.
你可以透過將文字加粗或斜體來增加強調。

## Bold 粗體

To bold text, add two asterisks or underscores before and after a word or phrase. To bold the middle of a word for emphasis, add two asterisks without spaces around the letters.
要加粗文字，可以在單詞或短語前後加上兩個星號或下劃線。若要加強單詞中間的文字，則在不加空格的情況下，在字母兩旁加上兩個星號。

| Markdown                                                   | HTML                                                                                 |
| ---------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| `I just love **bold text**.`<br>I just love **bold text**. | `I just love <strong>bold text</strong>.`<br>I just love <strong>bold text</strong>. |
| `I just love __bold text__.`<br>I just love __bold text__. | `I just love <strong>bold text</strong>.`<br>I just love <strong>bold text</strong>. |
| `Love**is**bold`<br>Love**is**bold                         | `Love<strong>is</strong>bold`<br>Love<strong>is</strong>bold                         |

### Bold Best Practices  粗體最佳實踐 [](https://www.markdownguide.org/basic-syntax/#bold-best-practices)

Markdown applications don’t agree on how to handle underscores in the middle of a word. For compatibility, use asterisks to bold the middle of a word for emphasis.
Markdown 應用程式對於單詞中間的底線處理方式並不一致。為了兼容性，使用星號來加強單詞中間的底線以達到強調的效果。

## Italic 斜體

To italicize text, add one asterisk or underscore before and after a word or phrase. To italicize the middle of a word for emphasis, add one asterisk without spaces around the letters.
為了斜體文字，在單詞或短語前後加上一個星號或底線。若要為單詞中間加強調，不加空格地在一個星號的周圍加上字母。

| Markdown                                                                       | HTML                                                                                         |
| ------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------- |
| `Italicized text is the *cat's meow*.`<br>Italicized text is the *cat's meow*. | `Italicized text is the <em>cat's meow</em>.`<br>Italicized text is the <em>cat's meow</em>. |
| `Italicized text is the _cat's meow_.`<br>Italicized text is the _cat's meow_. | `Italicized text is the <em>cat's meow</em>.`<br>Italicized text is the <em>cat's meow</em>. |
| `A*cat*meow`<br>A*cat*meow<br>                                                 | `A<em>cat</em>meow`<br>A<em>cat</em>meow                                                     |
| `A_cat_meow`<br>A_cat_meow                                                     |                                                                                              |

### Italic Best Practices  斜體最佳實踐

Markdown applications don’t agree on how to handle underscores in the middle of a word. For compatibility, use asterisks to italicize the middle of a word for emphasis.
Markdown 應用程式對於單詞中間的底線處理方式並不一致。為了兼容性，請使用星號來斜體化單詞中間的部分以強調。

## Bold and Italic  粗體與斜體

To emphasize text with bold and italics at the same time, add three asterisks or underscores before and after a word or phrase. To bold and italicize the middle of a word for emphasis, add three asterisks without spaces around the letters.
要同時使用粗體和斜體來強調文字，可以在單詞或短語的前後加上三個星號或下劃線。若要強調單詞中的中間部分，則在字母周圍不加空格的情況下加上三個星號。

| Markdown                                                                             | HTML                                                                                                                         |
| ------------------------------------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------- |
| `This text is ***really important***.`<br>This text is ***really important***.       | `This text is <em><strong>really important</strong></em>.`<br>This text is <em><strong>really important</strong></em>.       |
| `This text is ___really important___.`<br>This text is ___really important___.       | `This text is <em><strong>really important</strong></em>.`<br>This text is <em><strong>really important</strong></em>.       |
| `This text is __*really important*__.`<br>This text is __*really important*__.       | `This text is <em><strong>really important</strong></em>.`<br>This text is <em><strong>really important</strong></em>.       |
| `This text is **_really important_**.`<br>This text is **_really important_**.       | `This text is <em><strong>really important</strong></em>.`<br>This text is <em><strong>really important</strong></em>.       |
| `This is really***very***important text.`<br>This is really***very***important text. | `This is really<em><strong>very</strong></em>important text.`<br>This is really<em><strong>very</strong></em>important text. |

> **Note:** The order of the `em` and `strong` tags might be reversed depending on the Markdown processor you're using.
> 注意： `em` 和 `strong` 標籤的順序可能會根據您使用的 Markdown 处理器而有所不同。

### Bold and Italic Best Practices 粗體與斜體最佳實踐

Markdown applications don’t agree on how to handle underscores in the middle of a word. For compatibility, use asterisks to bold and italicize the middle of a word for emphasis.
Markdown 應用程式對於單詞中間的底線處理方式並不一致。為了兼容性，使用星號來粗體和斜體單詞中間的部分以強調。

---

# Blockquote 區塊引用

To create a blockquote, add a `>` in front of a paragraph.
要建立一個引文，請在段落前加上一個 `>` 。

```plantext
> Dorothy followed her through many of the beautiful rooms in her castle.
```

The rendered output looks like this:
渲染後的輸出看起來像這樣：

> Dorothy followed her through many of the beautiful rooms in her castle.

## Blockquotes with Multiple Paragraphs 引用多段落

Blockquotes can contain multiple paragraphs. Add a `>` on the blank lines between the paragraphs.
引用可以包含多個段落。在段落之間的空白行加上 `>` 。

```plantext
> Dorothy followed her through many of the beautiful rooms in her castle.
>
> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.
```

The rendered output looks like this:
渲染後的輸出看起來像這樣：

> Dorothy followed her through many of the beautiful rooms in her castle.
>
> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.

## Nested Blockquotes  嵌套引用

Blockquotes can be nested. Add a `>>` in front of the paragraph you want to nest.
引用可以嵌套。在您想要嵌套的段落前加上 `>>` 。

```plantext
> Dorothy followed her through many of the beautiful rooms in her castle.
>
>> The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.
```

The rendered output looks like this:
渲染後的輸出看起來像這樣：

> Dorothy followed her through many of the beautiful rooms in her castle.
>
> > The Witch bade her clean the pots and kettles and sweep the floor and keep the fire fed with wood.

## Blockquotes with Other Elements 區塊引言與其他元素

Blockquotes can contain other Markdown formatted elements. Not all elements can be used — you’ll need to experiment to see which ones work.
引號內可以包含其他 Markdown 格式的元素。不是所有元素都可以使用——您需要進行實驗以查看哪些元素可以正常使用。

```plantext
> #### The quarterly results look great!
>
> - Revenue was off the chart.
> - Profits were higher than ever.
>
>  *Everything* is going according to **plan**.
```

The rendered output looks like this:
渲染後的輸出看起來像這樣：

> #### The quarterly results look great!
>
> - Revenue was off the chart.
> - Profits were higher than ever.
> 
> _Everything_ is going according to **plan**.

## Blockquotes Best Practices 引用區塊最佳實踐

For compatibility, put blank lines before and after blockquotes.
為了兼容性，在引文前后加上空白行。

## Blockquotes with Other Elements 與其他元素的區塊引用

> #### The quarterly results look great!
>
> - Revenue was off the chart.
> - Profits were higher than ever.
>
>   _Everything_ is going according to **plan**.

---

# Lists  清單

You can organize items into ordered and unordered lists.
您可以將項目組織成有序和無序清單。

## Ordered Lists  有序清單

To create an ordered list, add line items with numbers followed by periods. The numbers don’t have to be in numerical order, but the list should start with the number one.
要建立有序清單，請加入帶有數字和句點的行項。數字不必按數字順序排列，但清單應從數字一開始。

| Markdown                                                                                                                                             | HTML                                                                                                                                                                                                                                                                       |
| ---------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `1. First item`<br>`2. Second item`<br>`3. Third item`<br>`4. Fourth item`<br><br>1. First item<br>2. Second item<br>3. Third item<br>4. Fourth item | `<ol>`<br>`    <li>First item</li>`<br>`    <li>Second item</li>`<br>`    <li>Third item</li>`<br>`    <li>Fourth item</li>`<br>`</ol>`<br><br><ol><br>    <li>First item</li><br>    <li>Second item</li><br>    <li>Third item</li><br>    <li>Fourth item</li><br></ol> |
1. First item
2. Second item
3. Third item
4. Fourth item

<ol>
    <li>First item</li>
    <li>Second item</li>
    <li>Third item</li>
    <li>Fourth item</li>
</ol>

| Markdown                                                                                                                                             | HTML                                                                                                                                                                                                                                                                       |
| ---------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `1. First item`<br>`1. Second item`<br>`1. Third item`<br>`1. Fourth item`<br><br>1. First item<br>1. Second item<br>1. Third item<br>1. Fourth item | `<ol>`<br>`    <li>First item</li>`<br>`    <li>Second item</li>`<br>`    <li>Third item</li>`<br>`    <li>Fourth item</li>`<br>`</ol>`<br><br><ol><br>    <li>First item</li><br>    <li>Second item</li><br>    <li>Third item</li><br>    <li>Fourth item</li><br></ol> |
1. First item
2. Second item
3. Third item
4. Fourth item

<ol>
    <li>First item</li>
    <li>Second item</li>
    <li>Third item</li>
    <li>Fourth item</li>
</ol>

| Markdown                                                                                                                                             | HTML                                                                                                                                                                                                                                                                       |
| ---------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `1. First item`<br>`8. Second item`<br>`3. Third item`<br>`5. Fourth item`<br><br>1. First item<br>8. Second item<br>3. Third item<br>5. Fourth item | `<ol>`<br>`    <li>First item</li>`<br>`    <li>Second item</li>`<br>`    <li>Third item</li>`<br>`    <li>Fourth item</li>`<br>`</ol>`<br><br><ol><br>    <li>First item</li><br>    <li>Second item</li><br>    <li>Third item</li><br>    <li>Fourth item</li><br></ol> |
1. First item
2. Second item
3. Third item
4. Fourth item

<ol>
    <li>First item</li>
    <li>Second item</li>
    <li>Third item</li>
    <li>Fourth item</li>
</ol>

| Markdown                                                                                                                                                                                                                                                 | HTML                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `1. First item`<br>`2. Second item`<br>`3. Third item`<br>`    1. Indented item`<br>`    2. Indented item`<br>`4. Fourth item`<br><br>1. First item<br>2. Second item<br>3. Third item<br>    1. Indented item<br>    2. Indented item<br>4. Fourth item | `<ol>`<br>`    <li>First item</li>`<br>`    <li>Second item</li>`<br>`    <li>Third item`<br>`        <ol>`<br>`            <li>Indented item</li>`<br>`            <li>Indented item</li>`<br>`        </ol>`<br>`    </li>`<br>`    <li>Fourth item</li>`<br>`</ol>`<br><br><ol><br>    <li>First item</li><br>    <li>Second item</li><br>    <li>Third item<br>        <ol><br>            <li>Indented item</li><br>            <li>Indented item</li><br>        </ol><br>    </li><br>    <li>Fourth item</li><br></ol> |
1. First item
2. Second item
3. Third item
    1. Indented item
    2. Indented item
4. Fourth item

<ol>
    <li>First item</li>
    <li>Second item</li>
    <li>Third item
        <ol>
            <li>Indented item</li>
            <li>Indented item</li>
        </ol>
    </li>
    <li>Fourth item</li>
</ol>

### Ordered List Best Practices 有序清單最佳實踐

CommonMark and a few other lightweight markup languages let you use a parenthesis (`)`) as a delimiter (e.g., `1) First item`), but not all Markdown applications support this, so it isn’t a great option from a compatibility perspective. For compatibility, use periods only.
CommonMark 以及其他一些輕量級標記語言允許您使用括號（ `)` ）作為分隔符（例如， `1) First item` ），但並非所有 Markdown 應用程序都支持這一功能，因此從兼容性角度來看，這不是一個很好的選擇。為了兼容性，請只使用句點。

## Unordered List 無序清單

To create an unordered list, add dashes (`-`), asterisks (`*`), or plus signs (`+`) in front of line items. Indent one or more items to create a nested list.
要建立無序列表，請在行項前加上破折號（ `-` ）、星號（ `*` ）或加號（ `+` ）。為了建立嵌套列表，請縮進一個或多個項目。

| Markdown                                                                                                                                     | HTML                                                                                                                                                                                                                                                                       |
| -------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `- First item`<br>`- Second item`<br>`- Third item`<br>`- Fourth item`<br><br>- First item<br>- Second item<br>- Third item<br>- Fourth item | `<ul>`<br>`    <li>First item</li>`<br>`    <li>Second item</li>`<br>`    <li>Third item</li>`<br>`    <li>Fourth item</li>`<br>`</ul>`<br><br><ul><br>    <li>First item</li><br>    <li>Second item</li><br>    <li>Third item</li><br>    <li>Fourth item</li><br></ul> |
- First item
- Second item
- Third item
- Fourth item

<ul>
    <li>First item</li>
    <li>Second item</li>
    <li>Third item</li>
    <li>Fourth item</li>
</ul>

| Markdown                                                                                                                                     | HTML                                                                                                                        |
| -------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `* First item`<br>`* Second item`<br>`* Third item`<br>`* Fourth item`<br><br>* First item<br>* Second item<br>* Third item<br>* Fourth item | `<ul>`<br>`	<li>First item</li>`<br>`	<li>Second item</li>`<br>`	<li>Third item</li>`<br>`	<li>Fourth item</li>`<br>`</ul>` |
* First item
* Second item
* Third item
* Fourth item

| Markdown                                                                                                                                     | HTML                                                                                                                        |
| -------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------- |
| `+ First item`<br>`+ Second item`<br>`+ Third item`<br>`+ Fourth item`<br><br>+ First item<br>+ Second item<br>+ Third item<br>+ Fourth item | `<ul>`<br>`	<li>First item</li>`<br>`	<li>Second item</li>`<br>`	<li>Third item</li>`<br>`	<li>Fourth item</li>`<br>`</ul>` |
+ First item
+ Second item
+ Third item
+ Fourth item

| Markdown                                                                                                                                                                                                                                     | HTML                                                                                                                                                                                                                                                                                                                                                                                                                         |
| -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `- First item`<br>`- Second item`<br>`- Third item`<br>`    - Indented item`<br>`    - Indented item`<br>`- Fourth item`<br><br>- First item<br>- Second item<br>- Third item<br>    - Indented item<br>    - Indented item<br>- Fourth item | `<ul>`<br>`	<li>First item</li>`<br>`	<li>Second item</li>`<br>`	<li>Third item`<br>`	<ul>`<br>`		<li>Indented item</li>`<br>`		<li>Indented item</li>`<br>`	</ul>`<br>`	</li>`<br>`	<li>Fourth item</li>`<br>`</ul>`<br><br><ul><br>	<li>First item</li><br>	<li>Second item</li><br>	<li>Third item<br>	<ul><br>		<li>Indented item</li><br>		<li>Indented item</li><br>	</ul><br>	</li><br>	<li>Fourth item</li><br></ul> |
- First item
- Second item
- Third item
    - Indented item
    - Indented item
- Fourth item

---

## Starting Unordered List Items With Numbers 從數字開始的無序列表項目

If you need to start an unordered list item with a number followed by a period, you can use a backslash (`\`) to [escape](https://www.markdownguide.org/basic-syntax/#escaping-characters) the period.
若您需要在無序列表項目中開始於數字後跟隨一個句號，可以使用反斜杠（ `\` ）來跳過句號。

| Markdown                                                                                                                       | HTML                                                                                                                                                                                                           |
| ------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `- 1968\. A great year!`<br>`- I think 1969 was second best.`<br><br>- 1968\. A great year!<br>- I think 1969 was second best. | `<ul>`<br>`    <li>1968. A great year!</li>`<br>`    <li>I think 1969 was second best.</li>`<br>`</ul>`<br><br><ul><br>    <li>1968. A great year!</li><br>    <li>I think 1969 was second best.</li><br></ul> |
- 1968\. A great year!
- I think 1969 was second best.

<ul>
    <li>1968. A great year!</li>
    <li>I think 1969 was second best.</li>
</ul>

## Unordered List Best Practices 無序列表最佳實踐

Markdown applications don’t agree on how to handle different delimiters in the same list. For compatibility, don’t mix and match delimiters in the same list — pick one and stick with it.
Markdown 應用程式對於如何處理同一清單中的不同分隔符並無共識。為了兼容性，請勿在相同清單中混用分隔符——選擇一個並堅持使用。

## Adding Elements in Lists[](https://www.markdownguide.org/basic-syntax/#adding-elements-in-lists)

To add another element in a list while preserving the continuity of the list, indent the element four spaces or one tab, as shown in the following examples.
若要在清單中新增另一個元素並保持清單的連續性，請將元素縮排四個空格或一個製表符，如下例所示。

 **Tip:** If things don't appear the way you expect, double check that you've indented the elements in the list four spaces or one tab.
**提示:** 如果顯示結果與您預期的不符，請仔細檢查清單中元素的縮排是否為四個空格或一個製表符。

### Paragraphs

```plantext
* This is the first list item.
* Here's the second list item.

    I need to add another paragraph below the second list item.

* And here's the third list item.
```

The rendered output looks like this:

- This is the first list item.
- Here’s the second list item.

    I need to add another paragraph below the second list item.
    
- And here’s the third list item.

### Blockquotes

```plantext
* This is the first list item.
* Here's the second list item.

    > A blockquote would look great below the second list item.

* And here's the third list item.
```

* This is the first list item.
* Here's the second list item.

    > A blockquote would look great below the second list item.

* And here's the third list item.

### Code Blocks

Code blocks are normally indented four spaces or one tab. When they’re in a list, indent them eight spaces or two tabs.

```plantext
1. Open the file.
2. Find the following code block on line 21:

        <html>
          <head>
            <title>Test</title>
          </head>

3. Update the title to match the name of your website.
```

The rendered output looks like this:

1. Open the file.
2. Find the following code block on line 21:

        <html>
          <head>
            <title>Test</title>
          </head>

3. Update the title to match the name of your website.

### Images

```plantext
1. Open the file containing the Linux mascot.
2. Marvel at its beauty.

    ![Tux, the Linux mascot](https://mdg.imgix.net/assets/images/tux.png)

3. Close the file.
```

The rendered output looks like this:

1. Open the file containing the Linux mascot.
2. Marvel at its beauty.

    ![Tux, the Linux mascot](https://mdg.imgix.net/assets/images/tux.png)
    
3. Close the file.

### Lists

You can nest an unordered list in an ordered list, or vice versa.

```plantext
1. First item
2. Second item
3. Third item
    - Indented item
    - Indented item
4. Fourth item
```

The rendered output looks like this:

1. First item
2. Second item
3. Third item
    - Indented item
    - Indented item
4. Fourth item

---

## Code 代碼

To denote a word or phrase as code, enclose it in backticks (`` ` ``).
要標示單詞或短語為代碼，請將其用反引號 (`` ` ``) 包圍。

| Markdown                                                                           | HTML                                                                                                   |
| ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------ |
| ``At the command prompt, type `nano`.``<br><br>At the command prompt, type `nano`. | `At the command prompt, type <code>nano</code>.`<br><br>At the command prompt, type <code>nano</code>. |
At the command prompt, type `nano`.
At the command prompt, type <code>nano</code>.

### Escaping Backticks 逃逸反引號

If the word or phrase you want to denote as code includes one or more backticks, you can escape it by enclosing the word or phrase in double backticks (` `` `).
如果您想標記的單詞或短語中包含一個或多個反引號，您可以使用雙反引號將單詞或短語包圍起來來逃逸它（ ` `` ` ）。

| Markdown                                                                                   | HTML                                                                                                     |
| ------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------- |
| ``` ``Use `code` in your Markdown file.`` ```<br><br>``Use `code` in your Markdown file.`` | ``<code>Use `code` in your Markdown file.</code>``<br><br><code>Use `code` in your Markdown file.</code> |
``Use `code` in your Markdown file.``
<code>Use `code` in your Markdown file.</code>

### Code Blocks 程式碼區塊

To create code blocks, indent every line of the block by at least four spaces or one tab.
為了建立程式碼區塊，至少縮進區塊內的每一行四個空格或一個制表符。

    <html>
      <head>
        <title>Test</title>
      </head>

The rendered output looks like this:

        <html>
          <head>
            <title>Test</title>
          </head>

> **Note:** To create code blocks without indenting lines, use [fenced code blocks](#fenced-code-blocks).
> 注意：若要建立不縮進行的程式碼區塊，請使用圍欄程式碼區塊。

---

# Horizontal Rule 水平線

To create a horizontal rule, use three or more asterisks (`***`), dashes (`---`), or underscores (`___`) on a line by themselves.
要建立水平線，請在行上使用三個或更多星號（ `***` ）、破折號（ `---` ）或下劃線（ `___` ）單獨使用。

```plantext
***

---

_________________
```

***

---

_________________

## Horizontal Rule Best Practices

水平線最佳實踐

For compatibility, put blank lines before and after horizontal rules.
為了兼容性，請在水平線前後放置空白行。

---

## Link 鏈接

To create a link, enclose the link text in brackets (e.g., `[Duck Duck Go]`) and then follow it immediately with the URL in parentheses (e.g., `(https://duckduckgo.com)`).
要建立連結，將連結文字放在括號中（例如， `[Duck Duck Go]` ）並立即跟隨括號中的網址（例如， `(https://duckduckgo.com)` ）。

```plantext
My favorite search engine is [Duck Duck Go](https://duckduckgo.com).
```

The rendered output looks like this:

My favorite search engine is [Duck Duck Go](https://duckduckgo.com/).

> **Note:** To link to an element on the same page, see [linking to heading IDs](https://www.markdownguide.org/extended-syntax/#linking-to-heading-ids). To create a link that opens in a new tab or window, see the section on [link targets](#link-targets).
>注意：要連結到同一頁面的元素，請參考連結到標題 ID 的部分。要建立一個在新標籤或視窗中打開的連結，請參考連結目標的章節。

## Adding Titles  添加標題

You can optionally add a title for a link. This will appear as a tooltip when the user hovers over the link. To add a title, enclose it in quotation marks after the URL.
您可以選擇性地為鏈接添加標題。當用戶將滑鼠移至鏈接上時，這將顯示為工具提示。要添加標題，請將其放在 URL 之後的引號中。

```plantext
My favorite search engine is [Duck Duck Go](https://duckduckgo.com "The best search engine for privacy").
```

The rendered output looks like this:

My favorite search engine is [Duck Duck Go](https://duckduckgo.com/ "The best search engine for privacy").

## URLs and Email Addresses 網址與電子郵件地址

To quickly turn a URL or email address into a link, enclose it in angle brackets.
為了快速將 URL 或電子郵件位址轉換為連結，將其包圍在尖括號中。

```plantext
<https://www.markdownguide.org>
<fake@example.com>
```

The rendered output looks like this:

[https://www.markdownguide.org](https://www.markdownguide.org/)
[fake@example.com](mailto:fake@example.com)

## Formatting Links 格式化鏈接

To [emphasize](https://www.markdownguide.org/basic-syntax/#emphasis) links, add asterisks before and after the brackets and parentheses. To denote links as [code](https://www.markdownguide.org/basic-syntax/#code), add backticks in the brackets.
為了強調連結，在括號前後加上星號。為了將連結標記為代碼，在括號中加上反引號。

```plantext
I love supporting the **[EFF](https://eff.org)**.
This is the *[Markdown Guide](https://www.markdownguide.org)*.
See the section on [`code`](#code).
```

The rendered output looks like this:

I love supporting the **[EFF](https://eff.org)**.
This is the *[Markdown Guide](https://www.markdownguide.org)*.
See the section on [`code`](#code).

## Reference-style Links 參考樣式鏈接

Reference-style links are a special kind of link that make URLs easier to display and read in Markdown. Reference-style links are constructed in two parts: the part you keep inline with your text and the part you store somewhere else in the file to keep the text easy to read.
參考式鏈接是一種特殊的鏈接，它讓在 Markdown 中顯示和閱讀 URL 更為簡便。參考式鏈接由兩部分構成：一部分與您的文本內文並行，另一部分則存放在文件的其他位置，以保持文本的易讀性。

### Formatting the First Part of the Link 格式化鏈接的第一部分

The first part of a reference-style link is formatted with two sets of brackets. The first set of brackets surrounds the text that should appear linked. The second set of brackets displays a label used to point to the link you’re storing elsewhere in your document.
參考式鏈接的第一部分使用兩組括號進行格式化。第一組括號包圍應該出現為鏈接的文本。第二組括號顯示一個用於指向您在文件其他位置存儲的鏈接的標籤。

Although not required, you can include a space between the first and second set of brackets. The label in the second set of brackets is not case sensitive and can include letters, numbers, spaces, or punctuation.
不強制性，但您可以在第一組和第二組方括號之間加入空格。第二組方括號中的標籤不區分大小寫，可以包含字母、數字、空格或標點符號。

This means the following example formats are roughly equivalent for the first part of the link:
這意味著以下範例格式對於鏈接的第一部分來說大致相等：

- `[hobbit-hole][1]`
- `[hobbit-hole] [1]`

### Formatting the Second Part of the Link 格式化鏈接的第二部分

The second part of a reference-style link is formatted with the following attributes:
參考風格的超連結的第二部分以以下屬性格式化：

1. The label, in brackets, followed immediately by a colon and at least one space (e.g., `[label]:` ).
    在括號中的標籤，隨即接著冒號和至少一個空格（例如， `[label]:` ）。
2. The URL for the link, which you can optionally enclose in angle brackets.
    超連結的 URL，您可以選擇用尖括號括起來。
3. The optional title for the link, which you can enclose in double quotes, single quotes, or parentheses.
    超連結的可選標題，您可以將其用雙引號、單引號或括號括起來。

This means the following example formats are all roughly equivalent for the second part of the link:
這意味著以下範例格式對於鏈接的第二部分來說大致相等：

- `[1]: https://en.wikipedia.org/wiki/Hobbit#Lifestyle`
- `[1]: https://en.wikipedia.org/wiki/Hobbit#Lifestyle "Hobbit lifestyles"`
- `[1]: https://en.wikipedia.org/wiki/Hobbit#Lifestyle 'Hobbit lifestyles'`
- `[1]: https://en.wikipedia.org/wiki/Hobbit#Lifestyle (Hobbit lifestyles)`
- `[1]: <https://en.wikipedia.org/wiki/Hobbit#Lifestyle> "Hobbit lifestyles"`
- `[1]: <https://en.wikipedia.org/wiki/Hobbit#Lifestyle> 'Hobbit lifestyles'`
- `[1]: <https://en.wikipedia.org/wiki/Hobbit#Lifestyle> (Hobbit lifestyles)`

You can place this second part of the link anywhere in your Markdown document. Some people place them immediately after the paragraph in which they appear while other people place them at the end of the document (like endnotes or footnotes).
您可以在 Markdown 文件的任何位置放置此連結的第二部分。有些人將其放置在出現的段落之後，而有些人則將其放置在文件的末尾（如尾注或註釋）。

### An Example Putting the Parts Together 一個將各部分組合的範例

Say you add a URL as a [standard URL link](#links) to a paragraph and it looks like this in Markdown:
假設你將一個網址作為標準的網址鏈接添加到段落中，在 Markdown 中它看起來像這樣：

```plantext
In a hole in the ground there lived a hobbit. Not a nasty, dirty, wet hole, filled with the ends
of worms and an oozy smell, nor yet a dry, bare, sandy hole with nothing in it to sit down on or to
eat: it was a [hobbit-hole](https://en.wikipedia.org/wiki/Hobbit#Lifestyle "Hobbit lifestyles"), and that means comfort.
```

Though it may point to interesting additional information, the URL as displayed really doesn’t add much to the existing raw text other than making it harder to read. To fix that, you could format the URL like this instead:
不過，它可能會指向有趣的附加資訊，但顯示的網址實際上並沒有為現有的原始文字增添太多，反而讓閱讀變得更困難。為了解決這個問題，您可以將網址格式化如下：

```plantext
In a hole in the ground there lived a hobbit. Not a nasty, dirty, wet hole, filled with the ends
of worms and an oozy smell, nor yet a dry, bare, sandy hole with nothing in it to sit down on or to
eat: it was a [hobbit-hole][1], and that means comfort.

[1]: <https://en.wikipedia.org/wiki/Hobbit#Lifestyle> "Hobbit lifestyles"
```

In both instances above, the rendered output would be identical:
在上述兩個例子中，渲染的輸出將會完全相同：

> In a hole in the ground there lived a hobbit. Not a nasty, dirty, wet hole, filled with the ends of worms and an oozy smell, nor yet a dry, bare, sandy hole with nothing in it to sit down on or to eat: it was a [hobbit-hole](https://en.wikipedia.org/wiki/Hobbit#Lifestyle "Hobbit lifestyles"), and that means comfort.
> 在地面的一個洞裡，住著一個霍比特人。不是一個髒亂、濕潤、充滿蠕蟲尾巴和濕滑氣味的洞，也不是一個乾燥、裸露、沙質的洞，裡面沒有東西可以坐或吃：那是一個霍比特洞，這意味著舒適。

and the HTML for the link would be:
連結的 HTML 為：

`<a href="https://en.wikipedia.org/wiki/Hobbit#Lifestyle" title="Hobbit lifestyles">hobbit-hole</a>`

## Link Best Practices  鏈接最佳實踐

Markdown applications don’t agree on how to handle spaces in the middle of a URL. For compatibility, try to URL encode any spaces with `%20`. Alternatively, if your Markdown application [supports HTML](https://www.markdownguide.org/basic-syntax/#html), you could use the `a` HTML tag.
Markdown 應用程式對於 URL 中間的空格處理方式並不一致。為了兼容性，請嘗試使用 `%20` 對空格進行 URL 编碼。另可選，如果您的 Markdown 應用程式支援 HTML，您可以使用 `a` HTML 標籤。

| ✅  Do this                                                                                                            | ❌  Don't do this                                   |
| --------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------- |
| `[link](https://www.example.com/my%20great%20page)`<br><br>`<a href="https://www.example.com/my great page">link</a>` | `[link](https://www.example.com/my great page)   ` |

Parentheses in the middle of a URL can also be problematic. For compatibility, try to URL encode the opening parenthesis (`(`) with `%28` and the closing parenthesis (`)`) with `%29`. Alternatively, if your Markdown application [supports HTML](https://www.markdownguide.org/basic-syntax/#html), you could use the `a` HTML tag.
URL 中間的括號也可能會造成問題。為了兼容性，請嘗試使用 `(` 對開啟括號進行 URL 编碼，並使用 `%28` 對關閉括號進行 URL 编碼。另可選，如果您的 Markdown 應用程式支援 HTML，您可以使用 `)` HTML 標籤。

| ✅  Do this                                                                                                                                                                      | ❌  Don't do this                                                             |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------- |
| `[a novel](https://en.wikipedia.org/wiki/The_Milagro_Beanfield_War_%28novel%29)`<br><br>`<a href="https://en.wikipedia.org/wiki/The_Milagro_Beanfield_War_(novel)">a novel</a>` | `[a novel](https://en.wikipedia.org/wiki/The_Milagro_Beanfield_War_(novel))` |

---

# Images  圖片

To add an image, add an exclamation mark (`!`), followed by alt text in brackets, and the path or URL to the image asset in parentheses. You can optionally add a title in quotation marks after the path or URL.
要加入圖片，請加入一個驚嘆號（ `!` ），然後在括號中輸入替代文字，並在括號中提供圖片資產的路徑或 URL。您可以在路徑或 URL 之後選擇性地加入一個用引號括起的標題。

```plantext
![The San Juan Mountains are beautiful!](https://mdg.imgix.net/assets/images/san-juan-mountains.jpg "San Juan Mountains")
```

The rendered output looks like this:

![The San Juan Mountains are beautiful!](https://mdg.imgix.net/assets/images/san-juan-mountains.jpg "San Juan Mountains")

> **Note:** To resize an image, see the section on [image size](https://www.markdownguide.org/hacks/#image-size). To add a caption, see the section on [image captions](https://www.markdownguide.org/hacks/#image-captions).
> **注意:** 要調整圖片大小，請參考圖片大小相關部分。要加入圖片說明文字，請參考圖片說明文字相關部分。

## Linking Images  圖片鏈接

To add a link to an image, enclose the Markdown for the image in brackets, and then add the link in parentheses.
要加入圖片鏈接，請將圖片的 Markdown 語法用方括號括起來，然後在括號中加上鏈接。

```plantext
[![An old rock in the desert](https://mdg.imgix.net/assets/images/shiprock.jpg "Shiprock, New Mexico by Beau Rogers")
```

The rendered output looks like this:

[![An old rock in the desert](https://mdg.imgix.net/assets/images/shiprock.jpg "Shiprock, New Mexico by Beau Rogers")

---

# Escaping Characters  轉義字元 [](https://www.markdownguide.org/basic-syntax/#escaping-characters)

To display a literal character that would otherwise be used to format text in a Markdown document, add a backslash (`\`) in front of the character.
為了在 Markdown 文件中顯示原本用於格式化文字的字元，請在該字元前加上一個反斜杠（\）。

```plantext
\* Without the backslash, this would be a bullet in an unordered list.
```

The rendered output looks like this:

\* Without the backslash, this would be a bullet in an unordered list.

## Characters You Can Escape 可逃逸的字符

You can use a backslash to escape the following characters.
您可以使用反斜線來轉義以下字符。

| Character  字符 | Name  名稱                                                                                                                                         |
| ------------- | ------------------------------------------------------------------------------------------------------------------------------------------------ |
| \             | backslash  反斜線                                                                                                                                   |
| `             | backtick (see also [escaping backticks in code](https://www.markdownguide.org/basic-syntax/#escaping-backticks))  <br>反引號（請參考在代碼中逃逸反引號）          |
| *             | asterisk  星號                                                                                                                                     |
| _             | underscore  下劃線                                                                                                                                  |
| { }           | curly braces  花括號                                                                                                                                |
| [ ]           | brackets  括號                                                                                                                                     |
| < >  <>       | angle brackets  尖括號                                                                                                                              |
| ( )           | parentheses  括號                                                                                                                                  |
| #             | pound sign  井號                                                                                                                                   |
| +             | plus sign  加號                                                                                                                                    |
| -             | minus sign (hyphen)  減號（破折號）                                                                                                                     |
| .             | dot  點                                                                                                                                           |
| !             | exclamation mark  驚嘆號                                                                                                                            |
| \|            | pipe (see also [escaping pipe in tables](https://www.markdownguide.org/extended-syntax/#escaping-pipe-characters-in-tables))  <br>管線（參見表格中的管線轉義） |

---

## HTML

Many Markdown applications allow you to use HTML tags in Markdown-formatted text. This is helpful if you prefer certain HTML tags to Markdown syntax. For example, some people find it easier to use HTML tags for images. Using HTML is also helpful when you need to change the attributes of an element, like specifying the [color of text](https://www.markdownguide.org/hacks/#color) or changing the width of an image.
許多 Markdown 應用程式允許您在 Markdown 格式的文字中使用 HTML 標籤。如果您偏好使用某些 HTML 標籤而非 Markdown 語法，這將非常有幫助。例如，有些人發現使用 HTML 標籤來插入圖片更為簡單。當您需要更改元素的屬性時，例如指定文字顏色或更改圖片的寬度，使用 HTML 同樣很有幫助。

To use HTML, place the tags in the text of your Markdown-formatted file.
使用 HTML，將標籤放置在 Markdown 格式檔案的文本中。

```plantext
This **word** is bold. This <em>word</em> is italic.
```

The rendered output looks like this:

This **word** is bold. This _word_ is italic.

## HTML Best Practices  HTML 最佳實踐

For security reasons, not all Markdown applications support HTML in Markdown documents. When in doubt, check your Markdown application’s documentation. Some applications support only a subset of HTML tags.
由於安全性的考量，並非所有的 Markdown 應用程式都支援在 Markdown 文件中使用 HTML。如有疑問，請檢查您的 Markdown 應用程式文件。有些應用程式只支援 HTML 標籤的子集。

Use blank lines to separate block-level HTML elements like `<div>`, `<table>`, `<pre>`, and `<p>` from the surrounding content. Try not to indent the tags with tabs or spaces — that can interfere with the formatting.
使用空白行將塊級 HTML 元素（如 `<div>` 、 `<table>` 、 `<pre>` 和 `<p>` ）與周圍內容分開。請勿使用制表符或空格縮進標籤——這可能會干擾格式。

You can’t use Markdown syntax inside block-level HTML tags. For example, `<p>italic and **bold**</p>` won’t work.
您不能在塊級 HTML 標籤中使用 Markdown 語法。例如， `<p>italic and **bold**</p>` 不會生效。

<p>italic and **bold**</p>

---

# Extended Syntax

## Overview 概述

The [basic syntax](https://www.markdownguide.org/basic-syntax/) outlined in the original Markdown design document added many of the elements needed on a day-to-day basis, but it wasn’t enough for some people. That’s where extended syntax comes in.
原始 Markdown 設計文件中概述的基本語法為日常所需添加了許多元素，但對某些人來說並不夠。這就是擴展語法的用處所在。

Several individuals and organizations took it upon themselves to extend the basic syntax by adding additional elements like tables, code blocks, syntax highlighting, URL auto-linking, and footnotes. These elements can be enabled by using a lightweight markup language that builds upon the basic Markdown syntax, or by adding an extension to a compatible Markdown processor.
許多個人和組織自發地擴展了基本的語法，增加了表格、代碼塊、語法高亮、網址自動鏈接和腳註等元素。這些元素可以通過使用基於基本 Markdown 語法的輕量級標記語言來啟用，或者通過為兼容的 Markdown 处理器添加擴展來實現。

# Availability  可用性

Not all Markdown applications support extended syntax elements. You’ll need to check whether or not the lightweight markup language your application is using supports the extended syntax elements you want to use. If it doesn’t, it may still be possible to enable extensions in your Markdown processor.
並非所有的 Markdown 應用程序都支持擴展語法元素。您需要檢查您所使用的輕量級標記語言是否支持您想要使用的擴展語法元素。如果它不支持，您可能仍然可以在 Markdown 处理器中啟用擴展。

# Lightweight Markup Languages 輕量級標記語言

There are several lightweight markup languages that are _supersets_ of Markdown. They include basic syntax and build upon it by adding additional elements like tables, code blocks, syntax highlighting, URL auto-linking, and footnotes. Many of the most popular Markdown applications use one of the following lightweight markup languages:
有許多輕量級標記語言是 Markdown 的超集。它們包括基本的語法並在其上增加額外的元素，如表格、代碼塊、語法高亮、網址自動鏈接和腳注。許多最受歡迎的 Markdown 應用程序都使用以下輕量級標記語言之一：

- [CommonMark](https://commonmark.org/)
- [GitHub Flavored Markdown (GFM)](https://github.github.com/gfm/)
- [Markdown Extra](https://michelf.ca/projects/php-markdown/extra/)
- [MultiMarkdown](https://fletcherpenney.net/multimarkdown/)
- [R Markdown](https://rmarkdown.rstudio.com/)

# Markdown Processors  Markdown 處理器 [](https://www.markdownguide.org/extended-syntax/#markdown-processors)

There are [dozens of Markdown processors](https://github.com/markdown/markdown.github.com/wiki/Implementations) available. Many of them allow you to add extensions that enable extended syntax elements. Check your processor’s documentation for more information.
Markdown 處理器有數十種。許多處理器允許您添加擴展功能，以啟用擴展語法元素。請檢查您處理器的文件以獲取更多信息。

---

# Table 表格

To add a table, use three or more hyphens (`---`) to create each column’s header, and use pipes (`|`) to separate each column. For compatibility, you should also add a pipe on either end of the row.
要添加表格，使用三個或更多短橫線（ `---` ）來創建每個列的標題，並使用管道符號（ `|` ）來分隔每個列。為了兼容性，您還應該在行的兩端添加一個管道符號。

```markdown
| Syntax      | Description |
| ----------- | ----------- |
| Header      | Title       |
| Paragraph   | Text        |
```

The rendered output looks like this:

| Syntax    | Description |
| --------- | ----------- |
| Header    | Title       |
| Paragraph | Text        |

Cell widths can vary, as shown below. The rendered output will look the same.
表格的欄寬可以有所不同，如下所示。渲染後的輸出將會看起來一樣。

```markdown
| Syntax | Description |
| --- | ----------- |
| Header | Title |
| Paragraph | Text |
```

> **Tip:** Creating tables with hyphens and pipes can be tedious. To speed up the process, try using the [Markdown Tables Generator](https://www.tablesgenerator.com/markdown_tables) or [AnyWayData Markdown Export](https://anywaydata.com/). Build a table using the graphical interface, and then copy the generated Markdown-formatted text into your file.
> **小技巧:** 使用破折號和管道符號建立表格可能會有些麻煩。為了加速過程，可以嘗試使用 Markdown Tables Generator 或 AnyWayData Markdown Export。使用圖形介面建立表格，然後將產生的 Markdown 格式化文字複製到您的檔案中。

# Alignment  對齊

You can align text in the columns to the left, right, or center by adding a colon (`:`) to the left, right, or on both side of the hyphens within the header row.
您可以在欄位中對文字進行左對齊、右對齊或居中對齊，只需在標題行的破折號內左側、右側或兩側加上冒號（ `:` ）即可。

```markdown
| Syntax      | Description | Test Text     |
| :---        |    :----:   |          ---: |
| Header      | Title       | Here's this   |
| Paragraph   | Text        | And more      |
```

The rendered output looks like this:

| Syntax    | Description |   Test Text |
| :-------- | :---------: | ----------: |
| Header    |    Title    | Here's this |
| Paragraph |    Text     |    And more |

# Formatting Text in Tables 在表格中格式化文字

You can format the text within tables. For example, you can add [links](https://www.markdownguide.org/basic-syntax/#links), [code](https://www.markdownguide.org/basic-syntax/#code-1) (words or phrases in backticks (`` ` ``) only, not [code blocks](https://www.markdownguide.org/basic-syntax/#code-blocks)), and [emphasis](https://www.markdownguide.org/basic-syntax/#emphasis).
您可以在表格中格式化文字。例如，您可以加入連結、代碼（只有反引號中的單詞或短語，不是代碼塊）以及強調。

You can’t use headings, blockquotes, lists, horizontal rules, images, or most HTML tags.
您不能使用標題、引言、清單、水平線、圖片或大多數 HTML 標籤。

> **Tip:** You can use HTML to create [line breaks](https://www.markdownguide.org/hacks/#line-breaks-within-table-cells) and add [lists](https://www.markdownguide.org/hacks/#lists-within-table-cells) within table cells.
> **提示:** 您可以使用 HTML 來建立換行並在表格單元格內添加清單。

# Escaping Pipe Characters in Tables 在表格中逃逸管狀字符

You can display a pipe (`|`) character in a table by using its HTML character code (`&#124;`).
您可以在表格中顯示管狀（ `|` ）字符，通過使用其 HTML 字符代碼（ `&#124;` ）。

---

# Fenced Code Block 多行程式碼區塊 / 圍欄代碼區塊

The basic Markdown syntax allows you to create [code blocks](https://www.markdownguide.org/basic-syntax/#code-blocks) by indenting lines by four spaces or one tab. If you find that inconvenient, try using fenced code blocks. Depending on your Markdown processor or editor, you’ll use three backticks (` ``` `) or three tildes (`~~~`) on the lines before and after the code block. The best part? You don’t have to indent any lines!
基本的 Markdown 語法允許您透過將行首縮進四個空格或一個制表符來建立程式碼塊。如果您覺得這樣不方便，可以嘗試使用圍繞式程式碼塊。根據您的 Markdown 解析器或編輯器，您可以在程式碼塊前後的行使用三個反引號 ( ` ``` ` ) 或三個波浪號 ( `~~~` )。最好的部分？您不需要縮進任何行！

````plantext
```
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}
```
````

The rendered output looks like this:

```csharp
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}
```

> **Tip:** Need to display backticks inside a code block? See [this section](https://www.markdownguide.org/basic-syntax/#escaping-backticks) to learn how to escape them.
> **小貼士:** 需要在程式碼塊中顯示反引號？請參考本節以了解如何跳脫它們。

---

# Syntax Highlighting 語法高亮

Many Markdown processors support syntax highlighting for fenced code blocks. This feature allows you to add color highlighting for whatever language your code was written in. To add syntax highlighting, specify a language next to the backticks before the fenced code block.
許多 Markdown 處理器支援為隔離程式碼區塊添加語法高亮。此功能可讓您為程式碼添加顏色高亮，無論該程式碼區塊是用什麼語言編寫的。若要新增語法高亮，請在隔離程式碼區塊前的反引號旁指定語言。

````text
```json
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}
```
````

The rendered output looks like this:

```json
{
  "firstName": "John",
  "lastName": "Smith",
  "age": 25
}
```

```CSharp
/// <summary>
/// 有效位數格式化處理。
/// </summary>
/// <param name="value">待格式化的double值。</param>
/// <param name="digit">小數點有效位數。</param>
/// <returns>格式化完成後的double值。</returns>
public static double Format(this double value, int digit)
{
    return double.IsNaN(value)
        ? double.NaN
        : Math.Round(value, digit, MidpointRounding.AwayFromZero);
}
```

```CPP
void MyWidget::on_ColorButton_clicked() {
    // 靜態呼叫, Cancel 返回 QColor(Invalid)
    QColor color = QColorDialog::getColor(Qt::red, this, tr("顏色對話框"), QColorDialog::ShowAlphaChannel);

    qDebug() << "color: " << color;

    // https://blog.csdn.net/u010847519/article/details/64441577
    QString msg = QString("A: %1\nR: %2\nG: %3\nB: %4")
                      .arg(QString::number(color.alpha()), QString::number(color.red()), QString::number(color.green()),
                           QString::number(color.blue()));
    QMessageBox::information(NULL, "Selected color", msg);
}
```

```XML
<Grid>
    <TextBlock Text="Text" />
</Grid>
```

## Language grammars listed 語言語法列表

https://shiki.matsu.io/languages

| Name                        | ID                   | Alias                     | Preview |
| --------------------------- | -------------------- | ------------------------- | ------- |
| ABAP                        | `abap`               |                           |         |
| ActionScript                | `actionscript-3`     |                           |         |
| Ada                         | `ada`                |                           |         |
| Angular HTML                | `angular-html`       |                           |         |
| Angular TypeScript          | `angular-ts`         |                           |         |
| Apache Conf                 | `apache`             |                           |         |
| Apex                        | `apex`               |                           |         |
| APL                         | `apl`                |                           |         |
| AppleScript                 | `applescript`        |                           |         |
| Ara                         | `ara`                |                           |         |
| AsciiDoc                    | `asciidoc`           | `adoc`                    |         |
| Assembly                    | `asm`                |                           |         |
| Astro                       | `astro`              |                           |         |
| AWK                         | `awk`                |                           |         |
| Ballerina                   | `ballerina`          |                           |         |
| Batch File                  | `bat`                | `batch`                   |         |
| Beancount                   | `beancount`          |                           |         |
| Berry                       | `berry`              | `be`                      |         |
| BibTeX                      | `bibtex`             |                           |         |
| Bicep                       | `bicep`              |                           |         |
| Blade                       | `blade`              |                           |         |
| 1C (Enterprise)             | `bsl`                | `1c`                      |         |
| C                           | `c`                  |                           |         |
| Cadence                     | `cadence`            | `cdc`                     |         |
| Cairo                       | `cairo`              |                           |         |
| Clarity                     | `clarity`            |                           |         |
| Clojure                     | `clojure`            | `clj`                     |         |
| CMake                       | `cmake`              |                           |         |
| COBOL                       | `cobol`              |                           |         |
| CODEOWNERS                  | `codeowners`         |                           |         |
| CodeQL                      | `codeql`             | `ql`                      |         |
| CoffeeScript                | `coffee`             | `coffeescript`            |         |
| Common Lisp                 | `common-lisp`        | `lisp`                    |         |
| Coq                         | `coq`                |                           |         |
| C++                         | `cpp`                | `c++`                     |         |
| Crystal                     | `crystal`            |                           |         |
| C#                          | `csharp`             | `c#``cs`                  |         |
| CSS                         | `css`                |                           |         |
| CSV                         | `csv`                |                           |         |
| CUE                         | `cue`                |                           |         |
| Cypher                      | `cypher`             | `cql`                     |         |
| D                           | `d`                  |                           |         |
| Dart                        | `dart`               |                           |         |
| DAX                         | `dax`                |                           |         |
| Desktop                     | `desktop`            |                           |         |
| Diff                        | `diff`               |                           |         |
| Dockerfile                  | `docker`             | `dockerfile`              |         |
| dotEnv                      | `dotenv`             |                           |         |
| Dream Maker                 | `dream-maker`        |                           |         |
| Edge                        | `edge`               |                           |         |
| Elixir                      | `elixir`             |                           |         |
| Elm                         | `elm`                |                           |         |
| Emacs Lisp                  | `emacs-lisp`         | `elisp`                   |         |
| ERB                         | `erb`                |                           |         |
| Erlang                      | `erlang`             | `erl`                     |         |
| Fennel                      | `fennel`             |                           |         |
| Fish                        | `fish`               |                           |         |
| Fluent                      | `fluent`             | `ftl`                     |         |
| Fortran (Fixed Form)        | `fortran-fixed-form` | `f``for``f77`             |         |
| Fortran (Free Form)         | `fortran-free-form`  | `f90``f95``f03``f08``f18` |         |
| F#                          | `fsharp`             | `f#``fs`                  |         |
| GDResource                  | `gdresource`         |                           |         |
| GDScript                    | `gdscript`           |                           |         |
| GDShader                    | `gdshader`           |                           |         |
| Genie                       | `genie`              |                           |         |
| Gherkin                     | `gherkin`            |                           |         |
| Git Commit Message          | `git-commit`         |                           |         |
| Git Rebase Message          | `git-rebase`         |                           |         |
| Gleam                       | `gleam`              |                           |         |
| Glimmer JS                  | `glimmer-js`         | `gjs`                     |         |
| Glimmer TS                  | `glimmer-ts`         | `gts`                     |         |
| GLSL                        | `glsl`               |                           |         |
| Gnuplot                     | `gnuplot`            |                           |         |
| Go                          | `go`                 |                           |         |
| GraphQL                     | `graphql`            | `gql`                     |         |
| Groovy                      | `groovy`             |                           |         |
| Hack                        | `hack`               |                           |         |
| Ruby Haml                   | `haml`               |                           |         |
| Handlebars                  | `handlebars`         | `hbs`                     |         |
| Haskell                     | `haskell`            | `hs`                      |         |
| Haxe                        | `haxe`               |                           |         |
| HashiCorp HCL               | `hcl`                |                           |         |
| Hjson                       | `hjson`              |                           |         |
| HLSL                        | `hlsl`               |                           |         |
| HTML                        | `html`               |                           |         |
| HTML (Derivative)           | `html-derivative`    |                           |         |
| HTTP                        | `http`               |                           |         |
| Hurl                        | `hurl`               |                           |         |
| HXML                        | `hxml`               |                           |         |
| Hy                          | `hy`                 |                           |         |
| Imba                        | `imba`               |                           |         |
| INI                         | `ini`                | `properties`              |         |
| Java                        | `java`               |                           |         |
| JavaScript                  | `javascript`         | `js``cjs``mjs`            |         |
| Jinja                       | `jinja`              |                           |         |
| Jison                       | `jison`              |                           |         |
| JSON                        | `json`               |                           |         |
| JSON5                       | `json5`              |                           |         |
| JSON with Comments          | `jsonc`              |                           |         |
| JSON Lines                  | `jsonl`              |                           |         |
| Jsonnet                     | `jsonnet`            |                           |         |
| JSSM                        | `jssm`               | `fsl`                     |         |
| JSX                         | `jsx`                |                           |         |
| Julia                       | `julia`              | `jl`                      |         |
| KDL                         | `kdl`                |                           |         |
| Kotlin                      | `kotlin`             | `kt``kts`                 |         |
| Kusto                       | `kusto`              | `kql`                     |         |
| LaTeX                       | `latex`              |                           |         |
| Lean 4                      | `lean`               | `lean4`                   |         |
| Less                        | `less`               |                           |         |
| Liquid                      | `liquid`             |                           |         |
| LLVM IR                     | `llvm`               |                           |         |
| Log file                    | `log`                |                           |         |
| Logo                        | `logo`               |                           |         |
| Lua                         | `lua`                |                           |         |
| Luau                        | `luau`               |                           |         |
| Makefile                    | `make`               | `makefile`                |         |
| Markdown                    | `markdown`           | `md`                      |         |
| Marko                       | `marko`              |                           |         |
| MATLAB                      | `matlab`             |                           |         |
| MDC                         | `mdc`                |                           |         |
| MDX                         | `mdx`                |                           |         |
| Mermaid                     | `mermaid`            | `mmd`                     |         |
| MIPS Assembly               | `mipsasm`            | `mips`                    |         |
| Mojo                        | `mojo`               |                           |         |
| Move                        | `move`               |                           |         |
| Narrat Language             | `narrat`             | `nar`                     |         |
| Nextflow                    | `nextflow`           | `nf`                      |         |
| Nginx                       | `nginx`              |                           |         |
| Nim                         | `nim`                |                           |         |
| Nix                         | `nix`                |                           |         |
| nushell                     | `nushell`            | `nu`                      |         |
| Objective-C                 | `objective-c`        | `objc`                    |         |
| Objective-C++               | `objective-cpp`      |                           |         |
| OCaml                       | `ocaml`              |                           |         |
| Pascal                      | `pascal`             |                           |         |
| Perl                        | `perl`               |                           |         |
| PHP                         | `php`                |                           |         |
| Pkl                         | `pkl`                |                           |         |
| PL/SQL                      | `plsql`              |                           |         |
| Gettext PO                  | `po`                 | `pot``potx`               |         |
| Polar                       | `polar`              |                           |         |
| PostCSS                     | `postcss`            |                           |         |
| PowerQuery                  | `powerquery`         |                           |         |
| PowerShell                  | `powershell`         | `ps``ps1`                 |         |
| Prisma                      | `prisma`             |                           |         |
| Prolog                      | `prolog`             |                           |         |
| Protocol Buffer 3           | `proto`              | `protobuf`                |         |
| Pug                         | `pug`                | `jade`                    |         |
| Puppet                      | `puppet`             |                           |         |
| PureScript                  | `purescript`         |                           |         |
| Python                      | `python`             | `py`                      |         |
| QML                         | `qml`                |                           |         |
| QML Directory               | `qmldir`             |                           |         |
| Qt Style Sheets             | `qss`                |                           |         |
| R                           | `r`                  |                           |         |
| Racket                      | `racket`             |                           |         |
| Raku                        | `raku`               | `perl6`                   |         |
| ASP.NET Razor               | `razor`              |                           |         |
| Windows Registry Script     | `reg`                |                           |         |
| RegExp                      | `regexp`             | `regex`                   |         |
| Rel                         | `rel`                |                           |         |
| RISC-V                      | `riscv`              |                           |         |
| ROS Interface               | `rosmsg`             |                           |         |
| reStructuredText            | `rst`                |                           |         |
| Ruby                        | `ruby`               | `rb`                      |         |
| Rust                        | `rust`               | `rs`                      |         |
| SAS                         | `sas`                |                           |         |
| Sass                        | `sass`               |                           |         |
| Scala                       | `scala`              |                           |         |
| Scheme                      | `scheme`             |                           |         |
| SCSS                        | `scss`               |                           |         |
| 1C (Query)                  | `sdbl`               | `1c-query`                |         |
| ShaderLab                   | `shaderlab`          | `shader`                  |         |
| Shell                       | `shellscript`        | `bash``sh``shell``zsh`    |         |
| Shell Session               | `shellsession`       | `console`                 |         |
| Smalltalk                   | `smalltalk`          |                           |         |
| Solidity                    | `solidity`           |                           |         |
| Closure Templates           | `soy`                | `closure-templates`       |         |
| SPARQL                      | `sparql`             |                           |         |
| Splunk Query Language       | `splunk`             | `spl`                     |         |
| SQL                         | `sql`                |                           |         |
| SSH Config                  | `ssh-config`         |                           |         |
| Stata                       | `stata`              |                           |         |
| Stylus                      | `stylus`             | `styl`                    |         |
| Svelte                      | `svelte`             |                           |         |
| Swift                       | `swift`              |                           |         |
| SystemVerilog               | `system-verilog`     |                           |         |
| Systemd Units               | `systemd`            |                           |         |
| TalonScript                 | `talonscript`        | `talon`                   |         |
| Tasl                        | `tasl`               |                           |         |
| Tcl                         | `tcl`                |                           |         |
| Templ                       | `templ`              |                           |         |
| Terraform                   | `terraform`          | `tf``tfvars`              |         |
| TeX                         | `tex`                |                           |         |
| TOML                        | `toml`               |                           |         |
| TypeScript with Tags        | `ts-tags`            | `lit`                     |         |
| TSV                         | `tsv`                |                           |         |
| TSX                         | `tsx`                |                           |         |
| Turtle                      | `turtle`             |                           |         |
| Twig                        | `twig`               |                           |         |
| TypeScript                  | `typescript`         | `ts``cts``mts`            |         |
| TypeSpec                    | `typespec`           | `tsp`                     |         |
| Typst                       | `typst`              | `typ`                     |         |
| V                           | `v`                  |                           |         |
| Vala                        | `vala`               |                           |         |
| Visual Basic                | `vb`                 | `cmd`                     |         |
| Verilog                     | `verilog`            |                           |         |
| VHDL                        | `vhdl`               |                           |         |
| Vim Script                  | `viml`               | `vim``vimscript`          |         |
| Vue                         | `vue`                |                           |         |
| Vue HTML                    | `vue-html`           |                           |         |
| Vue Vine                    | `vue-vine`           |                           |         |
| Vyper                       | `vyper`              | `vy`                      |         |
| WebAssembly                 | `wasm`               |                           |         |
| Wenyan                      | `wenyan`             | `文言`                      |         |
| WGSL                        | `wgsl`               |                           |         |
| Wikitext                    | `wikitext`           | `mediawiki``wiki`         |         |
| WebAssembly Interface Types | `wit`                |                           |         |
| Wolfram                     | `wolfram`            | `wl`                      |         |
| XML                         | `xml`                |                           |         |
| XSL                         | `xsl`                |                           |         |
| YAML                        | `yaml`               | `yml`                     |         |
| ZenScript                   | `zenscript`          |                           |         |
| Zig                         | `zig`                |                           |         |

## Special Languages   特殊語言

### Plain Text   純文字

You can set lang to `text` to bypass highlighting. This is useful as the fallback when you receive user specified language that are not available. For example:
您可以將語言設定為 `text` 以停用高亮顯示。這在收到用戶指定的語言但語言不可用時非常有用。例如：

```txt
import { codeToHtml } from 'shiki'

const html = codeToHtml('console.log("Hello World")', {
  lang: 'text', // [!code hl]
  theme: 'vitesse-light',
})
```

`txt`, `plain` are provided as aliases to `text` as well.
`txt` 和 `plain` 也是 `text` 的別名。

### ANSI   美國國家標準協會

A special processed language `ansi` is provided to highlight terminal outputs. For example:
系統提供了一種特殊的處理語言 `ansi` 用於高亮顯示終端輸出。例如：

```ansi
[0;32mcolored foreground[0m
[0;42mcolored background[0m

[0;1mbold text[0m
[0;2mdimmed text[0m
[0;4munderlined text[0m
[0;7mreversed text[0m
[0;9mstrikethrough text[0m
[0;4;9munderlined + strikethrough text[0m
```

Check the [raw markdown of code snippet above](https://github.com/shikijs/shiki/blob/main/docs/languages.md?plain=1#L35).
請查看 [上方程式碼片段的原始 Markdown](https://github.com/shikijs/shiki/blob/main/docs/languages.md?plain=1#L35) 程式碼。

## 一般文字

## Markdown 語法高亮對一般文字的可用選擇與差異

Markdown 的語法高亮是由「程式碼區塊語言標籤」控制的。
在 GitHub、VS Code、Typora 等常見環境中，對「一般文字」的語法高亮主要差別是：**顏色呈現、字型樣式、格式化規則**。

### 一、常見標籤與用途

1. **plaintext**
    - 表示純文字，不進行語法高亮。
    - 顯示效果與預設文字最接近。
    - 適合展示說明、日誌、命令輸出等。

```plaintext
This is just text.
No colors, no highlights.
```

2. **text**
    - 有些渲染器與 plaintext 等效。
    - 可作為兼容標籤，用於簡單文件。

```text
A plain text block.
```

3. **ansi**
    - 支援 ANSI 顏色轉義序列（如終端機彩色輸出）。
    - 若渲染器支援，會顯示彩色控制碼效果。

```ansi
[32mGreen text[0m and [31mred text[0m
```

4. **asciidoc / ansidoc**
    - 為 AsciiDoc 文件語法。
    - 有標題、強調、代碼塊等格式規則。
    - 顯示時會有簡單的文字結構高亮。

```asciidoc
= Document Title
== Section
*Bold* and _italic_ text
```

5. **markdown**
    - 會對標準 Markdown 語法加上顏色。
    - 適合展示 Markdown 文件教學。

```markdown
# Heading
**Bold text**
```

6. **none**
    - 部分系統使用 `none` 或空白作為無高亮模式。
    - 與 plaintext 類似，但有時樣式略不同。

```none
Just raw text here.
```

### 二、顯示差異總結

| 語法標籤                   | 高亮程度  | 用途建議           |
| ---------------------- | ----- | -------------- |
| `plaintext`            | 無     | 純說明文字或範例輸出     |
| `text`                 | 無或極輕微 | 相容用途           |
| `ansi`                 | 有終端色彩 | 模擬命令列結果        |
| `asciidoc` / `ansidoc` | 中度    | 文件結構展示         |
| `markdown`             | 高     | 教學 Markdown 語法 |
| `none`                 | 無     | 類似 plaintext   |

### 三、學習建議

在學習或寫筆記時：

1. **用 `plaintext`** 讓焦點放在內容，不被顏色干擾。
2. **用 `asciidoc` 或 `markdown`** 當想展示文件結構時。
3. **用 `ansi`** 模擬指令結果或彩色輸出。

---

# Footnote 腳註

腳註參考格式：<code>[^標識符]</code>。
標識符可以是數字或單詞，但不能包含空格或 TAB。
識符僅將腳註引用與腳註本身相關聯 - 在輸出中，腳註按順序編號。

腳註格式：<code>[^標識符]: 腳註內容</code>。
您不必在文檔末尾添加腳註。 您可以將它們放在除了列表、區塊引用和表格等其他元素內之外的任何位置。

Here's a simple footnote,[^1] and here's a longer one.[^bignote]

[^1]: This is the first footnote.
[^bignote]:
    Here's one with multiple paragraphs and code.
    Indent paragraphs to include them in the footnote.
    `{ my code }`
    Add as many paragraphs as you like.

---

# Heading ID 自定義標題 ID (註：在 HTML 中叫做錨點)

## My Great Heading {#heading-ids1} <!-- omit from toc -->

<h5 id="heading-ids2">My Great Heading</h3>

---

# Linking to Heading IDs 連結到自定義標題 ID (註：跳至錨點處)

[Heading IDs](#heading-ids1)

<a href="#heading-ids2">Heading IDs</a>

---

# Definition List 定義列表

First Term
: This is the definition of the first term.

Second Term
: This is one definition of the second term.
: This is another definition of the second term.

<dl>
  <dt>First Term</dt>
  <dd>This is the definition of the first term.</dd>
  <dt>Second Term</dt>
  <dd>This is one definition of the second term. </dd>
  <dd>This is another definition of the second term.</dd>
</dl>

---

# Strikethrough 刪除線

~~The world is flat.~~ We now know that the world is round.

---

# Task List 任務列表

- [x] Write the press release
- [ ] Update the website
- [ ] Contact the media

---

# Emoji 表情符號

## 方法 1：複製貼上

使用類似 [Emojipedia](https://emojipedia.org/) 來直接複製貼上 ❤️。

提示：如果您使用靜態站點生成器，請確保將 HTML 頁面編碼為 UTF-8。

## 方法 2：短碼加冒號

使用如 [markdown emoji markup](https://gist.github.com/rxaviers/7360908) 所列出的短碼，前後加冒號。

Gone camping! :tent: Be back soon.

That is so funny! :joy:

(See also [Copying and Pasting Emoji](https://www.markdownguide.org/extended-syntax/#copying-and-pasting-emoji))

---

# Highlight 強調

I need to highlight these ==very important words==.

I need to highlight these <mark>very important words</mark>.

---

# Subscript 下標

H~2~O

H<sub>2</sub>O

---

# Superscript 上標

X^2^

X<sup>2</sup>

---

# Automatic URL Linking 自動網址連結

有些編輯器在不使用角括弧時，仍可視為鏈接。

https://www.google.com/

---

# Disabling Automatic URL Linking 禁用網址自動連結

該鏈結將不會自動連結到該網址。

`http://www.example.com`

---

---

# Hacks 小技巧

## Overview 概述 <!-- omit from toc -->

大多數使用 Markdown 的人會發現基本和擴展語法元素滿足了他們的需求。 但如果你使用 Markdown 足夠長的時間，你將不可避免地發現它不支持你需要的東西。

提示：這些技巧不能保證在您的 Markdown 應用程序中有效。 如果您需要經常使用這些技巧，您應該考慮使用 Markdown 以外的其他工具進行編寫。

---

## Underline 底線

[Bear](https://www.markdownguide.org/tools/bear/) 和 [Simplenote](https://www.markdownguide.org/tools/simplenote/) 等一些應用程序提供對文本下劃線的支持，但 Markdown 本身並不原生支持下劃線。

Some of these words <ins>will be underlined</ins>.

---

## Indent (Tab) 縮進（TAB）

TAB 和空格在 Markdown 中具有特殊含義。Markdown 並沒有提供一種簡單的方法來做到縮進段落這一點。<br/>最好的選擇可能是使用支持縮進的 Markdown 編輯器。 這在更面向桌面發布的應用程序中很常見。<br/>如果您的 Markdown 處理器支持 HTML，另一種選擇是使用 HTML 實體作為不間斷空格 (&nbsp;)。這可能應該是您最後的選擇，因為它可能會變得尷尬。

&nbsp;&nbsp;&nbsp;&nbsp;This is the first sentence of my indented paragraph.

---

## Center 置中

不幸的是，Markdown 沒有任何文本對齊的概念（一個可能的例外是使用表格時）。好消息是您可以使用一個 HTML 標籤：\<center>。

<center>This text is centered.</center>

\<center> HTML 標記在技術上受支持，但正式已棄用，這意味著它目前可以使用，但您不應該使用它。不幸的是，沒有其他純 HTML 替代方案。<br/>如果您使用的應用程序提供 CSS 支持，您可以嘗試使用 CSS 替代方案，這裡有一個 \<center> 標籤的替代方案：

<p style="text-align:center">Center this text</p>

---

## Color 顏色

Markdown 不允許您更改文本的顏色，但如果您的 Markdown 處理器支持 HTML，則可以使用 \<font> HTML 標籤。

<font color="red">This text is red!</font>

\<font> HTML 標記在技術上受支持，但正式已棄用，這意味著它目前可以使用，但您不應該使用它。不幸的是，沒有其他純 HTML 替代方案。<br/>如果您使用的應用程序提供 CSS 支持，您可以嘗試使用 CSS 替代方案，這裡有一個 \<font> 標籤的替代方案：

<p style="color:red">Make this text blue.</p>

---

## Comments 註釋

Markdown 本身並不原生支持註釋，但一些有進取心的人設計了一個解決方案：<br/>將文本放在方括號內，後跟冒號、空格和井號（例如 [comment]: #）。您應該在註釋之前和之後添加空行。<br/>提示：此提示來自 [Stack Overflow](https://stackoverflow.com/questions/4823468/comments-in-markdown)。它已經過同行評審並被數千人使用！

Here's a paragraph that will be visible.

[This is a comment that will be hidden.]: #

And here's another paragraph that's visible.

---

## Admonitions 警告

Markdown 不提供特殊的警告語法，並且大多數 Markdown 應用程序不提供對警告的支持（[MkDocs](https://www.markdownguide.org/tools/mkdocs/) 是一個例外）。

如果您需要添加警告，您可以使用帶有表情符號和強調的區塊引用來創建與您在其他網站上看到的警告類似的內容。

> :warning: **Warning:** Do not push the big red button.

> :memo: **Note:** Sunrises are beautiful.

> :bulb: **Tip:** Remember to appreciate the little things in life.

---

## Image Size 圖片尺寸

圖像的 Markdown 語法不允許指定圖像的寬度和高度。如果需要調整圖像大小並且 Markdown 處理器支持 HTML，則可以使用帶有 width 和 height 屬性的 img HTML 標籤來設置圖像的尺寸（以像素為單位）。

<img src="https://upload.wikimedia.org/wikipedia/commons/a/af/Tux.png" width="100" height="120">

---

## Image Captions 圖片說明

Markdown 本身不原生支持圖像標題，但有兩種可能的解決方法。

### 方法一：使用 HTML

如果您的 Markdown 應用程序支持 HTML，您可以使用 figure 和 figcaption HTML 標籤為圖像添加標題。

<figure>
    <img src="https://upload.wikimedia.org/wikipedia/commons/a/af/Tux.png" alt="Tux">
    <figcaption>This is Tux.</figcaption>
</figure>

### 方法二：使用強調語法

如果您的 Markdown 應用程序不支持 HTML，您可以嘗試將標題直接放在圖像下方並使用強調。

![Tux](https://upload.wikimedia.org/wikipedia/commons/a/af/Tux.png)
_This is Tux._

---

## Link Targets

Markdown 的鏈接不允許您指定目標屬性，但如果您的 Markdown 處理器支持 HTML，則可以使用 HTML 創建這些鏈接。

<a href="https://www.markdownguide.org" target="_blank">Learn Markdown!</a>

---

## Symbols 符號

Markdown 不提供特殊的符號語法。但是，在大多數情況下，您可以將要使用的任何符號複製並粘貼到 Markdown 文檔中。<br/>或者，如果您的 Markdown 應用程序支持 HTML，您可以將 HTML 實體用於您想要使用的任何符號。

| HTML 實體的部分列表             |          |
| ------------------------ | -------- |
| Copyright (©)            | \&copy;  |
| Registered trademark (®) | \&reg;   |
| Trademark (™)            | \&trade; |
| Euro (€)                 | \&euro;  |
| Left arrow (←)           | \&larr;  |
| Up arrow (↑)             | \&uarr;  |
| Right arrow (→)          | \&rarr;  |
| Down arrow (↓)           | \&darr;  |
| Degree (°)               | \&#176;  |
| Pi (π)                   | \&#960;  |

完整 HTML 實體列表參考 [Wikipedia’s page](https://en.wikipedia.org/wiki/List_of_XML_and_HTML_character_entity_references#List_of_character_entity_references_in_HTML)。

---

## Table Formatting 表格格式化

Markdown 表格是出了名的挑剔。您不能使用許多 Markdown 語法元素來格式化表格單元格中的文本。但至少有兩個常見表格問題的解決方法：

### Line Breaks Within Table Cells 在表格單元格內換行

| Syntax    | Description                            |
| --------- | -------------------------------------- |
| Header    | Title                                  |
| Paragraph | First paragraph.<br/>Second paragraph. |

### Lists Within Table Cells 表格單元格內的列表

| Syntax | Description                                                  |
| ------ | ------------------------------------------------------------ |
| Header | Title                                                        |
| List   | Here's a list! <ul><li>Item one.</li><li>Item two.</li></ul> |

---

## Table of Contents 目錄

如果您的 Markdown 應用程序支持標題 ID，您可以使用列表和一些鏈接為 Markdown 文件創建目錄。

---

## Videos 影片

如果您的 Markdown 應用程序支持 HTML，您應該能夠通過複製並粘貼 YouTube 或 Vimeo 等視頻網站提供的 HTML 代碼來將視頻嵌入到您的 Markdown 文件中。

如果您的 Markdown 應用程序不支持 HTML，則無法嵌入視頻，但可以通過添加圖像和視頻鏈接來接近嵌入視頻。您幾乎可以對任何視頻服務上的任何視頻執行此操作。

由於 YouTube 使這一切變得簡單，我們將使用它們作為示例。以這個視頻網址為例：https://www.youtube.com/watch?v=TeQ_TTyLGMs 。URL 的最後部分 (TeQ_TTyLGMs) 是視頻的 ID。我們可以獲取該 ID 並將其放入以下模板中：

```markdown
[![Image alt text](https://img.youtube.com/vi/YOUTUBE-ID/0.jpg)](https://www.youtube.com/watch?v=YOUTUBE-ID)
```

[![Do You Want to Build a Snowman?](https://img.youtube.com/vi/TeQ_TTyLGMs/0.jpg)](https://www.youtube.com/watch?v=TeQ_TTyLGMs)

### Youtube 影片的縮圖

![](https://img.youtube.com/vi/TeQ_TTyLGMs/default.jpg) (120\*90)
![](https://img.youtube.com/vi/TeQ_TTyLGMs/mqdefault.jpg) (320\*180)
![](https://img.youtube.com/vi/TeQ_TTyLGMs/hqdefault.jpg) (480\*360)
![](https://img.youtube.com/vi/TeQ_TTyLGMs/sddefault.jpg) (640\*480)
![](https://img.youtube.com/vi/TeQ_TTyLGMs/maxresdefault.jpg) (1920\*1080)
![](https://img.youtube.com/vi/TeQ_TTyLGMs/0.jpg) (480\*360)
![](https://img.youtube.com/vi/TeQ_TTyLGMs/1.jpg) (120\*90 影片開頭的小截圖)
![](https://img.youtube.com/vi/TeQ_TTyLGMs/2.jpg) (120\*90 是 0.jpg 的縮小截圖)
![](https://img.youtube.com/vi/TeQ_TTyLGMs/3.jpg) (120\*90 影片結尾的小截圖)

`img.youtube.com` 可用稍短的主機名稱 `i3.ytimg.com` 取代。

---

# Markdown Preview Enhanced

- [GitHub](https://github.com/shd101wyy/vscode-markdown-preview-enhanced)
- [Last Releases](https://github.com/shd101wyy/vscode-markdown-preview-enhanced/releases)

---

## Admonition 警告

- [Admonition Reference](https://squidfunk.github.io/mkdocs-material/reference/admonitions/)
  (註：測試結果似乎顯示與官方範例不太一樣！，可改用 [Admonitions 警告](#admonitions-警告))

!!! note This is the admonition title
This is the admonition body

---

## 數學

- 使用 [KaTeX](https://github.com/Khan/KaTeX) 或者 [MathJax](https://github.com/mathjax/MathJax) 來渲染數學表達式。
- `$...$` 或者 `\(...\)` 中的數學表達式將會在行內顯示。
  $f(x)=sin(x)+12$
- `$$...$$` 或者 `\[...\]` 或者 ` ```math` 中的數學表達式將會在塊內顯示。
  $$\sum_{n=1}^{100} n$$

### $\KaTeX$

- [官方網站](https://katex.org/)
  - [GitHub](https://github.com/KaTeX/KaTeX)

---

## Last Releases

### 0.7.3

- Supported pandoc-like code blocks, for example:
  The first class in the {...} will be regarded as the language.

```{.python}
def add(x, y):
  return x + y
```

```{.mermaid}
graph LR
%%这是一条注释，在渲染图中不可见
    A[Hard edge] -->|Link text| B(Round edge)
    B --> C{Decision}
    C -->|One| D[Result one]
    C -->|Two| E[Result two]
```

### 0.7.0

- Added [Kroki](https://kroki.io/) support to render diagrams. This is a beta feature. For example:

  ```ditaa {kroki=true}
  +--------+   +-------+    +-------+
  |        | --+ ditaa +--> |       |
  |  Text  |   +-------+    |diagram|
  |Document|   |!magic!|    |       |
  |     {d}|   |       |    |       |
  +---+----+   +-------+    +-------+
      :                         ^
      |       Lots of work      |
      +-------------------------+
  ```

- Updated mermaid to version 10.4.0, and supported rendering [zenuml](https://mermaid.js.org/syntax/zenuml.html) chart using mermaid.
  (2023/09/08 註：使用 zenuml 後，清單會顯示異常！)

# 筆記

## Basic

- [ ] to-do
- [/] incomplete
- [x] done
- [-] canceled
- [>] forwarded
- [<] scheduling

## Extras

- [?] question
- [!] important
- [*] star
- ["] quote
- [l] location
- [b] bookmark
- [i] information
- [S] savings
- [I] idea
- [p] pros
- [c] cons
- [f] fire
- [k] key
- [w] win
- [u] up
- [d] down
- [D] draft pull request
- [P] open pull request
- [M] merged pull request
