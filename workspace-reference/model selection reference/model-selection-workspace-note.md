# Model Selection for Workspace Configuration

This note records the three OpenRouter models selected for regular development work in this workspace.

## Selected models

- **Xiaomi MiMo-V2.5-Pro** — primary model for regular work.[web:36]
- **DeepSeek V4 Pro** — secondary model for more complex work and confidence checking.[web:37][web:38]
- **Qwen3 VL 235B A22B Instruct** — image model for uploading and interpreting screenshots.[web:60][web:63]

## Why this set was chosen

This is a practical three-model setup rather than a "one model for everything" approach. It keeps day-to-day work simple, gives a stronger fallback when extra confidence is needed, and adds screenshot interpretation without forcing that job onto the coding models.[web:36][web:37][web:60]

MiMo-V2.5-Pro is the regular model because it combines large context, strong general coding ability, and low OpenRouter pricing, which makes it suitable for continuous use across planning, coding, refactoring, and documentation work.[web:36][web:38][web:47]

DeepSeek V4 Pro is the escalation model because it is well suited to harder reasoning and tougher coding problems, while its OpenRouter pricing is currently very close to MiMo.[web:37][web:38][web:40] That makes it a sensible second opinion model when the task is difficult or when extra confidence is needed before accepting output.[web:31][web:38]

Qwen3 VL 235B A22B Instruct is the image interpretation model because it supports vision input on OpenRouter and is priced low enough to use routinely for screenshot analysis.[web:60][web:63] The Instruct variant is the right fit for this role because it is tuned for direct instruction-following and practical visual interpretation rather than heavier reasoning overhead.[web:68][web:71][web:73]

## Role of each model

### MiMo-V2.5-Pro

**Use for:** normal development sessions, drafting, refactoring, file-by-file implementation, and general workspace assistance.[web:36][web:38]

**Why it fits:**
- Strong general coding model.[web:36][web:38]
- Very large context window for bigger working sessions.[web:36][web:38]
- OpenRouter pricing is low enough for regular use.[web:36][web:47]

**Where it is weaker:**
- Not the preferred model for screenshot interpretation.[web:60][page:1]
- Not the first choice when the task needs the strongest possible reasoning pass.[web:31][web:38]

### DeepSeek V4 Pro

**Use for:** complex reasoning, harder coding tasks, difficult refactors, and validation passes where confidence matters.[web:31][web:37][web:38]

**Why it fits:**
- Strong coding and reasoning performance.[web:31][web:38]
- Good option for checking or challenging work produced elsewhere.[web:31][web:37]
- Current OpenRouter pricing is close to MiMo, so it is realistic to keep as an active backup rather than a rare emergency option.[web:37][web:38][web:40]

**Where it is weaker:**
- Can use more reasoning tokens on hard tasks, so the practical cost per task can still rise even when token pricing is similar.[web:31][web:37]
- Not the right tool for screenshot upload and visual interpretation.[page:1]

### Qwen3 VL 235B A22B Instruct

**Use for:** uploading screenshots, interpreting UI layouts, extracting visible issues, and converting visual input into text that can be handed to a coding model.[web:60][web:63]

**Why it fits:**
- Supports image input on OpenRouter.[web:60][web:63]
- Economical enough for routine screenshot work.[web:60]
- The Instruct variant is better suited than the Thinking variant for direct UI interpretation tasks.[web:68][web:71]

**Where it is weaker:**
- Not the main coding model for regular implementation work.[web:60][web:68]
- Best used as the visual intake step, then hand the interpreted result to MiMo or DeepSeek for coding or review.[web:60][web:68]

## Working rule

- Start with **MiMo-V2.5-Pro** for normal work.[web:36]
- Escalate to **DeepSeek V4 Pro** for harder work or when confidence needs to be checked.[web:37][web:38]
- Use **Qwen3 VL 235B A22B Instruct** when a screenshot or UI image must be interpreted before coding begins.[web:60][web:68]

This keeps the workflow simple: one regular model, one stronger verification model, and one visual interpretation model.[web:36][web:37][web:60]
