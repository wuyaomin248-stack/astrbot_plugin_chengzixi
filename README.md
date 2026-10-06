# astrbot_plugin_chengzixi

> 橙子汐全系统算力与工程能力进阶插件 (AstrBot)

## 架构规划
原双轨算力路由已收敛至 AstrBot 原生 SubAgent（deepdiver_07 独立底座）调度，本插件聚焦于上下文工程与自愈工具链：
1. **context_engine/**: SillyTavern 预设编排与 In-Chat @ Depth 动态注入
2. **tools/**: 标准化 Tool Bus（Playwright 网页无头渲染、SSH 运维探针）
3. **memory/**: 状态机与 SkillMemory 反思自愈网络

---

## 实操落地指南：自底向上，敏捷演化

采用“自底向上，先骨架后血肉”的敏捷工程路径，分流脏活完全交由 SubAgent 处理，本体专注三层核心：

### 第一步：工程骨架与模块解耦 (Day 1)
精简后的插件目录结构：
```text
astrbot_plugin_chengzixi/
├── main.py                 # AstrBot 插件生命周期入口与消息拦截点
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

### 第二步：重构上下文渲染引擎 (Day 2-4)
- **引入模板引擎**：使用 `Jinja2` 或 Handlebars 管理预设。
- **实现 Prompt Order**：
  1. `templates/` 下建立 `system.hbs`、`scenario.hbs`。
  2. `renderer.py` 按 `[main] -> [charDescription] -> [scenario] -> [chatHistory] -> [jailbreak]` 顺序组装。
- **动态注入 (@ Depth)**：支持把报警、关键记忆或环境状态钉入 `[chatHistory]` 倒数第 0-1 轮，保证模型生成必见。

### 第三步：标准化 Tool Bus (Day 5-8)
- **基类规范**：强制统一返回 `{"status": bool, "data": dict, "error_code": str, "rollback_id": str}`。
- **接入组件**：Playwright 无头渲染与 Git 事务修改。

### 第四步：闭环测试与 SkillMemory 沉淀 (Day 9-14)
- **状态机拦截**：执行修改后进入内部 `VERIFY` 状态由模型二次校验。
- **自动回滚与记忆提取**：失败自动原子回滚，成功将特征特征沉淀进本地 SkillMemory。
