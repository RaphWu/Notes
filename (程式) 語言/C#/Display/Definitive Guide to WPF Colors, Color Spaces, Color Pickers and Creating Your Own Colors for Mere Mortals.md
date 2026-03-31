---
date: 2021-03-17
update:
author: Peter Huber SG
language: C#, XAML
sourceurl: https://medium.com/design-bootcamp/definitive-guide-to-wpf-colors-color-spaces-color-pickers-and-creating-your-own-colors-for-mere-f480935c6e94
tags:
  - WPF
  - Color
---

# Definitive guide to WPF colors, color spaces, color pickers, and creating your own colors

WPF 顏色、色彩空間、顏色選擇器以及建立自訂顏色的權威指南

PeterHuberSg

I don’t know about you, but I struggled now for many years with the limited number of colors available in the Colors class, trying to get matching colors with ColorPickers and understanding the various color models. To simplify my life, I wrote few small methods which allow me to change any color towards white and black and another one to mix colors. With this, I get nicely matching colors, a bit like gradients as in GradientBrush.
我不知道你們的情況如何，但我多年來一直苦於 Colors 類中有限的顏色數量，嘗試用 ColorPickers 調出匹配的顏色，還要理解各種顏色模型。為了簡化操作，我編寫了幾個小方法，一個可以將任何顏色轉換為白色或黑色，另一個可以混合顏色。這樣一來，我就能得到漂亮的搭配顏色，有點像 GradientBrush 中的漸層效果。

Then I had the bad idea to write an article about this. To explain how my methods work, I had no choice but to investigate in detail what is going on. So the biggest part of this article is about colors, color models, hue, brightness and stuff, but in easy terms a software developer can understand without a math of physics degree.
然後我突發奇想，決定寫篇文章來解釋這件事。為了說明我的方法是如何運作的，我不得不深入研究背後的原理。所以這篇文章的大部分內容都圍繞著顏色、顏色模型、色調、亮度等等展開，但會用淺顯易懂的語言，即使是沒有數學或物理學位的軟體開發人員也能理解。

Color space HSB: Hue, Saturation and Brightness
色彩空間 HSB：色相、飽和度與亮度
As we probably all know, color on a computer screen gets created by pixels and each pixel consists of 3 dots which can emit the light Red, Green and Blue, which explains the names R, G and B of these dots. However, here is already the first misunderstanding, because in actual fact, G is not Colors.Green but Colors.Lime. The human eye has three different kind of receptors for colors and the colors (hue) of R, G and B are chosen to match well with these receptors, which happens to be the lime green and not the "normal" green. With RGB pixels, a monitor can produce most colors a human eye can differentiate. Well, almost. Of course, the "strength" (brightness) of these colors are rather limited when compared to other light sources as for example, the sun.
我們都知道，電腦螢幕上的顏色是由像素產生的，每個像素由三個點組成，分別可以發出紅、綠、藍三種顏色的光，這也解釋了為什麼這些點被命名為 R、G、B。然而，這裡就存在著第一個誤解，因為實際上，G 並非 Colors.Green ，而是 Colors.Lime 。人眼有三種不同的顏色感受器，R、G、B 的顏色（色調）經過精心選擇，以更好地與這些感受器相匹配，而這種色調恰好是青檸綠，而非「普通」的綠色。借助 RGB 像素，顯示器可以顯示人眼能夠區分的大部分顏色。當然，與太陽等其他光源相比，這些顏色的「強度」（亮度）相當有限。

Each color is defined by how much light each of the 3 dots emits. One dot can have a value between 0 (emitting nothing) and 255 (or 0xFF in hexadecimal) emitting at full strength. To see one of the primary colors, for example red, R gets set to 255 and the G and B to zero, which gives the brightest red possible. If we want a darker red, we just lower the value of R. Once R is 0, the resulting color is black, since no dot is emitting anything. This is an example of changing the brightness of a color, one of the three properties every color has. Interestingly, brightness is not defined from 0 to 255, but from 0 to 1 or 0 to 100%.
每種顏色都由三個點各自發出的光量決定。每個點的亮度值介於 0（完全不發光）和 255（十六進位 0xFF）之間，255 表示最大亮度。例如，要顯示紅色，需要將 R 值設為 255，並將 G 和 B 值設為 0，這樣就能得到最亮的紅色。如果想要更暗的紅色，只需降低 R 值即可。當 R 值為 0 時，由於所有點都不發光，因此得到的顏色是黑色。這就是改變顏色亮度的一個例子，亮度是每種顏色都具有的三個屬性之一。有趣的是，亮度值並非定義在 0 到 255 之間，而是定義在 0 到 1 或 0% 到 100% 之間。

