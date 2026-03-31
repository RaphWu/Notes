---
aliases:
date: 2026-02-03
update:
author: DavidKo
language:
sourceurl: https://www.facebook.com/share/p/1Bn9kc3Ms8/
tags:
---

# RD 的真心話：我們不是不寫測試，是不知道要測什麼

![[RD 的真心話：我們不是不寫測試，是不知道要測什麼.jpg]]

在 Code Review 或 Sprint Planning 上，最尷尬的沉默往往發生在這一刻：

「這張票的 Test Case 這樣夠了嗎？」
「你有考慮到 User 如果先按 A 再按 B 會怎樣嗎？」

這時候，RD 通常會愣住，然後心虛地回：「呃...那我再補一下。」

這背後藏著一個 RD 很誠實、但這輩子幾乎不敢對 PM 或主管說出口的秘密：「我根本不知道 User 會那樣用，因為我看不到 Business Flow 的全貌。」

測試寫不出來，不是技術問題，是「知識斷層」
很多時候，測試案例開天窗，真的不是 RD 懶，也不是能力差。是因為我們的分工模式出了問題：

1. 規格書是開心農場
   PM 給的 Spec 往往只畫了正常流程。至於例外？邊界？那是留給 RD 通靈的。
2. 盲人摸象的開發體驗
   RD 接到的 Ticket 常常是被切碎的碎片。RD 不知道這個功能的前世今生，當然只能測碎片。
3. 多做多錯的生存法則
   如果 Spec 沒寫，RD 自己腦補了情境 A，結果被說你想太多、或是想錯了。幾次之後，大家就學乖了：照圖施工，保平安。

Test Case 開不起來，本質上是「業務知識」沒有流進 RD 的腦袋裡。
怎麼解？打破「知識斷層」的三帖藥
要解決這個問題，你需要的是改變協作方式：

1. 拒絕文字獄，把流程「畫」出來
   文字規格書是產生誤解的溫床。試試看 User Story Mapping 或簡單的白板討論。把 PM、RD、QA 關在一起，把 User 從進來到離開的流程貼在牆上。
   當 RD 親眼看到：「原來 User 付款失敗後，系統還要自動發信？」這時候你不用逼他，他自己就會知道要補「發信失敗」的測試案例了。
2. 開發前的「三劍客會議」
   不要等到 Code 寫完才問測什麼。在開始撰寫程式前，PM（講需求）、RD（看可行性）、QA（找麻煩）花 15 分鐘快速對焦，討論可能的場景，釐清例外狀況要如何處理。
3. 讓 QA 成為 RD 的「業務導師」
   在很多團隊，QA 其實比 RD 更懂整個系統的邏輯（因為他們測得最痛苦）。RD 寫不出測項時，不該去問 PM「要測什麼」，而該去問 QA「你會怎麼測壞它」。 建立這個請益的管道，比任何文件都有效。

「不知道測什麼」不是 RD 的錯，是「協作模式」的訊號燈。
各位 RD，你有過這種「看著 Spec 卻不知道該測多深」的無力感嗎？
各位 PM，你們通常怎麼確保 RD 懂那些「沒寫出來的細節」？

(1) AI 協作的系統化測試案例設計實戰 (2026/3/21)
[https://agile3uncles.com/products/t003/](https://agile3uncles.com/products/t003/?fbclid=IwZXh0bgNhZW0CMTAAYnJpZBExVGJYNWlIUHF5Mno0MGVXU3NydGMGYXBwX2lkEDIyMjAzOTE3ODgyMDA4OTIAAR5IPMNgVRq6nwaWco6jXyKWcV4W_hMJPq9bIV3a4292B7PW2lz6c0KL1jEYnA_aem_Fr4wstjjuALEl2SNGbVMVA)

(2) 如何寫出人人有共識的需求 - 範例描述需求篇 (2026/3/22)
[https://agile3uncles.com/products/t002/](https://agile3uncles.com/products/t002/?fbclid=IwZXh0bgNhZW0CMTAAYnJpZBExVGJYNWlIUHF5Mno0MGVXU3NydGMGYXBwX2lkEDIyMjAzOTE3ODgyMDA4OTIAAR4k4kUbU7fxskmET0a2wC7CzeKzHX1b63vvUQInKV3O-vbtnd1RjZ-FsMlqRA_aem_3erHDipKy6DmAUnlLve74A)

---

[Kriz](https://www.facebook.com/cheng.en.yu.2024?comment_id=Y29tbWVudDoxNDc4NTE4MzMwOTQ4NjI3XzEyMjgxOTM4NjIxMTQ3MzE%3D&__cft__[0]=AZaTG0pLrXak_f9excr78Q7f3yw5adrch3s4e3Tok34b0YMvsdGrvJTx9_7WKBX6CyEQnbCsyKLRlr6xSqpForzuiBhKB3KGwl3nkr2OpgrJKdILOytCLI8QoNrXlZy30IhO3m_BD45Nkgzchxk_PM8bL7HC_vV7bf82fyachT8IKquR8_UZCDAyWLKqY_xrWZItsUAYTkBM4Sok0cbyeWO4&__tn__=R]-R)

Product Manager 跟 Project Manager 思考的東西不一樣，常被混著用
沒幾個 "PM" 能夠真的把 Spec 、情境、邊界列清楚，Product Manager 通常尤為如此，整個最後就像許願池
