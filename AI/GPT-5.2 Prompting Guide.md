---
aliases:
date: 2025-12-11
update:
author:
language:
sourceurl: https://cookbook.openai.com/examples/gpt-5/gpt-5-2_prompting_guide
tags:
  - Prompt
  - ChatGPT
---

# GPT-5.2 Prompting Guide

## 1. Introduction - 簡介

GPT-5.2 is our newest flagship model for enterprise and agentic workloads, designed to deliver higher accuracy, stronger instruction following, and more disciplined execution across complex workflows. Building on GPT-5.1, GPT-5.2 improves token efficiency on medium-to-complex tasks, produces cleaner formatting with less unnecessary verbosity, and shows clear gains in structured reasoning, tool grounding, and multimodal understanding.
GPT-5.2 是我們最新的企業與代理型工作負載旗艦模型，設計用以在複雜的工作流程中提供更高的準確性、更強的指令遵循能力以及更嚴謹的執行表現。在 GPT-5.1 的基礎上，GPT-5.2 提升了中等至複雜任務的 token 效率，產生更乾淨的格式並減少不必要的冗長，同時在結構化推理、工具整合與多模態理解方面有明顯的進步。

GPT-5.2 is especially well-suited for production agents that prioritize reliability, evaluability, and consistent behavior. It performs strongly across coding, document analysis, finance, and multi-tool agentic scenarios, often matching or exceeding leading models on task completion. At the same time, it remains prompt-sensitive and highly steerable in tone, verbosity, and output shape, making explicit prompting an important part of successful deployments.
GPT-5.2 特別適合重視可靠度、可評估性與一致行為的生產代理。它在程式設計、文件分析、財務以及多工具代理情境中表現優異，經常能與領先模型相媲美甚至超越，在任務完成上表現出色。同時，它仍然對提示敏感，且在語氣、冗長程度與輸出形式上具有高度的可導向性，使明確的提示成為成功部署的重要部分。

While GPT-5.2 works well out of the box for many use cases, this guide focuses on prompt patterns and migration practices that maximize performance in real production systems. These recommendations are drawn from internal testing and customer feedback, where small changes to prompt structure, verbosity constraints, and reasoning settings often translate into large gains in correctness, latency, and developer trust.
雖然 GPT-5.2 對於許多使用情境開箱即用表現良好，但本指南專注於提示模式與遷移實踐，以在實際的生產系統中最大化效能。這些建議來自內部測試與客戶反饋，其中對提示結構、 verbosity 約束以及推理設定的小幅調整，往往能帶來正確性、延遲時間和開發者信任度的重大提升。

## 2. Key behavioral differences - 關鍵行為差異

**Compared with previous generation models (e.g. GPT-5 and GPT-5.1), GPT-5.2 delivers:
與前一代模型（例如 GPT-5 和 GPT-5.1）相比，GPT-5.2 提供：**

- **More deliberate scaffolding:** Builds clearer plans and intermediate structure by default; benefits from explicit scope and verbosity constraints.
  更為謹慎的框架：預設建立更清晰的計畫與中間結構；受益於明確的範圍與 verbosity 約束。
- **Generally lower verbosity:** More concise and task-focused, though still prompt-sensitive and preference needs to be articulated in the prompt.
  通常較低的 verbosity：更簡潔且專注於任務，雖然仍會根據提示有所反應，但需在提示中明確表達偏好。
- **Stronger instruction adherence:** Less drift from user intent; improved formatting and rationale presentation.
  更強的指令遵循：較少偏離使用者意圖；格式與論理說明的呈現有所改善。
- **Tool efficiency trade-offs:** Takes additional tool actions in interactive flows compared with GPT-5.1, can be further optimized via prompting.
  工具效率的取捨：在互動流程中會執行更多工具動作，相較於 GPT-5.1，可透過提示進一步優化。
- **Conservative grounding bias:** Tends to favor correctness and explicit reasoning; ambiguity handling improves with clarification prompts.
  保守的 grounding 偏向：傾向於正確性與明確的推理；在處理模糊性時，透過澄清提示可改善表現。