Another property of a color is called hue. Hue assigns values for yellow, orange, red, etc. We can produce hues at their purest when one of the 3 dots is 255, one is 0 and the “middle” (third) one can have any value. For example, R 225, G 255, B 0 combines red and green and the result is ? Strangely enough, the result is yellow ! The reason is that when 2 lights shine on the same spot, the spot gets brighter not darker. If we mix R, G and B shining at their maximum, we get a white light. This is the opposite of painting colors. If we mix many painting colors together, we get something dark grayish and ugly.
顏色的另一個屬性叫做色相 。色相賦予黃色、橙色、紅色等顏色不同的數值。當三個點中一個為 255，一個為 0，而「中間」（第三個）點可以取任意值時，我們可以得到最純正的色相。例如，R 225、G 255、B 0，紅色和綠色混合，結果是什麼？奇怪的是，結果是黃色！原因是，當兩束光照射到同一個點上時，這個點會變亮而不是變暗。如果我們混合 R、G 和 B 三個點的最大亮度，我們會得到白光。這與繪畫顏色的混合方式相反。如果我們混合多種顏料，我們會得到暗灰色且難看的顏色。

R:FF G:FF B:00 = Yellow
R:FF G:FF B:00 = 黃色

R:00 G:FF B:FF = Cyan
R:00 G:FF B:FF = 青色

R:FF G:00 B:FF = Magenta
R:FF G:00 B:FF = 洋紅色

These, by the way, are the brightest colors a monitor can produce, since they use 2 dots fully turned on as opposed to red, green and blue, where only 1 dot is used. For all other hues, the second strongest dot emits at less than 255. It is this “middle” dot, which is not 255 and not 0 which defines the hue. By incrementing this middle dot slowly from 0 to 255, we get all possible hues, for example, between red (FF0000) and yellow (FFFF00). In total, there are 6 such transitions:
順便一提，這些是顯示器能產生的最亮的顏色，因為它們使用了兩個完全點亮的像素點，而紅、綠、藍三色則只使用一個像素點。對於所有其他色調，亮度第二高的像素點的亮度都低於 255。正是這個既不是 255 也不是 0 的「中間」像素點定義了色調。透過將這個中間像素點的亮度從 0 緩慢增加到 255，我們可以得到所有可能的色調，例如，介於紅色 (FF0000) 和黃色 (FFFF00) 之間的色調。總共有 6 種這樣的過渡：

255 Red and some Green: Red to Yellow
255 Red and some Blue: Red to Violet
255 Green and some Red: Green to Yellow
255 Green and some Blue: Green to BlueGreen
255 Blue and some Green: Blue to BlueGreen
255 Blue and some Red: Blue to Violet
When we vary R, G or B by small increments, we get something like a rainbow:
當我們以微小的增量改變 R、G 或 B 的值時，就會得到類似彩虹的效果：

Press enter or click to view image in full size

On the left and right end, there is each time a red. For this reason, the rainbow is often drawn as a circle. Hue is defined in degrees with red being a 0 and 360 degrees.
彩虹的左右兩端總是紅色。因此，彩虹通常被畫成一個圓。色調以度數定義，紅色對應0度到360度。

The third color property is called saturation. So far, we have dealt only with fully saturated colors, i.e., the darkest dot was 0. If we want to make a fully saturated color brighter and bring it finally close to white, we need to lower the saturation by increasing the intensity of all three dots proportionally closer to 255. To decrease the saturation from 100% to 50%, we have to half the difference between the present value to 255. Example:
第三個顏色屬性稱為飽和度 。到目前為止，我們只處理了完全飽和的顏色，即最暗的點為 0。如果我們想要讓完全飽和的顏色更亮，最終接近白色，我們需要降低飽和度，方法是按比例增加所有三個點的強度，使其更接近 255。要將飽和度從 100% 降低到 50%，我們需要將目前值與 255 之間的差異減半。例如：

Present value: R 255, G 128, B 0

Decrease saturation from 1 to .5
New R value = 255 + 0.5 _ (255-255) = 255
New G value = 128 + 0.5 _ (255-128) = 192 (or 191 depending on rounding)
New B value = 0 + 0.5 \* (255-0) = 128 (or 127 depending on rounding)

