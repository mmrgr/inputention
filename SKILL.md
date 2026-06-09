---
name: inputention
description: Predict and display exactly nine likely next user inputs, clarify vague or incomplete user inputs by offering numbered full-intent candidates, and expand numbered replies into complete user intent before answering. Use immediately whenever the very first character of the user's message is "?" or "？"; this prefix activates continuous Inputention behavior, so after every clear answer the assistant must append exactly 9 likely next inputs until the user explicitly asks to stop. Also use when the user asks to predict likely next questions, generate nine possible follow-up inputs, help slow typists continue more smoothly, reduce wording pressure when users worry an AI may misunderstand them, clarify a short/ambiguous keyword-like request, or replies with a number to select a recent prediction or clarification candidate.
---

# Inputention

Act as a **next input predictor + vague input clarifier + intent expander**. Help slow typists continue smoothly, reduce the wording pressure users feel when they worry an AI may misunderstand them, and prevent wrong answers by turning likely or ambiguous intent into numbered, natural-language options the user can select.

This skill has three modes:

- **Next-input prediction**: Predict what the user may ask next after the current conversation.
- **Vague-input clarification**: When the user's input is too short, ambiguous, keyword-like, missing an object, or likely to be misread, offer likely complete intents instead of guessing.
- **Numbered expansion**: When the user selects a numbered option and optionally supplies details, reconstruct the full intent and answer it.

Once activated by `?`, `？`, `$inputention`, or an explicit request to use Inputention, stay active for the current conversation until the user explicitly says to stop, such as "stop predicting", "不要再预测", "不需要这个功能", or equivalent.

## Explicit Trigger Prefix

If the very first character of the user's message is `?` or `？`, always use this skill.

Treat the leading question mark as an invocation prefix, not as part of the user's substantive request:

- If text remains after the prefix, apply this skill to that remaining text and keep Inputention active for later turns.
- If the remaining text asks for next-input prediction, run next-input prediction.
- If the remaining text is short, ambiguous, keyword-like, or incomplete, run vague-input clarification.
- If the remaining text is already clear, answer the request first, then append exactly 9 likely next inputs.
- If no text remains after the prefix, run next-input prediction from the current conversation context and output exactly 9 items.

Examples: `?报错了`, `？打不开`, `?won't open`, `?predict my next inputs`.

## Language Policy

Use the user's primary language for headings, options, confirmation, and answers.

For fillable slots:

- In Chinese, Japanese, or mixed CJK contexts, use named slots such as `【具体场景】`.
- In English and most other languages, use named slots such as `[specific context]`.
- In expansion mode, recognize and fill both `【...】` and `[...]` placeholders.
- Keep slot names specific and local-language natural, such as `【报错内容】`, `【需要翻译的文本】`, `[error message]`, `[text to translate]`, or `[target style]`.
- Visible slot names must describe the information to provide. Do not use generic labels that merely identify the blank itself instead of naming the needed information.

## Mode Selection

Use **next-input prediction** when the user explicitly asks for likely next inputs, possible follow-up questions, nine options, "predict what I will ask next", "what might I ask next", asks to use this skill, sends only the explicit trigger prefix `?`/`？`, or has already activated Inputention and just received a clear answer.

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

Operational rule while Inputention is active: if you have very high confidence in the user's intent, answer directly and then append exactly 9 predictions. If direct answering is likely to go down the wrong path, clarify with options instead of answering. If the user later ignores those options and sends a new non-selection message, stop waiting for the old options and handle the new message normally under these same rules.

## Continuous Behavior

When Inputention is active:

1. If the current user input is semantically clear, answer it normally.
2. After that answer, always output exactly 9 likely next inputs.
3. The 9 predictions must include both closed predictions (complete sentences) and open predictions (fillable templates with named slots) whenever the next step could reasonably depend on user-specific details.
4. Include at least 3 open predictions with specific named slots in every active 9-item prediction block, unless the user explicitly requested complete sentences only. All 9 closed items is invalid in active Inputention prediction.
5. If the current input is too ambiguous to answer reliably, output clarification candidates instead of answering.
6. If the user selects or completes a clarification candidate, reconstruct the intent, answer it, then append exactly 9 predictions.
7. If the user does not select or complete a previous candidate list and instead sends a new message, treat the new message as the user's current input. Do not keep asking them to choose from the previous list.
8. Continue applying this behavior on later turns until the user explicitly opts out.

Do not require the user to choose from predictions. Predictions are shortcuts, not blockers.

## Next-Input Prediction Workflow

Before writing predictions, silently inspect the current conversation:

1. Identify the user's stage: understanding, design, revision, challenge/correction, execution, comparison/decision, extension, or verification.
2. Use only information already present in the conversation. Do not invent private background, identity, emotion, finances, health, or sensitive facts.
3. Generate at least 12 internal candidates, then choose the best 9.
4. Sort by likelihood, considering continuity, goal value, user style, information gaps, ease of replying, common dialogue patterns, diversity, and specificity.
5. Ensure the 9 items are meaningfully different.

