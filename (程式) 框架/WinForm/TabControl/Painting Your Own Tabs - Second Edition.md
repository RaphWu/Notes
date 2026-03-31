---
aliases:
date:
update:
author: The Man from U.N.C.L.E.
language: C#
sourceurl: https://www.codeproject.com/articles/Painting-Your-Own-Tabs-Second-Edition-2
tags:
---

# Introduction  介紹

![[newcustomtabcontrol.png|Sample Image]]

Some years ago, I wrote a custom tab control which has gained quite a following [here](http://www.codeproject.com/KB/dotnet/CustomTabControl.aspx)[[^](http://www.codeproject.com/KB/dotnet/CustomTabControl.aspx)] on Code Project. As with all such controls, it did what I needed it to do, and I haven't touched the code in years. Recently, I needed a tab control for another project so I dug out the old code, screamed a few times at how bad my coding was back then, re-wrote it from scratch and fixed all the bugs that had been reported. I also added missing functionality like alignment of the tabs, and other custom styles, because I needed them for my new project. So, time for a new article to explain what is new. By the way, the eye damaging background to the forms shown here are only used to highlight the transparency issues.
幾年前，我寫了一個自訂選項卡控件，它 [在](http://www.codeproject.com/KB/dotnet/CustomTabControl.aspx) Code Project 上頗受歡迎 [ [^](http://www.codeproject.com/KB/dotnet/CustomTabControl.aspx) ]。就像所有這類控制項一樣，它滿足了我的需求，之後我就沒再動過這段程式碼。最近，我在另一個專案中需要用到選項卡控件，於是我翻出了舊程式碼，一邊懊惱自己當時的糟糕程式碼，一邊從頭重寫，並修復了所有已知的 bug。我還添加了缺少的功能，例如選項卡的對齊方式和其他自訂樣式，因為我的新專案需要這些功能。所以，是時候寫一篇新文章來介紹新增內容了。順便說一句，這裡展示的表單背景圖看起來很刺眼，只是為了突出透明度問題。

I have tested this code against the following .NET Frameworks: 3.0 Sp1, 3.5 Sp1 & 4.0
我已使用以下 .NET Framework 版本測試過此程式碼：3.0 SP1、3.5 SP1 和 4.0。

While the C# version will compile against the 2.0 .NET Framework, the VB.NET version includes the use of inline functions to define a find predicate. As this is not supported in VB.NET 2.0, that line of code will have to be replaced by a `foreach` loop if you wish to compile it for VB.NET 2.0.
雖然 C# 版本可以編譯透過 .NET Framework 2.0，但 VB.NET 版本使用了內嵌函數來定義尋找謂詞。由於 VB.NET 2.0 不支援內聯函數，因此如果要將其編譯通過 VB.NET 2.0，則必須將該行程式碼替換為 `foreach` 循環。

**The compiled assembly has been compiled against .NET Framework 3.5.
編譯後的組件是針對 .NET Framework 3.5 編譯的。**

# The Problem  問題

First, let me explain the problems with the built in `System.Windows.Forms.TabControl`. As you can see from the picture below, the rendering of the tab control is okay when aligned to the top, but even then, it is not perfect.
首先，讓我解釋一下內建的 `System.Windows.Forms.TabControl` 存在的問題。如下圖所示，當選項卡控制項與頂部對齊時，其渲染效果尚可，但即使如此，也不完美。

![[newcustomtabcontrol1.webp|The native tab control]]

The issues are as follows:
問題如下：

1. When set to `TabAlignment.Top` or `TabAlignment.Bottom`, the tab page area has an unsightly white shadow to the bottom right.
    當設定為 `TabAlignment.Top` 或 `TabAlignment.Bottom` 時，選項卡頁面區域的右下角會出現難看的白色陰影。
2. When set to `TabAlignment.Bottom`, the tabs are not attached to the page area. The tab strip is the same as from the top, just displayed at the bottom, whereas the tabs should hang off the bottom of the page area.
    當設定為 `TabAlignment.Bottom` 時，選項卡不會固定在頁面區域內。選項卡條與頂部對齊時相同，僅顯示在底部，而選項卡應該懸垂在頁面區域底部之外。
3. When set to `TabAlignment.Top` or `TabAlignment.Bottom`, the tab page area border is not XP themed.
    設定為 `TabAlignment.Top` 或 `TabAlignment.Bottom` 時，選項卡頁面區域邊框不會採用 XP 主題。
4. When set to `TabAlignment.Left` or `TabAlignment.Right` the tab page area border becomes 3D.
    當設定為 `TabAlignment.Left` 或 `TabAlignment.Right` 時，選項卡頁面區域邊框將變為 3D 立體效果。
5. When set to `TabAlignment.Left` or `TabAlignment.Right` the background becomes a solid gray, rather than transparent.
    當設定為 `TabAlignment.Left` 或 `TabAlignment.Right` 時，背景會變成純灰色，而不是透明的。
6. When set to `TabAlignment.Left` or `TabAlignment.Right` the tabs lose any pretence of styling.
    當設定為 `TabAlignment.Left` 或 `TabAlignment.Right` 時，選項卡將失去所有樣式。
7. When viewed on Windows XP set to `TabAlignment.Left` or `TabAlignment.Right`, the text of the tabs disappears altogether!
    在 Windows XP 系統中，如果將 `TabAlignment.Left` 設為 Left 或 `TabAlignment.Right` ，則製表符的文字會完全消失！
8. It is impossible to turn off hot tracking.
    無法關閉熱追蹤功能。

In addition:  另外：

1. No support for modern styles
    不支援現代風格
2. Not possible to hide the tabs
    無法隱藏標籤頁。
3. No support for disabled tabs
    不支援禁用標籤頁。
4. No drag'n'drop support  不支援拖放功能

# The Solution  解決方案

I did make a few design changes up front this time round. The first of these was to move the control code into the `System` namespaces. I know these are usually reserved for stuff supplied by Microsoft, however [JosÃ© Manuel MenÃ©ndez Poo](http://www.codeproject.com/KB/toolbars/WinFormsRibbon.aspx)[[^](http://www.codeproject.com/KB/toolbars/WinFormsRibbon.aspx)] wrote a brilliant ribbon control and in it, made the observation that having control based stuff in the `System` namespaces is a big advantage when you are coding. No special imports, swap the classes with the underlying .NET classes with ease, and so on. The second was to place all native code interaction in a separate class, just as Microsoft has done inside the .NET Framework. I also decided to try passing the [FXCop](http://msdn.microsoft.com/en-us/library/bb429476%28VS.80%29.aspx)[[^](http://msdn.microsoft.com/en-us/library/bb429476%28VS.80%29.aspx)] test. Most projects I have collaborated on would take years to make them pass [FXCop](http://msdn.microsoft.com/en-us/library/bb429476%28VS.80%29.aspx)[[^](http://msdn.microsoft.com/en-us/library/bb429476%28VS.80%29.aspx)] tests, but I did this one from the start, so the code is theoretically better for it.
這次我提前做了一些設計上的改變。首先，我將控制項程式碼移到了 `System` 命名空間。我知道這些命名空間通常是留給微軟提供的，但是 [José Manuel Menéndez Poo](http://www.codeproject.com/KB/toolbars/WinFormsRibbon.aspx) [ [^](http://www.codeproject.com/KB/toolbars/WinFormsRibbon.aspx) ] 寫了一個非常棒的 Ribbon 控件，並在其中指出，將控件相關的代碼放在 `System` 命名空間中對編碼來說是一個很大的優勢。這樣就不需要特殊的導入語句，可以輕鬆地將類別與底層的 .NET 類別進行替換等等。其次，我把所有與本機程式碼的互動都放在了一個單獨的類別中，就像微軟在 .NET Framework 中所做的一樣。我還決定嘗試通過 [FXCop](http://msdn.microsoft.com/en-us/library/bb429476%28VS.80%29.aspx) [ [^](http://msdn.microsoft.com/en-us/library/bb429476%28VS.80%29.aspx) ] 測試。我參與過的大多數專案都需要花費數年時間才能通過 [FXCop](http://msdn.microsoft.com/en-us/library/bb429476%28VS.80%29.aspx) [ [^](http://msdn.microsoft.com/en-us/library/bb429476%28VS.80%29.aspx) ] 測試，但這個專案我從一開始就做了，所以理論上程式碼會更好。

## Transparency  透明度

First priority was to sort out the transparency issue.
首要任務是解決透明度問題。

In the original version of this control, I did some clever coding to paint the control underneath the tab control to replicate transparency. There are several better ways to achieve transparency, and all of them use far less code! Unfortunately, I soon discovered that they all introduce no end of flicker, so I have ended up with a hybrid solution, using some of the transparency code from the old project.
在這個控制項的原始版本中，我編寫了一些巧妙的程式碼，將控制項繪製在選項卡控制項下方，以模擬透明效果。其實有很多更好的方法可以實現透明效果，而且它們所需的程式碼量都少很多！可惜的是，我很快就發現它們都會引入嚴重的閃爍問題，所以最終我採用了一種混合方案，並藉鑑了舊專案中的一些透明效果程式碼。

```csharp
protected void PaintTransparentBackground(Graphics graphics, Rectangle clipRect)
{
    graphics.Clear(Color.Transparent);
    if (this.Parent != null) {
        clipRect.Offset(this.Location);
        PaintEventArgs e = new PaintEventArgs(graphics, clipRect);
        GraphicsState state = graphics.Save();
        graphics.SmoothingMode = SmoothingMode.HighSpeed;
        try {
            graphics.TranslateTransform((float)-this.Location.X, (float)-this.Location.Y);
            this.InvokePaintBackground(this.Parent, e);
            this.InvokePaint(this.Parent, e);
        }
        finally {
            graphics.Restore(state);
            clipRect.Offset(-this.Location.X, -this.Location.Y);
        }
    }
}
```

This fills the area with a transparent background, then if the parent object is available, we offset the paint origin and call the parent to paint the area under the tabs.
這樣就用透明背景填滿了該區域，然後，如果父物件可用，我們就偏移繪製原點並呼叫父物件來繪製選項卡下的區域。

The normal tricks of ignoring the `WM_ERASEBKGRND` message, changing the `createParams`, or setting the `Region` all fail because they introduce far too much flicker.
忽略 `WM_ERASEBKGRND` 訊息、更改 `createParams` 或設定 `Region` 等常規方法都失敗了，因為它們會引入過多的閃爍。

To fix the alignment rendering, I had to resort to custom painting, but as this is the whole point of the article, I shall deal with it in detail later. For now, here is how the tabs look in the default style with a transparent background, and the painting corrected for all alignments.
為了修復對齊渲染問題，我不得不採用自訂繪製，但由於這正是本文的重點，我將在後面詳細討論。現在，這裡展示的是預設樣式下標籤頁的顯示效果（背景透明），以及針對所有對齊方式修正後的繪製效果。

![[newcustomtabcontrol3.png|A better tab control]]

## Custom Painting  客製化噴漆

Clearly the custom painting of the tabs is the main point of this article. As I explained in my previous article on this subject, the .NET Framework SDK explains how you can set the `TabControl Drawmode` to `OwnerDrawFixed` to paint the tabs yourself; however, the tabs do not resize for longer captions, and there is a really annoying border painted on each tab that you just can't get rid of. I found that most people must have tried the .NET SDK way of painting the tabs and found it didn't work. They then proceeded to write their own tab controls, with all the problems of getting the design time experience just right, and so on. Although it took me a while to stumble across this solution, I think it is much cleaner (but then, I am biased in that respect). I retained the design time experience and the functionality of the underling .NET control, and I made it paint just the way I wanted. I found that you can set the control style to `UserPaint` and do it all yourself. This keeps the auto sizing tabs, and all the tab page functionality remains intact.
顯然，本文的重點在於自訂選項卡的繪製。正如我在之前的文章中所述，.NET Framework SDK 提供了將 `TabControl Drawmode` 設定為 `OwnerDrawFixed` 來手動繪製選項卡的方法；然而，這種方法無法使選項卡自動調整大小以適應較長的標題，並且每個選項卡上都會出現一個非常惱人的邊框，而且無法去除。我發現大多數人可能都嘗試過 .NET SDK 提供的選項卡繪製方法，但發現並不奏效。於是，他們轉而編寫自己的選項卡控件，卻又面臨著如何完美實現設計時體驗等諸多問題。雖然我花了一段時間才偶然發現這個解決方案，但我認為它更加簡潔（當然，我在這方面可能有些偏愛它）。我保留了底層 .NET 控制項的設計時體驗和功能，並且實作了我想要的繪製方式。我發現，只需將控制項樣式設為 `UserPaint` ，就可以完全自訂繪製。這樣可以保留自動調整大小的標籤頁，並且所有標籤頁功能都保持不變。

Before I get stuck in to the implementation details, I should mention that many improvements found their way into the code, thanks to posters on the original article.
在深入探討實作細節之前，我應該提到，由於原始文章下的評論者，程式碼中得到了許多改進。

- Thanks to [Bloggins](https://www.codeproject.com/script/Membership/View.aspx?mid=417076)[[^](https://www.codeproject.com/script/Membership/View.aspx?mid=417076)] for pointing out that you need to paint the entire tab strip every time to make overlapping tabs work.
    感謝 [Bloggins](https://www.codeproject.com/script/Membership/View.aspx?mid=417076) [ [^](https://www.codeproject.com/script/Membership/View.aspx?mid=417076) ] 指出，每次都需要繪製整個標籤條才能使重疊標籤生效。
- Thanks to [martin.riepl](https://www.codeproject.com/script/Membership/View.aspx?mid=1796696)[[^](https://www.codeproject.com/script/Membership/View.aspx?mid=1796696)] who suggested a font sizing fix.
    感謝 [martin.riepl](https://www.codeproject.com/script/Membership/View.aspx?mid=1796696) [ [^](https://www.codeproject.com/script/Membership/View.aspx?mid=1796696) ] 提出的字體大小調整建議。
- Thanks to [Tasosval](https://www.codeproject.com/script/Membership/View.aspx?mid=869405)[[^](https://www.codeproject.com/script/Membership/View.aspx?mid=869405)] for the suggested `ImageClick` event.
    感謝 [Tasosval](https://www.codeproject.com/script/Membership/View.aspx?mid=869405) [ [^](https://www.codeproject.com/script/Membership/View.aspx?mid=869405) ] 建議的 `ImageClick` 事件。
- And a big thank you to [Mick Doherty](https://www.codeproject.com/script/Membership/View.aspx?mid=1597644)[[^](https://www.codeproject.com/script/Membership/View.aspx?mid=1597644 "New Window")] whose work on the `TabControl` provided many [tips](http://dotnetrix.co.uk/tabcontrol.htm)[[^](http://dotnetrix.co.uk/tabcontrol.htm)] that I have integrated into this solution.
    非常感謝 [Mick Doherty](https://www.codeproject.com/script/Membership/View.aspx?mid=1597644) [ [^](https://www.codeproject.com/script/Membership/View.aspx?mid=1597644 "New Window") ]，他在 `TabControl` 方面的工作提供了許多 [技巧](http://dotnetrix.co.uk/tabcontrol.htm) [ [^](http://dotnetrix.co.uk/tabcontrol.htm) ]，我已將這些技巧融入到此解決方案中。

To enhance the rendering, I played around with double buffering using the control styles, the .NET 2.0 `BufferedGraphics` class and all sorts, but the only buffering that worked was painting into an in memory bitmap then pushing that to the screen in one hit. Anything else resulted in a loss of the transparent background, or excessive flicker.
為了提升渲染效果，我嘗試了各種方法，包括使用控制項樣式、.NET 2.0 `BufferedGraphics` 類別等等，但唯一有效的緩衝方式是先將影像繪製到記憶體中的點陣圖，然後一次性將其推送到螢幕。其他任何方法都會導致透明背景遺失或出現過度閃爍。

The original version of this control painted the background, each tab, the tab page border, and finally repainted the selected tab to place it on top. It also had special code to paint the first tab differently and to only paint the overlap for the selected tab. As this new version supports any level of overlap, or none at all, I decided that all tabs must be treated equally. Therefore this version paints each tab except the selected one, going right to left, and finished with the selected tab. This means that all tabs cap paint their overlapping parts and be happy in the knowledge that the next tab will cover up the bits that should be hidden away. Painting a tab therefore includes the tab page, complete border, tab background, text and image (if required). The tab page with its border, along with the text, image, and tab background colouring are standard throughout the styles, so the only part to customise for a new style is the path describing the border of the tab for each orientation.
此控制項的原始版本會繪製背景、每個選項卡、選項卡頁邊框，最後重繪選取的選項卡並將其置於最上方。它還包含特殊程式碼，用於以不同的方式繪製第一個選項卡，並僅繪製選取選項卡的重疊部分。由於新版本支援任何程度的重疊（包括完全不重疊），我決定對所有選項卡一視同仁。因此，此版本會從右到左繪製除選取選項卡之外的每個選項卡，最後繪製選取選項卡。這意味著所有選項卡都會繪製其重疊部分，並且下一個選項卡會覆蓋應該隱藏的部分。因此，繪製選項卡包括選項卡頁面、完整邊框、選項卡背景、文字和圖像（如果需要）。選項卡頁面及其邊框，以及文字、圖像和選項卡背景顏色在所有樣式中都是標準的，因此對於新樣式，唯一需要自訂的部分是描述每個方向選項卡邊框的路徑。

Note that if the tab page is set to `Enabled = false` then the tab is painted greyed out and is not selectable. However, this is a runtime check so if the initial tab is disabled, it will still appear selected on starting your application.
請注意，如果選項卡頁面的 `Enabled = false` ，則該標籤將顯示為灰色且不可選。但是，這是一個運行時檢查，因此即使初始選項卡處於停用狀態，在啟動應用程式時它仍然會顯示為選取狀態。

The standard method on the `TabControl` to get the tab rectangle is too small for our purposes. We take this rectangle and expand it to include the border edge of the tab page. The first tab must then be moved a couple of pixels as it defaults to start at the edge of the control not at the border of the tab page. We then reduce the size of non-selected tabs, and stretch all except for the first tab to allow for any overlap. This complete tab rectangle is then passed to an overrideable method to generate the appropriate shape for the tab.
`TabControl` 的標準方法所取得的選項卡矩形太小，不符合我們的需求。我們取得這個矩形，並將其擴展到包含選項卡頁面的邊框。由於第一個選項卡預設從控制項邊緣而非選項卡頁面邊框開始，因此需要將其移動幾個像素。然後，我們縮小未選取選項卡的尺寸，並拉伸除第一個選項卡之外的所有選項卡，以允許重疊。最後，將這個完整的選項卡矩形傳遞給可重寫的方法，以產生適當的選項卡形狀。

A further problem I uncovered is how to paint the tabs correctly when in multiline mode. The problems are twofold. Firstly, you need to paint the tabs on the outer rows before you paint the inner rows or they will vanish when over-painted. Secondly, it would be nice to get all rows to line up correctly on the left hand side.
我發現的另一個問題是如何在多行模式下正確繪製製表符。問題有兩個面向。首先，必須先繪製外側行的製表符，然後再繪製內側行的製表符，否則內側行的製表符會被覆蓋而消失。其次，最好能讓所有行在左側正確對齊。

Help is at hand with a new method `GetTabRow`. This enables us to paint the tabs row by row. An additional check for being the left most tab in the row is easily implemented, though I have included a method to get the row/column of the tab in the multirow tab array.
新增的 `GetTabRow` 方法可以幫我們解決這個問題。它允許我們逐行繪製選項卡。雖然很容易實現檢查選項卡是否為該行中最左側選項卡的功能，但我還是提供了一個方法來獲取選項卡在多行選項卡數組中的行/列。

Another useful addition is the ability to make hide individual tabs using `HideTab` and `ShowTab`. While not strictly speaking part of the painting process, they none-the-less effect the display. Most important is that unless used, they incur no overhead as the backup copy of the tab references is only initialised on hiding a tab for the first time. Care must also be taken to restore tabs in their original positions relative to the currently visible tabs.
另一個實用功能是可以使用 `HideTab` 和 `ShowTab` 隱藏單一標籤頁。雖然嚴格來說它們並非繪製過程的一部分，但仍會影響顯示效果。最重要的是，除非使用，否則它們不會產生任何額外開銷，因為標籤頁引用的備份副本僅在首次隱藏標籤頁時初始化。此外，還必須注意將標籤頁恢復到相對於目前可見標籤頁的原始位置。

## A More Versatile and Extendable Styling Solution

更通用、可擴展的造型解決方案

As I developed this control further, there was a growing need for more flexibility, meaning more properties, and styles. Clearly putting all this into the one class was going to become unmanageable so I extracted a large part of the Tab painting code into another class. This `TabStyleProvider` class acts as the base class for any new style, and has a factory method for getting instances of style providers. The `TabControl` now has only the `DisplayStyle` property, which changes the provider. Properties on the provider can then be customised further in the designer as required.
隨著我對這個控制項的進一步開發，對靈活性的需求日益增長，這意味著需要更多的屬性和樣式。顯然，將所有這些放在一個類別中會變得難以管理，所以我將大部分選項卡繪製程式碼提取到另一個類別中。這個 `TabStyleProvider` 類別作為所有新樣式的基類，並包含一個用於獲取樣式提供者實例的工廠方法。現在， `TabControl` 只剩下 `DisplayStyle` 屬性，用於更改提供者。之後，可以根據需要在設計器中進一步自訂提供者的屬性。

![[newcustomtabcontrol4.png|class diagram]]

Here are a few examples of the kinds of tab styles you can achieve.
以下是一些您可以實現的標籤樣式範例。

The currently supported styles via the `DisplayStyle` property are:
目前透過 `DisplayStyle` 屬性支援的樣式有：

- None - _No tabs visible_
    無 - _未顯示任何標籤頁_
- Default - _Identical to the .NET default rendering on Vista_
    預設 - _與 Vista 系統上 .NET 的預設渲染方式相同。_
- VisualStudio - _Imitating the Visual Studio 2005 tabs_
    Visual Studio - 模仿 Visual Studio 2005 的選項卡

    ![[newcustomtabcontrol5.webp]]
    
- Chrome - _Imitating the Google Chrome Tabs_
    Chrome - 模仿 Google Chrome 標籤頁

    ![[newcustomtabcontrol6.webp]]
    
- IE8 - _Imitating the Internet Explorer 8 Tabs_
    IE8 - 模仿 Internet Explorer 8 的標籤頁

    ![[newcustomtabcontrol7.webp]]
    
- Rounded - _My personal favourite, which looks good aligned left_
    圓角－我個人最喜歡這種形狀，左對齊效果很好。

    ![[newcustomtabcontrol2.webp]]
    
- Angled - Not quite like _Google Chrome_
    傾斜的——不太像谷歌瀏覽器

    ![[newcustomtabcontrol8.webp]]
    
- VS2010 - _Imitating the Visual Studio 2010 Tabs_
    VS2010 - 模仿 Visual Studio 2010 的選項卡

    ![[newcustomtabcontrol9.webp]]

# Using the Code  使用程式碼

To use this code just copy the contents of the _TabControl_ folder, and sub folders into your project and use it. For your convenience, I have included code in C# and in VB.NET, although the VB.NET version does not have the complete demo code, just the complete control code. The files for the VB.NET version are the same names as for C# just with _.vb_ at the end.
要使用此程式碼，只需將 _TabControl_ 資料夾及其子資料夾的內容複製到您的專案中即可。為了方便起見，我提供了 C# 和 VB.NET 程式碼，但 VB.NET 版本僅包含完整的控制項程式碼，不包含完整的示範程式碼。 VB.NET 版本的檔案名稱與 C# 版本相同，只是在檔案名稱末尾新增了 _.vb_ 。

If you prefer to use Managed C++, I used a tool to convert an early version of this control into a Managed C++ project for Visual Studio 2005. I have tested it works be referencing, and using the resulting DLL, but as my knowledge of C++ is limited, I won't pretend I can support it in any way, and I won't be updating that one with changes. But it should be enough to get things started for you.
如果您更傾向於使用託管 C++，我使用了一個工具將此控制項的早期版本轉換為適用於 Visual Studio 2005 的託管 C++ 專案。我已經通過引用和使用生成的 DLL 測試過它的功能，但由於我的 C++ 知識有限，我無法提供任何支持，也不會對其進行更新。不過，它應該足以幫助您入門。

Alternatively, download the [compiled assembly](https://www.codeproject.com/articles/newcustomtabcontrol/newcustomtabcontrolassembly.zip)[[^](https://www.codeproject.com/articles/newcustomtabcontrol/newcustomtabcontrolassembly.zip)] and just add a reference to it, or add it to your toolbox.
或者，下載 [已編譯的程序集](https://www.codeproject.com/articles/newcustomtabcontrol/newcustomtabcontrolassembly.zip) [ [^](https://www.codeproject.com/articles/newcustomtabcontrol/newcustomtabcontrolassembly.zip) ]，然後添加對它的引用，或將其添加到您的工具箱中。

Special note for VB.NET developers. As [T_uRRiCA_N](http://www.codeproject.com/script/Membership/View.aspx?mid=3974557)[[^](http://www.codeproject.com/script/Membership/View.aspx?mid=3974557)] noted, VB.NET and C# are different in how they use the project properties, specifically the **Root Namespace** value in the Project Properties. In C# projects, this property indicates the namespace that will be automatically added to the source code of any class you create in the project after setting the root namespace value. In VB.NET, the same property is not used at design time, but at compile time. The VB.NET compiler prepends this namespace to every class and resource in the assembly. So then class, `System.Windows.Forms.CustomTabControl` in C# could compile as `MyApplication.System.Windows.Forms.CustomTabControl` in VB.NET. There are two ways round this issue. The first is to not set the root namespace for VB.NET projects. This is my preferred option as I want to control my namespaces, not be at the whim of the compiler. The second option is to place your code in a custom namespace and ensure you add the appropriate `Imports` to each file.
VB.NET 開發人員特別注意。如 [T_uRRiCA_N](http://www.codeproject.com/script/Membership/View.aspx?mid=3974557) [ [^](http://www.codeproject.com/script/Membership/View.aspx?mid=3974557) ] 指出的，VB.NET 和 C# 在使用專案屬性（特別是專案屬性中的**根命名空間**值）時有所不同。在 C# 專案中，此屬性指示在設定根命名空間值後，將自動新增至專案中建立的任何類別的原始程式碼中的命名空間。在 VB.NET 中，該屬性不在設計時使用，而是在編譯時使用。 VB.NET 編譯器會將此命名空間加入到程式集中的每個類別和資源的前面。因此，C# 中的類別 `System.Windows.Forms.CustomTabControl` 在 VB.NET 中可能會編譯為 `MyApplication.System.Windows.Forms.CustomTabControl` 。有兩種方法可以解決這個問題。第一種方法是不對 VB.NET 專案設定根命名空間。這是我的首選方法，因為我希望控制命名空間，而不是受制於編譯器。第二種方法是將程式碼放在自訂命名空間中，並確保為每個檔案新增對應的 `Imports` 。

# Properties and Events Exposed by the Control

控制項公開的屬性和事件

Some properties, such as `HotTrack` and `Padding` I have moved to the `TabStyle` provider classes. Others, such as the `Appearance` property I have totally hidden as it is no use to use here. The properties exposed by the `TabStyleProvider` include:
一些屬性，例如 `HotTrack` 和 `Padding` 我已經移到了 `TabStyle` 提供者類別中。另一些屬性，例如 `Appearance` 屬性，我已經完全隱藏了，因為它在這裡沒有用。 TabStyleProvider 公開 `TabStyleProvider` 屬性包括：

- `BorderColor` - controls the border colour of non-selected tabs
    `BorderColor` - 控制未選取標籤頁的邊框顏色
- `BorderColorHot` - controls the border colour of tabs under the mouse when `HotTracking` is on
    `BorderColorHot` - 控制啟用 `HotTracking` 時滑鼠指標下方標籤頁的邊框顏色
- `BorderColorSelected` - controls the border colour of selected tabs. Defaults to an XP Themed border colour matching the standard textbox border
    `BorderColorSelected` - 控制選取標籤頁的邊框顏色。預設值為與標準文字方塊邊框顏色相符的 XP 主題邊框顏色。
- `CloserColor` - controls the colour of the closer cross if displayed on tabs
    `CloserColor` - 控制標籤頁上顯示的關閉十字的顏色
- `CloserColorActive` - controls the colour of closer cross when the mouse if over the closer area on tabs
    `CloserColorActive` - 控制滑鼠停留在標籤頁的「關閉」區域時，關閉十字的顏色。
- `FocusColor` - controls the colour of the focus indicator if required
    `FocusColor` - 控制對焦指示器的顏色（如果需要）
- `FocusTrack` - controls whether tabs display a focus indicator when the tab control has focus
    `FocusTrack` - 控制標籤控制項取得焦點時是否顯示焦點指示器。
- `TextColor` - controls the colour of the text displayed on tabs
    `TextColor` - 控制標籤頁上顯示的文字顏色
- `TextColorDisabled` - controls the colour of the text displayed on disabled tabs
    `TextColorDisabled` - 控制停用標籤頁上顯示的文字顏色
- `TextColorSelected` - controls the colour of the text displayed on selected tabs
    `TextColorSelected` - 控制選取標籤頁上顯示的文字顏色
- `HotTrack` - controls whether tabs change colour when the mouse is over them
    `HotTrack` - 控制滑鼠停留在標籤頁上時標籤頁是否改變顏色。
- `ImageAlign` - controls the position of any image displayed on the tabs
    `ImageAlign` - 控制標籤頁上顯示的任何影像的位置
- `Opacity` - controls the opacity of the entire tab control
    `Opacity` - 控制整個選項卡控制項的不透明度
- `Overlap` - controls how far the tabs extend to the left (or top) covering the previous tab
    `Overlap` - 控制標籤頁向左（或向上）延伸的距離，以覆蓋前一個標籤頁。
- `Padding` - controls the spacing around the text, giving extra height or width to the tabs
    `Padding` - 控製文字周圍的間距，為製表符增加額外的高度或寬度。
- `Radius` - controls the curvature of rounded tabs, or the spread of angled tabs
    `Radius` - 控制圓角標籤的曲率或斜角標籤的展開角度
- `ShowTabCloser` - controls whether tabs display a cross to close the tab
    `ShowTabCloser` - 控制標籤頁是否顯示關閉按鈕（叉號）

I have also added a couple of new events and properties to the `tabcontrol` itself.
我還為選項 `tabcontrol` 本身添加了一些新的事件和屬性。

- `ActiveIndex` - returns the index of the `TabPage` related to the tab currently under the mouse, or `-1` if the tab is disabled or the mouse is not near a tab
    `ActiveIndex` - 傳回與目前滑鼠懸停的選項卡相關的 `TabPage` 的索引；如果選項卡已停用或滑鼠不在選項卡附近，則傳回 `-1`
- `ActiveTab` - returns the `TabPage` related to the tab currently under the mouse, or `null` if the tab is disabled or the mouse is not near a tab
    `ActiveTab` - 傳回與目前滑鼠指標指向的選項卡相關的 `TabPage` ，如果選項卡已停用或滑鼠指標不在選項卡附近，則傳回 `null`

- `HScroll` - fired when the tab scroller is clicked
    `HScroll` - 點擊標籤滾動條時觸發
- `TabImageClick` - fired when an image on a tab is clicked
    `TabImageClick` - 當點擊標籤頁上的圖片時觸發。
- `TabClosing` - fired when the closer on a tab is clicked. This event can be cancelled.
    `TabClosing` - 當點擊標籤頁上的關閉按鈕時觸發。此事件可以取消。

And few new methods are also exposed:
此外，也鮮有新方法被提出：

- `GetTabPosition` - returns the row and column of the tab within the multi row tab array as a `Point`
    `GetTabPosition` - 傳回多行選項卡陣列中選項卡所在的行和列，以 `Point` 類型傳回。
- `GetTabRow` - returns the row of the tab within the multi row tab array
    `GetTabRow` - 傳回多行選項卡數組中選項卡所在的行
- `isFirstTabInRow` - returns `true` if the specified tab index is the first tab in its row
    `isFirstTabInRow` - 如果指定的選項卡索引是其所在行中的第一個選項卡，則傳回 `true`
- `HideTab` - Removes the tab specified by reference, key or index from the visible tabs
    `HideTab` - 從可見製表符清單中移除由參考、鍵或索引指定的製表符。
- `ShowTab` - Restores the tab specified by reference, key or index to the visible tabs, or adds the tab if it is not present
    `ShowTab` - 將透過引用、鍵或索引指定的選項卡還原到可見選項卡清單中，如果選項卡不存在，則新增該選項卡。

# Tips  尖端

## Padding  填充

In order to gain space for the tab closer, and allow for the curvature of the tabs I have had to adjust the actual `Padding` of the tabs. The apparent `Padding` is from before this adjustment. In some cases, you may find that the text does not wrap in the correct place, in which case adjust the `Padding` appropriately.
為了讓標籤頁閉合器騰出空間，並適應標籤頁的弧度，我必須調整標籤頁的實際內 `Padding` 。圖中顯示的內 `Padding` 是調整前的數值。在某些情況下，您可能會發現文字沒有正確換行，此時請相應地調整 `Padding` 。

## Context Menus  內容選單

I have not customised the tab scroller, in this version. However, you could use the workaround from [Mick Doherty](http://www.codeproject.com/script/Membership/Profiles.aspx?mid=1597644)[[^](http://www.codeproject.com/script/Membership/Profiles.aspx?mid=1597644 "New Window")] for adding navigation buttons at [Add a Custom Scroller to TabControl](http://www.dotnetrix.co.uk/tabcontrols.html)[[^](http://www.dotnetrix.co.uk/tabcontrols.html "New Window")].
在這個版本中，我沒有自訂選項卡捲軸。但是，您可以參考 [Mick Doherty](http://www.codeproject.com/script/Membership/Profiles.aspx?mid=1597644) [ [^](http://www.codeproject.com/script/Membership/Profiles.aspx?mid=1597644 "New Window") ] 的方法，在 [「向 TabControl 新增自訂捲軸」](http://www.dotnetrix.co.uk/tabcontrols.html) [ [^](http://www.dotnetrix.co.uk/tabcontrols.html "New Window") ] 中新增導覽按鈕。

For tab closing, I personally like to add a context menu to the tab control, though I have now implemented tab closer functionality. In the context menu Opening event, you add the following code to stop it opening for disabled tabs, and select the active tab on right mouse actions. Your should always set the selected tab to be the active tab in this scenario as by the time you click the close option, your mouse will have moved away from the tab,
對於標籤頁關閉，我個人喜歡在標籤頁控制項中新增上下文選單，不過現在我已經實現了標籤頁關閉功能。在上下文選單的開啟事件中，您可以新增以下程式碼，以防止在標籤頁被停用時開啟上下文選單，並在滑鼠右鍵操作時選擇目前活動的標籤頁。在這種情況下，您應該始終將選定的標籤頁設定為活動標籤頁，因為當您按一下關閉選項時，滑鼠可能已經移開了。

```cs
void ContextMenuStripOpening(object sender, CancelEventArgs e)
{
    if (this.customTabControl1.ActiveIndex > -1){
        this.customTabControl1.SelectedIndex = this.customTabControl1.ActiveIndex;
    } else {
        e.Cancel = true;
    }
}
```

I would then have a '**Close**' option on the context menu and in the `Click` event, place the following code:
然後，我會在上下文選單中新增「 **關閉** 」選項，並在 `Click` 事件中放入以下程式碼：

```cs
void CloseToolStripMenuItemClick(object sender, EventArgs e)
{
    TabPage pageToRemove = this.customTabControl1.SelectedTab;
    if (pageToRemove != null){
        this.customTabControl1.TabPages.Remove(pageToRemove);
        pageToRemove.Dispose();
    }
} 
```

## Implementing Drag 'n' Drop

實現拖放功能

The default `TabControl` does not support drag 'n' drop. I had implemented it in a separate application, so I have included it here also. To turn it on, just set `AllowDrop` to `true` on any instance of the `CustomTabControl` that you want to drag 'n' drop. You can then drag tabs around on the control, or drag them from one `CustomTabControl` to another.
預設的 `TabControl` 控制項不支援拖放功能。我已在另一個應用程式中實現了拖放功能，因此也將其包含在此處。要啟用此功能，只需將要支援拖放的 `CustomTabControl` 實例的 `AllowDrop` 設為 `true` 即可。之後，您就可以在控制項上拖曳選項卡，或將選項卡從一個 `CustomTabControl` 拖曳到另一個。

The basics of drag 'n' drop are the same for all controls. Start the drag on mouse down, enable drag effects when the mouse is over areas that can accept the drop, and handle the drop to move stuff around. You can find a good example of it on the [Microsoft support site](http://support.microsoft.com/kb/307966)[[^](http://support.microsoft.com/kb/307966 "New Window")]. Of interest however is how to move the tabs around.
所有控制的拖放基本原理都相同。滑鼠按下時開始拖曳，滑鼠懸停在可放置區域時啟用拖曳效果，然後透過拖曳操作來移動元素。您可以在 [微軟支援網站](http://support.microsoft.com/kb/307966) 上找到一個很好的範例 [ [^](http://support.microsoft.com/kb/307966 "New Window") ]。不過，我們更感興趣的是如何移動選項卡。

As you can see from the example code below we face two problems. Firstly, we must ensure we remove the tab from its parent, not the current `CustomTabControl`. This is vital for dragging from one `CustomTabControl` to another. Secondly, we must insert the tab in front of the tab currently under the mouse, remembering that if the tab came from a point to the left, then removing it will change the positions of all the tabs.
如下面的範例程式碼所示，我們面臨兩個問題。首先，我們必須確保從其父控制項（而非目前的 `CustomTabControl` 中移除選項卡。這對於從一個 `CustomTabControl` 拖曳到另一個 CustomTabControl 至關重要。其次，我們必須將選項卡插入到滑鼠目前指向的選項卡的前面，並記住，如果選項卡是從左側移動過來的，那麼移除它會改變所有選項卡的位置。

```cs
protected override void OnDragDrop(DragEventArgs drgevent)
{
    base.OnDragDrop(drgevent);
    
    //	Test for a TabPage in the drag data
    if (drgevent.Data.GetDataPresent(typeof(TabPage))){
        drgevent.Effect = DragDropEffects.Move;
        
        //	Extract the TabPage from the drag data
        TabPage dragTab = (TabPage)drgevent.Data.GetData(typeof(TabPage));
        
        //	Do not drop on the place you started
        if (this.ActiveTab == dragTab){
        return;
    }
    
    //	Capture insert point and adjust for removal of tab
    //	We cannot assess this after removal as differing tab sizes will cause
    //	inaccuracies in the activeTab at insert point.
    //	We will insert just before the active tab.
    int insertPoint = this.ActiveIndex;
    if (dragTab.Parent.Equals(this) && this.TabPages.IndexOf(dragTab) < insertPoint){
        insertPoint --;
    }
    if (insertPoint < 0){
        insertPoint = 0;
    }
    
    //	Remove from current position (could be another tabcontrol)
    ((TabControl)dragTab.Parent).TabPages.Remove(dragTab);
    
    //	Add to current position
    this.TabPages.Insert(insertPoint, dragTab);
    this.SelectedTab = dragTab;
    }
}
```

## Mono Support  單聲道支持

The downloads posted here are all compiled against .NET Framework 3.5, however I have tested a separate build against Mono 2.7.
這裡發布的下載檔案都是基於 .NET Framework 3.5 編譯的，但我已經針對 Mono 2.7 測試了一個單獨的版本。

The Mono implementation of the `System.Windows.Forms` namespace is not a perfect replica of the Microsoft implementation. When compiling against Mono, you will discover the following differences in operation of this control.
Mono 對 `System.Windows.Forms` 命名空間的實作並非 Microsoft 實作的完美復刻。在 Mono 環境下編譯時，您會發現此控制項在操作上有以下差異。

1. The Mono implementation of the `TabControl` does not support HotTracking so it is best to set `HotTrack` to `false` or you will get some odd effects.
    `TabControl` 的 Mono 實作不支援 HotTracking，因此最好將 `HotTrack` 設為 `false` ，否則會出現一些奇怪的效果。
2. Because of the lack of `Hottracking`, you will find that Drag'n'Drop does not work either.
    由於缺少 `Hottracking` ，您會發現拖放功能也無法使用。

If anyone is working with Mono and finds fixes for these issues that are compatible with the standard implementation, please let me know and I will integrate them into the main source here.
如果有人在使用 Mono 並且找到了與標準實作相容的這些問題的修復方法，請告訴我，我會將它們合併到這裡的主原始程式碼中。

Even so, the look of the standard Mono implementation is terrible, making this control a massive improvement, even in the default style!
即便如此，標準的 Mono 實現的外觀仍然很糟糕，因此即使是預設樣式，這個控制也是一個巨大的改進！

Having stated the limitations, there was only one change I had to make in order to make this control Mono portable. I had to remove all P/Invoke calls. To do this, I turned the `UserPaint` style back on, with the associated need to handle font changes correctly. This is because my previous implementation used `BeginPaint` and `EndPaint` to perform my own painting. I also wrote my own implementation of `SendMessage`, which is used to get the active tab, and for font updates.
在說明了這些限制之後，為了讓這個控制能夠移植到 Mono 平台，我只需要做一個改動：移除所有 P/Invoke 呼叫。為此，我重新啟用了 `UserPaint` 樣式，同時也需要正確處理字體變更。這是因為我之前的實作使用了 `BeginPaint` 和 `EndPaint` 來執行自訂繪製操作。我還編寫了自己的 `SendMessage` 實現，用於獲取當前活動標籤頁以及進行字體更新。

As you can see below, all we do is create the message object and invoke `WndProc` on the control. This ensures we are on the correct thread, and means no unmanaged code. However, this only works in our specific implementation here as the control is our control. This method cannot be used for sending messages to unmanaged controls.
如下所示，我們所做的只是建立訊息物件並在控制項上呼叫 `WndProc` 。這確保了我們在正確的執行緒上執行操作，並且意味著沒有使用非託管程式碼。但是，這僅適用於我們此處的特定實現，因為控制項是我們自己的控制項。此方法不能用於向非託管控制項傳送訊息。

```cs
public static IntPtr SendMessage (IntPtr hWnd, int msg, IntPtr wParam, IntPtr lParam)
{
    // This Method replaces the User32 method SendMessage, but will only work for sending
    // messages to Managed controls.
    Control control = Control.FromHandle(hWnd);
    if (control == null){
        return IntPtr.Zero;
    }
    
    Message message = new Message();
    message.HWnd = hWnd;
    message.LParam = lParam;
    message.WParam = wParam;
    message.Msg = msg;
    
    MethodInfo wproc = control.GetType().GetMethod("WndProc"
        , BindingFlags.NonPublic 
        | BindingFlags.InvokeMethod 
        | BindingFlags.FlattenHierarchy 
        | BindingFlags.IgnoreCase 
        | BindingFlags.Instance);
    
    object[] args = new object[] {message};
    wproc.Invoke(control, args);
    
    return ((Message)args[0]).Result;
}
```

## Mnemonic Support  記憶輔助

Mnemonic support is built in to this implementation of the `TabControl`, however the Mnemonic characters are only displayed, and only respond to key presses if the `KeyPreview` property is set to `true` on the parent form.
`TabControl` 的此實現內建了助記符支持，但是只有當父窗體上的 `KeyPreview` 屬性設為 `true` 時，助記符字元才會顯示並回應按鍵。

## Creating a New Style Provider

創建一個新的風格提供商

As an example of how to implement a style provider, here is how I implemented the Visual Studio Provider.
以下是我實作 Visual Studio Provider 的一個範例，說明如何實作樣式提供者。

The first thing to do is subclass the `TabStyleProvider`, creating the `TabStyleVisualStudioProvider` class. Add a new item to the `TabStyle` enum, and add an appropriate case to the `CreateProvider` factory method in the `TabStyleProvider` class as shown below:
首先，需要對 `TabStyleProvider` 進行子類化，建立 `TabStyleVisualStudioProvider` 類別。然後，向 `TabStyle` 枚舉新增一個新項，並在 `TabStyleProvider` 類別的 `CreateProvider` 工廠方法中加入對應的 case，如下所示：

```cs
public static TabStyleProvider CreateProvider(CustomTabControl tabControl)
{
    TabStyleProvider provider;
    // Depending on the display style of the tabControl generate an appropriate provider.
    switch (tabControl.DisplayStyle) {
    // New case for Visual Studio style
    case TabStyle.VisualStudio:
    provider = new TabStyleVisualStudioProvider(tabControl);
    break;
    case TabStyle.None:
    provider = new TabStyleNoneProvider(tabControl);
// ...
```

We need to set a few defaults in the constructor of our new provider. Images should align right, careful measurement shows that the tabs overlap by seven pixels, and we need to insert some padding to allow for the slope of the leading edge. Of course, these tabs are also not as tall as the standard tabs, so we drop the vertical padding right down.
我們需要在新提供者的建構函數中設定一些預設值。影像應該右對齊，仔細測量後發現標籤頁重疊了 7 個像素，我們需要添加一些內邊距來適應前緣的傾斜角度。當然，這些標籤頁的高度也比標準標籤頁矮，所以我們需要大幅降低垂直內邊距。

```cs
public TabStyleVisualStudioProvider(CustomTabControl tabControl) : base(tabControl)
{
    this._ImageAlign = ContentAlignment.MiddleRight; this._Overlap = 7;
    // Must set after the _Radius as this is used in the calculations of the actual padding
    this.Padding = new Point(11, 1);
}
```

Finally, we override the `AddTabBorder` method in our new provider class. This supplies the shape for the tabs. Visual Studio 2005 tabs have a leading edge that slopes at 45 degrees before curving into the top line of the tab. There is also a roundness to all the corners.
最後，我們在新的提供者類別中重寫了 `AddTabBorder` 方法。此方法用於設定選項卡的形狀。 Visual Studio 2005 的選項卡具有一個前緣，該前緣以 45 度角傾斜，然後彎曲過渡到選項卡的頂線。另外，所有角都是圓角。

```cs
public override void AddTabBorder(GraphicsPath path, Rectangle tabBounds)
{
    switch (this._TabControl.Alignment) {
        case TabAlignment.Top:
            // bottom left of tab up the leading slope
            path.AddLine(tabBounds.X, tabBounds.Bottom, tabBounds.X + 
                         tabBounds.Height - 4, tabBounds.Y + 2);
            // along the top, leaving a gap that is auto completed for us
            path.AddLine(tabBounds.X + tabBounds.Height, 
                         tabBounds.Y, tabBounds.Right - 3, tabBounds.Y);
            // round the top right corner
            path.AddArc(tabBounds.Right - 6, tabBounds.Y, 6, 6, 270, 90);
            // back down the right end
            path.AddLine(tabBounds.Right, tabBounds.Y + 3, tabBounds.Right, tabBounds.Bottom);
            // no need to complete the figure as this path joins into the border for the entire tab
        break;
// ...
```

I have only included the code for top aligned tabs, but as you can see from the other providers included, you need to supply code for each alignment. It is simplest if you imagine drawing round clockwise. Check out one of the existing providers for an example.
我只提供了頂部對齊製表符的程式碼，但正如您從其他範例提供者看到的，您需要為每種對齊方式提供程式碼。最簡單的理解方式是想像順時針畫圓。您可以參考現有提供者的範例。

As you can see adding a new style is simple, the main dificulty is getting the outline correct for all orientations. As an additional complication some styles, such as the IE8 style include overriding methods to paint the tabs in different colours, and custom painting of the closer button.
如您所見，添加新樣式很簡單，主要困難在於如何確保所有方向的輪廓都正確。此外，某些樣式（例如 IE8 樣式）還包含重寫標籤頁顏色和自訂關閉按鈕顏色的方法，這增加了複雜性。

# History  歷史

- 21/9/2010
    - Fixed bugs in resizing code
        修復了調整大小代碼中的錯誤
    - Corrected direction of Images and default alignment of Images and Closers for `RightToLeftLayout` tabs
        修正了 `RightToLeftLayout` 標籤中影像的方向以及影像和關閉按鈕的預設對齊方式。
- 14/9/2010
    - `RightToLeftLayout`, `RightToLeft` and Mnemonic support added
        新增 `RightToLeftLayout` 、 `RightToLeft` 和助記符支持
    - Improved painting for better Mono compatability
        改良的塗裝工藝，提升了與單色系統的兼容性
- 11/9/2010
    - Removed calls to P/Invoke functions in order to be portable to the Mono Framework
        為了能夠移植到 Mono 框架，移除了對 P/Invoke 函數的呼叫。
    
- 10/9/2010
    - Drag'n'Drop support added
        新增拖放支援
- 8/9/2010
    - Release of new style, VS2010
        發布新款式，VS2010
    - Updated Visual Studio style to correct the colouring of the closer
        更新了 Visual Studio 樣式，以修正關閉按鈕的顏色。
    - New properties added to control the text colour, and border colour
        新增屬性用於控製文字顏色和邊框顏色
    - Improved glass effect  玻璃效果改善
- 7/9/2010
    - Fixed bug affecting the `TabControl` size when anchored inside an MDI child form that has been maximized prior to calling `Show`
        修正了當 `TabControl` 錨定在已最大化的 MDI 子視窗中時，影響其大小的錯誤 `Show`
- 2/9/2010
    - Fixed bug where resizing the control left an artefact at the bottom
        修正了調整控制項大小後底部出現瑕疵的錯誤。
    - Fixed bug where anchored controls on a `TabPage` would move on re-opening the designer
        修正了 `TabPage` 上錨定控制項在重新開啟設計器時會移動的錯誤。
    - Refactored the `CustomTabControl` and associated code into a separate assembly
        將 `CustomTabControl` 及其相關程式碼重構到一個單獨的程式集中。
    - New methods added, `HideTab` and `ShowTab`
        新增方法： `HideTab` 和 `ShowTab`
- 23/8/2010
    - Large scale refactoring to a style provider model, and addition of tab closing capability
        大規模重構為樣式提供者模型，並添加了製表符關閉功能
- 23/7/2010
    - New properties, `FocusColor` and `FocusTrack` to better indicate focus
        新增屬性 `FocusColor` 和 `FocusTrack` 以便更好地指示焦點
    - Improved positioning of the images
        改進了影像定位
- 22/7/2010
    - Clipped the sided so that tabs do not paint beyond the edges of the tabPage
        裁剪了側邊，使標籤頁不會超出標籤頁邊緣。
    - In Default style, the selected tab is now bigger than the rest as in the underlying .NET control
        在預設樣式中，選取的選項卡現在比其他選項卡更大，這與底層 .NET 控制項中的情況相同。
- 21/7/2010
    - Two new properties, `BorderColor` and `SelectedBorderColor`
        新增兩個屬性： `BorderColor` 和 `SelectedBorderColor`
    - Update of rendering code to reduce flicker
        更新渲染程式碼以減少閃爍
- 20/7/2010
    - Update of article regarding the root namespace project property
        更新有關根命名空間項目屬性的文章
- 8/7/2010
    - Release of new CHROME style
        發布新款 CHROME 款式
    - Update to the handling of the painting to reduce flicker when changing tabs
        更新繪畫處理方式，以減少切換標籤頁時的閃爍。
- 7/7/2010
    - Order of painting corrected for `MultiLine` TabControls
        已修正 `MultiLine` 卡控制項的繪製順序
    - The overlap was running the wrong way
        重疊部分的方向錯了
- 6/7/2010
    - `MultiLine` TabControls now paint correctly in all alignments
        `MultiLine` 符控制項現在在所有對齊方式下都能正確繪製。
    - Method `GetTabPosition` added
        已加入 `GetTabPosition` 方法
    - Method `GetTabRow` added for more efficient painting
        增加了 `GetTabRow` 方法，以提高繪製效率。
- 5/7/2010
    - Hot tracking implemented (when `HotTrack` is set to `true`)
        熱追蹤已實現（當 `HotTrack` 設定為 `true` 時）
    - `TabImageClick` event added
        已新增 `TabImageClick` 事件
- 5/7/2010
    - Source for C++ version (DLL containing the `CustomTabControl` only) added
        已新增 C++ 版本原始碼（僅包含 `CustomTabControl` 的 DLL）。
- 2/7/2010
    - First release on CodeProject
        CodeProject 上的首次發布

# License  執照

This article, along with any associated source code and files, is licensed under [The Code Project Open License (CPOL)](http://www.codeproject.com/cpol)
本文及其相關原始碼和文件均根據 [Code Project 開源許可證 (CPOL)](http://www.codeproject.com/cpol) 授權。

(Now with support for Visual Studio 2010 Style, Drag'n'Drop, Mnemonics, `RightToLeftLayout`, and compilable against the Mono framework.)
（現在支援 Visual Studio 2010 樣式、拖曳、助記符、 `RightToLeftLayout` 佈局，並且可以針對 Mono 框架進行編譯。）

- [Download C# and VB.NET demo source code - 181.9 KB
- [Download C++ DLL source code - 2.03 MB
- [Download compiled CustomTabControl assembly - 18.33 KB
