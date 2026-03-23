---
aliases:
date: 2026-01-30
update:
author: Anthropic
language:
sourceurl: https://www.anthropic.com/research/AI-assistance-coding-skills
tags:
  - AI
---

# # How AI assistance impacts the formation of coding skills - 人工智慧輔助如何影響程式設計技能的形成

Research shows AI helps people do [parts](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566) of their job faster. In an observational [study](https://www.anthropic.com/research/estimating-productivity-gains) of [Claude.ai](http://claude.ai/redirect/website.v1.cdac1dd1-833d-4c26-a3b0-40c6e3c1894c) data, we found AI can speed up some tasks by 80%. But does this increased productivity come with trade-offs? Other research shows that when people use AI assistance, they become [less engaged with their work](https://www.nature.com/articles/s41598-025-98385-2) and [reduce](https://www.microsoft.com/en-us/research/wp-content/uploads/2025/01/lee_2025_ai_critical_thinking_survey.pdf) the effort they put into doing it—in other words, they offload their thinking to AI.
研究表明，人工智慧可以幫助人們更快地完成 [部分](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=4945566) 工作。在 [Claude.ai](http://claude.ai/redirect/website.v1.cdac1dd1-833d-4c26-a3b0-40c6e3c1894c) 數據的觀察性 [研究](https://www.anthropic.com/research/estimating-productivity-gains) 中，我們發現人工智慧可以將某些任務的速度提高 80%。但這種生產力的提升是否會帶來一些弊端呢？其他研究表明，當人們使用人工智慧輔助時，他們 [對工作的投入度會降低](https://www.nature.com/articles/s41598-025-98385-2) ，投入的精力也會 [減少](https://www.microsoft.com/en-us/research/wp-content/uploads/2025/01/lee_2025_ai_critical_thinking_survey.pdf) ——換句話說，他們把自己的思考工作交給了人工智慧。

It’s unclear whether this cognitive offloading can prevent people from growing their skills on the job, or—in the case of coding—understanding the systems they’re building. Our latest study, a randomized controlled trial with software developers as participants, investigates this potential downside of using AI at work.
目前尚不清楚這種認知卸載是否會阻礙人們在工作中提陞技能，或者——以程式為例——阻礙他們理解自己正在建構的系統。我們最新的研究是一項以軟體開發人員為參與者的隨機對照試驗，旨在調查在工作中使用人工智慧的這種潛在弊端。

This question has broad implications—for how to design AI products that facilitate learning, for how workplaces should approach AI policies, and for broader societal resilience, among others. We focused on coding, a field where AI tools have rapidly become standard. Here, AI creates a potential tension: as coding grows more automated and speeds up work, humans will still need the skills to catch errors, guide output, and ultimately provide oversight for AI deployed in high-stakes environments. Does AI provide a shortcut to _both_ skill development and increased efficiency? Or do productivity increases from AI assistance undermine skill development?
這個問題意義深遠——它關係到如何設計促進學習的人工智慧產品、企業應如何制定人工智慧政策，以及更廣泛的社會韌性等等。我們重點關注程式設計領域，人工智慧工具已迅速成為該領域的標準配置。然而，人工智慧也帶來了一種潛在的矛盾：隨著程式自動化程度的提高和工作效率的提升，人類仍然需要具備發現錯誤、指導輸出結果，並最終對部署在高風險環境中的人工智慧進行監督的技能。人工智慧是否能 _ 為 _ 技能發展和效率提升提供捷徑？或者，人工智慧輔助帶來的生產力提升是否會阻礙技能發展？

In a randomized controlled trial, we examined 1) how quickly software developers picked up a new skill (in this case, a Python library) with and without AI assistance; and 2) whether using AI made them less likely to understand the code they’d just written.
在一項隨機對照試驗中，我們研究了 1) 軟體開發人員在有 AI 幫助和沒有 AI 幫助的情況下，學習一項新技能（在本例中為 Python 庫）的速度有多快；以及 2) 使用 AI 是否會降低他們理解自己剛剛編寫的程式碼的可能性。

We found that using AI assistance led to a statistically significant decrease in mastery. On a quiz that covered concepts they’d used just a few minutes before, participants in the AI group scored 17% lower than those who coded by hand, or the equivalent of nearly two letter grades. Using AI sped up the task slightly, but this didn’t reach the threshold of statistical significance.
我們發現，使用人工智慧輔助會導致掌握程度顯著下降。在一項涵蓋幾分鐘前剛使用過的概念的測試中，人工智慧組的參與者得分比手動編碼組低 17%，相當於低了近兩個等級。使用人工智慧雖然略微加快了任務速度，但這種提升並未達到統計顯著性水準。