Decrease saturation from 1 to 0 (=white)
New R value = 255 + 1 _ (255-255) = 255
New G value = 128 + 1 _ (255-128) = 255
New B value = 0 + 1 \* (255-0) = 255
The color red drawn from 100% saturation to 0%:
紅色從飽和度 100% 逐漸​​過渡到 0%：

Now we can draw the rainbow again and include some saturation and brightness variation. The x-axis increases the hue from 0 to 360. In the middle is each hue shown with 100% saturation and 100% brightness. In the upper half, the brightness stays 100% and the saturation is decreased to 0%, which results in white. In the lower half, the saturation stays 100% and the brightness is decreased to 0%, which results in black.
現在我們可以再次繪製彩虹，並添加一些飽和度和亮度變化。 x 軸表示色調從 0 到 360 的遞增。中間部分顯示的是每個色調在 100% 飽和度和 100% 亮度下的狀態。在上半部分，亮度保持 100%，飽和度降低到 0%，從而得到白色。在下半部分，飽和度保持 100%，亮度降低到 0%，從而得到黑色。

Press enter or click to view image in full size

Notice how yellow, cyan and magenta can keep their color longer before they turn white or black than the other hues. They are the strongest colors, because 2 dots shine at full intensity.
注意觀察黃色、青色和洋紅色，它們比其他顏色更能保持顏色，不易變白或變黑。它們是顏色最鮮豔的顏色，因為兩個光點就能以最大強度發光。

A hue with close to 0% saturation and 100% brightness looks white. White has the value FFFFFF and the hue and saturation are undefined.
飽和度接近 0%、亮度接近 100% 的色調看起來是白色的。白色的值為 FFFFFF，色調和飽和度均未定義。

A hue with 100% saturation and close to 0% brightness looks black. Black has the value 000000 and the hue and saturation are undefined.
飽和度為 100%、亮度接近 0% 的色調看起來是黑色。黑色的值為 000000，色調和飽和度均未定義。

A color where all 3 dots shine with the same strength looks gray. A possible value could be 808080.
當三個點亮度相同時，顏色看起來是灰色。一個可能的數值是 808080。

Note: For gray, i.e., R, G and B have the same value, neither hue nor saturation is defined, only brightness has a meaningful value. We could also say, black, gray and white are not colors. Black 000000 has a brightness of 0, White FFFFFF has a brightness of 1. That brightness alone controls how white, gray and black look has a strange consequence as we can see in the next picture.
注意 ：對於灰色，即紅、綠、藍三色值相同，既不定義色調也不定義飽和度，只有亮度值有意義。我們也可以說，黑色、灰色和白色並非顏色。黑色 000000 的亮度為 0，白色 FFFFFF 的亮度為 1。僅憑亮度就決定了白色、灰色和黑色的外觀，這會產生一種奇特的後果，正如我們在下一張圖中看到的那樣。

Have we covered now all colors a monitor can display? Actually, we have shown less than 1% of all possible R, G and B combinations, i.e., only those where one dot is 255 (100% brightness) or one dot is 0 (100% saturation). Let’s say we change first the saturation of a color FF8000 (kind of orange red) to 50%, we get FFC080. When we then change the brightness to 50%, we get 806040. The hue is still orange red, but the color is now much closer to a dark gray.
我們是否已經涵蓋了顯示器可以顯示的所有顏色？實際上，我們只展示了不到 1% 的 R、G、B 組合，也就是說，我們只展示了其中一個點為 255（100% 亮度）或一個點為 0（100% 飽和度）的組合。假設我們先將顏色 FF8000（一種橘紅色）的飽和度改為 50%，得到 FFC080。然後，當我們將亮度改為 50% 時，得到 806040。色調仍然是橙紅色，但顏色現在更接近深灰色。

Here is a picture of the color Red with all possible saturation and brightness combinations:
下圖展示了紅色在所有可能的飽和度和亮度組合下的效果：

I guess this is the most confusing picture in this article. Basically, I wanted to change the color on the y axis (top to bottom) from red to black, meaning the brightness from 1 to 0, and on the x axis (left to right) from red to white, meaning saturation from 1 to 0. I would have expected that white and black would mix too and that the bottom right corner would be gray. But no such luck. As soon R, G, B have the same value, hue and saturation lose their meaning. Only brightness has an influence on the colors white, gray and black (right border). Even worse, the whole lower border is just black, because once brightness is 0, hue and saturation become meaningless again.
我想這應該是這篇文章裡最讓人困惑的圖了。我原本想把 y 軸（從上往下）的顏色從紅色變成黑色，也就是亮度從 1 變成 0；把 x 軸（從左到右）的顏色從紅色變成白色，也就是飽和度從 1 變成 0。我原本以為白色和黑色會混合，右下角會變成灰色。但事與願違。一旦 R、G、B 三個顏色值相同，色相和飽和度就失去了意義。只有亮度會影響白色、灰色和黑色（右邊緣）。更糟的是，整個下緣都變成了黑色，因為一旦亮度為 0，色相和飽和度就再次失效了。

