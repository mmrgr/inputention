# Predict Next Inputs

`predict-next-inputs` is a Codex skill for predicting likely next user inputs, clarifying vague requests, and expanding numbered replies into complete actionable intent.

It is designed to reduce user typing, prevent answers from drifting away from the user's real goal, and keep multi-turn conversations moving smoothly.

## What This Skill Does

This skill has three core capabilities:

1. **Predict next inputs**

   When the user asks what they are likely to ask next, the skill generates 9 likely next inputs based on the current conversation.

2. **Clarify vague input**

   When the user sends a short, ambiguous, keyword-like, or incomplete request, the skill generates 2-9 likely full-intent candidates instead of guessing and answering the wrong question.

3. **Expand numbered replies**

   When the user replies with a number, optionally followed by details, the skill fills placeholders in the chosen option, reconstructs the user's full intent, and then answers that reconstructed request.

## Typical Use Cases

Use this skill when a user wants to reduce typing or when their input is too vague to answer reliably.

Examples:

- The user asks: "Predict what I may ask next."
- The user asks: "Give me 9 possible next inputs."
- The user says: "VPN"
- The user says: "报错了"
- The user says: "won't open"
- The user says: "translate this" but provides no text.
- The user replies: `2; Python script; ModuleNotFoundError: No module named pandas`
- The user replies: `I choose 3`

## Installation Location

The skill is installed here:

```text
C:\Users\mmrgr\.codex\skills\predict-next-inputs
```

Main files:

```text
predict-next-inputs/
├── SKILL.md
├── README.md
└── agents/
    └── openai.yaml
```

`SKILL.md` contains the operational rules that Codex loads when the skill is triggered.

`agents/openai.yaml` contains UI-facing metadata such as display name, short description, default prompt, and implicit invocation policy.

## Core Concepts

### Next-Input Prediction

This mode is used when the user explicitly asks the assistant to predict possible follow-up inputs.

The skill should:

- Inspect the current conversation.
- Infer the user's current stage, such as understanding, design, revision, correction, execution, comparison, extension, or verification.
- Generate at least 12 internal candidates.
- Select the best 9.
- Sort them from most likely to less likely.
- Output exactly 9 items unless the user asks for a different number.

This mode is proactive and task-continuation oriented.

### Vague-Input Clarification

This mode is used when directly answering would likely answer the wrong question.

The skill should offer likely complete meanings rather than ask a generic question like "What do you mean?"

It is triggered by inputs such as:

- Keywords: `VPN`, `resume`, `paper`, `email`, `translate`, `optimize`
- Vague actions: "fix it", "help me look", "write it", "what should I do"
- Missing task object: "translate this" without text
- Unclear reference: "this is wrong", "it broke", "that issue"
- Multi-meaning phrases: "won't open", "not working", "can't connect"
- High-risk or high-cost domains with missing facts

Unlike prediction mode, clarification mode does not have to output 9 items. It should output the smallest useful set, usually 2-5, and at most 9.

### Numbered Expansion

This mode is used after the assistant has produced a numbered prediction or clarification list.

The user may reply with:

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

The skill then:

1. Finds the latest numbered option list.
2. Retrieves the selected item.
3. Extracts placeholders from left to right.
4. Fills placeholders using the user's supplied values.
5. Appends extra values as additional requirements.
6. Asks for missing required values only when they cannot be inferred and are needed.
7. Answers the reconstructed full request.

## Language Support

The skill is multilingual.

It should use the user's primary language for:

- Headings
- Candidate options
- Placeholder names
- Confirmation text
- Final answers

For Chinese, Japanese, or mixed CJK contexts, placeholders should use Chinese corner brackets:

```text
【报错内容】
【需要翻译的文本】
【目标风格】
```

For English and most other languages, placeholders should use square brackets:

```text
[error message]
[text to translate]
[target style]
```

Expansion mode must recognize both placeholder styles.

## Trigger Rules

### Trigger Prediction Mode

Use prediction mode when the user asks for:

- likely next inputs
- possible follow-up questions
- 9 next questions
- next prompt suggestions
- reduced typing via numbered choices

Example:

```text
请你预测我接下来最可能问的 9 个问题。
```

Example:

```text
Predict my most likely next inputs.
```

### Trigger Clarification Mode

Use clarification mode when the user input has high uncertainty.

High uncertainty means:

- The object is missing.
- The task type is unclear.
- Several interpretations are plausible.
- Direct answering would likely go down the wrong path.
- The answer depends on missing high-impact facts.

Example:

```text
打不开
```

Example:

```text
won't open
```

### Do Not Trigger Clarification Mode

Do not clarify when the request is already clear enough.

Example:

```text
帮我写一封向老师请病假的邮件。
```

The assistant should write the email directly.

Example:

```text
Translate this into English: 我今天身体不舒服，想请一天假。
```

The assistant should translate directly.

If the request has a small gap but a useful answer is still possible, make a reasonable assumption and proceed.