Importantly, using AI assistance didn’t guarantee a lower score. _How_ someone used AI influenced how much information they retained. The participants who showed stronger mastery used AI assistance not just to produce code but to build comprehension while doing so—whether by asking follow-up questions, requesting explanations, or posing conceptual questions while coding independently.
重要的是，使用人工智慧輔助並不意味著得分會降低。人工智慧的使用 *方式* 會影響參與者對資訊的保留程度。那些掌握程度較高的參與者不僅利用人工智慧輔助來編寫程式碼，而且還在編寫程式碼的過程中加深理解——無論是透過提出後續問題、請求解釋，還是在獨立編寫程式碼的同時提出概念性問題。

## Study design  研究設計

We recruited 52 (mostly junior) software engineers, each of whom had been using Python at least once a week for over a year. We also made sure they were at least somewhat familiar with AI coding assistance, and were unfamiliar with Trio, the Python library on which our tasks were based.
我們招募了 52 位（大部分是初級）軟體工程師，他們每個人都至少每週使用一次 Python，並且使用超過一年。我們也確保他們至少對人工智慧編碼輔助有一定的了解，並且不熟悉 Trio，我們任務所基於的 Python 函式庫。

We split the study into three parts: a warm-up; the main task consisting of coding two different features using Trio (which requires understanding concepts related to asynchronous programming, a skill often learned in a professional setting); and a quiz. We told participants that a quiz would follow the task, but encouraged them to work as quickly as possible.
我們將研究分為三個部分：熱身；主要任務，即使用 Trio 編寫兩個不同功能的程式碼（這需要理解非同步程式設計的相關概念，這是一項通常在專業環境中學習的技能）；以及測驗。我們告知參與者任務完成後會有測驗，但鼓勵他們盡可能快速完成。

We designed the coding task to mimic how someone might learn a new tool through a self-guided tutorial. Each participant was given a problem description, starter code, and a brief explanation of the Trio concepts needed to solve it. We used an online coding platform with an AI assistant in the sidebar which had access to participants’ code and could at any time produce the correct code if asked.1
我們設計的程式設計任務旨在模擬人們透過自學教程學習新工具的過程。每位參與者都會收到一份問題描述、一段初始代碼以及解決問題所需的 Trio 概念的簡要說明。我們使用了一個線上程式設計平台，其側邊欄中有一個人工智慧助手，可以存取參與者的程式碼，並在需要時隨時產生正確的程式碼 。

![[How AI assistance impacts the formation of coding skills-01.png]]

### Evaluation design  評估設計

