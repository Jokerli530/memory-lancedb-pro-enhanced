# memory-lancedb-pro-enhanced

> 基于 [win4r/memory-lancedb-pro](https://github.com/win4r/memory-lancedb-pro) 的增强版，融合了 OpenViking 的核心特性。

## 🎯 特性

### 原生特性（memory-lancedb-pro）
- ✅ Weibull 衰减引擎 — 智能遗忘，噪声自然淘汰
- ✅ Auto-Capture/Recall — 零配置自动学习
- ✅ Hybrid 检索 — 向量 + BM25 + Cross-Encoder
- ✅ 多 Scope 隔离 — global/agent/user/project

### 🆕 新增特性（来自 OpenViking）

| 特性 | 说明 |
|------|------|
| **🎯 Intent 分析** | 检索前用 LLM 分析查询意图，生成 0-3 个 typed queries |
| **📂 VikingFS 目录结构** | 记忆按路径组织：`user/memories/{category}/{subcategory}/` |
| **🔍 层级检索** | 分数传播 + 收敛检测，递归遍历子目录 |
| **🪜 L0/L1/L2 渐进摘要** | 按需加载，显著节省 token |

## 📊 L0/L1/L2 分层

| 层级 | 内容长度 | 用途 |
|------|---------|------|
| **L0** | 50-100 tokens | 一句概括（向量检索入口） |
| **L1** | 200-500 tokens | 结构化摘要 |
| **L2** | 完整内容 | 按需加载 |

## 📂 目录结构

```
user/memories/
├── profile/          ← 用户简介
├── preferences/      ← 偏好设置
├── entities/        ← 实体（人物/地点/品牌）
├── events/          ← 事件记录
├── cases/           ← 案例/决策
└── patterns/        ← 模式/规律
```

## 🔍 层级检索算法

```
childScore = 50% × embeddingScore + 50% × parentScore × 0.5^depth
```
- 收敛检测：连续两层分数变化 < 5% 时停止

## 🚀 安装

```bash
# 克隆仓库
git clone https://github.com/Jokerli530/memory-lancedb-pro-enhanced.git

# 安装依赖
cd memory-lancedb-pro-enhanced
npm install

# 配置 OpenClaw
# 将插件路径添加到 openclaw.json 的 plugins.load.paths
```

## ⚙️ 配置

在 `openclaw.json` 中配置：

```json
{
  "plugins": {
    "entries": {
      "memory-lancedb-pro": {
        "enabled": true,
        "config": {
          "embedding": {
            "provider": "openai-compatible",
            "apiKey": "dummy",
            "model": "nomic-embed-text",
            "baseURL": "http://127.0.0.1:11434/v1",
            "dimensions": 768
          },
          "autoCapture": true,
          "autoRecall": true,
          "smartExtraction": true,
          "intentAnalysisEnabled": true,
          "hierarchicalRetrieval": true,
          "sessionMemory": {
            "enabled": true
          }
        }
      }
    }
  }
}
```

## 📝 核心类型

```typescript
// Intent 分析
interface IntentQuery {
  type: 'skill' | 'resource' | 'memory' | 'preference';
  query: string;
}

// 层级检索结果
interface HierarchicalSearchResult extends MemorySearchResult {
  path: string;
  pathDepth: number;
  propagatedScore: number;
  parentScore: number;
}

// 记忆条目（含 L0/L1/L2）
interface MemoryEntry {
  id: string;
  category: 'profile' | 'preferences' | 'entities' | 'events' | 'cases' | 'patterns';
  content: string;
  path: string;           // VikingFS 路径
  l0_summary?: string;    // 50-100 tokens
  l1_summary?: string;    // 200-500 tokens
  l2_content?: string;    // 完整内容
  // ... 其他字段
}
```

## 🧪 测试

```bash
npm test
```

## 📜 License

MIT License

## 🙏 致谢

- [win4r/memory-lancedb-pro](https://github.com/win4r/memory-lancedb-pro) — 原始项目
- [volcengine/OpenViking](https://github.com/volcengine/OpenViking) — 灵感来源

---

*由 Nove AI 🤖 为李伟定制开发*
