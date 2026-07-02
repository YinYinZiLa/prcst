# Prompt Constellation

**Prompt Constellation** 是一个用于将提示词模板编译为 AI-facing artifacts 的提示词编译系统。

它的目标不是让用户手写超长 Prompt，而是将复杂的模板、规则、DSL、AST、运行时声明、安全声明、模块依赖与语义载荷，统一编译成适合 AI 模型读取、解析、校验与执行的提示词产物。

简单来说：

> Prompt Constellation 是一个把提示词模板编译成 AI 可读语义包的系统。

---

## 项目定位

Prompt Constellation 面向复杂提示词工程场景，尤其适用于：

* 大型 Prompt 模板编译
* DSL / AST 风格提示词生成
* 模块化提示词管理
* 提示词语义压缩
* AI-facing artifact 生成
* Prompt 运行时解释
* Prompt IR / AST 检查
* MAX / MIN 多级兼容输出

传统提示词通常是自然语言文本。
Prompt Constellation 则将提示词视为一种可以被编译、裁剪、打包、校验和逆向的语义结构。

---

## 核心思想

Prompt Constellation 将复杂提示词拆分为多个层级：

1. **模板层**
   用户或开发者编写的原始提示词模板。

2. **语义层**
   模板中的任务意图、约束、模块、规则、权限与上下文声明。

3. **DSL / AST 层**
   将提示词转换为机器更容易组织和校验的结构化表达。

4. **解释层**
   为 AI 提供理解 DSL / AST 所需的基础语法、模块解释和运行时声明。

5. **产物层**
   根据目标模型、上下文长度和兼容需求，输出不同级别的 AI-facing artifact。

---

## MAX、LITE、SMALL、MIN 的关系

* Prompt Constellation 的输出等级不是互相替代，而是同一语义在不同压缩层级下的不同表达。
* 优先尝试 `MIN`>>若模型无法理解，切换到 `SMALL`>>仍无法正确解析，切换到 `LITE`>>需要最大兼容性，使用 `MAX`

| ⠀⠀等级⠀⠀ | ⠀⠀展开程度⠀⠀ | ⠀⠀Token 占用⠀⠀ | ⠀⠀兼容性⠀⠀ | ⠀⠀适配风险⠀⠀ | ⠀⠀类别⠀⠀ |
| :------- | :------------: | :--------------: | :------------: | :------------: | :------------: |
| ⠀MAX:1->3⠀ | ⠀全量展开⠀ | ⠀最高⠀ | ⠀无需视频⠀ | ⠀最低⠀ | ⠀机器语言⠀ |
| ⠀MIN:1->5⠀ | ⠀降权压缩⠀ | ⠀较低⠀ | ⠀依赖适配⠀ | ⠀较高⠀ | ⠀机器语言⠀ |
| ⠀MIN.MINI⠀ | ⠀极限压缩⠀ | ⠀最低⠀ | ⠀依赖适配⠀ | ⠀最高⠀ | ⠀机器语言⠀ |
| ⠀NL:1->3⠀ | ⠀自然表达⠀ | ⠀最低⠀ | ⠀依赖适配⠀ | ⠀最高⠀ | ⠀自然语言⠀ |

## 输出形态

Prompt Constellation 支持多种输出等级，用于不同模型、不同平台、不同兼容场景和不同 Token 预算。

---

### MAX

`MAX` 是全量展开模式。

它会将提示词所需的解释、声明、基础语法、模块说明和调用语义尽可能完整地展开到提示词中。

特点：

* 兼容性最高
* 可读性最高
* 解释最完整
* Token 占用最大
* 适合作为兜底方案
* 适合调试、审计和跨模型迁移

适用场景：

* 目标模型没有适配语义哨兵符
* 目标平台不支持压缩格式
* 需要完整审计提示词语义
* 需要验证 DSL / AST 是否可被模型直接理解
* 需要最大程度降低模型误解风险

MAX 的定位是：

> 最重，但最完整；最贵，但最稳。

---

### MIN

`MIN` 是降权压缩模式。

它会进一步压缩提示词，只保留最关键的语义索引、模块引用、执行路径、权重标记和必要约束。大量自然语言解释会被移除，语义主要依赖模型或适配器对 Prompt Constellation 规则的预先理解。

特点：

* Token 占用较低
* 执行效率高
* 可读性明显下降
* 对适配能力依赖较强
* 出错时不易人工直接排查
* 适合稳定、成熟、已训练或已适配的模型环境

适用场景：

