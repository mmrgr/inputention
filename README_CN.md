# Inputention

> 面向 AI 对话的意图自动补全。预测下一步输入，澄清模糊表达，并把“序号 + 少量补充”还原成完整请求。

Inputention 是一个 Codex skill，用来降低用户和 AI 对话时的输入成本。它可以把“我接下来可能想问什么”或“我只输入了一个模糊关键词”转化成一组按可能性排序的自然语言候选项。用户只需要回复序号，必要时补充几个字段，助手就能拼合出完整意图并继续回答。

它适合用于智能助理、编程助手、客服问答、学习辅导、研究辅助、写作工作流，以及任何用户经常输入 `报错了`、`VPN`、`打不开`、`翻译一下`、`改一下` 这类片段的场景。

## 功能亮点

- **一个字符即可触发**：用户消息第一个字符是 `?` 或 `？` 时，立即启用 Inputention。
- **预测下一步输入**：根据当前对话生成 9 个最可能的后续输入。
- **澄清模糊输入**：把关键词、半句话、指代不明内容转成 2-9 个完整意图候选。
- **支持序号回复**：识别 `2`、`2；A；B`、`我选2`、`I choose 2`、`use #2` 等形式。
- **自动填充占位符**：按顺序填充 `【报错内容】`、`【目标风格】`、`[error message]` 等占位符。
- **多语言友好**：中文/CJK 场景使用 `【...】`，英文及多数其他语言使用 `[...]`。
- **低打扰设计**：用户意图清楚时直接回答；只有可能答偏时才通过候选项澄清。

## 为什么需要 Inputention

AI 对话最容易卡住的地方通常有两个：

1. 用户知道大方向，但不想写一整段完整提示词。
2. 用户只输入了一个片段，助手却猜错了真实任务。

Inputention 通过“候选项 + 序号补全”解决这两个问题：

```text
用户：?报错了

助手：
我还不能完全确定你的意思。你可能想表达的是：
1. 我在运行【软件/代码/命令】时出现报错：【报错内容】，请帮我分析原因。
2. 我使用的是【系统/环境】，在执行【具体操作】时出现【报错现象】，请帮我排查。
3. 请根据这个报错【报错内容】告诉我最可能的原因和解决方法。
4. 我不懂这个报错是什么意思，请用简单的话解释【报错内容】。

用户：1；Python 脚本；ModuleNotFoundError: No module named pandas

助手：
我将你的输入理解为：我在运行 Python 脚本时出现报错：ModuleNotFoundError: No module named pandas，请帮我分析原因。
...
```

## 核心模式

### 1. 下一步输入预测

当用户明确要求预测后续输入时，使用这个模式。

典型触发：

```text
请预测我接下来最可能输入的 9 句话。
给我 9 个可能继续问的问题。
Predict my most likely next inputs.
?
？
```

行为：

- 读取当前对话上下文。
- 判断用户处于理解、设计、修改、纠错、执行、比较、延展或验证等哪类阶段。
- 内部生成多个候选。
- 选出最合理的 9 个。
- 按可能性从高到低排序。
- 输出可直接复制或通过序号选择的完整句子。

### 2. 模糊输入澄清

当直接回答很可能答偏时，使用这个模式。

典型触发：

```text
?VPN
?打不开
?翻译一下
?报错了
?这个不对
?won't open
```

行为：

- 尽量保留用户原始关键词。
- 生成最小但够用的高质量候选，通常 2-5 个，最多 9 个。
- 用占位符表示缺失信息，而不是替用户编造事实。
- 只要能生成合理候选，就避免只问“你是什么意思？”。

### 3. 序号意图补全

当 Inputention 输出候选列表后，用户可以直接选择。

支持的回复形式：

```text
2
2；A；B
2; A; B
2: A
第2个
我选2
option 2
I choose 2
use #2
go with the second one
```

行为：

- 从最近一次 Inputention 列表中找到对应选项。
- 从左到右提取占位符。
- 按顺序填入用户补充的内容。
- 多出的内容作为附加要求。
- 缺少必要内容时只追问缺失项。
- 拼合完整请求后继续完成任务。

## 快速开始

把本仓库作为 Codex skill 文件夹使用，或把 `SKILL.md` 复制到你的 Codex skill 目录中。

当前项目位置：

```text
D:\my   github\inputention
```

典型结构：

```text
inputention/
├── agents/
│   └── openai.yaml
├── SKILL.md
├── README.md
└── README_CN.md
```

