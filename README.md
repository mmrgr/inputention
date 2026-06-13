# Inputention

> Intent autocomplete for AI conversations. Predict likely next prompts, clarify ambiguous input, and turn numbered replies into complete requests.

Inputention is a Codex skill that helps people communicate with AI assistants more smoothly, especially when typing is slow or when they feel pressure to phrase everything "perfectly" so the AI understands them. It turns "what I might type next" or "I only typed a vague keyword" into ranked, selectable, natural-language options. Users can then reply with a number, optionally add a few details, and the assistant reconstructs the full intent before answering.

It is especially useful for conversational agents, coding assistants, support workflows, research assistants, and any AI surface where users often type fragments such as `error`, `VPN`, `won't open`, `translate this`, or `fix it`.

## Highlights

- **One-character activation**: start a message with `?` or `？` to invoke Inputention directly.
- **Continuous predictions**: once activated, every clear answer is followed by predicted next inputs until the user opts out.
- **Default prediction count of 5**: use a lighter default than 9 to avoid interrupting the conversation.
- **Configurable count**: send only `??3` or `？？3` to make later prediction blocks contain 3 items.
- **Next-prompt prediction**: generate likely follow-up prompts from the current conversation.
- **Intent clarification before answering**: convert vague, short, or keyword-like input into 3-9 actionable intent candidates, then wait for the user's numbered choice before answering.
- **Numbered replies**: reply with `2`, `2; value A; value B`, `I choose 2`, or `use #2`.
- **Fillable slots**: fill `[error message]`, `[target style]`, `【报错内容】`, and other named slots left to right.
- **Less wording pressure**: users can choose or lightly edit a suggested intent instead of worrying about the perfect prompt.
- **Multilingual by design**: Chinese/CJK contexts use `【...】`; English and most other languages use `[...]`.
- **Low-friction UX**: the assistant answers directly when intent is clear, and asks through options only when guessing would be risky.

## Why Inputention?

Most AI conversations lose momentum in three places:

1. The user knows the direction but does not want to type a full prompt.
2. The user types slowly and needs a lower-friction way to continue.
3. The user worries the AI will misunderstand them, so they spend extra effort polishing the wording before sending anything.

Inputention solves these frictions by making intent selection lightweight:

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

Use this when the user explicitly asks for likely next inputs, or after any clear answer while Inputention is active.

Example triggers:

```text
Predict my most likely next inputs.
Give me possible follow-up prompts.
请预测我接下来最可能输入的话。
?
？
```

Behavior:

- Reads the current conversation.
- Infers the user's stage, such as understanding, design, revision, correction, execution, comparison, extension, or validation.
- Generates internal candidates.
- Selects and ranks the best items up to the current prediction count.
- Outputs complete prompts the user can copy or select by number.
- Outputs exactly the current prediction count. The default is 5.
- Must include at least `min(3, current prediction count)` open fillable templates with specific named slots whenever user-specific details could matter.

### Prediction Count

The prediction count controls only the next-input prediction block after clear answers or explicit prediction requests. It does not affect intent clarification, which still uses 3-9 candidates.

Rules:

- Default prediction count: 5.
- Valid range: 1-9.
- `??3` sets later prediction blocks to 3 items.
- `？？3` also sets later prediction blocks to 3 items.
- `?? 3` and `？？ 3` are accepted.
- The setting message only updates the count and gets a brief acknowledgement.
- The setting message does not trigger answering, clarification, numbered completion, or a prediction block.

### 2. Intent Clarification

Use this when the user input is not a complete, fully determined request. In this mode, Inputention should not answer yet; it should offer 3-9 likely intents and wait for the next turn.

Example triggers:

```text
?VPN
?won't open
?translate this
?报错了
?这个不对
?南瓜 空气炸锅 蒙赤
?1456238359@qq.com
```

Behavior:

- Preserves the user's original keyword when possible.
- Generates the smallest useful set of candidates, at least 3 and at most 9.
- Uses named fillable slots instead of inventing missing facts.
- Avoids generic clarification like "What do you mean?" when useful options can be offered.
- If the user ignores the options and sends a new message, treats the new message as the current request instead of insisting on the old list.
- Does not append a prediction block to a clarification list.

### 3. Numbered Intent Completion

Use this after Inputention has produced a numbered list.

Supported replies include:

```text
2
2; A; B
2；A；B
2: A
4；贝贝南瓜
8；200字
5；求职；正式
第2个
我选2
option 2
I choose 2
use #2
go with the second one
```