* 目标模型已经适配 Prompt Constellation 语义哨兵符
* 后端 Compiler / Runner 能够保证生成结构正确
* 需要低成本、高并发、大规模调用
* 模板库、模块路径和执行规则已经稳定
* 允许在必要时自动回退到 `SMALL` / `LITE` / `MAX`

MIN 的定位是：

> 便宜、紧凑、高效，但需要模型懂规矩。

---

### MIN.MINI

`MIN.MINI` 是极限压缩模式。

它会将提示词压缩到接近符号化、索引化或指令码化的状态，只保留最低限度的语义触发信息。它不是为通用模型准备的，而是为高度适配的模型、专用 Runner 或受控后端环境准备的。

特点：

* Token 占用最低
* 兼容性最低
* 适配风险最高
* 几乎不适合人工直接阅读
* 强依赖语义哨兵符、预置规则、上下文缓存和后端解析器
* 一旦适配成功，调用成本最低

适用场景：

* 模型已经深度适配 Prompt Constellation
* 后端拥有完整模板库、AST、IR 和执行上下文
* 输出只需要触发已知语义，不需要完整解释
* 极端 Token 预算限制
* 内部系统调用、批处理、Agent 间通信或专用推理链路

MIN.MINI 的定位是：

> 最小、最快、最省，但只适合完全受控的适配环境。

---

### NL

`NL` 是自然语言输出模式。

它不追求机器压缩，也不追求严格结构化，而是将 Prompt Constellation 的语义转换成人类更容易理解的自然语言表达。

特点：

* 可读性最高
* 适合说明、文档、审查和人工修改
* 不适合作为稳定的机器输入格式
* Token 占用可高可低，取决于说明密度
* 语义精度低于机器语言等级
* 不建议用于需要强一致性的自动化执行链路

适用场景：

* 给人类解释生成逻辑
* 输出文档说明
* 调试 Prompt Constellation 的设计意图
* 审查模板组合结果
* 将机器语言输出翻译成自然语言版本
* 面向非技术用户展示最终提示词含义

NL 的定位是：

> 给人看，不是给机器稳定解析。

---

## 模块系统

Prompt Constellation 支持模块化提示词结构。

常见模块包括：

* 基础语法模块
* DSL 解释模块
* AST 解释模块
* 安全声明模块
* 任务载荷模块
* 社区解释模块
* 自定义解释模块
* 临时解释模块

模块可以被完整展开，也可以被压缩为导入声明。

示例：

```text
#--> INPUT Basemodule-base.3.26 {%Pev:XXXXXXXX%}
```

模块导入的目标是：

> 不重复传递已经稳定的解释文本，只传递语义引用。

---

## 文件格式

Prompt Constellation 可以输出多种文件或文本产物。

### `.md` 裸全文 [目前仅支持裸全文]

直接展开为 Markdown 文本。
适合不支持专用文件格式的平台，但 Token 占用较高。

---

## 编译流程

典型编译流程如下：

```text
Prompt Template
    ↓
Semantic Analysis
    ↓
DSL / AST Generation
    ↓
Module Resolution
    ↓
Safety Declaration Binding
    ↓
Artifact Level Selection
    ↓
MAX / LITE / SMALL / MIN Output
```

也可以从已有产物逆向：

```text
.pev2 / .pevm2 / .pevmax3
    ↓
Reverse Parser
    ↓
Prompt IR / AST
    ↓
Natural Language Explanation
```

---

## 项目目标

Prompt Constellation 的目标是建立一套提示词编译和分发机制，使复杂提示词可以：

* 被模块化管理
* 被结构化表达
* 被压缩传递
* 被完整展开
* 被逆向审计
* 被版本化维护
* 被不同模型和平台逐步适配

最终目标是让复杂 Prompt 不再依赖人工复制大段文本，而是以稳定、可解析、可验证、可压缩的 AI-facing artifact 形式传递。

---

## 当前状态

本项目处于开源起步阶段。

---

# 它能干什么？

## 一句话介绍

**Prompt Constellation 是一个将提示词模板编译为 AI-facing artifacts 的提示词星座系统，支持从全量 MAX 到极简 MIN 的多级语义打包、压缩、导入和逆向。**

可以做的东西很大。**SPACE / Pev 如果和现在的 AI 体系结合，最直接的结果是：把“AI Agent 写代码”升级成“意图规格驱动的软件生成系统”。** 🌌