If you feel confused too, welcome to the club. But that’s how the HSB color scheme works. When manipulating just RGB values, it is kind of difficult to tell how the result will look like (remember that mixing R and G results in yellow ?). When manipulating colors in the HSB color space, yellow stays yellow, as long you only change saturation and brightness, until the brightness becomes 1 (white) or 0 (black), at which time the hue and saturation is lost.
如果你也感到困惑，別擔心，你不是一個人。這就是 HSB 色彩空間的工作原理。如果只調整 RGB 值，很難預估最終效果（還記得 R 和 G 混合會得到黃色嗎？）。在 HSB 色彩空間中調整顏色時，只要你只改變飽和度和亮度，黃色就會保持黃色，直到光澤變為 1（白色）或 0（黑色），此時色調和飽和度就會失去。

HSL Color Space HSL 色彩空間
There is another color space called HSL Hue, Saturation and Luminosity. Hue is the same as in HSB, Saturation doesn’t go towards white but gray and Luminosity goes from 0=black, 0.5=gray to 1= white. HSL was useful in the transition from black and white to color TVs. A black and white tv just displayed the L value, while a color tv used HSL.
還有一種叫做 HSL 的色彩空間，它包含色相、飽和度和亮度三個部分。色相與 HSB 相同，飽和度不是趨向白色而是趨向灰色，亮度則從 0（黑色）、0.5（灰色）到 1（白色）。 HSL 在黑白電視過渡到彩色電視的過程中發揮了重要作用。黑白電視只顯示亮度值（L 值），而彩色電視則使用 HSL。

Color Pickers 顏色選擇器
In the past, I always struggled to understand how the color pickers are supposed to work and why they failed sometimes. Understanding now hue, saturation and brightness and how they are related to RGB colors makes the color picker easier to understand.
過去，我一直很難理解顏色選擇器的工作原理以及它們有時會失效的原因。現在，我了解色調、飽和度和亮度以及它們與 RGB 顏色之間的關係，這讓顏色選擇器更容易理解了。

PowerPoint 2010 Color Picker
PowerPoint 2010 顏色選擇器

PowerPoint uses the HSL color space. In the color selection area, they display all hues horizontally, vertically they display the saturation. In the HSL color space, a saturation of 0 is gray, therefore the complete lower border is grey. On the right is a scrollbar which changes the luminosity, 0 meaning black, 128 meaning gray and 255 meaning white (it does not use 0–100% but 0–255). At 0 and 255, hue and saturation lose their meaning. The purest color is at luminosity 128.
PowerPoint 使用 HSL 色彩空間。在顏色選擇區域中，水平方向顯示所有色調，垂直方向顯示飽和度。在 HSL 色彩空間中，飽和度為 0 表示灰色，因此整個下方邊框為灰色。右側的捲軸用於調整亮度，0 表示黑色，128 表示灰色，255 表示白色（它不使用 0-100% 的範圍，而是使用 0-255）。亮度為 0 和 255 時，色調和飽和度不再有意義。最純淨的顏色亮度為 128。

Get PeterHuberSg’s stories in your inbox
訂閱 PeterHuberSg 的文章，直接傳送到您的信箱。
Join Medium for free to get updates from this writer.
免費加入 Medium 即可獲得這位作者的最新動態。

Enter your email
Subscribe 訂閱

Remember me for faster sign in
記住我以便更快登入

Choosing for example a blue in HSL, then reducing the luminosity to 0 and switching to RGB display displays properly 0,0,0. Increasing R a bit then setting it again to 0 and switching back to HSV shows now 0 for Hue and Sat, which should be undefined, of course. A hue of 0 means red, but black has no hue. The pointer in the color area still shows the originally chosen blue, instead of read which is hue 0. This was not just confusing for me, but I finally crashed PowerPoint when I kept playing with black, gray and white and switching between RGB and HSL view.
例如，在 HSL 色彩空間中選擇藍色，然後將亮度調至 0 並切換到 RGB 顯示模式，此時會正確顯示 (0,0,0)。稍微增加 R 值，然後再次將其調至 0 並切換回 HSV 色彩空間，此時色調和飽和度都顯示為 0，當然應該是未定義的 。色調 0 表示紅色，但黑色沒有色調。顏色區域中的指針仍然顯示最初選擇的藍色，而不是色調為 0 的紅色。這不僅讓我感到困惑，而且當我反覆嘗試黑色、灰色和白色並在 RGB 和 HSL 視圖之間切換時，最終導致 PowerPoint 崩潰。

