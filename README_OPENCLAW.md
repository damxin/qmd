# QMD OpenClaw 集成指南 (云端模式)

QMD (Query Markup Documents) 现在可以作为 **Model Context Protocol (MCP) Server** 无缝集成到 OpenClaw 中。通过使用云端后端（例如 SiliconFlow），你可以摆脱本地硬件限制，快速获得高性能的向量检索和重排序能力。

## 1. 核心优势

- **轻量化部署**：通过设置 `QMD_BACKEND=cloud`，QMD 将绕过本地 Llama.cpp 引擎，大幅降低资源占用，且不再有本地编译报错的烦恼。
- **高性能模型接入**：原生支持 SiliconFlow 提供的 `Qwen/Qwen3-Embedding-4B` (2560维) 和 `Qwen/Qwen3-Reranker-4B` 等最新模型。
- **多端点灵活配置**：允许为 Embedding、Rerank 和 Generation 独立指定不同的提供商或私有部署地址。

## 2. 环境变量配置

在 OpenClaw 的插件配置或系统环境中设置以下变量：

### 基础云端配置
```env
# 核心开关：切换到云端模式
QMD_BACKEND=cloud

# API Key (SiliconFlow 或其他 OpenAI 兼容平台)
QMD_CLOUD_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

# 默认基础 URL
QMD_CLOUD_BASE_URL=https://api.siliconflow.cn/v1
```

### 推荐的模型及多端点配置
你可以根据需要精细化控制各环节使用的模型：

```env
# 向量化模型 (Embedding)
QMD_CLOUD_EMBED_MODEL=Qwen/Qwen3-Embedding-4B
# 可选：独立地址
# QMD_CLOUD_EMBED_BASE_URL=https://api.siliconflow.cn/v1

# 重排序模型 (Rerank)
QMD_CLOUD_RERANK_MODEL=Qwen/Qwen3-Reranker-4B
# 可选：独立地址
# QMD_CLOUD_RERANK_BASE_URL=https://api.siliconflow.cn/v1

# 文本生成模型 (Generation)
QMD_CLOUD_GENERATE_MODEL=Qwen/Qwen3.5-9B
```

## 3. 在 OpenClaw 中配置 MCP Server

在 OpenClaw 的 `mcpServers` 配置部分添加如下内容：

```json
{
  "mcpServers": {
    "qmd-kb": {
      "command": "npx",
      "args": ["tsx", "src/cli/qmd.ts", "mcp"],
      "env": {
        "QMD_BACKEND": "cloud",
        "QMD_CLOUD_API_KEY": "你的_API_KEY",
        "QMD_CLOUD_BASE_URL": "https://api.siliconflow.cn/v1",
        "QMD_CLOUD_EMBED_MODEL": "Qwen/Qwen3-Embedding-4B",
        "QMD_CLOUD_RERANK_MODEL": "Qwen/Qwen3-Reranker-4B"
      }
    }
  }
}
```

## 4. 关键点：向量维度匹配

> [!IMPORTANT]
> QMD 的本地向量库 (`sqlite-vec`) 在初次索引时会锁定维度。
> - 如果你之前使用的是 1024 维模型 (如 BGE)，换成 2560 维模型 (如 Qwen3) 时会报错：`Embedding dimension mismatch`。
> - **解决方法**：运行以下命令强制重置向量表：
>   ```bash
>   npm run qmd embed -- --force
>   ```

## 5. 验证状态

接入前，请先在终端验证配置：

```bash
npm run qmd status
```
如果输出显示 `Mode: cloud` 且 API Key 脱敏处理正确，即可放心集成。