现在的 AI 体系已经有不少部件：Codex 这类云端软件工程 agent 会在沙箱里执行任务；Claude Code Agent SDK 提供 agent loop、上下文管理、读文件、跑命令、改代码等能力；GitHub Copilot cloud agent 能研究仓库、制定实现计划、改分支并发起 PR；LangGraph 则提供 durable execution、human-in-the-loop 等 agent 编排基础设施。([OpenAI Developers][1])

SPACE 接上这些东西后，核心价值不是“再造一个 Agent”，而是给它们加一个更高层的 **intent-spec control plane**。

---

# 1. 最直接能做：Pev-native Coding Agent

现在的 coding agent 通常是：

```text
Issue / Prompt
→ Agent 理解
→ 改代码
→ 跑测试
→ 出 diff / PR
```

接入 SPACE 后会变成：

```text
需求
→ Pev.mini
→ Pev.hard
→ AI-facing AST
→ 代码生成
→ 沙箱执行
→ Pevrunner 验证
→ Pev diff + Code diff
→ PR
```

这会解决一个关键问题：

```text
现在的 Agent 改代码后，只能解释“改了什么”；
SPACE 可以解释“为什么这样改”，因为 Pev diff 是业务原因源。
```

产品形态可以叫：

```text
Pev-native Software Engineer
```

它不是让 Agent 直接读代码猜需求，而是让 Agent 先维护 Pev，再由 Pev 生成代码。

---

# 2. 能做“规格先行”的 AI IDE

现在 IDE 里的 AI 多数是：

```text
补全代码
解释代码
修改代码
生成测试
```

SPACE 接入后，IDE 可以多一层：

```text
代码文件旁边显示 Pev 文件
```

也就是说，开发者看到的不只是：

```text
user_service.py
```

还会看到：

```text
user_service.pev.mini
user_service.pev.hard
user_service.ai_ast
user_service.pevh
```

开发体验会变成：

```text
改业务规则 → 改 Pev
改实现细节 → 改 Code
重构架构 → 改 Pev + 重新生成 Code
```

最有价值的功能是：

```text
Code diff ←→ Pev diff 对齐
```

比如 PR 页面里显示：

```text
代码变动：
- 增加订单状态 payment_verifying
- 修改 retry policy
- 增加幂等键构造

Pev 业务原因：
- 支付确认从同步变为异步
- 第三方支付网关存在延迟回调
- 重复回调必须被幂等处理
```

这比现在普通 AI code review 强很多。

---

# 3. 能做多模型代码搜索系统

现在多 Agent 并发通常是：

```text
让多个 Agent 各自尝试修 bug / 写方案
```

SPACE 可以把它规范化：

```text
Plan-1 → Pev-1 → Code-1 → Score-1 → Sandbox-1
Plan-2 → Pev-2 → Code-2 → Score-2 → Sandbox-2
Plan-3 → Pev-3 → Code-3 → Score-3 → Sandbox-3
```

最后注册表选：

```text
可用
被否
需修复
可合并
可继续展开
```

这可以做成：

```text
AI Code Search Farm
```

它不是搜索网页，而是搜索实现空间。

适合：

```text
算法实现
性能优化
复杂重构
跨语言迁移
多架构方案比较
高可靠模块生成
```

现在 Codex、Claude Code、Copilot agent 都已经在往并发 agent、沙箱执行、分支 PR 方向走；SPACE 的差异是把这些并发尝试全部挂到 Pev 规格和注册表上，而不是散落成多个 agent 会话。([OpenAI Developers][1])

---

# 4. 能做“自增式项目生成”

这是最强的部分。

普通 AI 生成代码是：

```text
需求 → 代码
```

SPACE 是：

```text
需求 → Pev → 代码 → 运行结果 → 新 Pev → 新代码 → 新结果
```

也就是说，项目可以围绕注册表持续长大：

```text
新增功能
→ 新 Pevh
→ 新 Pev Prompt
→ 新代码
→ 新沙箱验证
→ 合并进项目
```

每一次循环都留下：

```text
Pev.mini
Pev.hard
AI-facing AST
Pevh/s
Pevh/t
代码产物
评分记录
沙箱日志
Pevrunner 验证结果
```

这会形成一个 **项目生长账本**。

很适合：

```text
中后台系统
规则引擎
低代码平台
企业内部工具
多语言 SDK
业务流程自动化系统
测试用例生成系统
```

它不是“一次生成一个 app”，而是“项目按规格图不断生长”。

---

# 5. 能做跨语言重写/迁移

这个非常实际。

传统跨语言重写是：

```text
读 Python 旧代码
→ 猜业务逻辑
→ 写 Rust / Go / Java
```