Paint.net 4.2 Color Picker
Paint.net 4.2 顏色選擇器

Working with the color picker from paint.net proved to be easier. They use the HSV color model, which is the same as the HSB color model, they just changed the word brightness to volume. I appreciate that they display the RGB values and the HSV values in the same window, this makes it much easier to understand how changing one HSV value influences the RGB values. It displays all available hues on the border of the color circle and in the middle is white, meaning 0% saturation. To make colors darker, one has to change the slider of the Volume (brightness) parameter. Of course, it also displays a hue of zero for black, gray and white, but at least it didn’t crash when I played with these values.
使用 paint.net 的顏色選擇器更方便。它採用的是 HSV 色彩模型，與 HSB 色彩模型基本上相同，只是把 「亮度 」改成了 「飽和度」 。我很喜歡它在同一個視窗中同時顯示 RGB 值和 HSV 值，這樣更容易理解改變一個 HSV 值會如何影響 RGB 值。色環邊緣會顯示所有可用的色調，中間是白色，代表 0% 飽和度。要使顏色變暗，需要調整“飽和度”（亮度）參數的滑桿。當然，黑色、灰色和白色也會顯示為零飽和度，但至少在我調整這些值時程式並沒有崩潰。

WinUI ColorPicker WinUI 顏色選擇器
The sad fact is that WPF has no color picker. It’s like Microsoft gave up on WPF for many years, trying to force us to write UWP applications instead. Many developers thought “no thank you” and stayed with WPF. So now Microsoft is introducing XAML Islands allowing WPF projects using “newer” controls like a color picker, which should have been included in WPF right from the beginning. I haven’t used the WinUI ColorPicker in a project, but I run it in the XAML Controls Gallery:
令人遺憾的是，WPF 沒有顏色選擇器。微軟似乎多年來一直放棄 WPF，試圖強迫我們轉而編寫 UWP 應用程式。許多開發者對此不屑一顧，繼續使用 WPF。如今，微軟推出了 XAML Islands，允許 WPF 專案使用「更新」的控件，例如顏色選擇器——而顏色選擇器本應從一開始就包含在 WPF 中。我雖然沒有在專案中使用過 WinUI ColorPicker ，但我在 XAML 控制項庫中運行過它：

It works a bit like the PowerPoint color picker but using HSV (HSB) instead, meaning the lower part of the color area is white, not gray. The lower scrollbar allows to change the V value (brightness). When set to black, Hue and Saturation stick to their latest values, even when a different color gets chosen in the color area later on. When Value (brightness) gets increased, the circle in the color area jumps back to the old hue. Strange, but at least no crashing.
它的工作方式有點像 PowerPoint 的顏色選擇器，但使用的是 HSV（HSB）色彩空間，這意味著顏色區域的下半部是白色，而不是灰色。底部的滾動條可以調整 V 值（亮度）。當設定為黑色時，色相和飽和度會保持其最新值，即使之後在顏色區域中選擇了不同的顏色。當亮度值增加時，顏色區域中的圓圈會跳回先前的色相。雖然有點奇怪，但至少不會崩潰。

The Colors Class 顏色課
The Colors class gives some standard colors. They were chosen by committees mixing some different color schemes with sometimes strange results. For example, Colors.Gray is darker than Colors.DarkGray. Strange, right?
Colors 類別提供了一些標準顏色。這些顏色是由委員會混合了各種不同的配色方案後選出的，有時會產生一些奇怪的結果。例如， Colors.Gray 比 Colors.DarkGray 更深 。很奇怪，對吧？

Or 2 different names represent actually the same color, like Aqua (00FFFF) and Cyan (00FFFF) or Fuchsia (FF00FF) and Magenta (FF00FF). Unfortunately, the Colors help page displays the colors alphabetically, which makes them easy to find if you know the name, but one has great difficulties to tell which colors are close to each other or matching:
或者，兩個不同的名稱實際上代表的是同一種顏色，例如 Aqua (00FFFF) 和 Cyan (00FFFF)，或 Fuchsia (FF00FF) 和 Magenta (FF00FF)。遺憾的是，「顏色」幫助頁面是按字母順序顯示顏色的，如果您知道名稱，這很容易找到它們，但很難判斷哪些顏色彼此接近或相同：

