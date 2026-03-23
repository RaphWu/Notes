---
aliases:
date: 2016-07-05
update:
author: lukaseder
language:
sourceurl:
tags:
---

# Say NO to Venn Diagrams When Explaining JOINs 拒絕使用維恩圖解釋 JOINs

In recent times, there have been a couple of tremendously popular blog posts explaining JOINs using [Venn Diagrams](https://en.wikipedia.org/wiki/Venn_diagram). After all, relational algebra and SQL are set oriented theories and languages, so it only makes sense to illustrate set operations like JOINs using Venn Diagrams. Right?
近年來，有幾篇非常受歡迎的博客文章使用維恩圖解釋 JOINs。畢竟，關係代數和 SQL 都是集合論的理論和語言，所以用維恩圖來說明集合運算如 JOINs 是合理的。對吧？

[Google seems to say so:](https://www.google.com/search?num=10&sca_esv=9bd0e26604fdbc2f&sxsrf=AE3TifNuMbhXNszKCG1CAUNImUKHEB6JhA:1763754910971&udm=2&fbs=AIIjpHybaGNnaZw_4TckIDK59RtxQXhK6kI3AtAFLvuO8MTsfzzsG5eApDAJSjW6NQWuVcMzc2qyzfmQigsSwrF3PNK4UDPKHwrpFyoAE3gr27nJrsBxOOr2jIrqKAb8NJVvr4JyFw6-sgkGlaz1hC44rEB_wgwMqXCE_9MuLApjEuXaNlfuN1Jm0JGcgvC6Cjb4GOYrmPtG&q=visual+join+explanation&sa=X&ved=2ahUKEwisycGHg4SRAxX1bvUHHZvbGpoQtKgLegQIFBAB&biw=1200&bih=1759&dpr=1)
Google 似乎也這麼認為：

[![venn-google](https://i0.wp.com/blog.jooq.org/wp-content/uploads/2016/07/venn-google.png?resize=700%2C632&ssl=1)](https://i0.wp.com/blog.jooq.org/wp-content/uploads/2016/07/venn-google.png?ssl=1)

Everyone uses Venn Diagrams to explain JOINs. But that’s…
每個人都用維恩圖來解釋 JOINs。但那…

## PLAIN WRONG!  純粹錯誤

Venn Diagrams are perfect to illustrate … actual set operations! SQL knows three of them:
Venn 圖示法是解釋…實際集合運算的完美工具！SQL 知道其中三種：

- UNION
- INTERSECT
- EXCEPT

And they can be explained as such:
而且它們可以被解釋如下：

![[venn-union.webp]]

![[venn-intersection.webp]]

![[venn-difference.webp]]

([all of these slides are taken from our Data Geekery SQL Training, do check it out!](https://www.jooq.org/training))
（這些幻燈片都來自我們的 Data Geekery SQL 培訓，確實值得一看！）

Most of you use `UNION` occasionally. [`INTERSECT` and `EXCEPT` are more exotic, but do come in handy every now and then](https://blog.jooq.org/you-probably-dont-use-sql-intersect-or-except-often-enough/).
大多數人偶爾會使用 `UNION` 。 `INTERSECT` 和 `EXCEPT` 比較少用，但偶爾還是有用處。

The point here is: these set operations operate on sets of elements (tuples), which are all of the _same type_. As in the examples above, all elements are people with first and last names. This is also why `INTERSECT` and `EXCEPT` are more exotic, because they’re usually not very useful. `JOIN` is much more useful. For instance, you want to combine the set of actors with their corresponding set of films.
關鍵在於：這些集合運算運算的是元素（元組）的集合，而這些元素的*類型都相同*。就像上面的例子一樣，所有元素都是包含姓名的人名。這也是為什麼 `INTERSECT` 和 `EXCEPT` 比較少見的原因，因為它們通常用處不大。 `JOIN` 實用得多。例如，你可能想把演員集合和他們對應的電影集合合併起來。

A `JOIN` is really a [cartesian product](https://en.wikipedia.org/wiki/Cartesian_product) (also cross product) with a filter. Here’s a nice illustration of a cartesian product:
`JOIN` 實際上是一個帶有篩選條件的 [笛卡爾積](https://en.wikipedia.org/wiki/Cartesian_product) （也稱為叉積）。以下是笛卡爾積的一個清晰範例：

![[venn-cross-product.webp]]

## So, what’s a better way to illustrate JOIN operations? 那麼，還有什麼更好的方法來說明 JOIN 操作呢？

`JOIN` diagrams! Let’s look at `CROSS JOIN` first, because all other `JOIN` types can be derived from `CROSS JOIN`:
`JOIN` 圖表！我們先來看 `CROSS JOIN` ，因為其他所有 `JOIN` 類型都可以從 `CROSS JOIN` 演衍而來 ：

![[venn-cross-join1.webp]]

Remember, in a cross join (in SQL also written with a comma separated table list, historically) is just taking every item on the left side, and combines it with every item on the right side. When you `CROSS JOIN` a table of 3 rows with a table of 4 rows, you will get 3×4=12 result rows. See, I’m using an “x” character to write the multiplication. I.e. a “cross”.
記住，交叉連接（在 SQL 中，歷史上也用逗號分隔的表列表表示）就是將左側的每個元素與右側的每個元素合併。當你將一個包含 3 行的表格與一個包含 4 行的表格 `CROSS JOIN` 時，你會得到 3×4=12 行結果。你看，我用「x」字來表示乘法，也就是「交叉」。

## INNER JOIN  內連接

All other joins are still based on cross joins, but with additional filters, and perhaps unions. Here’s an explanation of each individual `JOIN` type.
所有其他連接仍然基於交叉連接，但增加了額外的篩選條件，可能也使用了聯合連接。以下是每種 `JOIN` 類型的詳細說明。

![[venn-join1.webp]]

In plain text, an `INNER JOIN` is a `CROSS JOIN` in which only those combinations are retained which fulfil a given predicate. For instance:
簡單來說， `INNER JOIN` 是一種 `CROSS JOIN` 但它只保留滿足給定謂詞的組合。例如：

```sql
-- "Classic" ANSI JOIN syntax
SELECT *
FROM author a
JOIN book b ON a.author_id = b.author_id
 
-- "Nice" ANSI JOIN syntax
SELECT *
FROM author a
JOIN book b USING (author_id)
 
-- "Old" syntax using a "CROSS JOIN"
SELECT *
FROM author a, book b
WHERE a.author_id = b.author_id
```

## OUTER JOIN  外連接

`OUTER JOIN` types help where we want to retain those rows from either the `LEFT` side or the `RIGHT` or both (`FULL`) sides, for which there was no matching row where the predicate yielded true.
當我們想要保留 `LEFT` 、 `RIGHT` 或兩側（ `FULL` ）中那些沒有與謂詞結果為真的匹配行時， `OUTER JOIN` 類型會有所幫助。

A `LEFT OUTER JOIN` in [relational algebra](https://en.wikipedia.org/wiki/Relational_algebra#Left_outer_join_.28.E2.9F.95.29) is defined as such:
在 [關係代數](https://en.wikipedia.org/wiki/Relational_algebra#Left_outer_join_.28.E2.9F.95.29) 中，左 `LEFT OUTER JOIN` 定義如下：

$$(R \bowtie S) \cup ((R - \pi_{r_1, r_2, \dots, r_n} (R \bowtie S)) \times \{(\omega, \dots \omega)\})$$

Or more verbosely in SQL:
或用更詳細的 SQL 表述：

```sql
SELECT *
FROM author a
LEFT JOIN book b USING (author_id)
```

This will produce all the authors and their books, but if an author doesn’t have any book, we still want to get the author with NULL as their only book value. So, it’s the same as writing:
這將列出所有作者及其著作，但如果某位作者沒有任何著作，我們仍然希望找到該作者，並取得其著作值為 NULL 的記錄。因此，這等同於編寫以下程式碼：

```sql
SELECT *
FROM author a
JOIN book b USING (author_id)
 
UNION
 
SELECT a.*, NULL, NULL, NULL, ..., NULL
FROM (
  SELECT a.*
  FROM author a
   
  EXCEPT
   
  SELECT a.*
  FROM author a
  JOIN book b USING (author_id)
) a
```

But no one wants to write that much SQL, so `OUTER JOIN` was implemented.
但沒有人願意寫那麼多 SQL 程式碼，所以實作了 `OUTER JOIN` 。

## Conclusion: Say NO to Venn Diagrams 結論：拒絕維恩圖

`JOIN`s are relatively easy to understand intuitively. And they’re relatively easy to explain using Venn Diagrams. But whenever you do that, remember, that you’re making a wrong analogy. A `JOIN` is not strictly a set operation that can be described with Venn Diagrams. A `JOIN` is always a cross product with a predicate, and possibly a `UNION` to add additional rows to the `OUTER JOIN` result.
`JOIN` 操作相對容易理解 `JOIN` 也比較容易用維恩圖解釋。但是，請記住，這樣做並不恰當。 JOIN 並非嚴格意義上的集合運算，不能用維恩圖來描述。 JOIN 總是帶有謂詞的 `JOIN` 積，也可能 `UNION` 將額外的行加入 `OUTER JOIN` 結果。

So, if in doubt, please use `JOIN` diagrams rather than Venn Diagrams. They’re more accurate and visually more useful.
所以，如有疑問，請使用 `JOIN` 圖而不是維恩圖。連結圖更準確，也更直觀易懂。

![venn-google-say-no](https://i0.wp.com/blog.jooq.org/wp-content/uploads/2016/07/venn-google-say-no.png?resize=700%2C630&ssl=1)

---

# 留言

**Bryce Deneen** says:
[July 5, 2016 at 15:34](https://blog.jooq.org/say-no-to-venn-diagrams-when-explaining-joins/#comment-145203)
The point of using a Venn Diagram is for illustrative purposes. If I’m using a Venn Diagram when I’m showing someone Joins, its probably their first exposure to joins (or nearly first). As soon as I say cartesian product I’ve lost them, and the graphic is Orders of magnitude harder to under stand than the venn diagram. While it may be imperfect, for someone new it is a lot clearer and less likely to push them away or scare them off from SQL.
使用維恩圖的目的是為了方便理解。如果我用維恩圖向別人講解連結（JOIN），那很可能是他們第一次接觸連結（或幾乎是第一次）。一旦我說到笛卡兒積，他們就聽不懂了，而且笛卡兒積的理解難度比維恩圖高出幾個數量級。雖然維恩圖可能不完美，但對於新手來說，它更清晰易懂，也更不容易讓他們望而卻步，甚至對 SQL 產生恐懼。

1. **VillageIdiot** says:
    [May 20, 2020 at 20:19](https://blog.jooq.org/say-no-to-venn-diagrams-when-explaining-joins/#comment-361535)
    lukaseder – Sometimes, we can take a middle ground here. Introduce beginners to Cartesian product & show a simple join very quickly. Then, show them joins by using Venn diagrams if that helps them to remember concepts easily. IMO, in this case, it is more important to remember the joins.
    lukaseder – 有時候，我們可以採取折衷方案。先向初學者介紹笛卡爾積，並快速展示一個簡單的連結。然後，如果維恩圖能幫助他們輕鬆記憶概念，再用維恩圖來展示連結。在我看來，在這種情況下，記住連結更重要。
    
    1. **[lukaseder](http://lukaseder.wordpress.com/)** says:
        [May 21, 2020 at 11:10](https://blog.jooq.org/say-no-to-venn-diagrams-when-explaining-joins/#comment-361673)
        Yes, there’s a middle ground. Venn diagrams appeal to intuition, but if you look at the screenshot of the google image search, _everyone_ uses Venn diagrams, and _no one_ uses a more accurate description.
        是的，兩者之間存在折衷方案。維恩圖迎合了直覺，但如果你看一下谷歌圖片搜尋的截圖，你會發現 _ 每個人都 _ 在使用維恩圖，而 _ 沒有人 _ 使用更準確的描述。

---

**[Carlos L Chacon Jr (@CarlosLChacon)](http://twitter.com/CarlosLChacon)** says:
[July 5, 2016 at 19:44](https://blog.jooq.org/say-no-to-venn-diagrams-when-explaining-joins/#comment-145206)
If you want to dig into the results a bit more, I can see where this is helpful. The nice thing about the Venn diagram is the uniformity of the illustration. I don’t have to connect boxes to get a feel for what is included in the result. Interesting thought.
如果你想更深入分析結果，我能理解這方面的幫助。維恩圖的優點在於其圖示的統一性。我不需要連接方框就能了解結果包含哪些內容。很有意思的想法。

1. **[lukaseder](http://lukaseder.wordpress.com/)** says:
    [July 5, 2016 at 22:22](https://blog.jooq.org/say-no-to-venn-diagrams-when-explaining-joins/#comment-145213)
    But this way of thinking works for equi join only. An `INNER JOIN`, for instance, may still yield a cartesian product, if not joining by a primary key / foreign key relationship. This will then be completely unexpected, when following only the Venn Diagram approach.
    但這種思路僅適用於等值連線。例如，如果 `INNER JOIN` 並非透過主鍵/外鍵關係連接，則仍可能產生笛卡爾積。如果僅遵循維恩圖方法，這種情況將完全出乎意料。

---

**Alexander Shopov** says:
[July 5, 2016 at 21:18](https://blog.jooq.org/say-no-to-venn-diagrams-when-explaining-joins/#comment-145208)
I also hate using Venn diagrams for explaning SQL joins. The set-bag discrepancy is one of the reasons. The other one is multiple joins. Venn diagrams completely break for anything above 3 sets/tables (2 joins) and may confuse you more than actually explain things.
我也討厭用維恩圖來解釋 SQL 連結。集合和集合之間的差異是原因之一。另一個原因是多重連結。維恩圖在超過 3 個集合/表（2 個連接）的情況下就完全失效了，反而會讓你更加困惑，而不是真正解釋清楚。

1. **[lukaseder](http://lukaseder.wordpress.com/)** says:
    [July 5, 2016 at 22:20](https://blog.jooq.org/say-no-to-venn-diagrams-when-explaining-joins/#comment-145211)
    Yes, indeed. It would be interesting to display an M:N relationship using a join table with the different diagram notations.
    沒錯。用不同的圖表符號，透過連結表來展示多對多關係，會很有意思。

---

**[Mark J. Reed](http://gravatar.com/markjreed)** says:
[July 5, 2016 at 21:55](https://blog.jooq.org/say-no-to-venn-diagrams-when-explaining-joins/#comment-145209)
The INNER JOIN visual results are all incorrect; each source row appears only once.
INNER JOIN 的視覺化結果全部錯誤；每個來源行只出現一次。

1. **[lukaseder](http://lukaseder.wordpress.com/)** says:
    [July 5, 2016 at 22:19](https://blog.jooq.org/say-no-to-venn-diagrams-when-explaining-joins/#comment-145210)
    Thanks, you’re right. There was a different diagram, originally ([https://lukaseder.files.wordpress.com/2016/07/venn-join.png](https://lukaseder.files.wordpress.com/2016/07/venn-join.png)), which was adapted to improve colours and remove the duplicates for simplicity. The text wasn’t adapted, though.
    謝謝，你說得對。最初使用的是另一個圖表（ [https://lukaseder.files.wordpress.com/2016/07/venn-join.png](https://lukaseder.files.wordpress.com/2016/07/venn-join.png) ），後來為了改進顏色和簡化圖表，對其進行了修改，並刪除了重複項。但文字部分沒有修改。
    ![[venn-join.png]]

---

**philipxy** says:
[July 18, 2016 at 00:28](https://blog.jooq.org/say-no-to-venn-diagrams-when-explaining-joins/#comment-145356)
Although Venn diagrams are unsuitable for explaining inner join, *one* diagram under a suitable interpretation is useful for comparing inner, left outer, right outer & full outer join: a left and right circle that are the tuples returned from left and right outer joins respectively. See my comments on various questions & answers at [https://stackoverflow.com/questions/38549/difference-between-inner-and-outer-joins](https://stackoverflow.com/questions/38549/difference-between-inner-and-outer-joins) and [https://stackoverflow.com/questions/17759687/cross-join-vs-inner-join-in-sql-server-2008/25957600#25957600](https://stackoverflow.com/questions/17759687/cross-join-vs-inner-join-in-sql-server-2008/25957600#25957600) . PS I came here via [http://www.dbdebunk.com/2016/07/this-week_10.html#more](http://www.dbdebunk.com/2016/07/this-week_10.html#more) .
雖然維恩圖不適合解釋內連接，但*一個*在適當解釋下的維恩圖可以用來比較內連接、左外連接、右外連接和全外連接：左圓和右圓分別代表左外連接和右外連接返回的元組。請參閱我在以下連結中對各種問題和答案的評論： [https://stackoverflow.com/questions/38549/difference-between-inner-and-outer-joins](https://stackoverflow.com/questions/38549/difference-between-inner-and-outer-joins) 和 [https://stackoverflow.com/questions/17759687/](https://stackoverflow.com/questions/17759687/cross-join-vs-inner-join-in-sql-server-2008/25957600#25957600) cross-join-vs-inner-join-in-sql-server-2008/2595760125。附：我是從 [http://www.dbdebunk.com/2016/07/this-week_10.html#more](http://www.dbdebunk.com/2016/07/this-week_10.html#more) 跳到這裡的。

1. **[lukaseder](http://lukaseder.wordpress.com/)** says:
    [July 18, 2016 at 07:56](https://blog.jooq.org/say-no-to-venn-diagrams-when-explaining-joins/#comment-145360)
    Thanks for your comments, and for the link to dbdebunk.com. Looks like a very nice resource!
    感謝您的評論，以及提供的 dbdebunk.com 連結。看起來是個很不錯的資源！

---

**Dominique Fortin** says:
[November 30, 2018 at 04:45](https://blog.jooq.org/say-no-to-venn-diagrams-when-explaining-joins/#comment-289655)
Your statement is a bit limited. It is true the UNION, INTERSECT, EXCEPT operates as set operation on rows, but all these operation can also be viewed as filters applied on the universe set. What is the UNION of A and B: it’s a filter on the universe set with the condition a is IN A plus b is in B but not in A. And as such are not very different from JOINs.
你的說法有些片面。誠然，UNION、INTERSECT 和 EXCEPT 操作是對行進行集合運算，但所有這些操作也可以看作是對整個集合套用篩選器。 A 和 B 的 UNION 操作是什麼？它實際上是對整個集合套用過濾器，條件是 a 屬於 A，且 b 屬於 B 但不屬於 A。因此，它們與 JOIN 操作並沒有本質差異。

To derive all JOINs, one Cartesian product is not enough. You need the UNION (not perhaps) of the following Cartesian products: (A x {}) U (A x B) U ({} x B) U ({} x {}), where {} is the empty set. Here, the universe set is made of tuples (a,b) where a is in A or is empty and b is in B or is empty.
要推導出所有 JOIN，僅一個笛卡爾積是不夠的。你需要以下笛卡爾積的並集（注意是並集，而不是可能）：(A × {}) U (A × B) U ({} × B) U ({} × {})，其中 {} 表示空集合。這裡，全集由元組 (a, b) 構成，其中 a 屬於 A 或為空集，b 屬於 B 或為空集合。

So the use of the Venn diagram, I think is perfectly legitimate although it is done on different universe sets.
所以，我認為使用維恩圖是完全合理的，即使是在不同的宇宙集合上進行的。

1. **[lukaseder](http://lukaseder.wordpress.com/)** says:
    [November 30, 2018 at 09:49](https://blog.jooq.org/say-no-to-venn-diagrams-when-explaining-joins/#comment-289728)
    Thanks for chiming in. Would you mind explaining those additional cartesian products. (A x {}) is an empty set. Did you mean to write e.g. (A x {(ω, ω, …, ω)}) to describe a left outer join?
    感謝您的參與。您能否解釋一下那些額外的笛卡爾積？ (A x {}) 是一個空集合。您是否想用例如 (A x {(ω, ω, …, ω)}) 來描述左外連接？

    Anyway, indeed Venn diagrams can be used to describe joins on a different level, but most people who do that will not use Venn diagrams this way, they use them in an “intuitive” way, which hides the fact that there is a cartesian product in there somewhere.
    總而言之，維恩圖確實可以用來描述不同層次的連接，但大多數使用維恩圖的人不會這樣使用，他們會以一種「直觀」的方式使用，這掩蓋了其中存在笛卡爾積的事實。

    In my SQL training, I always ask people to count all the actors that have the same first and last names (e.g. two actors called “John Doe”). A lot of trainees will inner join actor with itself on (a1.first_name, a1.last_name) = (a2.first_name, a2.last_name) and a1.id <> a2.id, not noticing their accidental cartesian product. It accidentally works for 2 John Does, but breaks for 3 or more.
    在我的 SQL 訓練中，我總是讓學員統計所有名字和姓氏相同的演員（例如，兩個都叫“John Doe”的演員）。許多學員會使用內連結將演員表本身與 (a1.first_name, a1.last_name) = (a2.first_name, a2.last_name) 和 a1.id <> a2.id，卻沒注意到他們無意中使用了笛卡兒積。這種方法對於兩個“John Doe”確實有效，但對於三個或更多“John Doe”來說就失效了。

    Once the cartesian product is understood, it can be interesting to get back to reasoning about set theory and Venn diagrams with respect to joins. But until then, I think they are part of the confusion around joins with a lot of people who aren’t writing SQL every day.
    一旦理解了笛卡爾積，再去探討集合論和維恩圖在連結運算中的應用就會很有趣。但在此之前，我認為對於許多不常寫 SQL 的人說來，這些概念是造成連線運算困惑的原因之一。

**Dominique Fortin** says:
[November 30, 2018 at 18:01](https://blog.jooq.org/say-no-to-venn-diagrams-when-explaining-joins/#comment-289916)
In ZFC set theory, a 2tuple set where (a,b) is an element of Ax{} is an empty set, but because all the domains of types in SQL include the null value, the free logic set theory must be used where a 2tuple set where (a,b) is an element of Ax{} is not an empty set. So, for example, if a, b and c are the non-null elements of A, A would be define by {null,a,b,c} and the empty set (or empty domain) is define {null} and the Cartesian product of A and {} is {(null,null),(a,null),(b.null),(c.null)}.
在 ZFC 集合論中，如果一個二元組集合 Ax{} 中包含 (a,b)，則該集合為空集合。但由於 SQL 中所有類型的領域都包含空值，因此必須使用自由邏輯集合論來定義 Ax{} 中包含 (a,b) 的二元組集合非空的情況。例如，如果 a、b 和 c 是 A 的非空元素，則 A 定義為 {null,a,b,c}，空集合（或空域）定義為 {null}，A 與 {} 的笛卡爾積為 {(null,null),(a,null),(b,null),(c,null)}。

1. **[lukaseder](http://lukaseder.wordpress.com/)** says:
    [December 3, 2018 at 11:56](https://blog.jooq.org/say-no-to-venn-diagrams-when-explaining-joins/#comment-291290)
    I see, thanks for the explanation!
    我明白了，謝謝你的解釋！

---

**A tech Dude** says:
[August 25, 2020 at 22:16](https://blog.jooq.org/say-no-to-venn-diagrams-when-explaining-joins/#comment-385271)
Be honest, the Venn diagram is way much clear than what you put here, sorry. Your explanation is excellent unless someone understand the math language you are using. Most people specially those think Scala is better than Java or node.js should be the tech stack used in backend….
說實話，維恩圖比你寫的清楚多了，抱歉。你的解釋很棒，除非有人懂你用的數學語言。大多數人，特別是那些認為 Scala 比 Java 好，或認為後端應該使用 Node.js 的人…

1. **[lukaseder](http://lukaseder.wordpress.com/)** says:
    [August 26, 2020 at 09:56](https://blog.jooq.org/say-no-to-venn-diagrams-when-explaining-joins/#comment-385364)
    Sure, I’m aware of this problem. Joins aren’t as simple as Venn diagrams make them look, hence the explanation of actual joins (via cartesian products) is also a bit less simple.
    當然，我意識到了這個問題。連接並不像維恩圖看起來那麼簡單，因此對實際連接（透過笛卡爾積）的解釋也稍微複雜一些。

    Most people appreciate the Venn diagrams because they help them remember what left and right mean in case of an outer join, and that’s fine. But in my SQL training, I ask delegates to write as simple query, to find the number of actors who have another actor by the same name (e.g. 2 Pierce Brosnan’s). ~40% of delegates will inner join using (first_name, last_name), and inadvertently produce a cartesian product, which leads to a wrong result!
    大多數人喜歡維恩圖，因為它能幫助他們記住外連接中「左」和「右」分別代表什麼，這當然很好。但在我的 SQL 訓練中，我要求學員編寫一個簡單的查詢語句，找出有多少位演員與另一位演員同名（例如，兩位皮爾斯布魯斯南）。大約 40% 的學員會使用 (first_name, last_name) 進行內連接，無意中產生了笛卡爾積，導致錯誤的結果！

    Understanding why this happens is _much more_ important than merely remembering the silly order of left/right join.
    理解為什麼會發生這種情況遠比僅僅記住左右連接的愚蠢順序重要 _ 得多 _ 。

    Of course, people can get quite far with SQL _without_ understanding the underlying relational algebra. But as with all things that are based on some fundamental theory, understanding the theory helps. Ignoring the theory means making the same mistakes over and over again.
    當然， *即使不了解*底層關係代數，人們也能在 SQL 上取得相當大的進展。但正如所有基於基本理論的事物一樣，理解理論至關重要。忽視理論意味著一而再、再而三地犯下同樣的錯誤。

    The main target audience of this article are not just beginners, but also folk who understand SQL, understand relational algebra, _and yet_, they resort to using Venn diagrams to explain SQL to others. I just think that’s not helpful.
    這篇文章的主要目標讀者不僅是初學者，還包括那些了解 SQL 和關係代數的人， *但他們仍然*選擇用維恩圖來解釋 SQL。我覺得這樣做沒什麼幫助。

---

# raddit 留言

https://www.reddit.com/r/programming/comments/4rc9uu/comment/d50mxez/

[bbalkenhol](https://www.reddit.com/user/bbalkenhol/)

I made a visualization last year that combines the approach shown in the article with Venn diagrams (link: [https://i.imgur.com/xLTcOGi.png](https://i.imgur.com/xLTcOGi.png)). Venns may be less accurate, but they do tend to be easier to understand.
去年我製作了一個視覺化圖表，結合了文章中展示的方法和維恩圖（連結： [https://i.imgur.com/xLTcOGi.png](https://i.imgur.com/xLTcOGi.png) ）。維恩圖可能不夠精確，但它們通常更容易理解。
![[SQL Joins and Set Operators by bbalkenhol.png]]

1. [lukaseder](https://www.reddit.com/user/lukaseder/)
    Very nice. Including "mini" cartesian products in the different joins.
    非常好。在不同的接合處都包含了「迷你」笛卡爾積。

    Any reason why you omitted EXCEPT ALL and INTERSECT ALL? And perhaps, if you're daring, how about including division? :)
    請問您為什麼省略了 EXCEPT ALL 和 INTERSECT ALL 這兩個運算子？如果您膽子夠大，不妨考慮一下除法運算子？ :)

    1. [bbalkenhol](https://www.reddit.com/user/bbalkenhol/)
        Mostly because they're not as widely supported, and `except all` is not as straightforward to visualize with a simple Venn diagram. Though I did set up the sample sets in a way that would let you clarify what makes them different.
        主要原因是它們的支持度不高， `except all` ，其他方面很難用簡單的維恩圖直觀地展現出來。不過，我設定樣本集的方式應該可以幫助你弄清楚它們之間的差異。

        Division is interesting but a bit more involved ([example](http://www.sqlfiddle.com/#!15/84d97/1)) and I prefer keeping the overview concise. Anyone is free to change/improve it as they see fit though. :)
        除法很有意思，但稍微複雜一些（ [例如](http://www.sqlfiddle.com/#!15/84d97/1) ），我更喜歡保持概述簡潔明了。當然，任何人都可以根據自己的想法進行修改/改進。 :)

2. [Scary-Actuator-8746](https://www.reddit.com/user/Scary-Actuator-8746/)
    Wow, pretty useful.  哇，真實用。
    The picture shows why actually preferable to use set theory instead of Venn diagrams.
    這張圖說明了為什麼使用集合論比使用維恩圖更好。
    For example, there is no obvious graphical difference between UNION and UNION ALL while using Venn diagrams. From the other side, set theory shows "uniqueness" of elements in case of UNION selection
    例如，在使用維恩圖時，UNION 和 UNION ALL 在圖形上沒有明顯的差異。另一方面，集合論表明，在 UNION 選擇的情況下，元素具有「唯一性」。

---

[burnblue](https://www.reddit.com/user/burnblue/)
I get the point, that the venn diagrams don't cover the combination of separate data into one. But I found them far nore intuituve and useful to understand what you end up with (eg filter left, inner) than your rectangle diagram, which was based on colors (unexplained). You're going to need a better diagram than that.
我明白你的意思，維恩圖無法將多個獨立資料合併成一個整體。但我發現，相較於你那個基於顏色（且未加解釋）的矩形圖，維恩圖更直觀、更實用，能幫助你理解最終結果（例如，篩選左側、內部）。你需要一個更好的圖表。

1. [rawrnnn](https://www.reddit.com/user/rawrnnn/)
    For real, the venn diagram isn't supposed to be a technical definition of a join, it's just a useful way of visualizing them. I've used them in interviews to succinctly summarize that I understand the differences without having to try to awkwardly stumble through in english.
    說真的，維恩圖並不是用來定義「連結」的技術細節，它只是一種便於理解的連結視覺化方式。我曾在面試中使用維恩圖，可以簡潔明了地表達我對連結概念的理解，而無需費力地用英語磕磕絆絆地解釋。
