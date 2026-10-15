# 实验 2-10 · 上下文压缩策略对比（运行记录）

> 运行环境：Windows 11 CPU 推理，Ollama + qwen3:0.6b（全本地）
> 任务：Research Agent 调研 "Find current affiliations of all OpenAI co-founders"
> 日期：2026-10-15；每策略独立完整运行一次，均以成功结束

## 汇总对比

| 策略 | 工具调用 | 上下文溢出 | 耗时 | 总 tokens | Prompt/Completion | 压缩效果 |
| :-- | :-- | :-- | :-- | :-- | :-- | :-- |
| no_compression | 1 | 0 | 97.0s | 2,252 | — | 127.7%（+268 字符） |
| individual（逐页摘要拼接） | 1 | 0 | 95.5s | 1,858 | — | 967→434 字符（44.9%） |
| combined（汇总摘要） | 2 | 0 | 150.5s | 2,360 | — | 100.8% |
| context_aware | 0 | 0 | **37.8s** | **1,055** | 427/628 | — |
| citations（感知+引用） | 0 | 0 | 43.4s | 1,177 | 427/750 | — |
| windowed（窗口截断） | 1 | 0 | 67.9s | 2,237 | — | 127.7%（+268 字符） |

## 关键观察

1. **"负压缩"**：mock 搜索结果仅 967 字符。no_compression 与 windowed 的"压缩"输出反而膨胀到 1,235 字符（127.7%，+268）——压缩引入标题与结构化开销，**上下文小于阈值时压缩不划算**。
2. **上下文感知类策略最快**：37~43s，约为其他策略一半。在写入时就裁剪，使后续每轮 prompt 更短；CPU 推理对该效应敏感。
3. **小模型幻觉**：context_aware 与 citations 两次运行中模型**跳过搜索工具**（0 次调用），直接凭参数记忆生成报告，内容含大量错误事实（如"Altman 于 2023 年离开 OpenAI 董事会"）。流程成功 ≠ 内容可靠——压缩策略需保留让模型核实信息的通道。
4. **摘要策略的代价**：combined 额外做汇总摘要（2 次工具调用、150.5s、2,360 tokens 全场最高）——压缩本身消耗推理预算，小模型上"压缩开销 > 压缩收益"。

## 复现命令

```powershell
$env:NO_PROXY = "127.0.0.1,localhost"
cd chapter2/context-compression
# .env: LLM_PROVIDER=ollama / MODEL_NAME=qwen3:0.6b
"n" | python main.py -s no_compression --no-streaming
"n" | python main.py -s individual     --no-streaming
"n" | python main.py -s combined       --no-streaming
"n" | python main.py -s context_aware  --no-streaming
"n" | python main.py -s citations      --no-streaming
"n" | python main.py -s windowed       --no-streaming
```