In our evaluation design, we drew on [research in computer science education](https://ieeexplore.ieee.org/document/9962584) to identify four types of questions commonly used to assess mastery of coding skills:
在我們的評估設計中，我們借鑒了 [電腦科學教育領域的研究](https://ieeexplore.ieee.org/document/9962584) ，確定了四種常用於評估程式設計技能掌握程度的問題類型：

- **Debugging**: The ability to identify and diagnose errors in code. This skill is crucial for detecting when AI-generated code is incorrect and understanding why it fails.
    **調試** ：識別和診斷代碼錯誤的能力。這項技能對於檢測人工智慧生成的程式碼何時出錯以及了解其失敗原因至關重要。
- **Code reading**: The ability to read and comprehend what code does. This skill enables humans to understand and verify AI-written code before deployment.
    **程式碼閱讀**能力：能夠閱讀並理解程式碼的功能。這項技能使人們能夠在部署前理解和驗證人工智慧編寫的程式碼。
- **Code writing:** The ability to write or select the correct approach to writing code. Low-level code writing, like remembering the syntax of functions, will be less important with the further integration of AI coding tools than high-level system design.
    **程式碼編寫：** 指編寫程式碼或選擇正確程式碼編寫方法的能力。隨著人工智慧編碼工具的進一步集成，底層程式碼編寫（例如記住函數語法）的重要性將不如高層系統設計。
- **Conceptual**: The ability to understand the core principles behind tools and libraries. Conceptual understanding is critical for assessing whether AI-generated code uses appropriate software design patterns that adhere to how the library is intended to be used.
    **概念理解** ：理解工具和函式庫背後的核心原理的能力。概念理解對於評估人工智慧產生的程式碼是否使用了符合庫預期用途的適當軟體設計模式至關重要。

Our assessment focused most heavily on debugging, code reading, and conceptual problems, as we considered these the most important for providing oversight of what is increasingly likely to be AI-generated code.
我們的評估主要側重於調試、程式碼閱讀和概念問題，因為我們認為這些對於監督越來越可能是人工智慧生成的程式碼最為重要。

## Results  結果

On average, participants in the AI group finished about two minutes faster, although the difference was not statistically significant. There was, however, a significant difference in test scores: the AI group averaged 50% on the quiz, compared to 67% in the hand-coding group—or the equivalent of nearly two letter grades (Cohen's _d_=0.738, _p_=0.01). The largest gap in scores between the two groups was on debugging questions, suggesting that the ability to understand when code is incorrect and why it fails may be a particular area of concern if AI impedes coding development.
平均而言，人工智慧組的參與者完成任務的速度比手工編碼組快了大約兩分鐘，儘管這種差異並不具有統計意義。然而，測驗成績卻有顯著差異：人工智慧組在測驗中的平均得分為 50%，而手工編碼組的平均得分為 67%，相當於近兩個等級的差距（Cohen's _d_ = 0.738， _p_ = 0.01）。兩組之間得分差距最大的是調試題，這表明，如果人工智慧阻礙了程式碼開發，那麼理解程式碼何時出錯以及出錯原因的能力可能是一個需要特別關注的領域。

![[How AI assistance impacts the formation of coding skills-02.webp]]

### Qualitative analysis: AI interaction modes - 定性分析：人工智慧互動模式

We were particularly interested in understanding _how_ participants went about completing the tasks we designed. In our qualitative analysis, we manually annotated screen recordings to identify how much time participants spent composing queries, what types of questions they asked, the types of errors they made, and how much time they spent actively coding.
我們特別感興趣的是了解參與者 _ 如何 _ 完成我們設計的任務。在定性分析中，我們手動標註了螢幕錄像，以確定參與者花費多少時間編寫查詢、他們提出的問題類型、他們犯的錯誤類型以及他們實際編碼的時間。

One surprising result was how much time participants spent interacting with the AI assistant. Several took up to 11 minutes (30% of the total time allotted) composing up to 15 queries. This helped to explain why, on average, participants using AI finished faster although the productivity improvement was not statistically significant. We expect AI would be more likely to significantly increase productivity when used on repetitive or familiar tasks.
一項令人驚訝的結果是參與者與人工智慧助理互動所花費的時間。一些參與者花了長達 11 分鐘（佔總分配時間的 30%），提出了多達 15 個問題。這有助於解釋為什麼平均而言，使用人工智慧的參與者完成任務的速度更快，儘管生產力的提升並不具有統計意義。我們預計，當人工智慧用於重複性或熟悉的任務時，更有可能顯著提高生產力。

Unsurprisingly, participants in the No AI group encountered more errors. These included errors in syntax and in Trio concepts, the latter of which mapped directly to topics tested on the evaluation. Our hypothesis is that the participants who encountered more Trio errors (namely, the control group) likely improved their debugging skills through resolving these errors independently.
不出所料，未使用人工智慧組的參與者遇到了更多錯誤。這些錯誤包括語法錯誤和 Trio 概念錯誤，後者與評估測驗的主題直接相關。我們假設，遇到更多 Trio 錯誤的參與者（即對照組）可能透過獨立解決這些錯誤提高了調試技能。

We then grouped participants by how they interacted with AI, identifying distinct patterns that led to different outcomes in completion time and learning.
然後，我們根據參與者與人工智慧的互動方式對他們進行分組，識別出導致完成時間和學習結果不同的獨特模式。

**Low-scoring interaction patterns**: The low-scoring patterns generally involved a heavy reliance on AI, either through code generation or debugging. The average quiz scores in this group were less than 40%. They showed less independent thinking and more cognitive offloading. We further separated them into:
**低分互動模式** ：低分模式通常嚴重依賴人工智慧，無論是透過程式碼產生還是調試。該組的平均測驗分數低於 40%。他們表現出較少的獨立思考能力和較多的認知卸載。我們進一步將他們分為：

- **AI delegation** (_n_=4): Participants in this group wholly relied on AI to write code and complete the task. They completed the task the fastest and encountered few or no errors in the process.
    **AI 委託組** （ _n_ =4）：此組參與者完全依賴 AI 編寫程式碼並完成任務。他們以最快的速度完成了任務，並且在過程中幾乎沒有遇到任何錯誤。
- **Progressive AI reliance** (_n_=4): Participants in this group started by asking one or two questions but eventually delegated all code writing to the AI assistant. They scored poorly on the quiz largely due to not mastering any of the concepts on the second task.
    **漸進式人工智慧依賴組** （ _n_ =4）：該組參與者最初只問一兩個問題，但最終將所有程式碼編寫工作都委託給了人工智慧助理。他們在測驗中得分很低，主要是因為他們沒有掌握第二個任務中的任何概念。
- **Iterative AI debugging** (_n_=4): Participants in this group relied on AI to debug or verify their code. They asked more questions, but relied on the assistant to solve problems, rather than to clarify their own understanding. They scored poorly as a result, and were also slower at completing the two tasks.
    **迭代式 AI 調試** （ _n_ =4）：此組參與者依賴 AI 來調試或驗證程式碼。他們提出的問題更多，但更依賴 AI 助手來解決問題，而不是自己去澄清理解。因此，他們的得分較低，完成兩項任務的速度也較慢。

**High-scoring interaction patterns:** We considered high-scoring quiz patterns to be behaviors where the average quiz score was 65% or higher. Participants in these clusters used AI both for code generation and conceptual queries.
**高分互動模式：** 我們將平均測驗分數達到或超過 65% 的行為定義為高分測驗模式。這些群體中的參與者既使用人工智慧進行程式碼生成，也使用人工智慧進行概念查詢。

- **Generation-then-comprehension** (_n_=2): Participants in this group first generated code and then manually copied or pasted the code into their work. After their code was generated, they asked the AI assistant follow-up questions to improve understanding. These participants were not particularly fast when using AI, but showed a higher level of understanding on the quiz. Interestingly, this approach looked nearly the same as that of the AI delegation group, except for the fact that they used AI to check their own understanding.
    **先生成後理解** （ _n_ =2）：此組參與者首先產生代碼，然後手動複製或貼上程式碼到自己的作品中。程式碼生成後，他們向人工智慧助理提出後續問題以加深理解。這些參與者在使用人工智慧時速度並不快，但在測試中表現出更高的理解程度。有趣的是，這種方法與人工智慧委託組的方法幾乎相同，差別在於他們使用人工智慧來檢驗自己的理解。
- **Hybrid code-explanation** (_n_=3): Participants in this group composed hybrid queries in which they asked for code generation along with explanations of the generated code. Reading and understanding the explanations they asked for took more time, but helped in their comprehension.
    **混合代碼 - 解釋** （ _n_ =3）：此組參與者編寫混合查詢，既要求產生程式碼，也要求對生成的程式碼進行解釋。閱讀和理解他們所要求的解釋需要更多時間，但有助於他們理解程式碼。
- **Conceptual inquiry** (_n_=7): Participants in this group only asked conceptual questions and relied on their improved understanding to complete the task. Although this group encountered many errors, they also independently resolved them. On average, this mode was the fastest among high-scoring patterns and second fastest overall, after AI delegation.
    **概念探究** （ _n_ =7）：此組參與者僅提出概念性問題，並依靠自身不斷加深的理解來完成任務。儘管該組參與者遇到了許多錯誤，但他們也獨立解決了這些問題。平均而言，這種模式在所有高分模式中速度最快，整體速度僅次於人工智慧委託模式，則排名第二。

Our qualitative analysis does not draw a causal link between interaction patterns and learning outcomes, but it does point to behaviors associated with different learning outcomes.
我們的定性分析並沒有建立互動模式與學習成果之間的因果關係，但它確實指出了與不同學習成果相關的行為。

## Conclusion  結論

Our results suggest that incorporating AI aggressively into the workplace, particularly with respect to software engineering, comes with trade-offs. The findings highlight that not all AI-reliance is the same: the way we interact with AI while trying to be efficient affects how much we learn. Given time constraints and organizational pressures, junior developers or other professionals may rely on AI to complete tasks as fast as possible at the cost of skill development—and notably the ability to debug issues when something goes wrong.
我們的研究結果表明，在工作場所（尤其是在軟體工程領域）積極引入人工智慧會帶來一些權衡取捨。研究結果強調，並非所有對人工智慧的依賴都相同：我們在追求效率的同時與人工智慧互動的方式會影響我們的學習效果。考慮到時間限制和組織壓力，初級開發人員或其他專業人員可能會依賴人工智慧以最快的速度完成任務，但這會犧牲技能發展——尤其是在出現問題時調試問題的能力。

Though preliminary, these results suggest important considerations as companies transition to a greater ratio of AI-written to human-written code. Productivity benefits may come at the cost of skills necessary to validate AI-written code if junior engineers’ skill development has been stunted by using AI in the first place. Managers should think intentionally about how to deploy AI tools at scale, and consider systems or intentional design choices that ensure engineers continue to learn as they work—and are thus able to exercise meaningful oversight over the systems they build.
儘管這些結果尚屬初步，但它們提示了一些重要的考量因素，尤其是在企業向人工智慧程式碼與人工程式碼比例不斷上升的轉型過程中。如果初級工程師的技能發展因人工智慧的廣泛應用而受到阻礙，那么生產力的提升可能會以犧牲驗證人工智慧程式碼所需的技能為代價。管理者應認真思考如何大規模部署人工智慧工具，並考慮如何透過系統或有意為之的設計選擇，確保工程師在工作中持續學習，以便能夠對他們所建構的系統進行有效的監督。

For novice workers in software engineering or any other industry, our study can be viewed as a small piece of evidence toward the value of intentional skill development with AI tools. Cognitive effort—and even getting painfully stuck—is likely important for fostering mastery. This is also a lesson that applies to how individuals choose to work with AI, and which tools they use. Major LLM services also provide learning modes (e.g., [Claude Code Learning and Explanatory](https://code.claude.com/docs/en/output-styles) mode or [ChatGPT Study Mode](https://openai.com/index/chatgpt-study-mode/)) designed to foster understanding. Knowing how people learn when using AI can also help guide how we design it; AI assistance should enable humans to work more efficiently _and_ develop new skills at the same time.
對於軟體工程或其他產業的初級從業人員而言，我們的研究可以被視為利用人工智慧工具進行有意識技能提升價值的一個初步證據。認知投入——甚至包括遇到難題——對於精通人工智慧至關重要。這項經驗同樣適用於個人如何選擇使用人工智慧以及使用哪些工具。主流的人工智慧學習管理（LLM）服務也提供旨在促進理解的學習模式（例如， [Claude Code 學習和解釋](https://code.claude.com/docs/en/output-styles) 模式或 [ChatGPT 學習模式](https://openai.com/index/chatgpt-study-mode/) ）。了解人們在使用人工智慧時的學習方式，也有助於我們設計人工智慧；人工智慧輔助應該能夠幫助人們更有效率地工作 _，_ 同時發展新技能。

Prior studies have found mixed results on whether AI [helps](https://arxiv.org/abs/2302.06590) or [hinders](https://arxiv.org/abs/2507.09089) coding productivity. Our own [research](https://www.anthropic.com/research/estimating-productivity-gains) found that AI can reduce the time it takes to complete some work tasks by 80%—a result that may seem in tension with the findings presented here. But the two studies ask different questions and use different methods: our earlier observational work measured productivity on tasks where participants already had the relevant skills, while this study examines what happens when people are learning something new. It is possible that AI both accelerates productivity on well-developed skills and hinders the acquisition of new ones, though more research is needed to understand this relationship.
過去研究對於人工智慧究竟是 [提升](https://arxiv.org/abs/2302.06590) 還是 [阻礙](https://arxiv.org/abs/2507.09089) 編碼效率得出了不同的結論。我們自身的 [研究](https://www.anthropic.com/research/estimating-productivity-gains) 發現，人工智慧可以將完成某些工作任務所需的時間縮短 80%——這一結果似乎與本文提出的發現相矛盾。但這兩項研究提出的問題不同，採用的方法也不同：我們早期的觀察性研究測量的是參與者已具備相關技能時的工作效率，而本研究則考察了人們學習新技能時的情況。人工智慧可能既能提高已熟練技能的效率，又能阻礙新技能的習得，但要理解這種關係，還需要更多的研究。

This study is only a first step towards uncovering how human-AI collaboration affects the experience of workers. Our sample was relatively small, and our assessment measured comprehension shortly after the coding task. Whether immediate quiz performance predicts longer-term skill development is an important question this study does not resolve. There remain many unanswered questions we hope future studies will investigate, for example: the effects of AI on tasks beyond coding, whether this effect dissipates longitudinally as engineers develop greater fluency, and whether AI assistance differs from human assistance while learning.
這項研究只是探討人機協作如何影響員工體驗的第一步。我們的樣本量相對較小，且評估是在編碼任務完成後不久進行的，僅測量了理解程度。即時測驗成績是否能預測長期技能發展，是本研究未能解答的重要議題。還有許多問題亟待解答，我們希望未來的研究能夠對此進行探討，例如：人工智慧對編碼以外任務的影響，這種影響是否會隨著工程師技能的提升而逐漸減弱，以及人工智慧輔助與人類輔助在學習過程中是否存在差異。

Ultimately, to accommodate skill development in the presence of AI, we need a more expansive view of the impacts of AI on workers. In an AI-augmented workplace, productivity gains matter, but so does the long-term development of the expertise those gains depend on.
歸根究底，為了在人工智慧時代促進技能發展，我們需要更全面地看待人工智慧對勞工的影響。在人工智慧增強的工作場所中，生產力的提升固然重要，但支撐這些提升的長期專業技能發展同樣至關重要。

Read the [full paper](https://arxiv.org/abs/2601.20245) for details.
請閱讀 [全文](https://arxiv.org/abs/2601.20245) 了解詳情。

Acknowledgments  致謝

This project was led by Judy Hanwen Shen and Alex Tamkin. Editorial support for this blog post was provided by Jake Eaton, Stuart Ritchie, and Sarah Pollack.
該項目由 Judy Hanwen Shen 和 Alex Tamkin 主導。 Jake Eaton、Stuart Ritchie 和 Sarah Pollack 為這篇部落格文章提供了編輯支援。

We would like to thank Ethan Perez, Miranda Zhang, and Henry Sleight for making this project possible through the Anthropic Safety Fellows Program. We would also like to thank Matthew Jörke, Juliette Woodrow, Sarah Wu, Elizabeth Childs, Roshni Sahoo, Nate Rush, Julian Michael, and Rose Wang for experimental design feedback.
我們要感謝 Ethan Perez、Miranda Zhang 和 Henry Sleight 透過人為安全研究員計畫使本計畫得以完成。我們也要感謝 Matthew Jörke、Juliette Woodrow、Sarah Wu、Elizabeth Childs、Roshni Sahoo、Nate Rush、Julian Michael 和 Rose Wang 對實驗設計的寶貴意見。

```python
@misc{aiskillformation2026,
  author = {Shen, Judy Hanwen and Tamkin, Alex},
  title = {How AI Impacts Skill Formation},
  year = {2026},
  eprint = {2601.20245},
  archivePrefix = {arXiv},
  primaryClass = {cs.LG},
  eprinttype = {arxiv}
}
```

Footnotes  註腳

1. Importantly, this setup is different from agentic coding products like Claude Code; we expect that the impacts of such programs on skill development are likely to be more pronounced than the results here.
    重要的是，這種設定與 Claude Code 等智慧編碼產品不同；我們預計此類程式對技能發展的影響可能會比這裡的結果更加顯著。

---

# 保哥

近期 Anthropic 發布的一項實驗數據，在工程師社群引發了不小的討論，這份研究直接觸及了當前產業界最不願面對的隱憂，也就是生成式 AI 究竟是在幫助我們進化，還是在削弱我們解決問題的能力。這項實驗邀請了五十二位初級工程師，讓他們在不同的環境下學習一個全新的 Python 函式庫 Trio，這是一個處理非同步寫程式的工具，具有一定的學習曲線。

研究結果相當令人不安，使用 AI 輔助的小組，在後續的概念測試中僅獲得百分之五十二的平均分數，而完全手動的小組卻能達到百分之六十八。這百分之十七的落差，並非單純的數值波動，而是反映了工程師在理解力與除錯能力上的實質下降。身為一名在產業現場觀察多年的工程師，我認為這份數據揭示了一個長期被忽視的結構性危機。

![📦](https://static.xx.fbcdn.net/images/emoji.php/v9/t3d/1/16/1f4e6.png) 既有學習模式與產業慣性的拆解

* 過往學習新技術的過程高度依賴閱讀官方文件與源碼，這是一個建立基礎認知的必要路徑。
* 傳統的寫程式過程強調透過不斷的嘗試與錯誤來修正邏輯，這種認知掙扎是形塑工程師心智模型的核心。
* 工程師習慣透過邏輯推理來進行除錯，而非依賴機率性的建議，這確保了問題解決的可重複性。
* 知識的獲取往往伴隨著痛苦的思考過程，這種摩擦力雖然降低了短期的產出速度，卻保證了長期技術深度。
* 產業中對於品質的評估，過去是基於開發者對每一行程式內容的掌控程度，而非僅僅看最後的運行結果。

![🛠️](https://static.xx.fbcdn.net/images/emoji.php/v9/tb9/1/16/1f6e0.png) 新變化帶來的結構性影響

1. 虛假的生產力成長

實驗數據顯示使用 AI 的開發者平均僅節省了兩分鐘的作業時間，這在統計學上幾乎可以忽略不計，這種現象就像是一條看似截彎取直的交流道，雖然縮短了表面上的導航里程，卻因為在匯流處產生了嚴重的資訊超載與驗證困難，導致交通整體流速反而因為頻繁的認知回堵而停滯不前。

2. 提示詞工程帶來的隱形成本

參與者花費了將近百分之三十的時間在琢磨如何與 AI 溝通，而非思考問題的核心邏輯，這代表工程師的注意力正在從解決技術難點轉向如何操控黑盒工具，猶如在管線系統中花費過多精力去修補漏水的接頭，卻遺忘了水流本身的流向與壓力設計是否符合邏輯。

3. 認知掙扎的全面消失

低分組的開發者傾向將所有問題丟給 AI 生成後直接複製答案，完全避開了理解原理的過程，這種行為讓學習過程失去了必要的阻力，就像是為了省力而拆除了所有的水泵與壓力閥，雖然表面上流體可以順暢流動，但整個系統卻失去了推動高層次邏輯運行的核心動力。

4. 除錯能力的退化與依賴

部分開發者僅在遇到問題時求助 AI 修復錯誤，而不去深究錯誤產生的根源，這導致他們的除錯視野被侷限在極小的局部範圍內，我推測這將導致未來系統維護的複雜度大幅提升，因為沒人能從全局視角理解這些由 AI 拼接起來的邏輯。

5. 學習曲線的崩塌

高分組的開發者是將 AI 作為問答對象來深挖概念，而非單純索取程式碼，這兩者之間的差異決定了技術實力的天花板，如果開發者一開始就習慣讓 AI 接管思考，可能會導致初級工程師晉升為資深開發者的路徑被徹底截斷。

![📊](https://static.xx.fbcdn.net/images/emoji.php/v9/taa/1/16/1f4ca.png) 影響評估與產業觀察

對於初級開發者而言，這份數據是一個嚴肅的警訊，如果我們在職涯初期就習慣於這種低摩擦力的開發模式，極有可能在不知不覺中喪失了處理複雜架構的能力。我認為未來軟體開發領域的貧富差距將會擴大，具備獨立思考能力並將 AI 視為輔助工具的工程師，將會與那些只會複製 AI 建議的工程師拉開極大的距離。

從商業角度來看，企業雖然希望透過 AI 提升開發品質與速度，但如果這份速度是以犧牲團隊整體的理解力為代價，那可能是一筆賠本生意。長期而言，企業可能需要重新定義培訓機制，甚至在教育訓練中刻意移除 AI 工具，以確保工程師能夠建立紮實的底層邏輯。我懷疑目前市場上過度吹捧 AI 開發工具的風氣，某種程度上掩蓋了人才素質可能大幅下滑的結構性風險。

這種影響也將波及到軟體外包與電商系統的穩定性，當大量的系統是由一群理解力下降百分之十七的工程師透過 AI 組裝而成時，系統崩潰後的修復成本將會呈指數級增長。對於產品經理與行銷人員來說，這代表未來技術團隊的交付品質可能變得更不穩定，即便交付的速度看起來變快了，隱藏的技術債可能早已在暗處堆積。

![🔒](https://static.xx.fbcdn.net/images/emoji.php/v9/t2e/1/16/1f512.png) 未來的核心挑戰

這項研究讓我們必須重新審視人機協作的本質，工具的演進應當是為了延伸人類的智慧，而不是取代思考的過程。如果我們無法在開發效率與深度學習之間找到新的平衡點，那麼這場由生成式 AI 帶來的技術革命，最終可能會變成一場集體的認知倒退。

我們是否應該在工程師的成長過程中，刻意保留一定程度的低效與痛苦，以換取更深層次的技術掌握力？