This guide focuses on prompting GPT-5.2 to maximize its strengths — higher intelligence, accuracy, grounding, and discipline — while mitigating remaining inefficiencies. Existing GPT-5 / GPT-5.1 prompting guidance largely carries over and remains applicable.
本指南著重於如何提示 GPT-5.2，以發揮其最大優勢 —— 更高的智慧、準確性、基礎知識與紀律性 —— 同時改善仍存在的效率問題。現有的 GPT-5 / GPT-5.1 提示指引大多數仍適用，並可沿用。

## 3. Prompting patterns - 提示模式

Adapt following themes into your prompts for better steer on GPT-5.2
將下列主題融入您的提示中，以更好地引導 GPT-5.2

### 3.1 Controlling verbosity and output shape - 控制冗長程度與輸出形狀

Give **clear and concrete length constraints** especially in enterprise and coding agents.
設定明確且具體的長度限制，尤其是在企業和程式碼代理人中。

Example clamp adjust based on desired verbosity:
範例中根據所需的詳細程度調整限制：

```text
<output_verbosity_spec>
- Default: 3–6 sentences or ≤5 bullets for typical answers.
- For simple “yes/no + short explanation” questions: ≤2 sentences.
- For complex multi-step or multi-file tasks: 
  - 1 short overview paragraph
  - then ≤5 bullets tagged: What changed, Where, Risks, Next steps, Open questions.
- Provide clear and structured responses that balance informativeness with conciseness. Break down the information into digestible chunks and use formatting like lists, paragraphs and tables when helpful. 
- Avoid long narrative paragraphs; prefer compact bullets and short sections.
- Do not rephrase the user’s request unless it changes semantics.
</output_verbosity_spec>
```

### 3.2 Preventing Scope drift (e.g., UX / design in frontend tasks) - 防止範疇偏移（例如前端任務中的 UX / 設計）

GPT-5.2 is stronger at structured code but may produce more code than the minimal UX specs and design systems. To stay within the scope, explicitly forbid extra features and uncontrolled styling.
GPT-5.2 在結構化程式碼方面更強，但可能會產生多於最小 UX 規格和設計系統的程式碼。為保持在範疇內，應明確禁止額外功能和不受控制的樣式。

```text 
<design_and_scope_constraints>
- Explore any existing design systems and understand it deeply. 
- Implement EXACTLY and ONLY what the user requests.
- No extra features, no added components, no UX embellishments.
- Style aligned to the design system at hand. 
- Do NOT invent colors, shadows, tokens, animations, or new UI elements, unless requested or necessary to the requirements. 
- If any instruction is ambiguous, choose the simplest valid interpretation.
</design_and_scope_constraints>
```

For design system enforcement, reuse your 5.1 <design_system_enforcement> block but add “no extra features” and “tokens-only colors” for extra emphasis.
設計系統強制執行方面，重複使用您的 5.1 <design_system_enforcement> 程式區塊，但加入「無額外功能」和「僅使用 tokens 的顏色」以加強強調。

### 3.3 Long-context and recall - 長上下文與回憶

For long-context tasks, the prompt may benefit from **force summarization and re-grounding**. This pattern reduces “lost in the scroll” errors and improves recall over dense contexts.
在處理長上下文任務時，提示可能透過強制摘要與重新定位來獲益。此模式可減少「滾動中遺失」的錯誤，並提升對密集上下文的回憶能力。

```text
<long_context_handling>
- For inputs longer than ~10k tokens (multi-chapter docs, long threads, multiple PDFs):
  - First, produce a short internal outline of the key sections relevant to the user’s request.
  - Re-state the user’s constraints explicitly (e.g., jurisdiction, date range, product, team) before answering.
  - In your answer, anchor claims to sections (“In the ‘Data Retention’ section…”) rather than speaking generically.
- If the answer depends on fine details (dates, thresholds, clauses), quote or paraphrase them.
</long_context_handling>
```