Press enter or click to view image in full size

So I spent some time and sorted them vertically by their hue, then horizontally by their brightness and their saturation. Here is the result listing exactly the same colors like in the Colors help page:
所以我花了一些時間，先按色調垂直排序，再按亮度和飽和度水平排序。以下是結果，其中列出的顏色與 Colors 幫助頁面中的顏色完全相同：

Press enter or click to view image in full size

Generating Your Own Color With Precision (Code of this Article)
精確產生您自己的顏色（本文程式碼）
Making Colors Brighter or Darker (Decreasing Saturation and or Brightness)
使顏色更亮或更暗（降低飽和度和/或亮度）
When I design a new application and decide about the color scheme to use, I usually cannot use the color palette provided by the Colors class. Often, I need the same hue with shades (different saturation and brightness). To do that takes surprisingly few lines of code. Here is the method to decrease the saturation (make it brighter) or decrease the brightness (make it darker) of any color:
當我設計一個新應用程式並決定使用哪種配色方案時，通常無法直接使用 Colors 類別提供的調色板。很多時候，我需要的是同一色相的不同深淺（不同的飽和度和亮度）。而實現這一點只需要很少的程式碼。以下是降低任何色彩飽和度（使其更亮）或降低亮度（使其更暗）的方法：

/// <summary>
/// Makes the color lighter if factor>0 and darker if factor<0. 1 returns white, -1 returns
/// black.
/// </summary>
public static Color GetBrighterOrDarker(this Color color, double factor) {
if (factor<-1) throw new Exception($"Factor {factor} must be greater equal -1.");
  if (factor>1) throw new Exception($"Factor {factor} must be smaller equal 1.");

if (factor==0) return color;

if (factor<0) {
//make color darker, changer brightness
factor += 1;
return Color.FromArgb(
color.A,
(byte)(color.R*factor),
(byte)(color.G*factor),
(byte)(color.B*factor));
} else {
//make color lighter, change saturation
return Color.FromArgb(
color.A,
(byte)(color.R + (255-color.R)*factor),
(byte)(color.G + (255-color.G)*factor),
(byte)(color.B + (255-color.B)*factor));
}
}
Amazing with how few lines saturation and brightness can be changed. A bit of a challenge is to get the saturation calculation right, for which this method comes in handy.
令人驚訝的是，只需幾行程式碼就能調整飽和度和亮度。要準確計算飽和度有點難度，而這個方法正好可以解決這個問題。

Red, Green, Blue with factor -1 to 1:
紅、綠、藍，係數為-1到1：

To get stronger shining colors, do not use green, which is not 100% saturated, but use yellow, magenta and cyan instead:
為了獲得更鮮豔的色彩，不要使用飽和度不是100%的綠色，而要改用黃色、洋紅色和青色：

Note that applying first a factor of 0.5 and then one of -0.5 does not result in the original color. The first call changes the saturation, the second the brightness.
注意，先應用 0.5 的係數，再應用 -0.5 的係數，並不能得到原始顏色。第一次呼叫改變的是飽和度，第二次呼叫改變的是亮度。

What I like about using this method is:
我喜歡這種方法的原因是：

I can increase, decrease the changes in small, controlled steps and see the results in the GUI.
我可以以小步、可控的方式增加或減少變化，並在圖形使用者介面中查看結果。
I can easily create shadows and highlights, which should have the same hue, but different saturation and brightness.
我可以輕鬆創建陰影和高光，它們應該具有相同的色調，但飽和度和亮度不同。
Mixing Hues 混合色調
Usually, a user interface should not use too many hues, but a few might be fine and some hues mixed from them. These two methods serve this purpose, the first mixing two colors half, half, the second allowing to use more of one color than the other:
通常，使用者介面不應使用過多色調，但幾種色調是可以接受的，並且可以從中混合一些色調。以下兩種方法可以實現這一目的：第一種方法是將兩種顏色各佔一半混合，第二種方法允許使用比另一種顏色更多的色調。

/// <summary>
/// Mixes 2 colors equally
/// </summary>
public static Color Mix(this Color color1, Color color2) {
return Mix(color1, 0.5, color2);
}

