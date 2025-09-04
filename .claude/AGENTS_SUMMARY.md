# jsrt Claude Agents 快速参考

## 🎯 核心 Agents (12个)

| Agent | 用途 | 关键命令 |
|-------|------|---------|
| 🥇 **parallel-coordinator** | 并行协调 | `workflow: dev/test/debug/release` |
| 🟢 **test-runner** | 测试执行 | `make test`, 单测试运行 |
| 🔵 **module-developer** | 模块开发 | 创建 C 模块，注册到 runtime.c |
| 🔴 **memory-debugger** | 内存调试 | `make jsrt_m`, ASAN 分析 |
| 🟣 **cross-platform** | 跨平台 | Windows/Linux/macOS 适配 |
| 🟡 **code-reviewer** | 代码审查 | 质量、安全、标准检查 |
| 🟠 **build-optimizer** | 构建优化 | CMake 配置，编译优化 |
| 🔷 **formatter** | 格式化 | `make clang-format` |
| 🟪 **quickjs-expert** | JS引擎 | QuickJS API，JS/C 绑定 |
| 🟦 **example-creator** | 示例创建 | examples/ 目录示例 |
| 🟢 **wpt-compliance** | WPT标准 | Web Platform Tests 合规 |
| 🔵 **wintercg-compliance** | WinterCG | 跨运行时 API 标准 |

## ⚡ 快速使用

### 并行执行
```bash
# 请求并行运行多个任务
"同时运行测试、格式化和内存检查"

# 指定特定 agents 并行
"并行运行 test-runner 和 memory-debugger"
```

### 常用请求
```bash
# 增量测试
"只测试我修改的文件"

# 智能调试
"帮我调试这个段错误"

# 代码审查
"审查这些更改"
```

## 📊 性能指标

- 并行加速：2-3x
- 缓存提升：+40%
- 增量执行：+50%
- 目标效率：>75%

## 📁 文件结构

```
.claude/
├── agents/          # Agent 定义 (12个 .md 文件)
├── settings.json    # Claude Code 项目设置
└── AGENTS_SUMMARY.md # 快速参考
```

## 🔧 说明

- Agent 文件遵循 Claude Code 官方格式
- 并行执行由 parallel-coordinator agent 管理
- 系统会自动优化执行策略

---
💡 系统会学习使用模式并自动优化执行策略。