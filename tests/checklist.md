# Inputention Test Checklist

Use this checklist to verify Inputention behavior after editing `SKILL.md`.

## Activation

- [ ] A message beginning with `?` activates Inputention.
- [ ] A message beginning with `？` activates Inputention.
- [ ] A standalone `?` produces a next-input prediction block.
- [ ] A standalone `？` produces a next-input prediction block.
- [ ] After activation, Inputention stays active until the user explicitly opts out.

## Prediction Count

- [ ] Default next-input prediction count is 5.
- [ ] After a clear complete request, the assistant answers first and then outputs exactly 5 predictions by default.
- [ ] A standalone `??3` changes later prediction blocks to exactly 3 items.
- [ ] A standalone `？？3` changes later prediction blocks to exactly 3 items.
- [ ] `?? 3` and `？？ 3` are accepted.
- [ ] `?3`, `???3`, `？？三`, and `？？3 请继续` are not treated as prediction-count settings.
- [ ] Prediction count setting messages do not trigger answering, clarification, numbered completion, or prediction output.
- [ ] Prediction counts outside 1-9 cause a brief request to choose a number from 1 to 9.

## Prediction Block Quality

- [ ] Every active prediction block contains exactly the current prediction count.
- [ ] With the default count of 5, at least 3 predictions are open templates with specific fillable slots when user-specific details may matter.
- [ ] With prediction count 3, at least 3 predictions are open templates when user-specific details may matter.
- [ ] With prediction count 2, at least 2 predictions are open templates when user-specific details may matter.
- [ ] Fillable slot names describe the missing information, such as `【饮食习惯】` or `[health goal]`.
- [ ] Visible slot names never use generic labels that merely name the blank itself.

## Intent Clarification

- [ ] `?南瓜 空气炸锅 蒙赤` produces 3-9 intent candidates and does not answer yet.
- [ ] `?1456238359@qq.com` produces 3-9 intent candidates and does not answer yet.
- [ ] `?报错了` produces 3-9 intent candidates and does not answer yet.
- [ ] Clarification candidates preserve the user's original keywords when useful.
- [ ] Clarification candidates use the same numbered selection and fillable-slot mechanism as prediction blocks.
- [ ] Clarification output does not append a next-input prediction block.

## Numbered Completion

- [ ] After a candidate list, `4` selects option 4.
- [ ] After a candidate list, `4；贝贝南瓜` fills the first slot in option 4.
- [ ] After a candidate list, `8；200字` fills the first slot in option 8.
- [ ] After a candidate list, `5；求职；正式` fills slots from left to right.
- [ ] Numbered completion has priority over treating the message as a new request.
- [ ] After completing and answering a selected intent, Inputention appends the current prediction count.
- [ ] If the selected item has unfilled required slots, the assistant asks only for the missing values.

## Ignoring Old Lists

- [ ] If the user does not select from a prior list and sends a new non-selection message, the assistant treats it as the current input.
- [ ] The assistant does not repeatedly force the user to choose from an old list.

## Opt-Out

- [ ] "stop predicting" disables continuous prediction.
- [ ] "不要再预测" disables continuous prediction.
- [ ] "不需要这个功能" disables continuous prediction.

## Documentation

- [ ] `README.md` states default prediction count is 5.
- [ ] `README_CN.md` states default prediction count is 5.
- [ ] Both READMEs document `??N` / `？？N`.
- [ ] Both READMEs state intent clarification uses 3-9 candidates and waits for the user's next turn.
- [ ] The project does not include `agents/openai.yaml`.
