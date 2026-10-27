# 剩余 ASR 模型：本地中文 CPU 8bit 对比

状态：VERIFIED_COMPLETE_WITH_RECOGNITION_FAILURES；已完成 12/12 个模型×测试集配置。全量尝试8784段，其中原生识别失败1段。

沿用 EPYC 9115、WSL2 Ubuntu 22.04、四线程参数、batch=1、贪心解码、已知中文、模型常驻。每段重置状态，一段预热后全量测一次。无 VAD、热词、额外上下文或文本纠错。

AISHELL 朗读集531段、20位说话人、2701.839秒；RAMC自然对话集933段、43场会话、86位说话人、2700.797秒。本轮结果使用相同清单及音频哈希，未抽样缩减。

## 完整配置的结果

包含此前两款 Q8 结果。CER越低越好。耗时为处理整套约45分钟音频的常驻请求总时间。

| 测试集 | 模型 | CER ↓ | 处理耗时 | RTF ↓ | 识别失败 | 峰值 RSS | 平均 CPU |
|---|---|---:|---:|---:|---:|---:|---:|
| AISHELL 朗读 | Qwen3-ASR-0.6B | 1.84% | 280.5秒 | 0.104 | 0 | 1698 MiB | 390% |
| AISHELL 朗读 | MOSS-Transcribe-Diarize | 2.21% | 2098.8秒 | 0.777 | 0 | 1333 MiB | 384% |
| AISHELL 朗读 | Fun-ASR-Nano | 2.63% | 208.9秒 | 0.077 | 0 | 1561 MiB | 376% |
| AISHELL 朗读 | SenseVoice-Small | 3.48% | 49.4秒 | 0.018 | 0 | 482 MiB | 389% |
| AISHELL 朗读 | GLM-ASR-Nano-2512 | 4.39% | 4124.6秒 | 1.527 | 1 | 2649 MiB | 393% |
| AISHELL 朗读 | Whisper-large-v3-turbo | 8.75% | 5769.4秒 | 2.135 | 0 | 1320 MiB | 396% |
| AISHELL 朗读 | Nemotron-3.5-ASR-0.6B | 12.11% | 136.5秒 | 0.051 | 0 | 1285 MiB | 357% |
| AISHELL 朗读 | MMS-1B-all | 30.87% | 241.2秒 | 0.089 | 0 | 5054 MiB | 386% |
| RAMC 自然对话 | Qwen3-ASR-0.6B | 8.29% | 383.5秒 | 0.142 | 0 | 1704 MiB | 389% |
| RAMC 自然对话 | Fun-ASR-Nano | 8.69% | 315.2秒 | 0.117 | 0 | 1564 MiB | 373% |
| RAMC 自然对话 | MOSS-Transcribe-Diarize | 9.58% | 3642.2秒 | 1.349 | 0 | 1338 MiB | 382% |
| RAMC 自然对话 | GLM-ASR-Nano-2512 | 9.98% | 6512.1秒 | 2.411 | 0 | 2657 MiB | 396% |
| RAMC 自然对话 | SenseVoice-Small | 11.21% | 49.7秒 | 0.018 | 0 | 461 MiB | 374% |
| RAMC 自然对话 | Nemotron-3.5-ASR-0.6B | 18.53% | 154.8秒 | 0.057 | 0 | 1280 MiB | 347% |
| RAMC 自然对话 | Whisper-large-v3-turbo | 20.68% | 10153.7秒 | 3.760 | 0 | 1330 MiB | 396% |
| RAMC 自然对话 | MMS-1B-all | 49.13% | 250.1秒 | 0.093 | 0 | 4476 MiB | 384% |

RTF=处理耗时÷音频时长，1为实时速度。平均CPU约400%代表平均使用约4个逻辑核；参数四线程不是整个进程严格的四核配额。RSS为100ms采样的Python驱动及其原生子进程合计，包含模型加载和预热阶段。

## 模型规模与能力概览

