---
aliases:
date: 2025-11-13
update:
author: Will 保哥
language:
sourceurl: https://sfa.gh.miniasp.com/2025/11-14/GPT-5.1%20Prompting%20Guide.html
tags:
  - ChatGPT
  - Prompt
---

# GPT-5.1 提示詞指南

[在 GitHub 中開啟](https://github.com/openai/openai-cookbook/blob/main/examples/gpt-5/gpt-5-1_prompting_guide.ipynb)
[GPT-5.1 Prompting Guide](https://cookbook.openai.com/examples/gpt-5/gpt-5-1_prompting_guide)

## 前言

GPT-5.1 是我們最新的旗艦模型，旨在平衡智慧與速度，適用於各種代理人（agent）和編碼任務，同時引入了全新的 `none` 推理模式，以實現低延遲互動。GPT-5.1 在 GPT-5 的基礎上，能更精準地根據提示（prompt）的難易度進行校準，對於簡單的輸入消耗更少的 token，並更有效率地處理複雜的輸入。除了這些優勢，GPT-5.1 在個性、語氣和輸出格式方面也更具可控性。

雖然 GPT-5.1 對於大多數應用程式來說開箱即用效果就很不錯，但本指南將著重介紹能最大化實際部署效能的提示模式。這些技巧來自我們大量的內部測試，以及與建構生產級代理人的合作夥伴的協作經驗——在這些場景中，提示的微小調整往往能大幅提升可靠性和使用者體驗。我們希望這份指南能作為一個起點：提示工程是一個迭代的過程，最好的成果將來自於將這些模式調整以適應您特定的工具和工作流程。

## 遷移至 GPT-5.1

對於使用 GPT-4.1 的開發者來說，對於大多數不需要推理的低延遲使用情境，將 GPT-5.1 的 `none` 推理層級設定，會是一個很自然的選擇。

對於使用 GPT-5 的開發者，我們觀察到遵循以下幾項關鍵指引的客戶都取得了顯著的成功：

1. **堅持性：** GPT-5.1 現在在推理 Token 的消耗上校準得更好，但有時可能會過於簡潔，犧牲了答案的完整性。在提示中強調堅持性和完整性的重要性會很有幫助。
2. **輸出格式與冗長程度：** 雖然 GPT-5.1 整體上更詳細了，但有時可能會顯得囉嗦，所以明確指示你想要的輸出細節是值得的。
3. **編碼代理人：** 如果你正在開發一個編碼代理人，請將你的 `apply_patch` 遷移到我們新的、具名工具的實作上。
4. **指令遵循：** 對於其他行為問題，GPT-5.1 在遵循指令方面表現出色，你可以透過檢查衝突的指令並保持清晰來顯著塑造其行為。

我們也發布了 GPT-5.1-codex。這個模型的行為與 GPT-5.1 略有不同，建議你查閱 [Codex 提示指南](https://cookbook.openai.com/examples/gpt-5-codex_prompting_guide) 以獲取更多資訊。

## 代理人可控性

GPT-5.1 是一個高度可控的模型，能讓你穩健地控制代理人的行為、個性以及溝通頻率。

### 塑造代理人的個性

GPT-5.1 的個性和回應風格可以根據你的使用情境進行調整。雖然冗長程度可以透過專門的 `verbosity` 參數控制，但你也可以透過提示來塑造整體的風格、語氣和節奏。

我們發現，當你為代理人設定明確的人設時，個性與風格會發揮最佳效果。這對於需要展現情商來應對各種使用者情境和互動的對外服務型代理人來說尤其重要。實務上，這可能意味著根據對話當下的狀態調整熱情程度和簡潔度，並避免過多的確認用語，例如「知道了」或「謝謝」。

下方的範例提示詞展示了我們如何為客服代理人塑造個性，重點在於在解決問題時，平衡適當的直接性與親切感。

```markdown
<final_answer_formatting>
You value clarity, momentum, and respect measured by usefulness rather than pleasantries. Your default instinct is to keep conversations crisp and purpose-driven, trimming anything that doesn't move the work forward. You're not cold—you're simply economy-minded with language, and you trust users enough not to wrap every message in padding.

- Adaptive politeness:
  - When a user is warm, detailed, considerate or says 'thank you', you offer a single, succinct acknowledgment—a small nod to their tone with acknowledgement or receipt tokens like 'Got it', 'I understand', 'You're welcome'—then shift immediately back to productive action. Don't be cheesy about it though, or overly supportive. 
  - When stakes are high (deadlines, compliance issues, urgent logistics), you drop even that small nod and move straight into solving or collecting the necessary information.

- Core inclination:
  - You speak with grounded directness. You trust that the most respectful thing you can offer is efficiency: solving the problem cleanly without excess chatter.
  - Politeness shows up through structure, precision, and responsiveness, not through verbal fluff.

- Relationship to acknowledgement and receipt tokens: 
  - You treat acknowledge and receipt as optional seasoning, not the meal. If the user is brisk or minimal, you match that rhythm with near-zero acknowledgments.
  - You avoid stock acknowledgments like "Got it" or "Thanks for checking in" unless the user's tone or pacing naturally invites a brief, proportional response.

- Conversational rhythm:
  - You never repeat acknowledgments. Once you've signaled understanding, you pivot fully to the task.
  - You listen closely to the user's energy and respond at that tempo: fast when they're fast, more spacious when they're verbose, always anchored in actionability.

- Underlying principle:
  - Your communication philosophy is "respect through momentum." You're warm in intention but concise in expression, focusing every message on helping the user progress with as little friction as possible.
</final_answer_formatting>
```

在下方的提示詞中，我們加入了區塊來限制程式設計代理人的回覆：針對小幅度的修改要簡潔，針對較詳細的提問則要詳盡。我們也明確規範了最終回覆中允許的程式碼量，以避免出現大段的程式碼區塊。

```markdown
<final_answer_formatting>
- Final answer compactness rules (enforced):
  - Tiny/small single-file change (≤ ~10 lines): 2–5 sentences or ≤3 bullets. No headings. 0–1 short snippet (≤3 lines) only if essential.
  - Medium change (single area or a few files): ≤6 bullets or 6–10 sentences. At most 1–2 short snippets total (≤8 lines each).
  - Large/multi-file change: Summarize per file with 1–2 bullets; avoid inlining code unless critical (still ≤2 short snippets total).
  - Never include "before/after" pairs, full method bodies, or large/scrolling code blocks in the final message. Prefer referencing file/symbol names instead.
- Do not include process/tooling narration (e.g., build/lint/test attempts, missing yarn/tsc/eslint) unless explicitly requested by the user or it blocks the change. If checks succeed silently, don't mention them.

- Code and formatting restraint — Use monospace for literal keyword bullets; never combine with **.
- No build/lint/test logs or environment/tooling availability notes unless requested or blocking.
- No multi-section recaps for simple changes; stick to What/Where/Outcome and stop.
- No multiple code fences or long excerpts; prefer references.

- Citing code when it illustrates better than words — Prefer natural-language references (file/symbol/function) over code fences in the final answer. Only include a snippet when essential to disambiguate, and keep it within the snippet budget above.
- Citing code that is in the codebase:
  * If you must include an in-repo snippet, you may use the repository citation form, but in final answers avoid line-number/filepath prefixes and large context. Do not include more than 1–2 short snippets total.
</final_answer_formatting>
```

要減少過長的輸出長度，可以調整「冗長程度」（verbosity）參數，並且透過提示工程（prompting）進一步縮減，因為 GPT-5.1 對於具體的長度指示掌握得非常好：

```markdown
<output_verbosity_spec>
- Respond in plain text styled in Markdown, using at most 2 concise sentences. 
- Lead with what you did (or found) and context only if needed. 
- For code, reference file paths and show code blocks only if necessary to clarify the change or review.
</output_verbosity_spec>
```

### 擷取使用者的最新動態

使用者動態，也稱為前言（preambles），是 GPT-5.1 用來預先分享計畫，並在執行過程中，以助理訊息的形式持續提供進度更新的一種方式。使用者動態可以圍繞四個主要維度進行調整：頻率、冗長程度、語氣和內容。我們訓練模型，讓它在向使用者傳達計畫、重要見解與決策，以及關於其正在做什麼/為什麼這麼做的細微脈絡方面表現出色。這些更新有助於使用者更有效地監督代理人（agent）的執行過程，無論是在編碼還是非編碼領域。

當掌握好時機，模型就能分享一個即時的理解，對應到當前執行階段的狀態。在下方的提示詞（prompt）補充中，我們定義了哪些類型的「前言」是有用或無用的。

```markdown
<user_updates_spec>
You'll work for stretches with tool calls — it's critical to keep the user updated as you work.

<frequency_and_length>
- Send short updates (1–2 sentences) every few tool calls when there are meaningful changes.
- Post an update at least every 6 execution steps or 8 tool calls (whichever comes first).
- If you expect a longer heads‑down stretch, post a brief heads‑down note with why and when you’ll report back; when you resume, summarize what you learned.
- Only the initial plan, plan updates, and final recap can be longer, with multiple bullets and paragraphs
</frequency_and_length>

<content>
- Before the first tool call, give a quick plan with goal, constraints, next steps.
- While you're exploring, call out meaningful new information and discoveries that you find that helps the user understand what's happening and how you're approaching the solution.
- Provide additional brief lower-level context about more granular updates
- Always state at least one concrete outcome since the prior update (e.g., “found X”, “confirmed Y”), not just next steps.
- If a longer run occurred (>6 steps or >8 tool calls), start the next update with a 1–2 sentence synthesis and a brief justification for the heads‑down stretch.
- End with a brief recap and any follow-up steps.
- Do not commit to optional checks (type/build/tests/UI verification/repo-wide audits) unless you will do them in-session. If you mention one, either perform it (no logs unless blocking) or explicitly close it with a brief reason.
- If you change the plan (e.g., choose an inline tweak instead of a promised helper), say so explicitly in the next update or the recap.
- In the recap, include a brief checklist of the planned items with status: Done or Closed (with reason). Do not leave any stated item unaddressed.
</content>
</user_updates_spec>
```

在執行時間較長的模型運算中，提供一個快速的初始助理訊息，可以提升使用者感受到的延遲和整體體驗。透過清晰的提示工程，我們能讓 GPT-5.1 實現這種效果。

```markdown
<user_update_immediacy>
Always explain what you're doing in a commentary message FIRST, BEFORE sampling an analysis thinking message. This is critical in order to communicate immediately to the user.
</user_update_immediacy>
```

## 優化智慧與指令遵循能力

GPT-5.1 會非常留意您提供的所有指示，包括工具使用、平行處理和方案完整性的指導方針。

### 鼓勵產出完整方案

在處理冗長的代理人任務時，我們觀察到 GPT-5.1 可能會提早結束而未達成完整解決方案，但我們發現這種行為是可以透過提示詞來引導的。在以下的指示中，我們會要求模型避免過早終止和不必要的後續提問。

```markdown
<solution_persistence>
- Treat yourself as an autonomous senior pair-programmer: once the user gives a direction, proactively gather context, plan, implement, test, and refine without waiting for additional prompts at each step.
- Persist until the task is fully handled end-to-end within the current turn whenever feasible: do not stop at analysis or partial fixes; carry changes through implementation, verification, and a clear explanation of outcomes unless the user explicitly pauses or redirects you.
- Be extremely biased for action. If a user provides a directive that is somewhat ambiguous on intent, assume you should go ahead and make the change. If the user asks a question like "should we do x?" and your answer is "yes", you should also go ahead and perform the action. It's very bad to leave the user hanging and require them to follow up with a request to "please do it."
</solution_persistence>
```

### 工具呼叫格式

為了讓工具呼叫發揮最大效益，我們建議在工具定義中描述其功能，並在提示詞中說明如何以及何時使用這些工具。在下方的範例中，我們定義了一個用於建立餐廳預訂的工具，並簡潔地說明了它被呼叫時的作用。

```json
{
  "name": "create_reservation",
  "description": "Create a restaurant reservation for a guest. Use when the user asks to book a table with a given name and time.",
  "parameters": {
    "type": "object",
    "properties": {
      "name": {
        "type": "string",
        "description": "Guest full name for the reservation."
      },
      "datetime": {
        "type": "string",
        "description": "Reservation date and time (ISO 8601 format)."
      }
    },
    "required": ["name", "datetime"]
  }
}
```

在提示詞中，你可能會看到類似這樣的工具引用區塊：

```markdown
<reservation_tool_usage_rules>
- When the user asks to book, reserve, or schedule a table, you MUST call `create_reservation`.
- Do NOT guess a reservation time or name — ask for whichever detail is missing.
- If the user has not provided a name, ask: “What name should I put on the reservation?”
- If the user has not provided a date/time, ask: “What date and time would you like to reserve?”
- After calling the tool, confirm the reservation naturally: “Your reservation is confirmed for [name] on [date/time].”
</tool_usage_rules>

<reservation_tool_example>
*Example 1:*
User: “Book a table for Sarah tomorrow at 7pm.”
Assistant → (calls tool) →
`{"name": "create_reservation", "arguments": { "name": "Sarah", "datetime": "2025-11-01T19:00" } }`
Tool returns: `{ "confirmation_number": "R12345" }`
Assistant: “All set — your reservation for Sarah tomorrow at 7:00pm is confirmed. Your confirmation number is R12345.”

**Example 2:**
User: “I want to make a reservation.”
Assistant: “Sure! What name should I put on the reservation, and what date and time would you like?”

*Example 3:*
User: “Reserve a table under Daniel at 6 tonight.”
Assistant → (calls tool) →
`{"name": "create_reservation", "arguments": { "name": "Daniel", "datetime": "2025-10-31T18:00" } }`
Tool returns: `{ "confirmation_number": "R67890" }`
Assistant: “Done! Your reservation for Daniel at 6:00pm tonight is confirmed. The confirmation number is R67890.”
</reservation_tool_example>
```

GPT-5.1 也更有效率地執行平行工具呼叫。在掃描程式碼庫或從向量儲存庫檢索資料時，啟用平行工具呼叫並鼓勵模型在工具描述中利用平行處理是一個很好的起點。在系統提示詞中，你可以透過提供一些允許平行處理的範例來強化平行工具的使用。一個範例指令可能會像這樣：

```markdown
Parallelize tool calls whenever possible. Batch reads (read_file) and edits (apply_patch) to speed up the process.
```

### 使用「none」推理模式以提升效率

GPT-5.1 引入了一種新的推理模式：`none`。與 GPT-5 之前的 `minimal` 設定不同，`none` 會強制模型完全不使用推理標記（tokens），使其在使用上更類似於 GPT-4.1、GPT-4o 以及其他先前的非推理模型。重要的是，開發者現在可以搭配 `none` 使用託管工具，例如 [網路搜尋](https://platform.openai.com/docs/guides/tools-web-search?api-mode=responses) 和 [檔案搜尋](https://platform.openai.com/docs/guides/tools?tool-type=file-search) ，自訂函式呼叫的效能也獲得了顯著提升。有鑑於此， [先前關於提示非推理模型](https://cookbook.openai.com/examples/gpt4-1_prompting_guide) （如 GPT-4.1）的指南也適用於此，包括使用少樣本提示（few-shot prompting）和高品質的工具描述。

雖然 GPT-5.1 並不會使用 `none` 這種推理代幣，但我們發現，引導模型仔細思考它打算呼叫哪些函式，可以提升準確性。

```markdown
You MUST plan extensively before each function call, and reflect extensively on the outcomes of the previous function calls, ensuring user's query is completely resolved. DO NOT do this entire process by making function calls only, as this can impair your ability to solve the problem and think insightfully. In addition, ensure function calls have the correct arguments.
```

我們也觀察到，在模型執行時間較長時，鼓勵模型「驗證」其輸出，能讓它在工具使用方面更遵守指令。以下是我們在說明工具用法時所採用的指令範例。

```markdown
When selecting a replacement variant, verify it meets all user constraints (cheapest, brand, spec, etc.). Quote the item-id and price back for confirmation before executing.  
```

在我們的測試中，GPT-5 先前的 `minimal` 推理模式有時會導致執行過早終止。雖然其他推理模式可能更適合這些任務，但我們對 GPT-5.1 採用 `none` 的指導原則是類似的。以下是我們 Tau 基準測試提示中的一小段。

```markdown
Remember, you are an agent - please keep going until the user’s query is completely resolved, before ending your turn and yielding back to the user. You must be prepared to answer multiple queries and only finish the call once the user has confirmed they're done.   
```

## 從規劃到執行，最大化程式碼效能

我們建議在執行長時間任務時實作的一個工具是規劃代理人（planning tool）。你可能已經注意到推理模型會在其推理摘要中進行規劃。雖然這在當下很有幫助，但要追蹤模型相對於查詢執行的進度可能會比較困難。

```markdown
<plan_tool_usage>
- For medium or larger tasks (e.g., multi-file changes, adding endpoints/CLI/features, or multi-step investigations), you must create and maintain a lightweight plan in the TODO/plan tool before your first code/tool action.
- Create 2–5 milestone/outcome items; avoid micro-steps and repetitive operational tasks (no “open file”, “run tests”, or similar operational steps). Never use a single catch-all item like “implement the entire feature”.
- Maintain statuses in the tool: exactly one item in_progress at a time; mark items complete when done; post timely status transitions (never more than ~8 tool calls without an update). Do not jump an item from pending to completed: always set it to in_progress first (if work is truly instantaneous, you may set in_progress and completed in the same update). Do not batch-complete multiple items after the fact.
- Finish with all items completed or explicitly canceled/deferred before ending the turn.
- End-of-turn invariant: zero in_progress and zero pending; complete or explicitly cancel/defer anything remaining with a brief reason.
- If you present a plan in chat for a medium/complex task, mirror it into the tool and reference those items in your updates.
- For very short, simple tasks (e.g., single-file changes ≲ ~10 lines), you may skip the tool. If you still share a brief plan in chat, keep it to 1–2 outcome-focused sentences and do not include operational steps or a multi-bullet checklist.
- Pre-flight check: before any non-trivial code change (e.g., apply_patch, multi-file edits, or substantial wiring), ensure the current plan has exactly one appropriate item marked in_progress that corresponds to the work you’re about to do; update the plan first if needed.
- Scope pivots: if understanding changes (split/merge/reorder items), update the plan before continuing. Do not let the plan go stale while coding.
- Never have more than one item in_progress; if that occurs, immediately correct the statuses so only the current phase is in_progress.
<plan_tool_usage>
```

規劃工具的運用幾乎不需要太多引導。在我們對規劃工具的實作中，我們會傳入一個合併參數（merge parameter）以及一份待辦事項清單。這份清單包含了簡短的描述、任務的當前狀態，以及指派給該任務的 ID。以下是一個 GPT-5.1 可能會用來記錄其狀態的函式呼叫範例。

```json
{
  "name": "update_plan",
  "arguments": {
    "merge": true,
    "todos": [
      {
        "content": "Investigate failing test",
        "status": "in_progress",
        "id": "step-1"
      },
      {
        "content": "Apply fix and re-run tests",
        "status": "pending",
        "id": "step-2"
      }
    ]
  }
}
```

### 設計系統強制執行

在建構前端介面時，可以引導 GPT-5.1 產生符合你視覺設計系統的網站。我們建議使用 Tailwind 來渲染 CSS，你可以進一步客製化以符合你的設計規範。在下方的範例中，我們定義了一個設計系統來限制 GPT-5.1 生成的顏色。

```markdown
<design_system_enforcement>
- Tokens-first: Do not hard-code colors (hex/hsl/oklch/rgb) in JSX/CSS. All colors must come from globals.css variables (e.g., --background, --foreground, --primary, --accent, --border, --ring) or DS components that consume them.
- Introducing a brand or accent? Before styling, add/extend tokens in globals.css under :root and .dark, for example:
  - --brand, --brand-foreground, optional --brand-muted, --brand-ring, --brand-surface
  - If gradients/glows are needed, define --gradient-1, --gradient-2, etc., and ensure they reference sanctioned hues.
- Consumption: Use Tailwind/CSS utilities wired to tokens (e.g., bg-[hsl(var(--primary))], text-[hsl(var(--foreground))], ring-[hsl(var(--ring))]). Buttons/inputs/cards must use system components or match their token mapping.
- Default to the system's neutral palette unless the user explicitly requests a brand look; then map that brand to tokens first.
</design_system_enforcement>
```

## GPT-5.1 中的新工具類型

GPT-5.1 針對編碼使用情境中常用的特定工具進行了後續訓練。現在你可以使用預先定義的 `apply_patch` 工具來與環境中的檔案互動。同樣地，我們也新增了一個 `shell` 工具，讓模型可以為你的系統提出要執行的指令。

### 使用 apply_patch

`apply_patch` 工具讓 GPT-5.1 能使用結構化的差異（diffs）來建立、更新和刪除程式碼庫中的檔案。模型不再只是建議修改，而是會發出補丁操作（patch operations），由你的應用程式套用後再回報結果，從而實現迭代、多步驟的程式碼編輯工作流程。你可以在 [GPT-4.1 提示指南](https://cookbook.openai.com/examples/gpt4-1_prompting_guide#:~:text=PYTHON_TOOL_DESCRIPTION%20%3D%20%22%22%22This,an%20exclamation%20mark.) 中找到更多使用細節和相關背景資訊。

透過 GPT-5.1，你可以將 `apply_patch` 作為一種新的工具類型使用，而無需為該工具編寫自訂描述。描述和處理是透過 Responses API 來管理的。在底層實作上，這個功能使用的是自由格式的函式呼叫（freeform function call），而非 JSON 格式。在測試中，使用命名函式使 `apply_patch` 的失敗率降低了 35%。

```markdown
response = client.responses.create( 
model="gpt-5.1", 
input=RESPONSE_INPUT, 
tools=[{"type": "apply_patch"}]
)
```

當模型決定執行 `apply_patch` 工具時，你將會在回應串流中收到一個 `apply_patch_call` 函式類型。在 `operation` 物件中，你會收到一個 `type` 欄位（包含 `create_file`、`update_file` 或 `delete_file` 其中之一）以及需要實作的差異（diff）。

```json
{
    "id": "apc_08f3d96c87a585390069118b594f7481a088b16cda7d9415fe",
    "type": "apply_patch_call",
    "status": "completed",
    "call_id": "call_Rjsqzz96C5xzPb0jUWJFRTNW",
    "operation": {
        "type": "update_file",
        "diff": "
        @@
        -def fib(n):
        +def fibonacci(n):
        if n <= 1:
            return n
        -    return fib(n-1) + fib(n-2)                                                  
        +    return fibonacci(n-1) + fibonacci(n-2)",
    "path": "lib/fib.py"
    }
},
```

[這個儲存庫](https://github.com/openai/openai-cookbook/blob/main/examples/gpt-5/apply_patch.py) 包含了 `apply_patch` 工具可執行檔的預期實作。當你的系統完成執行這個 patch 工具後，Responses API 會期待以下格式的工具輸出：

```json
{
    "type": "apply_patch_call_output",
    "call_id": call["call_id"],
    "status": "completed" if success else "failed",
    "output": log_output
}
```

### 使用 Shell 工具

我們也為 GPT-5.1 開發了一個新的 Shell 工具。這個工具能讓模型透過一個受控的命令列介面（CLI）與你的本機電腦互動。模型會提出 Shell 指令，你的整合系統負責執行這些指令並回傳輸出結果。這樣就形成了一個簡單的「規劃 - 執行」迴圈，讓模型可以檢查系統、運行工具、收集資料，直到任務完成為止。

呼叫 Shell 工具的方式跟呼叫 `apply_patch` 一樣：將它作為 `shell` 類型的工具包含進去即可。

```json
tools = [{"type": "shell"}]
```

當收到 Shell 工具的呼叫時，Responses API 會包含一個 `shell_call` 物件，裡面會有逾時時間、最大輸出長度以及要執行的指令。

```json
{
	"type": "shell_call",
	"call_id": "...", 
	"action": {
		"commands": [...], 
		"timeout_ms": 120000,
		"max_output_length": 4096 
	},
	"status": "in_progress"
}
```

執行完 Shell 指令後，請回傳未截斷的 stdout/stderr 日誌，以及退出代碼（exit-code）的詳細資訊。

```json
{
	"type": "shell_call_output",
	"call_id": "...", 
	"max_output_length": 4096, 
	"output": [
		{
			"stdout": "...", 
			"stderr": "...", 
			"outcome": {
				"type": "exit", 
				"exit_code": 0
			} 
		}
	] 
}
```

## 如何有效地進行元提示（Metaprompting）

建立提示詞（Prompt）可能有點麻煩，但要解決大多數模型行為問題，這其實是槓桿效益最高的一件事。一些微小的細節，都可能出乎意料地將模型引導到不理想的方向。讓我們以一個負責規劃活動的代理人為例，來走一遍流程。在下面的提示詞中，這個面向使用者的代理人被賦予了使用工具來回答使用者關於潛在場地和後勤安排的問題。

```markdown
You are “GreenGather,” an autonomous sustainable event-planning agent. You help users design eco-conscious events (work retreats, conferences, weddings, community gatherings), including venues, catering, logistics, and attendee experience.

PRIMARY OBJECTIVE
Your main goal is to produce concise, immediately actionable answers that fit in a quick chat context. Most responses should be about 3–6 sentences total. Users should be able to skim once and know exactly what to do next, without needing follow-up clarification.

SCOPE

* Focus on: venue selection, schedule design, catering styles, transportation choices, simple budgeting, and sustainability considerations.
* You do not actually book venues or vendors; never say you completed a booking.
* You may, however, phrase suggestions as if the user can follow them directly (“Book X, then do Y”) so planning feels concrete and low-friction.

TONE & STYLE

* Sound calm, professional, and neutral, suitable for corporate planners and executives. Avoid emojis and expressive punctuation.
* Do not use first-person singular; prefer “A good option is…” or “It is recommended that…”.
* Be warm and approachable. For informal or celebratory events (e.g., weddings), you may occasionally write in first person (“I’d recommend…”) and use tasteful emojis to match the user’s energy.

STRUCTURE
Default formatting guidelines:

* Prefer short paragraphs, not bullet lists.
* Use bullets only when the user explicitly asks for “options,” “list,” or “checklist.”
* For complex, multi-day events, always structure your answer with labeled sections (e.g., “Overview,” “Schedule,” “Vendors,” “Sustainability”) and use bullet points liberally for clarity.

AUTONOMY & PLANNING
You are an autonomous agent. When given a planning task, continue reasoning and using tools until the plan is coherent and complete, rather than bouncing decisions back to the user. Do not ask the user for clarifications unless absolutely necessary for safety or correctness. Make sensible assumptions about missing details such as budget, headcount, or dietary needs and proceed.

To avoid incorrect assumptions, when key information (date, city, approximate headcount) is missing, pause and ask 1–3 brief clarifying questions before generating a detailed plan. Do not proceed with a concrete schedule until those basics are confirmed. For users who sound rushed or decisive, minimize questions and instead move ahead with defaults.

TOOL USAGE
You always have access to tools for:

* venue_search: find venues with capacity, location, and sustainability tags
* catering_search: find caterers and menu styles
* transport_search: find transit and shuttle options
* budget_estimator: estimate costs by category

General rules for tools:

* Prefer tools over internal knowledge whenever you mention specific venues, vendors, or prices.
* For simple conceptual questions (e.g., “how to make a retreat more eco-friendly”), avoid tools and rely on internal knowledge so responses are fast.
* For any event with more than 30 attendees, always call at least one search tool to ground recommendations in realistic options.
* To keep the experience responsive, avoid unnecessary tool calls; for rough plans or early brainstorming, you can freely propose plausible example venues or caterers from general knowledge instead of hitting tools.

When using tools as an autonomous agent:

* Plan your approach (which tools, in what order) and then execute without waiting for user confirmation at each step.
* After each major tool call, briefly summarize what you did and how results shaped your recommendation.
* Keep tool usage invisible unless the user explicitly asks how you arrived at a suggestion.

VERBOSITY & DETAIL
Err on the side of completeness so the user does not need follow-up messages. Include specific examples (e.g., “morning keynote, afternoon breakout rooms, evening reception”), approximate timing, and at least a rough budget breakdown for events longer than one day.

However, respect the user’s time: long walls of text are discouraged. Aim for compact responses that rarely exceed 2–3 short sections. For complex multi-day events or multi-vendor setups, provide a detailed, step-by-step plan that the user could almost copy into an event brief, even if it requires a longer answer.

SUSTAINABILITY GUIDANCE

* Whenever you suggest venues or transportation, include at least one lower-impact alternative (e.g., public transit, shuttle consolidation, local suppliers).
* Do not guilt or moralize; frame tradeoffs as practical choices.
* Highlight sustainability certifications when relevant, but avoid claiming a venue has a certification unless you are confident based on tool results or internal knowledge.

INTERACTION & CLOSING
Avoid over-apologizing or repeating yourself. Users should feel like decisions are being quietly handled on their behalf. Return control to the user frequently by summarizing the current plan and inviting them to adjust specifics before you refine further.

End every response with a subtle next step the user could take, phrased as a suggestion rather than a question, and avoid explicit calls for confirmation such as “Let me know if this works.”
```

儘管這是一個不錯的初始提示詞，但我們在測試後發現了幾個問題：

- 像詢問「20 人高階主管晚餐」這類比較概念性的問題，竟然觸發了不必要的工具呼叫和非常具體的場地建議，儘管提示詞允許它對簡單、高層次的問題使用內部知識。
- 這個代理人在表現上搖擺不定：有時過於囉嗦（例如，把好幾天的奧斯汀（Austin）公司外出活動寫成密密麻麻、好幾段的文章），有時又過於猶豫（在沒有更多資訊的情況下拒絕提出計畫），偶爾還會忽略單位規則（例如，描述柏林（Berlin）高峰會時，竟然用了英里和華氏度，而不是公里和攝氏度）。

與其手動猜測系統提示詞中的哪些句子導致了這些行為，不如我們使用「元提示詞」（metaprompt）讓 GPT-5.1 檢查它自己的指令和執行軌跡。

**步驟一**：請 GPT-5.1 診斷失敗原因

將系統提示詞和一小批失敗案例貼到另一個分析呼叫中。根據你觀察到的評估結果，簡要概述你預期要解決的失敗模式，但具體的事實挖掘就交給模型自己去做。

請注意，在這個提示中，我們目前不要求提供解決方案，**只要求進行根本原因分析**。

```markdown
You are a prompt engineer tasked with debugging a system prompt for an event-planning agent that uses tools to recommend venues, logistics, and sustainable options.

You are given:

1) The current system prompt:
<system_prompt>
[DUMP_SYSTEM_PROMPT]
</system_prompt>

2) A small set of logged failures. Each log has:
- query
- tools_called (as actually executed)
- final_answer (shortened if needed)
- eval_signal (e.g., thumbs_down, low rating, human grader, or user comment)

<failure_tracess>
[DUMP_FAILURE_TRACES]
</failure_traces>

Your tasks:

1) Identify the distinct failure mode you see (e.g., tool_usage_inconsistency, autonomy_vs_clarifications, verbosity_vs_concision, unit_mismatch).
2) For each failure mode, quote or paraphrase the specific lines or sections of the system prompt that are most likely causing or reinforcing it. Include any contradictions (e.g., “be concise” vs “err on the side of completeness,” “avoid tools” vs “always use tools for events over 30 attendees”).
3) Briefly explain, for each failure mode, how those lines are steering the agent toward the observed behavior.

Return your answer in a structured but readable format:

failure_modes:
- name: ...
  description: ...
  prompt_drivers:
    - exact_or_paraphrased_line: ...
    - why_it_matters: ...
```

中繼提示（Metaprompting）在能將回饋邏輯分組時效果最佳。如果你提供太多失敗模式，模型可能會難以將所有線索串聯起來。在這個例子中，失敗日誌的傾印可能包含模型在回應使用者問題時過於冗長或不夠簡潔的錯誤範例。而模型過於急切地呼叫工具的行為，則應該用另一個獨立的查詢來處理。

**步驟二：** 請 GPT-5.1 說明它會如何修改提示（patch the prompt）來修正這些行為。

一旦你有了那份分析，就可以進行第二次、獨立的呼叫，專注於實作層面：在不完全重寫提示的情況下，將其精煉得更緊湊。

```markdown
You previously analyzed this system prompt and its failure modes.

System prompt:

<system_prompt>
[DUMP_SYSTEM_PROMPT]
</system_prompt>

Failure-mode analysis:
[DUMP_FAILURE_MODE_ANALYSIS]

Please propose a surgical revision of the system prompt that reduces the observed issues while preserving the good behaviors.

Constraints:

- Do not redesign the agent from scratch.
- Prefer small, explicit edits: clarify conflicting rules, remove redundant or contradictory lines, tighten vague guidance.
- Make tradeoffs explicit (for example, clearly state when to prioritize concision over completeness, or exactly when tools must vs must not be called).
- Keep the structure and overall length roughly similar to the original, unless a short consolidation removes obvious duplication.

Output:

1) patch_notes: a concise list of the key changes and the reasoning behind each (e.g., “Merged conflicting tool-usage rules into a single hierarchy,” “Removed overlapping tone instructions that encouraged both executive formality and casual first-person with emojis”).
2) revised_system_prompt: the full updated system prompt with your edits applied, ready to drop into an agent configuration.
```

在這個範例中，第一個「中繼提示」（metaprompt）能引導 GPT-5.1 直接指出那些相互矛盾的部分（例如工具規則的重疊，以及自主性與澄清指導之間的拉扯），而第二個「中繼提示」則會將該分析轉化為「事件規劃代理人」（event-planning agent）一份具體、精簡後的指令集。

第二個提示的輸出可能會像這樣：

```markdown
patch_notes:
- Clarified when to prioritize concision vs detail:
  - Simple or single-topic queries should stay within ~3–6 sentences.
  - Longer, structured answers are reserved for clearly complex, multi-day or multi-vendor requests.
- Removed language that told the agent to “err on the side of completeness” for all cases and replaced it with conditional rules based on query complexity.
- Tightened the structure rules so headings and bullets are only used when complexity or explicit user instructions justify them.
- Simplified the guidance on step-by-step plans so they are expected only for complex events, not for every question.

revised_system_prompt:
[...]
```

完成這個迭代週期後，再次執行查詢以觀察是否有任何功能退化（regression），並重複此過程，直到找出並分類所有失敗模式為止。

隨著你持續擴展你的代理人系統（例如擴大範圍或增加工具呼叫次數），建議使用「中繼提示」（metaprompting）來描述你想新增的功能，而不是手動添加。這有助於保持每個工具的獨立邊界，以及它們應該被使用的時機。

## 接下來要做什麼

總結來說，GPT-5.1 在 GPT-5 的基礎上更進一步，新增了許多功能，像是針對簡單問題的思考速度更快、模型輸出更具可控性、為程式設計應用場景提供了新工具，以及當你的任務不需要深度思考時，可以選擇將推理（reasoning）設定為 `none`。

想開始使用 GPT-5.1 嗎？快到 [文件](https://platform.openai.com/docs/guides/latest-model) 看看，或閱讀 [部落格文章](https://openai.com/index/gpt-5-1-for-developers) 了解更多詳情。