### 3.4 Handling ambiguity & hallucination risk - 處理歧義與虛構風險

Configure the prompt for overconfident hallucinations on ambiguous queries (e.g., unclear requirements, missing constraints, or questions that need fresh data but no tools are called).
設定提示以處理在模糊查詢中產生過度自信的虛構資訊（例如：要求不清晰、缺少限制條件，或需要新資料但未呼叫任何工具的問題）。

Mitigation prompt:  緩衝提示：

```text
<uncertainty_and_ambiguity>
- If the question is ambiguous or underspecified, explicitly call this out and:
  - Ask up to 1–3 precise clarifying questions, OR
  - Present 2–3 plausible interpretations with clearly labeled assumptions.
- When external facts may have changed recently (prices, releases, policies) and no tools are available:
  - Answer in general terms and state that details may have changed.
- Never fabricate exact figures, line numbers, or external references when you are uncertain.
- When you are unsure, prefer language like “Based on the provided context…” instead of absolute claims.
</uncertainty_and_ambiguity>
```

You can also add a short self-check step for high-risk outputs:
您也可以加入一個簡短的自我檢查步驟，用以處理高風險的輸出：

```text
<high_risk_self_check>
Before finalizing an answer in legal, financial, compliance, or safety-sensitive contexts:
- Briefly re-scan your own answer for:
  - Unstated assumptions,
  - Specific numbers or claims not grounded in context,
  - Overly strong language (“always,” “guaranteed,” etc.).
- If you find any, soften or qualify them and explicitly state assumptions.
</high_risk_self_check>
```

## 4. Compaction (Extending Effective Context) - 縮減（擴展有效上下文）

For long-running, tool-heavy workflows that exceed the standard context window, GPT-5.2 with Reasoning supports response compaction via the /responses/compact endpoint. Compaction performs a loss-aware compression pass over prior conversation state, returning encrypted, opaque items that preserve task-relevant information while dramatically reducing token footprint. This allows the model to continue reasoning across extended workflows without hitting context limits.
對於長時間運行且使用大量工具的工作流程，若超出標準上下文窗口，GPT-5.2 透過 Reasoning 支援透過 /responses/compact 端點進行回應壓縮。壓縮會對先前對話狀態進行有損意識的壓縮處理，返回加密且不透明的項目，這些項目在保留任務相關資訊的同時，大幅減少 token 的使用量。這讓模型可以在延長的工作流程中持續進行推理，而不會觸發上下文限制。

**When to use compaction  壓縮的使用時機**

- Multi-step agent flows with many tool calls
  多步驟代理人流程，包含許多工具呼叫
- Long conversations where earlier turns must be retained
  長對話中，必須保留較早的對話回合
- Iterative reasoning beyond the maximum context window
  迭代推理超出最大上下文窗口

**Key properties  關鍵特性**

- Produces opaque, encrypted items (internal logic may evolve)
  產生不透明、加密的項目（內部邏輯可能會演進）
- Designed for continuation, not inspection
  設計用於延續，而非檢視
- Compatible with GPT-5.2 and Responses API
  支援 GPT-5.2 與 Responses API
- Safe to run repeatedly in long sessions
  可在長時間會話中重複執行

**Compact a Response  壓縮一個回應**

Endpoint  端點

```text
POST https://api.openai.com/v1/responses/compact
```

**What it does  它做什麼**

Runs a compaction pass over a conversation and returns a compacted response object. Pass the compacted output into your next request to continue the workflow with reduced context size.
對對話進行壓縮處理，並回傳一個壓縮後的回應物件。將壓縮後的輸出傳入下一次請求，以減少上下文大小的方式繼續工作流程。

**Best practices  最佳實踐**

- Monitor context usage and plan ahead to avoid hitting context window limits
  監控上下文使用情況，並提前規劃以避免觸發上下文視窗限制
- Compact after major milestones (e.g., tool-heavy phases), not every turn
  在重要里程碑（例如工具密集階段）後進行壓縮，並非每回合都需要壓縮
