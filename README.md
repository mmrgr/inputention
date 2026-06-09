# Inputention

> Intent autocomplete for AI conversations. Predict likely next prompts, clarify ambiguous input, and turn numbered replies into complete requests.

Inputention is a Codex skill that helps users communicate faster with AI assistants. It turns "what I might type next" or "I only typed a vague keyword" into ranked, selectable, natural-language options. Users can then reply with a number, optionally add a few details, and the assistant reconstructs the full intent before answering.

It is especially useful for conversational agents, coding assistants, support workflows, research assistants, and any AI surface where users often type fragments such as `error`, `VPN`, `won't open`, `translate this`, or `fix it`.

## Highlights

- **One-character activation**: start a message with `?` or `？` to invoke Inputention directly.
- **Next-prompt prediction**: generate 9 likely follow-up prompts from the current conversation.
- **Ambiguous-input clarification**: convert vague, short, or keyword-like input into 2-9 actionable intent candidates.
- **Numbered replies**: reply with `2`, `2; value A; value B`, `I choose 2`, or `use #2`.
- **Placeholder filling**: fill `[error message]`, `[target style]`, `【报错内容】`, and other placeholders left to right.
- **Multilingual by design**: Chinese/CJK contexts use `【...】`; English and most other languages use `[...]`.
- **Low-friction UX**: the assistant answers directly when intent is clear, and asks through options only when guessing would be risky.

## Why Inputention?

Most AI conversations lose momentum in two places:

1. The user knows the direction but does not want to type a full prompt.
2. The user types a fragment, and the assistant guesses the wrong task.

Inputention solves both by making intent selection lightweight:

```text
User: ?报错了

Assistant:
我还不能完全确定你的意思。你可能想表达的是：
1. 我在运行【软件/代码/命令】时出现报错：【报错内容】，请帮我分析原因。
2. 我使用的是【系统/环境】，在执行【具体操作】时出现【报错现象】，请帮我排查。
3. 请根据这个报错【报错内容】告诉我最可能的原因和解决方法。
4. 我不懂这个报错是什么意思，请用简单的话解释【报错内容】。

User: 1；Python 脚本；ModuleNotFoundError: No module named pandas

Assistant:
我将你的输入理解为：我在运行 Python 脚本时出现报错：ModuleNotFoundError: No module named pandas，请帮我分析原因。
...
```

## Core Modes

### 1. Next-Prompt Prediction

Use this when the user explicitly asks for likely next inputs.

Example triggers:

```text
Predict my most likely next inputs.
Give me 9 possible follow-up prompts.
请预测我接下来最可能输入的 9 句话。
?
？
```

Behavior:

- Reads the current conversation.
- Infers the user's stage, such as understanding, design, revision, correction, execution, comparison, extension, or validation.
- Generates internal candidates.
- Selects and ranks the best 9.
- Outputs complete prompts the user can copy or select by number.

### 2. Ambiguous-Input Clarification

Use this when answering directly would likely answer the wrong question.

Example triggers:

```text
?VPN
?won't open
?translate this
?报错了
?这个不对
```

Behavior:

- Preserves the user's original keyword when possible.
- Generates the smallest useful set of candidates, usually 2-5 and at most 9.
- Uses placeholders instead of inventing missing facts.
- Avoids generic clarification like "What do you mean?" when useful options can be offered.

### 3. Numbered Intent Expansion

Use this after Inputention has produced a numbered list.

Supported replies include:

```text
2
2; A; B
2；A；B
2: A
第2个
我选2
option 2
I choose 2
use #2
go with the second one
```

Behavior:

- Selects the matching option from the latest Inputention list.
- Extracts placeholders from left to right.
- Fills values in order.
- Appends extra values as additional requirements.
- Asks for missing values only when they are necessary.
- Answers the reconstructed request.

## Quick Start

Place this repository as a Codex skill folder, or copy `SKILL.md` into a Codex skill directory.

Current project path:

```text
D:\my   github\inputention
```

Typical Codex skill layout:

```text
inputention/
├── agents/
│   └── openai.yaml
├── SKILL.md
├── README.md
└── README_CN.md
```

The Codex skill name is `inputention`, so it can also be invoked explicitly as `$inputention` in environments that support named skill invocation.

