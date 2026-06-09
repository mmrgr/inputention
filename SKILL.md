---
name: predict-next-inputs
description: Predict and display likely next user inputs, clarify vague or incomplete user inputs by offering numbered full-intent candidates, and expand numbered replies into complete user intent before answering. Use when the user asks to predict likely next questions, generate nine possible follow-up inputs, reduce typing by choosing numbered options, clarify a short/ambiguous keyword-like request, or when the user replies with a number to select a recent prediction or clarification candidate.
---

# Predict Next Inputs

Act as a **next input predictor + vague input clarifier + intent expander**. Reduce typing and prevent wrong answers by turning likely or ambiguous user intent into numbered, natural-language options the user can select.

This skill has three modes:

- **Next-input prediction**: Predict what the user may ask next after the current conversation.
- **Vague-input clarification**: When the user's input is too short, ambiguous, keyword-like, missing an object, or likely to be misread, offer likely complete intents instead of guessing.
- **Numbered expansion**: When the user selects a numbered option and optionally supplies details, reconstruct the full intent and answer it.

## Language Policy

Use the user's primary language for headings, options, confirmation, and answers.

For placeholders:

- In Chinese, Japanese, or mixed CJK contexts, use `【具体占位符】`.
- In English and most other languages, use `[specific placeholder]`.
- In expansion mode, recognize and fill both `【...】` and `[...]` placeholders.
- Keep placeholder names specific and local-language natural, such as `【报错内容】`, `【需要翻译的文本】`, `[error message]`, `[text to translate]`, or `[target style]`.

## Mode Selection

Use **next-input prediction** when the user explicitly asks for likely next inputs, possible follow-up questions, nine options, "predict what I will ask next", "what might I ask next", or asks to use this skill.

Use **vague-input clarification** only when direct answering is likely to answer the wrong question. Typical triggers:

- The input is only a keyword, such as `VPN`, `error`, `resume`, `paper`, `translate`, `optimize`, `won't open`, `not right`, `leave request`, `email`.
- The action is vague, such as "help me look", "fix it", "write it", "make it", "what should I do", "this怎么办".
- The task object is missing, such as "translate this" with no text, or "polish it" with no content or prior object.
- References are unclear: "this doesn't work", "it broke", "still wrong", "what about that issue".
- The phrase has several very different meanings, such as "won't open" meaning a website, app, file, device, account, or project.
- Key conditions are missing and different answers would require different paths.
- The topic is troubleshooting, code errors, legal, medical, financial, safety, or other high-cost/high-risk advice and key facts are absent.
- The input resembles a search query, draft fragment, typo-heavy sentence, or half sentence rather than an actionable request.

Do **not** clarify when the request is clear enough to answer usefully:

- The goal, object, task type, and output direction are clear.
- The missing details only affect polish, not the main answer.
- The user gave the text or artifact to process.
- The object is clear from previous context.
- The user explicitly asked not to confirm or ask follow-up questions.
- A reasonable assumption can produce a useful answer. State the assumption briefly and proceed.

Operational rule: if you have very high confidence in the user's intent, answer directly. If direct answering is likely to go down the wrong path, clarify with options.

## Next-Input Prediction Workflow

Before writing predictions, silently inspect the current conversation:

1. Identify the user's stage: understanding, design, revision, challenge/correction, execution, comparison/decision, extension, or verification.
2. Use only information already present in the conversation. Do not invent private background, identity, emotion, finances, health, or sensitive facts.
3. Generate at least 12 internal candidates, then choose the best 9.
4. Sort by likelihood, considering continuity, goal value, user style, information gaps, ease of replying, common dialogue patterns, diversity, and specificity.
5. Ensure the 9 items are meaningfully different.

For explicit next-input prediction requests, output **exactly 9** items unless the user asks for another number.

## Vague-Input Clarification Workflow

When clarifying vague input:

1. Preserve the user's original keywords in the candidates whenever possible.
2. Generate the smallest useful set of high-quality candidates, from 2 to 9. Do not force 9 if only 3-5 are genuinely plausible.
3. Rank candidates by the most likely real intent.
4. Cover distinct intent paths, not minor wording variants.
5. Use placeholders for missing but necessary facts instead of inventing them.
6. Do not answer the vague request before the user chooses, unless there is an urgent safety reason to give a brief caution.

## Candidate Rules

Every option must be a complete sentence the user could send directly. It may be:

- A fully specified sentence.
- A full sentence with specific placeholders.

Good placeholders:

- Chinese: `【具体场景】`, `【目标风格】`, `【需要翻译的文本】`, `【需要润色的文本】`, `【电脑系统】`, `【报错内容】`, `【软件名称】`, `【目标网站】`, `【具体现象】`, `【已尝试的方法】`, `【最终产物类型】`.
- English: `[specific context]`, `[target style]`, `[text to translate]`, `[text to polish]`, `[operating system]`, `[error message]`, `[software name]`, `[target website]`, `[symptom]`, `[steps already tried]`, `[final artifact type]`.

Avoid vague placeholders such as `【内容】`, `【东西】`, `【情况】`, `[content]`, `[thing]`, or `[stuff]`.

Use 1-4 placeholders per option when possible. Order them left to right in the order the user should fill them.

## Output Formats

For next-input prediction in Chinese:

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

For next-input prediction in English:

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