/// <summary>
/// Mixes factor*color1 with (1-factor)*color2.
/// </summary>
public static Color Mix(this Color color1, double factor, Color color2) {
if (factor<0) throw new Exception($"Factor {factor} must be greater equal 0.");
  if (factor>1) throw new Exception($"Factor {factor} must be smaller equal 1.");

if (factor==0) return color2;
if (factor==1) return color1;

var factor1 = 1 - factor;
return Color.FromArgb(
(byte)((color1.A _ factor + color2.A _ factor1)),
(byte)((color1.R _ factor + color2.R _ factor1)),
(byte)((color1.G _ factor + color2.G _ factor1)),
(byte)((color1.B _ factor + color2.B _ factor1)));
}
That’s all what’s needed to generate nicely matching colors. The first picture shows how each each “main” color gets mixed with each other “main” color to various degrees, again for red, green and blue:
這就是產生顏色匹配良好的所有步驟。第一張圖展示了每種「主色」如何與其他每種「主色」以不同程度混合，同樣以紅色、綠色和藍色為例：

However, it might be better if you use yellow, magenta and cyan instead:
不過，或許改用黃色、洋紅色和青色會更好：

Here I really feel the colors are nicer than mixing red, green and blue. Of course, they might be too pure. A GUI often uses kind of grayish hues, which can easily be made after mixing by using GetBrighterOrDarker().
我覺得這裡的顏色比紅、綠、藍混合起來更好看。當然，它們可能太純了。圖形使用者介面（GUI）經常使用一些偏灰的色調，這可以透過使用 GetBrighterOrDarker() 函數輕鬆實現。

Getting Hue, Saturation and Brightness of a RGB Color
取得 RGB 顏色的色調、飽和度和亮度
I made few more methods I needed for writing this article, which might be useful too. The first calculates hue, saturation and brightness of a RGB color:
為了撰寫本文，我還寫了一些其他方法，這些方法或許也很有用。第一個方法可以計算 RGB 顏色的 hue 、 saturation 和 brightness ：

/// <summary>
/// Returns the hue, saturation and brightness of color
/// </summary>
public static (int Hue, double Saturation, double Brightness)GetHSB(this Color color) {
int max = Math.Max(color.R, Math.Max(color.G, color.B));
int min = Math.Min(color.R, Math.Min(color.G, color.B));
int hue = 0;//for black, gray or white, hue could be actually any number, but usually 0 is
//assign, which means red
if (max-min!=0) {
//not black, gray or white
int maxMinDif = max-min;
if (max==color.R) {
#pragma warning disable IDE0045 // Convert to conditional expression
if (color.G>=color.B) {
#pragma warning restore IDE0045
hue = 60 _ (color.G-color.B)/maxMinDif;
} else {
hue = 60 _ (color.G-color.B)/maxMinDif + 360;
}
} else if (max==color.G) {
hue = 60 _ (color.B-color.R)/maxMinDif + 120;
} else if(max == color.B) {
hue = 60 _ (color.R-color.G)/maxMinDif + 240;
}
}

double saturation = (max == 0) ? 0.0 : (1.0-((double)min/(double)max));

return (hue, saturation, (double)max/0xFF);
}
I copied this code from CodeProject: Manipulating colors in .NET Part 1, which actually won “Best C# article of May 2007” and “improved” it slightly. For example, I feel integer 0 to 360 are enough to enumerate all hues. The original code uses floating point numbers and there you can have an unlimited number of hues. This might make the calculation less susceptible to rounding errors, but I guess one can’t see the difference.
我從 CodeProject 的「.NET 色彩操作（第一部分）」這篇文章中複製了這段程式碼，這篇文章曾榮獲「2007 年 5 月最佳 C# 文章」獎，並對其進行了一些「改進」。例如，我認為使用 0 到 360 的整數就足以列舉所有色調。原始碼使用的是浮點數，因此色調的數量是無限的。這樣做或許能降低計算中出現捨入誤差的可能性，但我認為實際效果可能不明顯。

Increasing Saturation and Brightness to 100% for Any Color
將任意顏色的飽和度和亮度提高到 100%
In my approach of choosing matching colors, it helps to start with “pure” colors which have a saturation and brightness of 100%, which then later are used for mixing and making them darker or brighter. The following method takes any RGB color and returns a RGB color with the same hue, but 100% saturation and brightness:
在我選擇配色方案時，首先使用飽和度和亮度均為 100% 的“純色”，然後再用這些純色進行混合，使其變暗或變亮。以下方法接受任意 RGB 顏色，並傳回一個色調相同但飽和度和亮度均為 100% 的 RGB 顏色：

