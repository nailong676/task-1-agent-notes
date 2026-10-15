# ai-agent-book 学习笔记（第二章 · 上下文工程）

记录阅读 [bojieli/ai-agent-book](https://github.com/bojieli/ai-agent-book/tree/main) 第二章的学习心得，以及本地实验（2-1 完整跑通，另完成 2-2 / 2-3 / 2-6 / 2-7 / 2-9）的部署、运行与排障记录。

| 文件 | 内容 |
| :-- | :-- |
| [学习心得.md](学习心得.md) | 五个学习目标的逐条梳理：上下文工程概念、Prompt vs Context、Context Window 与 Token 限制、上下文选择/组织/管理、为何上下文决定 Agent 能力上限 |
| [实验2-1-部署与运行记录.md](实验2-1-部署与运行记录.md) | Windows + Ollama + qwen3:0.6b 环境搭建全记录，含 502 代理坑、模型迁移 D 盘、GitHub 限速下载等真实排障过程 |

## 环境

- Windows 11（原生，无 NVIDIA GPU → Ollama 路径）
- Python 3.12.15（uv 托管）+ 根目录 `ch2` extra（120 包）
- Ollama 0.40.4 + qwen3:0.6b（522 MB）

## 核心收获

> 上下文工程回答的是一个问题：**怎样以更低成本，向模型提供一个信息充分的 c_t？**

- 无状态 API → 每一轮都在"重新喂养"模型，构造 c_t 的质量决定 Agent 的下限
- 顺序即成本：稳定前缀 + KV Cache 友好布局
- 0.6B 小模型也能正确产出工具调用 JSON——工具调用行为主要由上下文结构驱动