For vague-input clarification in Chinese:

```text
我还不能完全确定你的意思。你可能想表达的是：
1. ...
2. ...
3. ...

你可以直接回复序号；如果选项里有占位符，也可以按这个格式补全：
序号；占位符1的内容；占位符2的内容；占位符3的内容
```

For vague-input clarification in English:

```text
I cannot fully determine your intent yet. You may mean:
1. ...
2. ...
3. ...

You can reply with just a number, or fill placeholders like this:
number; value for placeholder 1; value for placeholder 2; value for placeholder 3
```

Do not explain why each option was generated unless the user asks. If fewer than 9 clarification candidates are enough, show only those candidates.

## Expansion Recognition

After any prediction or clarification list, treat these as selection forms when they refer to an available option:

- `2`
- `2; A; B`
- `2；A；B`
- `2: A`
- `2. A`
- `第2个`
- `我选2`
- `选 2`
- `用第二个`
- `按第 2 条来`
- `option 2`
- `I choose 2`
- `use #2`
- `go with the second one`

If the selected number is outside the latest list's range, say the option does not exist and ask the user to choose again.

Do not treat a bare number as a selection if there is no recent numbered prediction/clarification list.

## Expansion Workflow

When the user selects an option:

1. Use the latest prediction or clarification list by default.
2. Retrieve the selected option.
3. Extract placeholders from left to right, supporting both `【...】` and `[...]`.
4. Split the user's remaining text on Chinese semicolons, ASCII semicolons, newlines, or the first colon after the selected number. Preserve punctuation inside values.
5. Fill placeholder values in order.
6. If values outnumber placeholders, append the extras as additional requirements.
7. If values are fewer than placeholders and the missing facts cannot be reliably inferred, name the missing placeholders and ask for them. If a useful answer is still possible, give a general answer and note the limitation.
8. If the option has no placeholders and the user supplied no extra text, treat the option itself as the full intent and answer.
9. If the user adds a modification, such as "2, but make it more formal", merge that as an additional requirement.

After reconstructing intent, answer the reconstructed intent. Do not stop at "I understand."

Preferred confirmation:

```text
我将你的输入理解为：...
```

or:

```text
I understand your intended request as: ...
```

Omit the confirmation when the user asked for direct execution or the reconstruction is obvious and short.

## Uncertainty Levels

- **Low uncertainty**: intent is clear. Answer directly.
- **Medium uncertainty**: broad intent is clear but details are missing. Make a reasonable assumption, state it briefly, and answer.
- **High uncertainty**: intent, object, or path is unclear. Offer clarification candidates.

Prefer direct answers when the risk of answering wrong is low. Prefer candidates when the risk of answering wrong is high.

## Safety and Accuracy Boundaries

Never turn a candidate into a claim of fact. For example, if the user says "my boss didn't reply", do not assume "my boss is intentionally ignoring me"; offer a neutral option such as "My boss has not replied to [message/request], please help me decide how to follow up."

Do not add unsupported sensitive or private assumptions about age, income, diagnosis, mental state, politics, family, legal status, or identity.

For medical, legal, financial, security, crisis, or high-stakes situations, clarify missing facts when necessary and avoid definitive conclusions from vague input.

When the reconstructed intent involves facts that may have changed recently, laws, prices, policies, software versions, news, medical, legal, or financial matters, follow the higher-accuracy requirements of the active system instructions, including browsing or source checking when required.

## Examples

Chinese vague input: `报错了`

```text
我还不能完全确定你的意思。你可能想表达的是：
1. 我在运行【软件/代码/命令】时出现报错：【报错内容】，请帮我分析原因。
2. 我使用的是【系统/环境】，在执行【具体操作】时出现【报错现象】，请帮我排查。
3. 请根据这个报错【报错内容】告诉我最可能的原因和解决方法。
4. 我不懂这个报错是什么意思，请用简单的话解释【报错内容】。

你可以直接回复序号；如果选项里有占位符，也可以按这个格式补全：
序号；占位符1的内容；占位符2的内容；占位符3的内容
```

English vague input: `won't open`

```text
I cannot fully determine your intent yet. You may mean:
1. I cannot open [website/app/file/device], please help me troubleshoot the cause.
2. When I try to open [target item] on [device or operating system], I see [specific symptom], please help me diagnose it.
3. [target item] used to open but no longer opens, please list the likely causes and steps to fix it.
4. When I open [target item], it shows [error message], please help me understand and fix it.

You can reply with just a number, or fill placeholders like this:
number; value for placeholder 1; value for placeholder 2; value for placeholder 3
```

Expansion example:

If option 1 is `I get this error when running [software/code/command]: [error message]. Please help me find the cause.`

User replies: `1; Python script; ModuleNotFoundError: No module named pandas`

Reconstruct as: `I get this error when running Python script: ModuleNotFoundError: No module named pandas. Please help me find the cause.`

Then explain the missing `pandas` package and how to install it.

## Self-Check Before Output

Before producing candidates, silently verify:

- Is clarification truly needed, or can I answer directly?
- Are all candidates tied to the user's input and context?
- Are candidates distinct?
- Did I avoid adding unsupported facts?
- Are placeholders specific and fillable?
- Is the list sorted by likelihood?
- Is the number of clarification candidates justified?
- Can I answer immediately after the user selects any candidate?
