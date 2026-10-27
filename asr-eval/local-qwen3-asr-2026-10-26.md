# Windows 本机 Qwen3-ASR-0.6B Q8_0 测试（2026-10-26）

这两次测量是对仓库现有 EPYC/WSL2 基准的本机补充，单独记录，不并入原主表排名。

## 结果

| 测试集 | 条数 | 音频时长 | 推理耗时 | RTF | 实时倍速 | CER | 请求错误 / 空结果 | 采样峰值 RSS |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| AISHELL-1 45min | 531 | 2701.839 s | 1072.570 s | 0.397 | 2.52x | 1.8820% | 0 / 0 | 1524.2 MiB |
| MagicData-RAMC 45min | 933 | 2700.797 s | 1396.895 s | 0.5172 | 1.93x | 8.3853% | 0 / 0 | 1103.9 MiB |

## 环境与口径

- CPU：Intel Core i7-1165G7；Windows x64；llama.cpp b11889；CPU 推理 4 线程，单并发槽，GPU offload 0。
- 上下文 4096，temperature 0，top-k 1；经 HTTP multipart API 串行逐条请求。两组分别计时，各自开始前做 1 条 warmup；模型启动和 warmup 不计入测量。两组复用同一个已加载模型服务。
- RTF = 整套推理耗时 ÷ 音频总时长。耗时包含逐条 HTTP 请求往返。CER 使用 NFKC、小写化、移除空白/标点/符号后按字符编辑距离计算。
- RSS 是每条请求后采样到的进程工作集峰值，可能漏过短暂的更高峰值。
- 本机清单 SHA-256 与本仓库 45 分钟测试清单一致：AISHELL `702e57057178c14385722c854adfe4dfe17e1e0f1595f459eacbdfb0c60353e9`；RAMC `f3b7a023b52c3384875b40a9a41431e778cbd58b711037f689fc1064b9e88739`。

## 与仓库主表的 Qwen 结果对照

仓库原基准记录 AISHELL 280.5 s / RTF 0.104 / CER 1.84%，RAMC 383.5 s / RTF 0.142 / CER 8.29%。本机 CER 接近，速度较慢。主基准使用 EPYC 9115 和 WSL2；本机使用 i7-1165G7 和 Windows，且本轮通过 HTTP API 计时，因此速度不可直接归因于单一因素。

原始逐集合指标见 [AISHELL JSON](local-qwen3-asr-2026-10-26-aishell.json) 和 [RAMC JSON](local-qwen3-asr-2026-10-26-ramc.json)。