Codex skill 名称是 `inputention`，因此在支持显式 skill 调用的环境中，也可以用 `$inputention` 调用。

触发逻辑主要写在 `SKILL.md` 的 frontmatter `description`、`agents/openai.yaml` 和正文的模式选择规则中。

## 显式 `?` 触发

Inputention 支持非常轻量的手动触发方式。

只要用户消息的**第一个字符**是 `?` 或 `？`，助手就应该启用这个 skill。

在支持显式 skill 调用的 Codex 环境中，也可以直接使用 `$inputention` 调用。

示例：

```text
?报错了
？打不开
?won't open
?邮件
?predict my next inputs
?
```

开头的问号被视为调用前缀，不属于用户真正要处理的内容。

规则：

- `?报错了` 表示对 `报错了` 进行模糊输入澄清。
- `?predict my next inputs` 表示预测下一步输入。
- 单独输入 `?` 表示基于当前对话预测接下来最可能输入什么。
- 即使 `?` 后面的文字已经比较清楚，也仍应输出 Inputention 风格的候选项，而不是绕过 skill。

## 输出示例

### 中文模糊输入

```text
用户：?打不开

助手：
我还不能完全确定你的意思。你可能想表达的是：
1. 我打不开【网站/软件/文件/设备】，请帮我排查原因。
2. 我在【设备或系统】上打开【目标对象】时出现【具体现象】，请帮我判断问题。
3. 【目标对象】之前能打开，现在打不开了，请告诉我可能原因和解决步骤。
4. 我打开【目标对象】时提示【错误信息】，请帮我分析。

你可以直接回复序号；如果选项里有占位符，也可以按这个格式补全：
序号；占位符1的内容；占位符2的内容；占位符3的内容
```

### 中文序号补全

```text
用户：1；PDF 文件；提示文件已损坏

助手：
我将你的输入理解为：我打不开 PDF 文件，请帮我排查原因。附加要求：提示文件已损坏。
...
```

### 英文模糊输入

```text
User: ?won't open

Assistant:
I cannot fully determine your intent yet. You may mean:
1. I cannot open [website/app/file/device], please help me troubleshoot the cause.
2. When I try to open [target item] on [device or operating system], I see [specific symptom], please help me diagnose it.
3. [target item] used to open but no longer opens, please list the likely causes and steps to fix it.
4. When I open [target item], it shows [error message], please help me understand and fix it.
```

## 占位符规范

只有在缺少必要信息、且助手不应该擅自补全时，才使用占位符。

推荐中文占位符：

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

推荐英文占位符：

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

避免使用模糊占位符：

```text
【内容】
【东西】
【情况】
[content]
[thing]
[stuff]
```

## 设计原则

Inputention 遵循一条简单规则：

- 用户表达清楚时，直接回答。
- 用户以 `?` 或 `？` 开头时，启用 Inputention。
- 用户表达不清且直接回答容易答偏时，给出排序后的意图候选。
- 用户选择候选后，补全真实意图并完成任务。

高质量候选项应该：

- 是完整自然语言请求。
- 按可能性排序。
- 彼此有实质区别。
- 贴合当前上下文。
- 不加入未经用户提供的私人推断。
- 用户选择后可以立即执行。

## 安全与准确性

Inputention 不能把猜测写成事实。

例如用户说：

```text
老板没回我
```

不要生成：

```text
老板故意冷落我，我该怎么办？
```

更合适的是：

```text
我的老板还没有回复【某条消息/某个请求】，请帮我判断下一步应该如何跟进。
```

在医疗、法律、金融、安全、心理危机等高风险领域，如果信息不足，应优先澄清关键事实，避免基于模糊输入给出确定性结论。

## 校验

修改后建议运行 skill 校验：

```powershell
$env:PYTHONUTF8='1'
python C:\Users\mmrgr\.codex\skills\.system\skill-creator\scripts\quick_validate.py "D:\my   github\inputention"
```

如果当前 Python 环境缺少 `PyYAML`，可以换用包含该依赖的环境，或临时安装后再校验。

## 路线图

- 增加一键安装到 Codex skills 目录的脚本。
- 增加触发行为测试集。
- 增加日文、西班牙文等多语言示例。
- 增加序号补全和占位符填充的回归测试。

## 许可证

当前仓库尚未指定许可证。公开发布前建议补充 LICENSE 文件。
