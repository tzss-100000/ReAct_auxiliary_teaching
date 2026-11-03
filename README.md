# ReAct 辅助教学案例

一个面向 AI Agent 初学者的中文 Jupyter Notebook，通过少量 Python 代码和逐步讲解，演示如何使用大模型完成多轮分析、保存历史步骤，并在满足结束条件时输出最终答案。

项目使用 OpenAI Python SDK 调用兼容接口，Notebook 中以 Moonshot / Kimi 为配置示例，无需安装额外的 Agent 框架。

> **实现范围**：这是一个帮助理解 ReAct 循环结构的简化教学示例。当前的 `action` 是模型生成的文字描述，`observation` 也由模型生成，没有实际执行搜索、计算器或其他外部工具。因此，它演示的是 ReAct 风格的多轮文本循环，而非完整的工具执行型 ReAct Agent。

## 可以学到什么

- 使用环境变量配置 API Key、接口地址和模型名称。
- 通过 Prompt 约定每轮返回的 JSON 结构。
- 将历史步骤重新组织为下一轮输入。
- 使用 `done` 和最大轮数控制循环结束。
- 观察并修改 Prompt，比较不同问题下的多轮输出。

## 仓库文件

| 文件 | 说明 |
| --- | --- |
| `ReAct.ipynb` | 中文概念介绍、核心代码及已保存的示例输出 |
| `requirements.txt` | Python 依赖 |
| `.gitignore` | 忽略环境变量文件、Python 缓存及虚拟环境等内容 |

## 快速开始

### 1. 下载项目并安装依赖

需要 Python 环境，建议使用 Python 3.12，与 Notebook 保存的内核版本一致。依赖文件标注 Python ≥ 3.8，但实际可用版本还受安装的 SDK 和 Jupyter 版本影响。

在 PowerShell 中执行：

```powershell
git clone https://github.com/tzss-100000/ReAct_auxiliary_teaching.git
cd ReAct_auxiliary_teaching
python -m pip install -r requirements.txt
```

如果已经有本地项目，直接进入对应文件夹安装依赖即可。

### 2. 配置环境变量

以下值来自 Notebook 中的配置示例；请按自己账号可用的模型和服务配置调整：

```powershell
$env:MOONSHOT_API_KEY="替换为你的API Key"
$env:MOONSHOT_BASE_URL="https://api.moonshot.cn/v1"
$env:KIMI_MODEL="kimi-k2.5"
```

这些变量仅在当前 PowerShell 会话及其后启动的子进程中生效。随后从**同一个终端**启动 Jupyter：

```powershell
python -m jupyter notebook
```

在浏览器中打开 `ReAct.ipynb`，从上到下执行所有单元格。

也可以使用 VS Code 的 Python、Jupyter 扩展打开 Notebook，并选择已安装依赖的 Python 内核。如果 VS Code 已经在运行，终端里临时设置的环境变量不会自动进入已有的 Notebook 内核；可以使用下面的用户级配置，再完全退出并重新打开 VS Code：

```powershell
[Environment]::SetEnvironmentVariable("MOONSHOT_API_KEY", "替换为你的API Key", "User")
[Environment]::SetEnvironmentVariable("MOONSHOT_BASE_URL", "https://api.moonshot.cn/v1", "User")
[Environment]::SetEnvironmentVariable("KIMI_MODEL", "kimi-k2.5", "User")
```

**代码通过 `os.getenv()` 读取环境变量，不会自动加载 `.env` 文件。** 不要将真实密钥写进 Notebook 或提交到 GitHub。API 调用的费用与额度取决于所用服务。

### 3. 修改问题并运行

找到设置问题的单元格，修改 `question`，例如：

```python
question = "请帮我制定一个为期两周的 Python 基础复习计划，每天学习一小时。"
```

按顺序执行后面的函数定义，最后运行：

```python
final_answer = run_react(question, react_prompt)
print(final_answer)
```

程序会逐轮打印 `Thought`、`Action`、`Observation`，并在模型返回 `done: true` 时打印和返回最终答案。不同运行的轮数与内容可能不同。

## 核心流程

每一轮执行以下步骤：

1. `format_history(history)`：将之前的步骤整理为文本。
2. `ask_llm(question, history, react_prompt)`：把问题、历史和系统提示词发送给模型，使用 JSON 模式请求结构化响应。
3. `run_react(...)`：打印并保存本轮结果，检查结束条件，再决定是否继续。

模型返回的数据结构如下：

```json
{
  "thought": "本轮的简短分析",
  "action": "准备进行的推理动作",
  "observation": "本轮生成的中间结果",
  "done": false
}
```

结束时返回 `"done": true`，并增加 `"final_answer"` 字段。

默认最多调用 10 轮，可以在最后的调用中调整：

```python
final_answer = run_react(question, react_prompt, max_steps=5)
```

达到最大轮数仍未结束时，函数返回“超过最大轮数还没得到最终答案。”。最大轮数限制的是调用次数，不是单次请求的耗时。

## 更换模型或接口

客户端使用 `OpenAI(api_key=api_key, base_url=base_url)`。若切换到其他兼容服务，可以修改三个环境变量的值；当前代码仍读取 `MOONSHOT_API_KEY`、`MOONSHOT_BASE_URL` 和 `KIMI_MODEL` 这三个名称。

目标服务需要支持代码使用的 Chat Completions 接口以及 `response_format={"type": "json_object"}`。接口兼容程度和模型输出行为需要实际验证，不能仅凭修改名称保证可运行。

## 常见问题

| 现象 | 排查方式 |
| --- | --- |
| `'NoneType' object has no attribute 'strip'` | 当前内核没有读取到 `MOONSHOT_API_KEY`；检查环境变量及启动顺序 |
| 提示未配置接口地址或模型 | 检查 `MOONSHOT_BASE_URL` 和 `KIMI_MODEL` |
| `ModuleNotFoundError: No module named 'openai'` | 在 Notebook 所选内核对应的 Python 环境中安装依赖 |
| 认证、额度或模型访问错误 | 检查密钥、服务账号额度及模型可用权限 |
| JSON 解析失败或字段缺失 | 检查模型响应；当前代码没有 JSON 修复、字段校验或自动重试 |
| 修改问题后输出仍不对应 | 重新执行设置 `question` 的单元格，再运行最后的调用单元格 |

## 学习时需要注意

- 当前示例没有外部事实验证，模型生成的中间结果可能出错；多轮输出和自我检查并不保证答案正确。
- Notebook 中保留的“火星旅游”输出包含虚构设施、价格以及未经验证的科学描述，仅用于观察循环行为，不能作为事实资料。
- 每轮都会重新发送历史步骤，轮数增加会使输入变长，也可能增加调用成本。
- 当前实现没有工具分发、RAG、持久化记忆或完整的异常处理；适合先理解基础流程，再逐步扩展。
- Notebook 已保存的输出中包含一次内核崩溃记录；这不代表每次运行都会崩溃，若复现应查看本地 Jupyter 日志。

可以进一步尝试：接入真实工具，让 Python 执行 `action`，并把工具返回值作为 `observation`；再添加输出校验、错误重试和日志记录。

## 参考资料

- [ReAct: Synergizing Reasoning and Acting in Language Models](https://arxiv.org/abs/2210.03629)

本仓库是用于理解循环结构的简化教学实现，并非原论文实验的完整复现。