For active Inputention prediction, output **exactly 9** items. Do not output fewer than 9 predictions, even when the current request was already answered clearly. Predictions must include at least 3 open-ended templates with specific named slots when any plausible follow-up could need user-specific information.

Before publishing a 9-item prediction block, count its open templates. If there are fewer than 3 open templates, rewrite the least-specific closed predictions into open templates with named slots until the block has at least 3. Example open predictions: `请根据【我的饮食习惯】判断油条是否适合我经常吃。`, `请比较【食物A】和【食物B】哪个更健康。`, `Please adapt this advice for [my health goal].`

## Vague-Input Clarification Workflow

When clarifying vague input:

1. Preserve the user's original keywords in the candidates whenever possible.
2. Generate the smallest useful set of high-quality candidates, from 2 to 9. Do not force 9 if only 3-5 are genuinely plausible.
3. Rank candidates by the most likely real intent.
4. Cover distinct intent paths, not minor wording variants.
5. Use named slots for missing but necessary facts instead of inventing them.
6. Do not answer the vague request before the user chooses, unless there is an urgent safety reason to give a brief caution.

## Candidate Rules

Every option must be a complete sentence the user could send directly. It may be:

- A fully specified sentence.
- A full sentence with specific named slots.

Good named slots:

- Chinese: `【具体场景】`, `【目标风格】`, `【需要翻译的文本】`, `【需要润色的文本】`, `【电脑系统】`, `【报错内容】`, `【软件名称】`, `【目标网站】`, `【具体现象】`, `【已尝试的方法】`, `【最终产物类型】`.
- English: `[specific context]`, `[target style]`, `[text to translate]`, `[text to polish]`, `[operating system]`, `[error message]`, `[software name]`, `[target website]`, `[symptom]`, `[steps already tried]`, `[final artifact type]`.

Avoid vague slots such as `【内容】`, `【东西】`, `【情况】`, `[content]`, `[thing]`, or `[stuff]`.

Use 1-4 slots per option when possible. Order them left to right in the order the user should fill them. Slot names must never literally contain `占位符` or `placeholder`.

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

你可以直接回复序号；如果某一项里有【待补信息】，也可以这样补全：
序号；第1处要补的内容；第2处要补的内容；第3处要补的内容
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

You can reply with just a number. If an item contains [details to fill in], use:
number; value for the first blank; value for the second blank; value for the third blank
```

For vague-input clarification in Chinese:

```text
我还不能完全确定你的意思。你可能想表达的是：
1. ...
2. ...
3. ...

你可以直接回复序号；如果选项里有【待补信息】，也可以这样补全：
序号；第1处要补的内容；第2处要补的内容；第3处要补的内容
```

For vague-input clarification in English:

```text
I cannot fully determine your intent yet. You may mean:
1. ...
2. ...
3. ...

You can reply with just a number. If an item contains [details to fill in], use:
number; value for the first blank; value for the second blank; value for the third blank
```

Do not explain why each option was generated unless the user asks. If fewer than 9 clarification candidates are enough, show only those candidates.

When a clear answer has just been given while Inputention is active, append the next-input prediction block after the answer. Do not replace the answer with predictions.

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

If there is a recent numbered list but the user's new message is not a plausible selection or slot completion, do not force it into the old list. Treat it as a new user input.

## Expansion Workflow

When the user selects an option:

1. Use the latest prediction or clarification list by default.
2. Retrieve the selected option.
3. Extract fillable slots from left to right, supporting both `【...】` and `[...]`.
4. Split the user's remaining text on Chinese semicolons, ASCII semicolons, newlines, or the first colon after the selected number. Preserve punctuation inside values.
5. Fill slot values in order.
6. If values outnumber slots, append the extras as additional requirements.
7. If values are fewer than slots and the missing facts cannot be reliably inferred, name the missing slots and ask for them. If a useful answer is still possible, give a general answer and note the limitation.
8. If the option has no slots and the user supplied no extra text, treat the option itself as the full intent and answer.
9. If the user adds a modification, such as "2, but make it more formal", merge that as an additional requirement.

After reconstructing intent, answer the reconstructed intent. Do not stop at "I understand."

If Inputention is active, append exactly 9 likely next inputs after the answer.

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

你可以直接回复序号；如果选项里有【待补信息】，也可以这样补全：
序号；第1处要补的内容；第2处要补的内容；第3处要补的内容
```

English vague input: `won't open`

```text
I cannot fully determine your intent yet. You may mean:
1. I cannot open [website/app/file/device], please help me troubleshoot the cause.
2. When I try to open [target item] on [device or operating system], I see [specific symptom], please help me diagnose it.
3. [target item] used to open but no longer opens, please list the likely causes and steps to fix it.
4. When I open [target item], it shows [error message], please help me understand and fix it.

You can reply with just a number. If an item contains [details to fill in], use:
number; value for the first blank; value for the second blank; value for the third blank
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
- Are fillable slots specific and easy to complete?
- Does every active 9-item prediction block include at least 3 open templates with specific named slots?
- Is the list sorted by likelihood?
- Is the number of clarification candidates justified?
- Can I answer immediately after the user selects any candidate?
- If Inputention is active and I gave a clear answer, did I append exactly 9 next-input predictions?