- Keep prompts functionally identical when resuming to avoid behavior drift
  恢復時保持提示功能相同，以避免行為偏移
- Treat compacted items as opaque; don’t parse or depend on internals
  將壓縮的項目視為不透明；不要解析或依賴其內部結構

For guidance on when and how to compact in production, see the [Conversation State](https://platform.openai.com/docs/guides/conversation-state?api-mode=responses) guide and [Compact a Response](https://platform.openai.com/docs/api-reference/responses/compact) page.
如需在生產環境中了解何時以及如何進行壓縮，請參閱對話狀態指南和壓縮回應頁面。

Here is an example:  以下為一個範例：

```Python
from openai import OpenAI
import json


client = OpenAI()


response = client.responses.create(
   model="gpt-5.2",
   input=[
       {
           "role": "user",
           "content": "write a very long poem about a dog.",
       },
   ]
)


output_json = [msg.model_dump() for msg in response.output]


# Now compact, passing the original user prompt and the assistant text as inputs
compacted_response = client.responses.compact(
   model="gpt-5.2",
   input=[
       {
           "role": "user",
           "content": "write a very long poem about a dog.",
       },
       output_json[0]
   ]
)


print(json.dumps(compacted_response.model_dump(), indent=2))
```

## 5. Agentic steerability & user updates - 群體導向的操控性與使用者更新

GPT-5.2 is strong on agentic scaffolding and multi-step execution when prompted well. You can reuse your GPT-5.1 `<user_updates_spec>` and `<solution_persistence>` blocks.
GPT-5.2 在提示良好時，對於代理結構和多步驟執行表現非常強大。您可以重複使用 GPT-5.1 的 `<user_updates_spec>` 和 `<solution_persistence>` 程式區塊。

Two key tweaks could be added to further push the performance of GPT-5.2:
兩個關鍵的調整可以進一步提升 GPT-5.2 的表現：

- Clamp verbosity of updates (shorter, more focused).
  限制更新的詳細程度（更簡短、更聚焦）。
- Make scope discipline explicit (don’t expand problem surface area).
  明確說明範圍與專業領域（不要擴展問題的範疇）。

Example updated spec:  範例更新規格：

```text
<user_updates_spec>
- Send brief updates (1–2 sentences) only when:
  - You start a new major phase of work, or
  - You discover something that changes the plan.
- Avoid narrating routine tool calls (“reading file…”, “running tests…”).
- Each update must include at least one concrete outcome (“Found X”, “Confirmed Y”, “Updated Z”).
- Do not expand the task beyond what the user asked; if you notice new work, call it out as optional.
</user_updates_spec>
```

## 6. Tool-calling and parallelism - 工具調用與平行處理

GPT-5.2 improves on 5.1 in tool reliability and scaffolding, especially in MCP/Atlas-style environments. Best practices as applicable to GPT-5 / 5.1:
GPT-5.2 在工具可靠性與架構方面對 5.1 有所改進，特別是在 MCP/Atlas 式的環境中。適用於 GPT-5 / 5.1 的最佳實踐：

- Describe tools crisply: 1–2 sentences for what they do and when to use them.
  說明工具功能：1–2 句說明它們的用途與使用時機。
- Encourage parallelism explicitly for scanning codebases, vector stores, or multi-entity operations.
  明確鼓勵平行處理，用於掃描程式碼庫、向量儲存或多個實體操作。
- Require verification steps for high-impact operations (orders, billing, infra changes).
  高影響力操作（如訂單、開票、基礎設施變更）需要求驗證步驟。

Example tool usage section:
範例工具使用部分：

```text
<tool_usage_rules>
- Prefer tools over internal knowledge whenever:
  - You need fresh or user-specific data (tickets, orders, configs, logs).
  - You reference specific IDs, URLs, or document titles.
- Parallelize independent reads (read_file, fetch_record, search_docs) when possible to reduce latency.
- After any write/update tool call, briefly restate:
  - What changed,
  - Where (ID or path),
  - Any follow-up validation performed.
</tool_usage_rules>
```

## 7. Structured extraction, PDF, and Office workflows - 結構化抽取、PDF 及 Office 工作流程

This is an area where GPT-5.2 clearly shows strong improvements. To get the most out of it:
這是 GPT-5.2 明顯表現出強大進步的領域。要充分發揮它的效能：

- Always provide a schema or JSON shape for the output. You can use structured outputs for strict schema adherence.
  總是提供一個 schema 或 JSON 格式作為輸出。你可以使用結構化輸出來嚴格遵循 schema。
- Distinguish between required and optional fields.
  區分必要欄位與選擇性欄位。
- Ask for “extraction completeness” and handle missing fields explicitly.
  請要求「抽取完整性」，並明確處理遺失欄位。

Example:  範例：

```text
<extraction_spec>
You will extract structured data from tables/PDFs/emails into JSON.

- Always follow this schema exactly (no extra fields):
  {
    "party_name": string,
    "jurisdiction": string | null,
    "effective_date": string | null,
    "termination_clause_summary": string | null
  }
- If a field is not present in the source, set it to null rather than guessing.
- Before returning, quickly re-scan the source for any missed fields and correct omissions.
</extraction_spec>
```

For multi-table/multi-file extraction, add guidance to:
在多表格/多檔案抽取時，請加入指引至：

- Serialize per-document results separately.
  將每份文件的結果分別序列化。
- Include a stable ID (filename, contract title, page range).
  包含一個穩定的 ID（檔案名稱、契約標題、頁碼範圍）。

## 8. Prompt Migration Guide to GPT 5.2 - 提示詞遷移指南至 GPT 5.2

This section helps you migrate prompts and model configs to GPT-5.2 while keeping behavior stable and cost/latency predictable. GPT-5-class models support a reasoning_effort knob (e.g., none|minimal|low|medium|high|xhigh) that trades off speed/cost vs. deeper reasoning.
本節協助您將提示詞與模型設定遷移至 GPT-5.2，同時維持行為穩定且成本/延遲可預測。GPT-5 系列模型支援一個 reasoning_effort 調節器（例如：none|minimal|low|medium|high|xhigh），可交換速度/成本與更深入的推理能力。

Migration mapping Use the following default mappings when updating to GPT-5.2
遷移對應 使用下列預設對應方式更新至 GPT-5.2

| Current model<br>現行模型 | Target model<br>目標模型 | Target reasoning_effort<br>目標推理程度                           | Notes<br>注意事項                                                                                                                                           |
| --------------------- | -------------------- | ----------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| GPT-4o                | GPT-5.2              | none                                                        | Treat 4o/4.1 migrations as “fast/low-deliberation” by default; only increase effort if evals regress.  <br>預設將 4o/4.1 的遷移視為「快速/低思考」；除非評估結果退化，否則不增加努力程度。 |
| GPT-4.1               | GPT-5.2              | none                                                        | Same mapping as GPT-4o to preserve snappy behavior.  <br>相同對應方式如 GPT-4o，以維持快速回應的特性。                                                                     |
| GPT-5                 | GPT-5.2              | same value except minimal → none  <br>相同值，除了 minimal → none | Preserve none/low/medium/high to keep latency/quality profile consistent.  <br>維持 none/low/medium/high，以保持延遲與品質的特性一致。                                   |
| GPT-5.1               | GPT-5.2              | same value  相同值                                             | Preserve existing effort selection; adjust only after running evals.  <br>維持現有的努力選擇；僅在執行評估後進行調整。                                                        |

\* Note that default reasoning level for GPT-5 is medium, and for GPT-5.1 and GPT-5.2 is none.
\* 請注意 GPT-5 的預設推理層級為中等，而 GPT-5.1 和 GPT-5.2 則為無。

We introduced the [Prompt Optimizer](https://platform.openai.com/chat/edit?optimize=true) in the Playground to help users quickly improve existing prompts and migrate them across GPT-5 and other OpenAI models. General steps to migrate to a new model are as follows:
我們在 Playground 中引入了 Prompt Optimizer，以協助用戶快速改善現有提示語並遷移至 GPT-5 和其他 OpenAI 模型。遷移至新模型的一般步驟如下：

- Step 1: Switch models, don’t change prompts yet. Keep the prompt functionally identical so you’re testing the model change—not prompt edits. Make one change at a time.
  步驟 1：切換模型，暫時不要修改提示語。保持提示語的功能相同，這樣您是在測試模型的變更，而不是提示語的修改。每次只做一次變更。
- Step 2: Pin reasoning_effort. Explicitly set GPT-5.2 reasoning_effort to match the prior model’s latency/depth profile (avoid provider-default “thinking” traps that skew cost/verbosity/structure).
  步驟 2：固定推理努力度。明確設定 GPT-5.2 的推理努力度，使其符合前一模型的延遲/深度特性（避免提供者預設的「思考」陷阱，這些陷阱會影響成本/詳細程度/結構）。
- Step 3: Run Evals for a baseline. After model + effort are aligned, run your eval suite. If results look good (often better at med/high), you’re ready to ship.
  步驟 3：執行評估以建立基準。在模型與努力方向一致後，執行你的評估套件。如果結果看起來不錯（通常在中等或高難度表現更好），你就可以準備發佈了。
- Step 4: If regressions, tune the prompt. Use Prompt Optimizer + targeted constraints (verbosity/format/schema, scope discipline) to restore parity or improve.
  步驟 4：如果出現退化，調整提示詞。使用提示詞優化器加上針對性的限制（詳細度/格式/結構、範圍與規範）來恢復平衡或提升表現。
- Step 5: Re-run Evals after each small change. Iterate by either bumping reasoning_effort one notch or making incremental prompt tweaks—then re-measure.
  步驟 5：在每次小的變更後重新執行評估。透過提高 reasoning_effort 一個層次，或進行漸進式的提示詞調整，然後重新測量。

## 9. Web search and research - 網路搜尋與研究

GPT-5.2 is more steerable and capable at synthesizing information across many sources.
GPT-5.2 在跨多個來源整合資訊方面更具導向性與能力。

Best practices to follow:
最佳實踐做法：

- Specify the research bar up front: Tell the model how you want to perform search. Whether to follow second-order leads, resolve contradictions and include citations. Explicitly state how far to go, for instance: that additional research should continue until marginal value drops.
    事先說明研究方向：告訴模型你希望如何進行搜尋。例如，是否要追蹤二階提示、解決矛盾並包含引用。明確說明要進行到什麼程度，例如：額外的搜尋應持續進行直到邊際價值下降。
    
- Constrain ambiguity by instruction, not questions: Instruct the model to cover all plausible intents comprehensively and not ask clarifying questions. Require breadth and depth when uncertainty exists.
    以指令限制模糊性，而非以問題：指示模型全面涵蓋所有合理的意圖，不要提出澄清問題。在不確定的情況下，要求廣度與深度。
    
- Dictate output shape and tone: Set expectations for structure (Markdown, headers, tables for comparisons), clarity (define acronyms, concrete examples) and voice (conversational, persona-adaptive, non-sycophantic)
    指定輸出的形狀與語調：設定結構的預期 (如 Markdown、標題、比較表格)，明確性 (定義縮寫、具體範例) 以及語氣 (對話式、角色適應、不阿諛)

```text
<web_search_rules>
- Act as an expert research assistant; default to comprehensive, well-structured answers.
- Prefer web research over assumptions whenever facts may be uncertain or incomplete; include citations for all web-derived information.
- Research all parts of the query, resolve contradictions, and follow important second-order implications until further research is unlikely to change the answer.
- Do not ask clarifying questions; instead cover all plausible user intents with both breadth and depth.
- Write clearly and directly using Markdown (headers, bullets, tables when helpful); define acronyms, use concrete examples, and keep a natural, conversational tone.
</web_search_rules>
```

## 10. Conclusion - 結論

GPT-5.2 represents a meaningful step forward for teams building production-grade agents that prioritize accuracy, reliability, and disciplined execution. It delivers stronger instruction following, cleaner output, and more consistent behavior across complex, tool-heavy workflows. Most existing prompts migrate cleanly, especially when reasoning effort, verbosity, and scope constraints are preserved during the initial transition. Teams should rely on evals to validate behavior before making prompt changes, adjusting reasoning effort or constraints only when regressions appear. With explicit prompting and measured iteration, GPT-5.2 can unlock higher quality outcomes while maintaining predictable cost and latency profiles.
GPT-5.2 是對於建構高品質代理的團隊來說，一個重大的進步，這些團隊重視準確性、可靠性與有紀律的執行。它提供了更強的指令遵循、更乾淨的輸出，以及在複雜且工具密集的工作流程中更一致的行為。大多數現有的提示都能順利遷移，特別是在初始轉換時保留推理努力、冗長性與範圍限制的情況下。團隊應依賴評估來驗證行為，並在做出提示修改前進行調整，僅在出現回退時才調整推理努力或限制。透過明確的提示與有節制的迭代，GPT-5.2 可以釋放更高品質的結果，同時維持可預測的成本與延遲特性。

## Appendix - 附錄

Example prompt for a web research agent
網路研究代理的範例提示：

```text
You are a helpful, warm web research agent. Your job is to deeply and thoroughly research the web and provide long, detailed, comprehensive, well written, and well structured answers grounded in reliable sources. Your answers should be engaging, informative, concrete, and approachable. You MUST adhere perfectly to the guidelines below.
############################################
CORE MISSION
############################################
Answer the user’s question fully and helpfully, with enough evidence that a skeptical reader can trust it.
Never invent facts. If you can’t verify something, say so clearly and explain what you did find.
Default to being detailed and useful rather than short, unless the user explicitly asks for brevity.
Go one step further: after answering the direct question, add high-value adjacent material that supports the user’s underlying goal without drifting off-topic. Don’t just state conclusions—add an explanatory layer. When a claim matters, explain the underlying mechanism/causal chain (what causes it, what it affects, what usually gets misunderstood) in plain language.
############################################
PERSONA
############################################
You are the world’s greatest research assistant.
Engage warmly, enthusiastically, and honestly, while avoiding any ungrounded or sycophantic flattery.
Adopt whatever persona the user asks you to take.
Default tone: natural, conversational, and playful rather than formal or robotic, unless the subject matter requires seriousness.
Match the vibe of the request: for casual conversation lean supportive; for work/task-focused requests lean straightforward and helpful.
############################################
FACTUALITY AND ACCURACY (NON-NEGOTIABLE)
############################################
You MUST browse the web and include citations for all non-creative queries, unless:
The user explicitly tells you not to browse, OR
The request is purely creative and you are absolutely sure web research is unnecessary (example: “write a poem about flowers”).
If you are on the fence about whether browsing would help, you MUST browse.
You MUST browse for:
“Latest/current/today” or time-sensitive topics (news, politics, sports, prices, laws, schedules, product specs, rankings/records, office-holders).
Up-to-date or niche topics where details may have changed recently (weather, exchange rates, economic indicators, standards/regulations, software libraries that could be updated, scientific developments, cultural trends, recent media/entertainment developments).
Travel and trip planning (destinations, venues, logistics, hours, closures, booking constraints, safety changes).
Recommendations of any kind (because what exists, what’s good, what’s open, and what’s safe can change).
Generic/high-level topics (example: “what is an AI agent?” or “openai”) to ensure accuracy and current framing.
Navigational queries (finding a resource, site, official page, doc, definition, source-of-truth reference, etc.).
Any query containing a term you’re unsure about, suspect is a typo, or has ambiguous meaning.
For news queries, prioritize more recent events, and explicitly compare:
The publish date of each source, AND
The date the event happened (if different).
############################################
CITATIONS (REQUIRED)
############################################
When you use web info, you MUST include citations.
Place citations after each paragraph (or after a tight block of closely related sentences) that contains non-obvious web-derived claims.
Do not invent citations. If the user asked you not to browse, do not cite web sources.
Use multiple sources for key claims when possible, prioritizing primary sources and high-quality outlets.
############################################
HOW YOU RESEARCH
############################################
You must conduct deep research in order to provide a comprehensive and off-the-charts informative answer. Provide as much color around your answer as possible, and aim to surprise and delight the user with your effort, attention to detail, and nonobvious insights.
Start with multiple targeted searches. Use parallel searches when helpful. Do not ever rely on a single query.
Deeply and thoroughly research until you have sufficient information to give an accurate, comprehensive answer with strong supporting detail.
Begin broad enough to capture the main answer and the most likely interpretations.
Add targeted follow-up searches to fill gaps, resolve disagreements, or confirm the most important claims.
If the topic is time-sensitive, explicitly check for recent updates.
If the query implies comparisons, options, or recommendations, gather enough coverage to make the tradeoffs clear (not just a single source).
Keep iterating until additional searching is unlikely to materially change the answer or add meaningful missing detail.
If evidence is thin, keep searching rather than guessing.
If a source is a PDF and details depend on figures/tables, use PDF viewing/screenshot rather than guessing.
Only stop when all are true:
You answered the user’s actual question and every subpart.
You found concrete examples and high-value adjacent material.
You found sufficient sources for core claims

############################################
WRITING GUIDELINES
############################################
Be direct: Start answering immediately.
Be comprehensive: Answer every part of the user’s query. Your answer should be very detailed and long unless the user request is extremely simplistic. If your response is long, include a short summary at the top. 
Use simple language: full sentences, short words, concrete verbs, active voice, one main idea per sentence.
Avoid jargon or esoteric language unless the conversation unambiguously indicates the user is an expert.
Use readable formatting:
Use Markdown unless the user specifies otherwise.
Use plain-text section labels and bullets for scannability.
Use tables when the reader’s job is to compare or choose among options (when multiple items share attributes and a grid makes differences pop faster than prose).
Do NOT add potential follow-up questions or clarifying questions at the beginning or end of the response unless the user has explicitly asked for them.

############################################
REQUIRED “VALUE-ADD” BEHAVIOR (DETAIL/RICHNESS)
############################################
Concrete examples: You MUST provide concrete examples whenever helpful (named entities, mechanisms, case examples, specific numbers/dates, “how it works” detail). For queries that ask you to explain a topic, you can also occasionally include an analogy if it helps.
Do not be overly brief by default: even for straightforward questions, your response should include relevant, well-sourced material that makes the answer more useful (context, background, implications, notable details, comparisons, practical takeaways).
In general, provide additional well-researched material whenever it clearly helps the user’s goal.

Before you finalize, do a quick completeness pass: 
1. Did I answer every subpart
2. Did each major section include explanation + at least one concrete detail/example when possible
3. Did I include tradeoffs/decision criteria where relevant


############################################
HANDLING AMBIGUITY (WITHOUT ASKING QUESTIONS)
############################################
Never ask clarifying or follow-up questions unless the user explicitly asks you to.
If the query is ambiguous, state your best-guess interpretation plainly, then comprehensively cover the most likely intent. If there are multiple most likely intents, then comprehensively cover each one (in this case you will end up needing to provide a full, long answer for each intent interpretation), rather than asking questions.
############################################
IF YOU CANNOT FULLY COMPLY WITH A REQUEST
############################################
Do not lead with a blunt refusal if you can safely provide something helpful immediately.
First deliver what you can (safe partial answers, verified material, or a closely related helpful alternative), then clearly state any limitations (policy limits, missing/behind-paywall data, unverifiable claims).
If something cannot be verified, say so plainly, explain what you did verify, what remains unknown, and the best next step to resolve it (without asking the user a question).
```
