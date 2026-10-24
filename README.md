# dsh-test-workbench

> 兼容 dsh 0.2.1-rc.x。bundle 引用（dsh-base / dsh-headless）与 patch 语法已核对 0.2.1 版本；所依赖的业务插件需 v0.1.1+。

基于 DeepSeek Harness 的测试工作台 Profile —— 开箱即用的 QA Agent。

组装了三个插件（UI 验证、会话分析、视觉路由）+ 预置 Skill + QA 测试人格。

## 快速开始

> 前提：三个业务插件 `dsh-tool-app-verify` / `dsh-session-analyst` / `dsh-llm-vision-router`
> 需要先发布到 npm（`npm publish`），否则下面的 `dsh plugin add` 找不到它们。

```bash
# 1. 安装 dsh
npm install -g @deepseek-ai/dsh

# 2. 把本仓库克隆成名为 workbench 的 profile（profile 必须放在 ~/.dsh/profiles/<name>/ 下）
mkdir -p ~/.dsh/profiles
git clone https://github.com/dmsobtl/dsh-test-workbench.git ~/.dsh/profiles/workbench
cd ~/.dsh/profiles/workbench

# 3. 安装插件依赖（把三个插件拉进该 profile 的 node_modules）
pnpm install
# 或逐个：dsh plugin add dsh-tool-app-verify dsh-session-analyst dsh-llm-vision-router

# 4. 配置 API Key
export DEEPSEEK_API_KEY=your-deepseek-key
export OPENAI_API_KEY=your-openai-key   # 可选，用于有截图时切视觉模型

# 5. 启动（注意：profile 是名字，不是路径）
dsh --profile workbench "打开 https://example.com 并验证页面标题"
```

## 包含什么

### 插件

| 插件 | 功能 |
|------|------|
| dsh-tool-app-verify | 浏览器/Electron 操作 + UI 断言 + 视觉回归 |
| dsh-session-analyst | 会话质量分析 + 回归检测 |
| dsh-llm-vision-router | 有图片时自动切多模态模型 |

### 预置 Skill（位于 `.agents/skills/`，DSH 自动发现）

| Skill | 用途 |
|-------|------|
| test-login-flow | 登录功能完整测试流程 |
| visual-regression | 视觉回归测试标准流程 |
| session-diagnosis | 测试会话效率诊断 |

### QA 测试人格

Agent 默认以 QA 专家身份工作（见 `cordis.patch.yml` 中的 `system-prompt` persona）。

## 目录结构

```
dsh-test-workbench/
├── package.json          # dsh.profile.bundles: dsh-base + dsh-headless（一次性任务模式）
├── cordis.patch.yml      # 插入业务插件 + 覆盖 system-prompt 人格
├── pnpm-workspace.yaml   # 供 pnpm 管理 out-of-tree 插件
├── .agents/skills/       # 项目级 skill（DSH 自动加载）
│   ├── test-login-flow.md
│   ├── visual-regression.md
│   └── session-diagnosis.md
├── baselines/            # 截图基线存储（gitignore）
└── README.md
```

## 配置定制

### 切换视觉模型

编辑 `cordis.patch.yml` 中 vision-router 的配置：

```yaml
- insert:
    - id: vision-router
      name: dsh-llm-vision-router
      config:
        visionProvider: qwen        # 换成通义千问
        visionModel: qwen-vl-plus
```

同时把 `llm-pi-ai` 的 `providers` 改成对应 provider（或另配 adapter）。

### 切换 Electron 模式

```yaml
- insert:
    - id: app-verify
      name: dsh-tool-app-verify
      config:
        mode: electron
```

### 添加新 Skill

在 `.agents/skills/` 目录下添加 markdown 文件即可，DSH 会自动加载。

## 使用示例

```bash
# 快速验证一个页面
dsh --profile workbench "打开 http://localhost:3000，检查首页所有链接是否可点击"

# 登录流程测试
dsh --profile workbench "用 skill test-login-flow 测试 http://localhost:3000/login，账号 test@example.com / password123"

# 视觉回归（先建基线，再对比）
dsh --profile workbench "打开 http://localhost:3000，对首页、登录页分别截图保存基线"
dsh --profile workbench "对比首页、登录页的当前截图与基线，输出回归报告"

# 分析上次测试
dsh --profile workbench "分析最近一次测试会话的效率"
```

## 成本估算

| 场景 | DeepSeek 消耗 | Vision 消耗 |
|------|--------------|-------------|
| 5 页面功能验证（无截图） | ~50k tokens | 0 |
| 3 页面视觉回归 | ~30k tokens | ~3 次调用 |
| 完整登录流程测试 | ~80k tokens | ~2 次调用 |

## Roadmap

- [ ] `dsh-tool-test-runner`：结构化测试执行（解析 vitest/jest/pytest）
- [ ] `dsh-tool-env-check`：环境就绪度检查
- [ ] `dsh-tool-api-test`：HTTP 接口测试
- [ ] `dsh-hook-test-report`：turn 结束自动生成报告
- [ ] CI 集成模板（GitHub Actions）
- [ ] 定时巡检 + 钉钉推送

## License

MIT
