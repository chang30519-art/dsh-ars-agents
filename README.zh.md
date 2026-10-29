# dsh-ars-agents

给 [DeepSeek Harness](https://github.com/deepseek-ai/deepseek-harness)（DSH）用的**学术科研智能体组**：7 个 agent preset，一个总控 + 6 个专项，装好后在新建会话的预设选择器里直接选。

不是「一个会读技能的 agent」，而是把学术研究的真实工作流拆成**互相派活的团队**：总控按阶段把任务交给调研 / 筛选 / 写作 / 评审 / 核验 / 实验六个专项 agent。

[English](README.md)

---

## 7 个 agent

| # | 预设名 | 定位 | 核心职能 |
|---|---|---|---|
| 1 | **ARS·总控** | 10 阶段编排器 | 判断该走哪个工作流，逐阶段派工，2.5 / 4.5 两道诚信门禁 |
| 2 | **ARS·调研** | 深度调研 | 6 阶段 13 角色 8 模式，Devil's Advocate 三道强制检查点 |
| 3 | **ARS·筛选** | 文献筛选 | 标题摘要 → 全文 → 冲突裁决，排除审计留痕 |
| 4 | **ARS·写作** | 论文撰写 | IMRAD 结构，引用必核实，`[CITATION-NEEDED]` 标记法 |
| 5 | **ARS·评审** | 审稿模拟 | 编辑判定 ACCEPT / MINOR / MAJOR / REJECT，逐行意见 |
| 6 | **ARS·核验** | 诚信核验 | 引用存在性、claim 对齐、论断强度漂移、token 守恒 |
| 7 | **ARS·实验** | 实验执行 | 实验设计、跑批、结果归因与图表 |

总控内置 `dispatch` 工具，只允许派给上面这 6 个专项 preset（`allowedAgentPresets` 白名单），不会跑到无关 agent 上。

## 前置条件

### 1. DSH 0.2.0-rc.2 或更高

低版本不支持 `agent-preset` 的当前 schema。

### 2. `academic-research-suite` 技能包（**必须**，不随本包分发）

这些 agent 的 persona 引用了该技能包的工作流结构和角色文档路径（例如 `<技能根>/ars/deep-research/WORKFLOW.md`）。

上游仓库：

```
https://github.com/Imbad0202/academic-research-skills
```

作者 Cheng-I Wu，许可 **CC-BY-NC-4.0（署名 + 非商业）**。

安装：把 `academic-research-suite` 目录放到你的 skills 目录下，默认是：

```
<DSH_HOME>/skills/academic-research-suite
```

如果放在别处，设置环境变量 `DSH_ARSS_SKILLS_DIR` 指向**包含所有技能的父目录**（不是技能本身）。

> 本包**不打包**该技能包：一是许可要求使用者自行遵守非商业条款，二是它有 46 MB、2756 个文件，不适合塞进插件包。

### 3. Reader 本地文献库（**可选**）

若你装了 [Reader](https://github.com/cynicalight/reader) 并用它管理文献，这些 agent 会自动把它当作第一证据层：先 `reader search` 查库，库内没有才走外网。

- Reader 需在运行时才能调用（`reader status` 会显示连接状态）
- `reader` 命令在 PATH 上时直接可用；否则设 `READER_CLI` 指向 `reader.exe`
- 没装 Reader 不影响 agent 工作，只是失去本地文献库检索能力

### 4. 同步注意：**不要把 API Key / 本地路径写进共享配置**

本包的 `cordis.patch.yml` 不含任何密钥或绝对路径。技能目录、Reader 路径都通过环境变量解析。

## 安装

### 方式一：本地目录（开发调试最方便）

```bash
git clone https://github.com/chang30519-art/dsh-ars-agents.git
```

然后在 DSH 的 **Plugins → Add plugin** 里填克隆下来的**本地绝对路径**，例如：

```
E:\GitHub\dsh-ars-agents
```

### 方式二：Git 地址

**Plugins → Add plugin** 里直接填：

```
https://github.com/chang30519-art/dsh-ars-agents
```

### 方式三：手动合并 patch

不想装插件、只想把预设拷进自己的 profile，把 `cordis.patch.yml` 的内容追加到：

```
<DSH_HOME>/profiles/desktop/cordis.patch.yml
```

追加前先备份。合并后重启 DSH。

> 注意：DSH 重启时会重写 profile 文件。手动合并的改动可能被覆盖，因此推荐用插件方式安装。

## 环境变量

| 变量 | 作用 | 默认值 |
|---|---|---|
| `DSH_HOME` | DSH 数据目录 | `%USERPROFILE%\.dsh` |
| `DSH_ARSS_SKILLS_DIR` | 技能父目录（覆盖默认） | `<DSH_HOME>/skills` |
| `READER_CLI` | `reader.exe` 完整路径 | 从 PATH 查找 |

## 验证安装

1. 重启 DSH
2. 新建会话，在预设选择器里应能看到 **ARS·总控 / 调研 / 筛选 / 写作 / 评审 / 核验 / 实验**
3. 选「ARS·总控」，问一句「帮我看下我文献库里关于 X 的内容」——它应该先调 `skill` 加载 `academic-research-suite`，再跑 `reader search`

若预设没出现，检查：

- Plugins 页面里 bundle 是否为 **On**
- `<DSH_HOME>/skills/academic-research-suite` 是否存在
- 启动日志里有没有 `cannot resolve` 之类的模块错误

## 架构说明

```
┌─────────────────────────────────────────┐
│  ARS·总控 (ars-pipeline, order 1)        │
│  10 阶段编排 + dispatch 工具             │
└──────────────┬──────────────────────────┘
               │ dispatch（白名单限制）
   ┌───────────┼───────────┬───────────┐
   ▼           ▼           ▼           ▼
 调研        筛选        写作        评审
 research   screener    writer     reviewer
   │           │           │           │
   └───────────┴─────┬─────┴───────────┘
                     ▼
                  核验 verify
                     │
                     ▼
                  实验 experiment
```

每个专项 agent 的 persona **就是该工作流的真实内容**（阶段、门禁、模式表、输出契约），不是「去读文档」的指针。深层细则才回落到技能包里的 `references/` 与 `templates/`。

## 已知限制

- **shell 工具分平台**：Windows 用 `tool-pwsh`，其它平台用 `tool-bash`，两者都按 `process.platform` 自动启停。Windows 上验证最充分。
- **persona 是中文**：这些 agent 用中文思考和输出。写英文论文正文时会按需切换，但指令层是中文。
- **依赖技能包结构**：如果上游 `academic-research-skills` 改了目录布局（例如 `ars/<workflow>/WORKFLOW.md` 改名），persona 里的路径引用需要同步更新。
- **不包含 WeKnora 集成**：本包只用 Reader CLI 做本地文献检索。如果你用 WeKnora，需要自己装插件并在 profile 里加行。

## 许可与署名

- 本包代码与 persona 文本：**MIT**（见 [LICENSE](LICENSE)）
- `academic-research-suite` 技能包：**CC-BY-NC-4.0**，作者 Cheng-I Wu，**不随本包分发**。使用、修改、再分发该技能包时须独立遵守其署名与非商业条款。

本包的 persona 参考并改编了该技能包的工作流结构与角色命名。若你把本包与该技能包一起分发，需要同时满足 CC-BY-NC-4.0。