Behavior:

- Selects the matching option from the latest Inputention list.
- Extracts fillable slots from left to right.
- Fills values in order.
- Appends extra values as additional requirements.
- Asks for missing values only when they are necessary.
- Answers the reconstructed request.
- If Inputention is active, appends the current prediction count after the answer.

This is the actual completion feature: the assistant must reconstruct the selected option and answer it. It should not treat `4；贝贝南瓜` or `8；200字` as a fresh unrelated prompt, and it should not respond with predictions only.

## Quick Start

Place this repository as a Codex skill folder, or copy `SKILL.md` into a Codex skill directory.

Current project path:

```text
D:\my   github\inputention
```

Typical Codex skill layout:

```text
inputention/
├── tests/
│   ├── checklist.md
│   └── cases.yaml
├── SKILL.md
├── README.md
└── README_CN.md
```

The Codex skill name is `inputention`, so it can also be invoked explicitly as `$inputention` in environments that support named skill invocation.

The skill's trigger logic lives in the frontmatter `description` and the mode-selection rules inside `SKILL.md`. This project intentionally does not include `agents/openai.yaml`.

## Explicit `?` Trigger

Inputention is designed to be easy to invoke.

If the **very first character** of the user's message is `?` or `？`, the assistant should use this skill.

You can also invoke it by name with `$inputention` when your Codex environment supports explicit skill invocation.

After `?` or `？` activates Inputention, the behavior continues on later turns until the user explicitly says to stop, such as "stop predicting", "no more predictions", or "不要再预测".

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
- If the text after `?` is already clear, the assistant answers it first and then appends the current prediction count.
- If the user does not choose from a previous prediction or clarification list, the assistant treats the next message as a new request and continues Inputention behavior.

## Changing Prediction Count

Send a standalone double-question-mark command:

```text
??3
```

or:

```text
？？3
```

This changes later prediction blocks to 3 items. Use any integer from 1 to 9.

This command is intentionally narrow: `?3`, `???3`, `？？三`, and `？？3 please continue` are not prediction-count settings.
- If the user starts with a valid option number and provides details, such as `4；贝贝南瓜`, the assistant completes the latest matching option before doing anything else.

## Output Examples

### English Intent Clarification

```text
User: ?won't open

Assistant:
I cannot fully determine your intent yet. You may mean:
1. I cannot open [website/app/file/device], please help me troubleshoot the cause.
2. When I try to open [target item] on [device or operating system], I see [specific symptom], please help me diagnose it.
3. [target item] used to open but no longer opens, please list the likely causes and steps to fix it.
4. When I open [target item], it shows [error message], please help me understand and fix it.

You can reply with just a number. If an item contains [details to fill in], use:
number; value for the first blank; value for the second blank; value for the third blank
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
2. 请给我一个用户选择序号并补全待补信息的完整示例。
3. 请说明这个 skill 在英文场景下应该如何输出。
4. 请把预测数量改成【目标数量】条，并说明这样更适合什么场景。
5. 请帮我检查这套规则是否存在容易误触发或漏触发的问题。
```

## Fillable Slot Conventions

Use fillable slots only for missing information that the assistant should not invent.

Recommended Chinese slots:

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

Recommended English slots:

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

Avoid vague slots:

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

- If Inputention is not active and the user is clear, answer directly.
- If the user explicitly starts with `?` or `？`, activate continuous Inputention behavior.
- If Inputention is active and the user is clear, answer directly and append the current prediction count of next-input predictions.
- If the user is unclear or not fully determined, offer 3-9 ranked intent candidates and wait for the user to choose or complete one.
- If the user selects a candidate, reconstruct the request and complete it.
- If the user ignores a candidate list and sends a new message, handle the new message instead of repeatedly asking them to choose.

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

## Tests

Inputention includes two lightweight test artifacts:

- `tests/checklist.md`: a human QA checklist for trigger behavior, count changes, intent clarification, numbered completion, and opt-out.
- `tests/cases.yaml`: machine-readable scenarios that can be used by a future test runner or reviewed manually.

## Roadmap Ideas

- Add packaged install metadata for one-click Codex installation.
- Add a small test corpus for trigger behavior.
- Add locale-specific examples for Japanese, Spanish, and other languages.
- Add regression tests for numbered expansion and placeholder filling.

## License

No license has been specified yet. Add one before publishing this repository publicly.
