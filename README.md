# dsh-test-workbench

基于 DeepSeek Harness 的测试工作台 Profile —— 开箱即用的 QA Agent。

## 快速开始

```bash
# 1. 安装 dsh
npm install -g @deepseek-ai/dsh

# 2. 克隆本仓库
git clone https://github.com/dmsobtl/dsh-test-workbench.git
cd dsh-test-workbench

# 3. 安装插件依赖
npm install

# 4. 配置 API Key
export DEEPSEEK_API_KEY=your-deepseek-key
export OPENAI_API_KEY=your-openai-key  # 可选，用于视觉路由

# 5. 启动
dsh --profile . "打开 https://example.com 并验证页面标题"
```

## 包含什么

### 插件

| 插件 | 功能 |
|------|------|
| dsh-tool-app-verify | 浏览器/Electron 操作 + UI 断言 + 视觉回归 |
| dsh-session-analyst | 会话质量分析 + 回归检测 |
| dsh-llm-vision-router | 有图片时自动切多模态模型 |

### 预置 Skill

| Skill | 用途 |
|-------|------|
| test-login-flow | 登录功能完整测试流程 |
| visual-regression | 视觉回归测试标准流程 |
| session-diagnosis | 测试会话效率诊断 |

### 测试人格 Prompt

Agent 默认以 QA 专家身份工作：
- 每步操作后自动验证
- 异常时自动截图取证
- 测试完成自动输出结构化报告

## 使用示例

### 快速验证一个页面

```bash
dsh --profile . "打开 http://localhost:3000，检查首页所有链接是否可点击"
```

### 视觉回归测试

```bash
# 首次：建立基线
dsh --profile . "打开 http://localhost:3000，对首页、登录页、仪表盘分别截图保存基线"

# 后续：对比
dsh --profile . "对比首页、登录页、仪表盘的当前截图与基线，输出回归报告"
```

### 登录流程测试

```bash
dsh --profile . "用 skill test-login-flow 测试 http://localhost:3000/login，测试账号 test@example.com / password123"
```

### 分析上次测试

```bash
dsh --profile . "分析最近一次测试会话的效率，找出可优化的地方"
```

### Web UI 模式（团队共享）

```bash
dsh --profile . web
# 打开 http://localhost:3080，团队成员通过浏览器下发测试任务
```

## 目录结构

```
dsh-test-workbench/
├── config/
│   ├── cordis.yml           # 主配置（插件组合）
│   └── cordis.patch.yml     # 测试人格 prompt
├── skills/                  # 测试知识库
│   ├── test-login-flow.md
│   ├── visual-regression.md
│   └── session-diagnosis.md
├── baselines/               # 截图基线存储（gitignore 或 LFS）
├── package.json
└── README.md
```

## 配置定制

### 切换视觉模型

编辑 `config/cordis.yml` 中的 vision-router：

```yaml
- id: vision-router
  plugin: dsh-llm-vision-router
  config:
    visionProvider: qwen        # 换成通义千问
    visionModel: qwen-vl-plus   # 更便宜
```

### 切换 Electron 模式

```yaml
- id: app-verify
  plugin: dsh-tool-app-verify
  config:
    mode: electron    # 测试桌面应用
```

### 添加新 Skill

在 `skills/` 目录下添加 markdown 文件即可，Agent 会自动加载。

## 成本估算

| 场景 | DeepSeek 消耗 | Vision 消耗 | 总计 |
|------|--------------|-------------|------|
| 5 页面功能验证（无截图） | ~50k tokens | 0 | ~¥0.05 |
| 3 页面视觉回归 | ~30k tokens | ~3 次调用 | ~¥0.1 |
| 完整登录流程测试 | ~80k tokens | ~2 次调用 | ~¥0.1 |

绝大部分消耗在便宜的 DeepSeek 上。

## Roadmap

- [ ] `dsh-tool-test-runner`：结构化测试执行（解析 vitest/jest/pytest）
- [ ] `dsh-tool-env-check`：环境就绪度检查
- [ ] `dsh-tool-api-test`：HTTP 接口测试
- [ ] `dsh-hook-test-report`：turn 结束自动生成报告
- [ ] CI 集成模板（GitHub Actions / AoneCI）
- [ ] 定时巡检 + 钉钉推送

## License

MIT