## Output Formats

### Chinese Prediction Output

```text
我预测你接下来最可能输入的是：
1. ...
2. ...
3. ...
4. ...
5. ...
6. ...
7. ...
8. ...
9. ...

你可以直接回复序号，也可以用这种格式补全占位符：
序号；占位符1的内容；占位符2的内容；占位符3的内容
```

### English Prediction Output

```text
I predict your most likely next inputs are:
1. ...
2. ...
3. ...
4. ...
5. ...
6. ...
7. ...
8. ...
9. ...

You can reply with just a number, or fill placeholders like this:
number; value for placeholder 1; value for placeholder 2; value for placeholder 3
```

### Chinese Clarification Output

```text
我还不能完全确定你的意思。你可能想表达的是：
1. ...
2. ...
3. ...

你可以直接回复序号；如果选项里有占位符，也可以按这个格式补全：
序号；占位符1的内容；占位符2的内容；占位符3的内容
```

### English Clarification Output

```text
I cannot fully determine your intent yet. You may mean:
1. ...
2. ...
3. ...

You can reply with just a number, or fill placeholders like this:
number; value for placeholder 1; value for placeholder 2; value for placeholder 3
```

## Placeholder Rules

Placeholders should be specific, fillable, and natural in the current language.

Good Chinese placeholders:

```text
【报错内容】
【软件名称】
【电脑系统】
【需要翻译的文本】
【需要润色的文本】
【目标风格】
【具体现象】
【已尝试的方法】
```

Good English placeholders:

```text
[error message]
[software name]
[operating system]
[text to translate]
[text to polish]
[target style]
[specific symptom]
[steps already tried]
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

Most candidates should use 1-4 placeholders. Too many placeholders increase user effort and make the option harder to choose.

## Expansion Examples

### Chinese Example

Assistant option:

```text
1. 我在运行【软件/代码/命令】时出现报错：【报错内容】，请帮我分析原因。
```

User reply:

```text
1；Python 脚本；ModuleNotFoundError: No module named pandas
```

Reconstructed intent:

```text
我在运行 Python 脚本时出现报错：ModuleNotFoundError: No module named pandas，请帮我分析原因。
```

The assistant should then explain that `pandas` is missing and provide installation/debugging steps.

### English Example

Assistant option:

```text
1. I cannot open [website/app/file/device], please help me troubleshoot the cause.
```

User reply:

```text
1; a PDF file; it says the file is damaged
```

Reconstructed intent:

```text
I cannot open a PDF file, please help me troubleshoot the cause. Additional requirement: it says the file is damaged.
```

The assistant should then give PDF repair and verification steps.

## Candidate Quality Standards

Every candidate should:

- Be a complete sentence.
- Sound like something a real user might type.
- Preserve the user's original keyword when useful.
- Be directly actionable after selection.
- Avoid unsupported facts or private assumptions.
- Be distinct from the other candidates.
- Be sorted by likelihood.
- Use precise placeholders when information is missing.

Candidates should not:

- Be labels such as "translation request" or "debugging".
- Repeat the same intent with tiny wording changes.
- Invent sensitive background details.
- Turn guesses into facts.
- Force 9 items in vague-input clarification mode when fewer options are better.

## Safety and Accuracy Rules

The skill must avoid overconfident interpretation.

For example, if the user says:

```text
my boss didn't reply
```

Do not create a candidate like:

```text
My boss is deliberately ignoring me, what should I do?
```

Use a neutral candidate:

```text
My boss has not replied to [message/request], please help me decide how to follow up.
```

For medical, legal, financial, security, crisis, or other high-stakes topics, the skill should clarify missing facts when necessary and avoid definitive conclusions from vague input.

If the reconstructed request involves facts that may have changed recently, the assistant should follow the active system instructions for higher-accuracy work, including source checking or web browsing when required.

## Maintenance Notes

When updating this skill:

1. Keep `SKILL.md` concise enough to load efficiently.
2. Preserve all three modes unless intentionally redesigning the skill:
   - next-input prediction
   - vague-input clarification
   - numbered expansion
3. Keep multilingual behavior explicit.
4. Add examples only when they teach a new edge case.
5. Avoid adding large fixed template libraries that make the skill mechanical.
6. Validate the skill after edits with the skill validation script.

Recommended validation command:

```powershell
$env:PYTHONUTF8='1'
python C:\Users\mmrgr\.codex\skills\.system\skill-creator\scripts\quick_validate.py C:\Users\mmrgr\.codex\skills\predict-next-inputs
```

If the default Python environment lacks `PyYAML`, use an environment that includes it or install it temporarily for validation.

## Design Principle

The skill should not replace the user's judgment. It should make likely intent easier to express.

When the user's intent is clear, answer directly.

When the user's intent is unclear but recoverable, offer high-quality numbered candidates.

When the user selects a candidate, reconstruct the request and complete the task.