参数量为模型卡标称值，Q8/INT8量化只改变权重表示，不改变参数个数。磁盘大小按下表路径统计本轮推理实际需要的量化权重及必要tokenizer/config文件，单位为十进制GB（1 GB=1,000,000,000 bytes）；不含运行环境、原始浮点副本、转换来源文件和未使用的备份。MMS仅计本次加载的简体中文INT8模型包，不计留在本机但未加载的原始权重。语言/方言特性引用模型卡，未经本轮测试集验证。

| 模型 | HF 地址 | 参数量 | 本轮推理模型包 | 主要特点 |
|---|---|---:|---:|---|
| Qwen3-ASR-0.6B | [https://huggingface.co/Qwen/Qwen3-ASR-0.6B](https://huggingface.co/Qwen/Qwen3-ASR-0.6B) | 0.6B | 1.019 GB | 30种语言与22种中文方言/变体（含粤语、吴语、闽南语）；支持离线和流式，官方流式接口走Qwen-ASR/vLLM工具链；另有独立强制对齐模型可输出时间戳。 |
| Fun-ASR-Nano | [https://huggingface.co/FunAudioLLM/Fun-ASR-Nano-2512-hf](https://huggingface.co/FunAudioLLM/Fun-ASR-Nano-2512-hf) | 0.8B | 1.058 GB | 中文、英语、日语；中文覆盖7类方言群和26种地域口音（吴、粤、闽、客家、赣、湘、晋）；支持热词提示。 |
| SenseVoice-Small | [https://huggingface.co/csukuangfj/sherpa-onnx-sense-voice-zh-en-ja-ko-yue-int8-2025-09-09](https://huggingface.co/csukuangfj/sherpa-onnx-sense-voice-zh-en-ja-ko-yue-int8-2025-09-09) | 234M | 0.237 GB | 普通话、粤语、英语、日语、韩语；除ASR外还可做语种识别、情绪识别和声音事件检测；模型卡未声明具体地域方言数量。 |
| Nemotron-3.5-ASR-0.6B | [https://huggingface.co/nvidia/nemotron-3.5-asr-streaming-0.6b](https://huggingface.co/nvidia/nemotron-3.5-asr-streaming-0.6b) | 600M | 0.751 GB | 40个语言/地区代码分级支持；普通话属广覆盖级，8种代码需领域适配；原生缓存感知流式RNNT，带标点和大小写，未列中文方言支持。 |
| MMS-1B-all | [https://huggingface.co/facebook/mms-1b-all](https://huggingface.co/facebook/mms-1b-all) | 1.0B | 1.027 GB | 官方模型卡列出1162种语言适配器；本测试专门使用简体中文适配器。语种覆盖广，但没有明确列出中文方言覆盖。 |
| MOSS-Transcribe-Diarize | [https://huggingface.co/OpenMOSS-Team/MOSS-Transcribe-Diarize](https://huggingface.co/OpenMOSS-Team/MOSS-Transcribe-Diarize) | 0.9B | 0.987 GB | 原模型卡称支持50+语种、最长90分钟，并联合输出转录、说话人标签、时间戳和声音事件，支持热词；本次GGUF运行说明明确覆盖英文/中文，测评只评分转录文本。 |
| GLM-ASR-Nano-2512 | [https://huggingface.co/zai-org/GLM-ASR-Nano-2512](https://huggingface.co/zai-org/GLM-ASR-Nano-2512) | 1.5B | 2.437 GB | 以普通话和英语为主，模型卡特别强调粤语及其他中文方言、低音量语音识别；没有宣称广泛多语种覆盖。 |
| Whisper-large-v3-turbo | [https://huggingface.co/openai/whisper-large-v3-turbo](https://huggingface.co/openai/whisper-large-v3-turbo) | 809M | 0.874 GB | 多语种识别，模型卡列99种语言；支持语音转写及翻译成英语，未公布具体中文方言清单。 |

## 推理方式与量化覆盖

| ID | 模型 | 推理框架 | INT8/Q8参数比例 |
|---|---|---|---:|
| SV08 | SenseVoice-Small | sherpa-onnx 1.13.8 / INT8 ONNX | 99.61% |
| NT08 | Nemotron-3.5-ASR-0.6B | transcribe.cpp / Q8_0 GGUF | 88.05% |
| MM08 | MMS-1B-all | Transformers 5.17.0 / PyTorch 2.10.0 CPU dynamic INT8 | 98.11% |
| MS08 | MOSS-Transcribe-Diarize | transcribe.cpp / Q8_0 GGUF | 99.42% |
| GL08 | GLM-ASR-Nano-2512 | CrispASR GGML / Q8_0 GGUF，修正特征前端 | 98.43% |
| WT08 | Whisper-large-v3-turbo | whisper.cpp / Q8_0 GGML | 98.95% |

比例按参数个数计算，表示权重存储覆盖。各实现保留卷积、归一化、偏置等部分浮点张量，Q8_0还有块缩放因子；不表示所有权重、激活与计算均为8bit。

- Nemotron：GGUF中约88.05%参数为Q8_0；原生RNNT解码器将权重镜像到F32，48个编码器逐点卷积F16权重在CPU上转F32。本次是该Q8模型文件的混合精度原生路径。使用zh-CN、chunked_limited注意力[56,13]和整段离线接口。
- MMS：先加载官方cmn-script_simplified普通话适配器，再将全部386个Linear权重做PyTorch动态INT8；98.11%参数量化，卷积/归一化等保留F32。简体中文词表4495，逐帧argmax CTC，无语言模型。
- SenseVoice：sherpa-onnx INT8 ONNX，language=zh，use_itn=false，每段新建流。
- MOSS：保留带时间戳、说话人和事件的原始生成。仅按官方元数据去除规则提取转录文字计算CER；原始输出及提取后文字均保存。这里不计算说话人分离或时间戳精度。
- Whisper：原生whisper.cpp的Q8_0 GGML格式，每段独立状态，固定中文、贪心、禁用温度回退；保留原始EOT与token序列，普通文本单独重建，合法空输出计为删除错误。
- GLM：CrispASR底层GGML接口，固定中文提示词，贪心、512个新token上限、每段清空KV。修正社区前端的400点FFT/512点FFT差异、波形补零和最后STFT帧规则；五种确定性信号与官方Transformers 5.17.0特征对照，最大绝对误差3.052e-5。这是带兼容补丁的原生实现，未据此宣称整个模型与官方浮点实现数值等价。

## 与此前 Transformers FP32 比较

此前仅Qwen/Fun两款同时测试了Transformers FP32与原生Q8。下表保留第一次的模型、推理方式、耗时与效果；本轮剩余模型没有对应的本地FP32完整基线，不能把跨模型差异解释为量化损失。

| 测试集 | 模型 | 首轮框架 | 首轮CER | 首轮耗时 | Q8框架 | Q8 CER | Q8耗时 |
|---|---|---|---:|---:|---|---:|---:|
| AISHELL 朗读 | Qwen3-ASR-0.6B | Transformers FP32 | 1.92% | 896.4秒 | 上游 llama.cpp / 编解码器 Q8_0 | 1.84% | 280.5秒 |
| AISHELL 朗读 | Fun-ASR-Nano | Transformers FP32 | 2.71% | 988.0秒 | FunASR llama.cpp / 编解码器 Q8_0 | 2.63% | 208.9秒 |
| RAMC 自然对话 | Qwen3-ASR-0.6B | Transformers FP32 | 8.24% | 1407.3秒 | 上游 llama.cpp / 编解码器 Q8_0 | 8.29% | 383.5秒 |
| RAMC 自然对话 | Fun-ASR-Nano | Transformers FP32 | 8.63% | 1558.2秒 | FunASR llama.cpp / 编解码器 Q8_0 | 8.69% | 315.2秒 |

更早的五款CPU模型还包括混合INT8和FP32、在线与离线路径，实际CPU占用也不同；原始结果保存在本地评测工作区，不混作本轮统一8bit配置排名。

## 与Qwen Q8的配对误差差异

差值=本轮候选CER−Qwen Q8 CER；负数更低。95%区间使用2000次聚类bootstrap，朗读集按20位说话人，自然对话按43场会话抽样。区间不含重复运行、量化版本或硬件变动的不确定性；未做多重比较校正。

| 测试集 | 候选 | CER差值（百分点） | 95%区间（百分点） |
|---|---|---:|---|
| AISHELL 朗读 | SenseVoice-Small | +1.64 | [+1.12, +2.22] |
| AISHELL 朗读 | Nemotron-3.5-ASR-0.6B | +10.26 | [+8.63, +11.82] |
| AISHELL 朗读 | MMS-1B-all | +29.03 | [+27.08, +30.85] |
| AISHELL 朗读 | MOSS-Transcribe-Diarize | +0.37 | [+0.05, +0.71] |
| AISHELL 朗读 | GLM-ASR-Nano-2512 | +2.54 | [+1.56, +3.66] |
| AISHELL 朗读 | Whisper-large-v3-turbo | +6.90 | [+5.70, +8.16] |
| RAMC 自然对话 | SenseVoice-Small | +2.92 | [+2.26, +3.70] |
| RAMC 自然对话 | Nemotron-3.5-ASR-0.6B | +10.24 | [+8.79, +11.69] |
| RAMC 自然对话 | MMS-1B-all | +40.84 | [+38.31, +43.41] |
| RAMC 自然对话 | MOSS-Transcribe-Diarize | +1.29 | [+0.50, +2.30] |
| RAMC 自然对话 | GLM-ASR-Nano-2512 | +1.69 | [+1.02, +2.42] |
| RAMC 自然对话 | Whisper-large-v3-turbo | +12.39 | [+10.61, +14.39] |

## 计时与适用范围

主耗时包含每段新状态、特征提取、编码、解码和结果转换；原生框架还包含本机IPC与缓存后的WAV重读。排除驱动预读/哈希、CER计算、日志写入、一次冷加载和一次预热。逐段另存native_compute_s，因此小的框架边界差异仍存在。

Whisper默认将短音频放入30秒编码窗口；GLM特征前端也使用固定30秒补零画布。本集含大量数秒语段，各段都独立调用；其处理时间反映该调用方式，不能直接外推连续长音频吞吐。只跑一次完整测量，未用最快轮次挑选结果。

CER沿用NFKC、小写、去空白及Unicode标点/符号，不转换数字读法、简繁或同音字。阿拉伯数字、繁体写法、漏掉语气词等都会影响分数。保留原始完整得分，不做参考文本驱动的纠错或解码调参。

RAMC是人工切好的单人非叠话语段，保留停顿、重复与语气词。此测试不能验证连续流式延迟、整场会议、VAD、重叠语音或说话人分离能力。模型和原生实现同时不同，不能据此孤立比较推理框架优劣。

## 失败记录与覆盖核验

Nemotron首轮在第120段因新增的过严每帧符号上限检查停止，未产生成功汇总。对照固定版本NeMo源码后确认：每帧达到10个符号时应前进到下一帧，继续处理全部编码帧；修复检查后从第1段全量重跑，采用remaining-q8v2目录。没有拼接首轮119条结果。达到该每帧上限的重复或错语种输出完整保留并计入CER，全局迭代上限/中止/生成截断仍拒绝成功。

GLM固定512-token预算下若以token_limit结束且未生成EOS，按原生识别失败单独记录；不计入成功转录，不使用部分文本，空假设按删除错误计入CER。该规则不更改提示词、token预算或解码参数。

MOSS首轮朗读集保存365/531段后测试会话停止，没有成功汇总。继续任务时WSL已重新启动，因此保留该未完成目录并排除，使用remaining-q8r2-aishell目录从第1段重新完整测量；未拼接跨会话耗时。

GLM/AISHELL曾在WSL重启前保存486/531段，未生成汇总；保留remaining-q8v2目录并排除。最终纳入的GLM/AISHELL使用remaining-q8v3从第1段重新完成，没有跨会话拼接。

- AISHELL 朗读 Nemotron：5段出现每帧符号上限前进。
- RAMC 自然对话 Nemotron：4段出现每帧符号上限前进。

已独立复算8784条本轮输出，其中1条原生识别失败按空假设计分；并复算此前2928条Q8与2928条HF输出；核对清单/音频/模型/实际使用的原生二进制与动态库SHA256、唯一ID、完成状态或明示的识别失败、以及逐段状态重置。此前172个文件哈希保持一致。未完成/失败/三段smoke均未进入上述完整结果。

本发布包包含汇总结果、完整性核验摘要、量化审计和配对统计，分别见同目录的 [summary.json](summary.json)、[verification.json](verification.json)、[precision-audit.json](precision-audit.json) 与 [paired-vs-qwen.json](paired-vs-qwen.json)。逐句转录文本、原始音频、逐段输出、差异例句及本机二进制/动态库哈希未随本报告公开。

## 来源与复现

[Qwen3-ASR](https://huggingface.co/Qwen/Qwen3-ASR-0.6B) · [Fun-ASR-Nano](https://huggingface.co/FunAudioLLM/Fun-ASR-Nano-2512-hf) · [SenseVoice](https://huggingface.co/FunAudioLLM/SenseVoiceSmall) · [Nemotron](https://huggingface.co/nvidia/nemotron-3.5-asr-streaming-0.6b) · [MMS](https://huggingface.co/facebook/mms-1b-all) · [MOSS](https://huggingface.co/OpenMOSS-Team/MOSS-Transcribe-Diarize) · [GLM](https://huggingface.co/zai-org/GLM-ASR-Nano-2512) · [Whisper](https://huggingface.co/openai/whisper-large-v3-turbo)。

逐模型下载 revision、上游文件哈希、运行时源代码和逐段记录保留在原始本地评测工作区。本次 Git 发布不包含模型权重、音频、逐句转录或本机运行二进制。

原清单中的PaddleOCR-VL为OCR模型；mms-300m-1130-forced-aligner依赖已有转录，均不加入独立ASR测试。Nvidia原版和Handy Nemotron GGUF按同一模型计数；Qwen/Fun已测，保留旧完整结果用于对照。

```bash
cd /path/to/ASR
# 同名结果目录拒绝覆盖；再次测量需使用新tag。
python scripts/remaining_cpu_offline.py \
  --model NT08 --manifest data/aishell1-45min/manifest.jsonl --tag your-new-tag
python scripts/report_remaining_asr.py
```

## Windows 本机补充测量（2026-10-26）

以下结果使用 Intel Core i7-1165G7、Windows x64、llama.cpp b11889，CPU 4 线程，经 HTTP API 串行运行；分别计时，各自 warmup 一条，模型启动和 warmup 不计入。每条请求后采样 RSS。与上方 EPYC/WSL2 主表分开呈现。

| 测试集 | 条数 | 推理耗时 | RTF | CER | 采样峰值 RSS |
|---|---:|---:|---:|---:|---:|
| AISHELL-1 45min | 531 | 1072.570 s | 0.397 | 1.8820% | 1524.2 MiB |
| MagicData-RAMC 45min | 933 | 1396.895 s | 0.5172 | 8.3853% | 1103.9 MiB |

清单 SHA-256 与上方数据集相同。完整口径见 [Windows 本机测试说明](local-qwen3-asr-2026-10-26.md)，原始指标见 [AISHELL JSON](local-qwen3-asr-2026-10-26-aishell.json) 和 [RAMC JSON](local-qwen3-asr-2026-10-26-ramc.json)。本机 CER 与主表接近，速度不能直接比较归因：硬件、系统及 HTTP 计时路径均不同。