The skill's trigger logic lives in the frontmatter `description`, `agents/openai.yaml`, and the mode-selection rules inside `SKILL.md`.

## Explicit `?` Trigger

Inputention is designed to be easy to invoke.

If the **very first character** of the user's message is `?` or `？`, the assistant should use this skill.

You can also invoke it by name with `$inputention` when your Codex environment supports explicit skill invocation.

Examples:

```text
?报错了
？打不开
?won't open
?email
?predict my next inputs
?
```

The leading question mark is treated as an invocation prefix, not as part of the actual request.

Rules:

- `?报错了` runs ambiguous-input clarification on `报错了`.
- `?predict my next inputs` runs next-prompt prediction.
- `?` by itself predicts likely next inputs from the current conversation.
- If the text after `?` is already clear, the prefix still forces Inputention-style options instead of bypassing the skill.

## Output Examples

### English Ambiguous Input

```text
User: ?won't open

Assistant:
I cannot fully determine your intent yet. You may mean:
1. I cannot open [website/app/file/device], please help me troubleshoot the cause.
2. When I try to open [target item] on [device or operating system], I see [specific symptom], please help me diagnose it.
3. [target item] used to open but no longer opens, please list the likely causes and steps to fix it.
4. When I open [target item], it shows [error message], please help me understand and fix it.

You can reply with just a number, or fill placeholders like this:
number; value for placeholder 1; value for placeholder 2; value for placeholder 3
```

### English Numbered Expansion

```text
User: 1; a PDF file; it says the file is damaged

Assistant:
I understand your intended request as: I cannot open a PDF file, please help me troubleshoot the cause. Additional requirement: it says the file is damaged.
...
```

### Chinese Next-Prompt Prediction

```text
User: ?

Assistant:
我预测你接下来最可能输入的是：
1. 请把这个方案整理成可以直接复制到 SKILL.md 的版本。
2. 请给我一个用户选择序号并补全占位符的完整示例。
3. 请说明这个 skill 在英文场景下应该如何输出。
...
9. 请帮我检查这套规则是否存在容易误触发或漏触发的问题。
```

## Placeholder Conventions

Use placeholders only for missing information that the assistant should not invent.

Recommended Chinese placeholders:

```text
【报错内容】
【软件名称】
【电脑系统】
【需要翻译的文本】
【需要润色的文本】
【目标风格】
【具体现象】
【已尝试的方法】
【最终产物类型】
```

Recommended English placeholders:

```text
[error message]
[software name]
[operating system]
[text to translate]
[text to polish]
[target style]
[specific symptom]
[steps already tried]
[final artifact type]
```

Avoid vague placeholders:

```text
【内容】
【东西】
【情况】
[content]
[thing]
[stuff]
```

## Design Principles

Inputention follows a simple decision rule:

- If the user is clear, answer directly.
- If the user explicitly starts with `?` or `？`, use Inputention.
- If the user is unclear and a direct answer would likely drift, offer ranked intent candidates.
- If the user selects a candidate, reconstruct the request and complete it.

Candidates should be:

- Complete natural-language requests.
- Ranked by likelihood.
- Distinct from one another.
- Grounded in the current conversation.
- Free of unsupported private assumptions.
- Ready to execute after selection.

## Safety Notes

Inputention must not turn guesses into facts.

For example, if a user says:

```text
my boss didn't reply
```

Do not generate:

```text
My boss is deliberately ignoring me. What should I do?
```

Prefer:

```text
My boss has not replied to [message/request], please help me decide how to follow up.
```

For medical, legal, financial, security, crisis, or other high-stakes domains, Inputention should clarify missing facts and avoid definitive conclusions from vague input.

## Validation

Validate the skill after editing:

```powershell
$env:PYTHONUTF8='1'
python C:\Users\mmrgr\.codex\skills\.system\skill-creator\scripts\quick_validate.py "D:\my   github\inputention"
```

If the active Python environment does not include `PyYAML`, use another environment or install it temporarily for validation.

## Roadmap Ideas

- Add packaged install metadata for one-click Codex installation.
- Add a small test corpus for trigger behavior.
- Add locale-specific examples for Japanese, Spanish, and other languages.
- Add regression tests for numbered expansion and placeholder filling.

## License

No license has been specified yet. Add one before publishing this repository publicly.