SPACE 可以变成：

```text
Pev
→ Python implementation
→ Rust implementation
→ Go implementation
→ TypeScript implementation
```

Pev 作为逻辑源，代码只是 target artifact。

这可以做：

```text
Legacy modernization
多语言 SDK 生成
Python → Rust 性能迁移
Node → Go 服务迁移
Java 单体 → 微服务拆分
```

最强的是：
不是从旧代码翻译新代码，而是从 Pev 重新生成新语言实现。

---

# 6. 能做企业级“合规代码生成”

Pev 本来就能量化：

```text
输入信任边界
字段敏感级别
日志级别
持久化级别
权限前置条件
状态迁移白名单
幂等键规则
重试安全规则
失败补偿规则
事务隔离级别
并发冲突策略
外部副作用规则
审计留痕要求
脱敏规则
数据保留规则
错误暴露策略
```

这些可以直接接入企业合规体系。

产品形态：

```text
Compliance-aware Code Generation
```

例如企业可以声明：

```text
所有用户手机号字段：
- 不允许明文日志
- 不允许跨境持久化
- 对外响应必须脱敏
- 只能在授权上下文读取
```

然后 Pevcompiler 把这些规则注入所有相关 Pevh。

这比让开发者记规范更可靠。

---

# 7. 能做“AI 生成代码的审计账本”

现在 AI 代码生成最大的治理问题之一是：

```text
谁让 AI 改了什么？
为什么改？
依据是什么？
有没有验证？
```

SPACE 可以天然记录：

```text
需求来源
方案注册表
Pev diff
Code diff
评分记录
沙箱运行日志
Pevrunner 验证结果
最终合并状态
```

这会形成：

```text
AI Software Provenance Ledger
```

用途：

```text
企业审计
金融/医疗/政企软件变更记录
AI 生成代码责任追踪
合规复盘
安全事故回溯
```

这正好补现在 agent 系统最缺的一层：**产物来源与意图来源绑定**。

---

# 8. 能做模型厂商的 Pev 原生通道

如果模型厂商支持 SPACE/Pev，就不需要用户把 MAX3 全塞进窗口。

可以变成：

```text
上传 Pev package
→ 平台识别 Pevhead
→ 后台解析 MAX1/2/3
→ 缓存语法解释
→ 抽取 task-local subset
→ 注入模型
→ 输出代码
→ 返回 runner-ready artifact
```

这对厂商有两个好处：

```text
降低 token 成本
提高合规可见性
```

厂商还可以要求：

```text
root intent manifest
Pevh lineage
Pevs custody proof
最终 artifact 审查
```

这样 SPACE 既能做生成，又能做治理。

---

# 9. 能做 Agent 之上的控制平面

现在 LangGraph 这种框架解决的是：

```text
Agent 节点怎么连？
状态怎么持久化？
什么时候 human-in-the-loop？
```

SPACE 可以坐在它上面：

```text
LangGraph / Agent Runtime = 控制执行流程
SPACE = 控制意图规格、代码产物、验证账本
```

两者关系可以是：

```text
LangGraph 管“流程图”
SPACE 管“语义图 + 规格图 + 代码图 + 验证图”
```

所以不是替代 workflow，而是升维：

```text
Workflow：节点编排
Agent：动作执行
SPACE：意图—语用—代码—验证闭环
```

---

# 10. 能做“AI 软件工厂”

最终形态可以是：

```text
输入：自然语言需求 / PRD / issue / 业务规则
输出：Pev 规格 + 代码 + 测试 + 审计记录 + 可运行模块
```

流水线：

```text
需求输入
→ 方案生成
→ Pev.mini 生成
→ Pev.hard 编译
→ AI-facing AST
→ 多模型代码生成
→ 评分
→ PevExecutor 沙箱执行
→ Pevrunner 验证
→ 注册表更新
→ PR / 部署 / 回滚
```

这就是：

```text
AI Software Factory
```

适合做：

```text
企业内部系统自动生成
SaaS 模块工厂
多租户业务规则引擎
插件市场
行业模板生成器
自动化 SDK 工厂
AI-native low-code / no-code
```

---

# 11. 和现有 AI 体系结合后的定位

可以这样定义：

```text
Model = 生成能力
Agent = 行动能力
Workflow = 编排能力
Sandbox = 执行隔离能力
CI = 验证能力
SPACE = 意图规格闭环能力
```

它填的是当前 AI 体系中间缺的一层：

```text
从“模型会写代码”
到“系统能持续维护业务意图与代码的一致性”
```