/// <summary>
/// Returns a color with the same hue, but brightness and saturation increased to 100%.
/// </summary>
public static Color ToFullColor(this Color color) {
//step 1: increase brightness to 100%
var max = Math.Max(color.R, Math.Max(color.G, color.B));
var min = Math.Min(color.R, Math.Min(color.G, color.B));
if (max==min) {
//for black, gray or white return white
return Color.FromArgb(color.A, 0xFF, 0xFF, 0xFF);
}

double rBright = (double)color.R _ 255 / max;
double gBright = (double)color.G _ 255 / max;
double bBright = (double)color.B \* 255 / max;

//step2: increase saturation to 100%
//lower smallest R, G, B component to zero and adjust second smallest color accordingly
//p = (smallest R, G, B component) / 255
//(255-FullColor.SecondComponent) \* p + FullColor.SecondComponent = color.SecondComponent
//FullColor.SecondComponent = (color.SecondComponent-255p)/(1-p)
if (color.R==max) {
if (color.G==min) {
double p = gBright / 255;
return Color.FromArgb(color.A, 0xFF, 0, (byte)((bBright-gBright)/(1-p)));
} else {
double p = bBright / 255;
return Color.FromArgb(color.A, 0xFF, (byte)((gBright-bBright)/(1-p)), 0);
}
} else if (color.G==max) {
if (color.R==min) {
double p = rBright / 255;
return Color.FromArgb(color.A, 0, 0xFF, (byte)((bBright-rBright)/(1-p)));
} else {
double p = bBright / 255;
return Color.FromArgb(color.A, (byte)((rBright-bBright)/(1-p)), 0xFF, 0);
}
} else {
if (color.R==min) {
double p = rBright / 255;
return Color.FromArgb(color.A, 0, (byte)((gBright-rBright)/(1-p)), 0xFF);
} else {
double p = bBright / 255;
return Color.FromArgb(color.A, (byte)((rBright-bBright)/(1-p)), 0, 0xFF);
}
}
}
I wrote this method myself. The math here is a little bit more demanding and I hope I got it right. You can easily verify it by using any color picker. Please let me know if you find any discrepancies.
這個方法是我自己寫的。這裡的數學計算稍微複雜一些，我希望我沒有出錯。你可以用任何顏色選擇器輕鬆驗證。如果你發現任何錯誤，請告訴我。

Download the Sample Code 下載範例程式碼
The article already contains the methods which help you to create your own colors. Download the sample code from CodeProject to see how they can be used and to see the graphics in this article shining brightly in their original size. They do look nice and you can adjust them to your own needs.
本文已包含建立自訂顏色的方法。您可以從 CodeProject 下載範例程式碼，查看其使用方法，並欣賞本文中的圖形在原始尺寸下的精美效果。這些圖形看起來很漂亮，您可以根據需要進行調整。

Download demo — 1.3 MB
下載試用版 — 1.3 MB
Recommended Reading 推薦閱讀
If you kept reading until here, you might be interested in other WPF related articles I wrote on CodeProject. The last two are not WPF related, but the email POP3/MIME article is my most popular(over 1 million reads) and the Debugging in real time (!) I find is my most amazing article:
如果你一直讀到這裡，你可能也會對我在 CodeProject 上寫的其他 WPF 相關文章感興趣。最後兩篇雖然與 WPF 無關，但其中一篇關於電子郵件 POP3/MIME 的文章是我最受歡迎的（閱讀量超過 100 萬），而另一篇關於實時調試的文章（！）則是我認為最精彩的文章：

Guide to WPF DataGrid Formatting Using Bindings
使用綁定進行 WPF DataGrid 格式化的指南
WPF DataGrid: Solving Sorting, ScrollIntoView, Refresh and Focus Problems
WPF DataGrid：解決排序、捲動到檢視內、刷新和焦點問題
Base WPF Window Functionality for Data Entry
用於資料輸入的 WPF 基本視窗功能
POP3 Email Client with full MIME Support (over 1 million views, Rating: 4.88/5)
支援完整 MIME 格式的 POP3 電子郵件用戶端 （瀏覽量超過 100 萬，評分：4.88/5）
Debugging Multithreaded Code in Real Time! (highly recommended to read, amazing code, Rating: 4.96/5)
即時調試多線程程式碼！ （強烈建議閱讀，代碼精彩，評分：4.96/5）
