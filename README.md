# astrbot_plugin_chengzixi

> 橙子汐全系统算力与工程能力进阶插件 (AstrBot)

## 架构规划
1. **router/**: 双轨算力调度引擎（Fast vs Deep 路由分流）
2. **context_engine/**: SillyTavern 预设编排与 In-Chat @ Depth 动态注入
3. **tools/**: 标准化 Tool Bus（Playwright 网页无头渲染、SSH 运维探针）
4. **memory/**: 状态机与 SkillMemory 反思自愈网络

---

## 实操落地指南：自底向上，敏捷演化

要将这份庞大的架构计划落地，最忌讳一开始就全面铺开。我们需要在 AstrBot 的插件框架下，采用“自底向上，先骨架后血肉”的敏捷工程路径。

以下是梳理的实操落地指南，直接从代码结构与核心组件的开发切入：

### 第一步：搭建工程骨架与基础目录 (Day 1)

作为 AstrBot 的一个高级插件，你需要先建立独立的模块作用域，隔离标准对话与复杂工程任务。

建议的仓库结构：

```text
astrbot_plugin_chengzixi/
├── main.py                 # AstrBot 插件生命周期入口与消息拦截点
├── router/
│   └── dual_track.py       # 算力调度引擎（Fast vs Deep）
├── context_engine/
│   ├── templates/          # 存放类似 SillyTavern 的 [main], [worldInfo] 模板
│   └── renderer.py         # Jinja2 或 Handlebars 渲染核心
├── tools/
│   ├── base.py             # Tool Contract 抽象基类
│   ├── playwright_env.py   # 无头浏览器工具
│   └── ssh_probe.py        # 探针与运维工具
└── memory/
    └── state_manager.py    # 状态机与 Git/JSON 记忆读写
```

### 第二步：实现双轨路由器与算力调度 (Day 2-3)

这是系统的“大脑”。利用 AstrBot 的消息预处理机制（如 `@event_message_type` 或中间件），在请求发往大模型前进行拦截分流。

* **指标打分系统**：在 `dual_track.py` 中写一个轻量级的评估函数，对用户的 prompt 提取关键词、判断上下文长度、是否包含代码块或报错信息。
* **模型级联**：
  * **Fast 轨道**：对于日常对话、简单的正则或文案生成，直接路由给低延迟模型（建议对接 **DeepSeek-V3** API，响应极快且成本极低）。
  * **Deep 轨道**：一旦识别到类似“排查服务器报错”、“重构某个类”、“生成并执行多步测试”的意图，立即切换至深度推理模型（建议对接 **DeepSeek-R1**，利用其内置的深度思考链来生成严谨的执行计划和诊断假设）。

### 第三步：重构上下文渲染引擎 (Day 4-5)

不要再将 prompt 简单地使用字符串拼接，需要复刻类似于酒馆（SillyTavern）的上下文组装逻辑。

* **引入模板引擎**：在 Python 中使用 `Jinja2` 或 `Chevron` (Handlebars 的 Python 实现) 来管理预设。
* **实现 Prompt Order**：
  1. 在 `templates/` 下建立 `system.hbs`、`scenario.hbs`。
  2. 编写 `renderer.py`，接收当前对话状态，按照 `[main] -> [charDescription] -> [scenario] -> [chatHistory] -> [jailbreak]` 的硬性顺序组装最终输入。
* **动态注入 (@ Depth)**：编写一个钩子，允许特定工具（如探针报警后）将报错摘要强制插入到 `[chatHistory]` 倒数第 0 轮或第 1 轮的位置，确保模型下一条回复一定会注意到。

### 第四步：建立标准化 Tool Bus (Day 6-8)

将外部能力抽象为标准接口，避免模型因为工具返回格式不同而产生幻觉。

* **定义基类 (Tool Contract)**：在 `base.py` 中强制规定每个工具必须返回 `{"status": bool, "data": dict, "error_code": str, "rollback_id": str}`。
* **接入核心组件**：
  * 使用 `playwright-python` 封装无头浏览器，只暴露出 `goto(url)`、`extract_dom()` 和 `screenshot()` 接口。
  * 使用 `GitPython` 库封装自迭代能力。规定每次修改代码前，工具自动执行 `git checkout -b auto-fix-<timestamp>`。

### 第五步：闭环测试与记忆沉淀 (Day 9-14)

让系统具备容错与学习能力。

* **状态机拦截**：在执行代码修改或服务器命令后，不要直接回复用户，而是进入一个内部的 `VERIFY` 状态。让 Fast 模型读取执行日志，判断是否成功。
* **自动回滚**：如果验证失败，触发 `git reset --hard` 或配置还原。
* **SkillMemory 写入**：一旦排障成功，提取故障特征和修复方案，序列化写入本地 JSON 或简单的向量库（如 ChromaDB）。下次路由引擎在评估阶段如果匹配到相似报错，直接将这段记忆作为 `[scenario]` 变量注入上下文。
