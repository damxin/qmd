# QMD Memory Engine 集成指南 (OpenClaw)

QMD 是 OpenClaw 极力推荐的高级内存引擎后端。相比 OpenClaw 内置的简单内存引擎，QMD 提供了**重排序 (Reranking)**、**查询扩展 (Query Expansion)** 以及对**本地外部文档**和**会话历史**的深度索引能力。

本指南将结合 OpenClaw 的官方规范与 QMD 最新的 **云端 (Cloud) 后端支持** 进行说明。

## 1. 核心特性

- **检索质量增强**：利用重排序和向量检索提升召回准确率。
- **扩展索引**：索引项目文档、团队笔记及磁盘上的任何 Markdown 资料。
- **全平台支持**：通过 `QMD_BACKEND=cloud`，Windows/macOS/Linux 用户均可跳过本地 C++ 编译，直接连接云端 API (如 SiliconFlow)。
- **自动降级**：如果 QMD 不可用，OpenClaw 会自动无缝切换回内置的 SQLite 引擎。

## 2. 准备工作

### 安装 QMD
```bash
npm install -g @tobilu/qmd
# 或者
bun install -g @tobilu/qmd
```

### 环境要求
1. **SQLite 扩展支持**：确保系统安装了支持加载扩展的 SQLite（如 macOS 的 `brew install sqlite`）。
2. **PATH 环境**：确保 `qmd` 命令在 OpenClaw Gateway 的执行路径中。

## 3. 在 OpenClaw 中启用 QMD

在 OpenClaw 的 `config.json` 或 Agent 配置中，将内存后端设置为 `qmd`：

```json
{
  "memory": {
    "backend": "qmd",
    "qmd": {
      "paths": [
        { "name": "docs", "path": "~/notes", "pattern": "**/*.md" }
      ],
      "sessions": { "enabled": true }
    }
  }
}
```

## 4. 后端配置：云端 vs 本地

### 方案 A：云端模式 (推荐，无需 GPU)
通过设置环境变量，让 QMD 使用 [SiliconFlow](https://siliconflow.cn/) 或其他 OpenAI 兼容 API。

```env
# 核心开关
QMD_BACKEND=cloud

# API 配置
QMD_CLOUD_API_KEY=sk-xxxx...
QMD_CLOUD_BASE_URL=https://api.siliconflow.cn/v1

# 推荐的高性能 Qwen3 模型配置 (自动适配前缀与维度)
QMD_CLOUD_EMBED_MODEL=Qwen/Qwen3-Embedding-4B
QMD_CLOUD_RERANK_MODEL=Qwen/Qwen3-Reranker-4B
QMD_CLOUD_GENERATE_MODEL=Qwen/Qwen3.5-9B
```

### 方案 B：本地模式 (GGUF)
如果您有本地 GPU 且希望完全隐身运行：
```env
# 默认即为本地模式
QMD_BACKEND=llama.cpp
QMD_EMBED_MODEL=hf:Qwen/Qwen3-Embedding-0.6B-GGUF/Qwen3-Embedding-0.6B-Q8_0.gguf
```

## 5. 高级：多端点独立配置 (New)

QMD 现在支持为不同功能指定不同的 API 基础地址：
- `QMD_CLOUD_EMBED_BASE_URL`: 专门用于向量化的 API 地址。
- `QMD_CLOUD_RERANK_BASE_URL`: 专门用于重排序的 API 地址。
- `QMD_CLOUD_GENERATE_BASE_URL`: 专门用于生成/对话的 API 地址。

> [!TIP]
> **兜底逻辑**：如果您没有配置上述专项地址，QMD 会自动回退并使用 `QMD_CLOUD_BASE_URL` 作为默认地址。这意味着如果您所有的模型都在同一个服务商（如 SiliconFlow）下，您只需配置一个 `QMD_CLOUD_BASE_URL` 即可。

## 6. 常见问题 (Troubleshooting)

### Dimension Mismatch (维度不匹配)
如果你从 1024 维模型 (如 BGE) 切换到了 2560 维模型 (如 Qwen3)，你会收到维度错误警告。
**解决方法**：运行强行嵌入命令重置数据库：
```bash
npm run qmd embed -- --force
```

### 验证状态
在接入 OpenClaw 前，请务必执行以下命令验证 QMD 是否能正常识别您的环境配置：
```bash
npm run qmd status
```

---
更多详情请参考 [OpenClaw 官方文档](https://docs.openclaw.ai/concepts/memory-qmd)。
