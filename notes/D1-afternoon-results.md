# D1 下午+D2 预热实验记录（Qwen3-8B 全链路，A2 / 910B3）

> 环境：CANNLab `cann_camp_d1d2`｜1×910B3（A2 系列，64 GB HBM）｜CANN 9.0.0｜torch 2.8.0+cpu / torch_npu 2.8.0.post4
> 执行方式：SSH 进实例，`jupyter nbconvert --execute` 无头跑，产物落 `/mnt/workspace`（持久化）
> 口径：bf16、`batch_size=1`、`input_truncated_len=256`、`max_new_tokens=64`、`enable_thinking=False`、`exe_mode=eager`（除第 7 章）
> 时间：02 于 16:22、03→04 于 16:22–16:28、07 于 16:30–16:37、08 于 16:37–16:43 完成

## 0. 结论速览

| 章节 | 优化手段 | 吞吐 (tok/s) | 相对基线 | 判定 |
|---|---|---|---|---|
| 02 Baseline | 无 | 6.23 | — | 基线成立 |
| 04 A/B | `enable_npu_rmsnorm` + `enable_add_rmsnorm` | 6.88 → **8.32** | **+20.94%** | 精度 3/3、误差 0.0、相似度 0.984 → 建议开启 |
| 07 图模式 | `ge_graph` | **31.62** | **≈4.5×** vs eager 6.97 | 更优后端 = ge_graph |
| 07 图模式 | `npugraph_ex` | ~20（50.1 ms/token） | 不如 ge_graph | 捕获开销吃掉收益 |
| 08 o_proj 预取 | 16 MiB | 34.61 → **34.98** | **+1.06%** | 可选，收益小 |
| 08 o_proj 预取 | 32 MiB | 34.26 | **−1.00%** | 不采用 |
| 05 量化 | AMCT W8A8 | ✖ 未执行 | — | 环境缺 AMCT（见 §5） |
| 06 自定义量化 matmul | `QmmCustom` | ✖ 未执行 | — | 依赖第 5 章量化权重 |

## 1. 第 2 章 Baseline

```
decode 平均 92.46 ms ｜ 总耗时 6.095 s ｜ 吞吐 6.23 tok/s
生成 38 token ｜ 吐字检查 通过 ｜ 融合状态 否/否
权重 Qwen/Qwen3-8B（16 files，13 s，ModelScope 缓存命中）
```

## 2. 第 3 章 Profiling 热点（Top 6）

| 算子 | 次数 | 总耗时 µs | 占比 |
|---|---|---|---|
| `aclnnMatmulWeightNz_MatMulCommon_MatMulV2` | 435 | 36476.9 | **68.97%** |
| `aclnnMul_MulAiCore_Mul` | 978 | 3610.1 | 6.83% |
| `aclnnPowTensorScalar_PowsAiCore_Pows` | 435 | 2755.3 | 5.21% |
| `aclnnMean_ReduceMeanAiCore_ReduceMean` | 435 | 2295.0 | 4.34% |
| `FusedInferAttentionScore` | 108 | 2209.0 | 4.18% |
| `aclnnInplaceCopy_CastAiCore_Cast` | 873 | 1425.8 | 2.70% |

读法：
- **MatMul 占 69%**（1.7B 只有 46%）→ 模型越大，矩阵乘越是绝对主项，这也是为什么任何只优化小算子链路的手段收益都被压；
- `Mul + Pow + Mean` 合计 16.4%，次数 435 = 层数 → RMSNorm 被拆成小算子链的直接证据（第 4 章融合的对象）；
- 对比 1.7B：那边这类归一化链路占 20.7%，这边只有 16.4% → **同样的融合，8B 的收益只有 1.7B 的一半（20.9% vs 43.4%）**，这就是 Amdahl 定律的现场版。

## 3. 第 4 章 A/B（Dense RMSNorm 融合）

| 组 | 总耗时 s | 吞吐 tok/s | 融合 | 精度 |
|---|---|---|---|---|
| baseline | 5.523 | 6.880 | 否/否 | — |
| optimized | **4.567** | **8.320** | 是/是 | RMSNorm 3/3、Add+RMSNorm 3/3，最大误差 **0.0**，阈值 0.01 |

- 吞吐提升 **+20.94%**，decode 时间下降约 17%；
- 文本相似度 **0.984**（唯一差异：`each key vector` → `the key vectors`），判定达标；
- notebook 结论：**建议开启**。

## 4. 第 7 章 图模式（Eager / GE Graph / NPU Graph EX）

| 模式 | 吞吐 tok/s | 平均 token 时延 | 正式推理耗时 s |
|---|---|---|---|
| eager | 6.97 | 143.5 ms | 5.454 |
| **ge_graph** | **31.62** | **31.6 ms** | — |
| npugraph_ex | ~20 | 50.1 ms | — |

- 三组「除 `exe_mode` 外关键字段一致」已校验，口径干净；
- GE/NPU Graph 都记录了 `compile_request_seen=true`、`graph_mode_verified=true`（证据 JSON 在 `benchmark_graph_mode_*/metrics/`）；
- 三种模式**生成文本完全一致**；
- notebook 结论：**更优后端 = ge_graph，部署建议 `adopt_ge_graph`**；
- 为什么 ge_graph 赢：decode 35 层每层都要 Host 下发一堆小算子，图模式把下发折叠成一次重放；`npugraph_ex` 的 aclGraph 捕获 + 重放的开销没盖住收益。

## 5. 第 8 章 GE 图下 o_proj 预取

| 组 | 吞吐 tok/s | decode s | 相对 off |
|---|---|---|---|
| off | 34.610 | 1.0282 | — |
| **16 MiB** | **34.976** | **1.0162** | **+1.06%**（decode −1.17%） |
| 32 MiB | 34.263 | 1.0221 | −1.00%（decode −0.59%） |

- 三组都固定「GE 图模式 + NPU RMSNorm + Add+RMSNorm」；Prefetch 依赖 `value_cache_after_scatter`，与 Fused Attention 并行搬 `self_attn.o_proj` 权重进 L2；
- 16 MiB 覆盖 32 MiB o_proj 的一半即拿到全部收益，**32 MiB 反而轻微负收益**（大概率是 L2 争用）；
- 输出相似度 3 组都是 **1.0**，数值无损；
- 结论：**32 MiB 不采用；16 MiB 可作可选优化，收益 <2%，别当主要卖点**。

## 6. 05 / 06 为什么没跑（可直接写进作业说明）

```
python3 -c "import amct_pytorch"   → ModuleNotFoundError
pip download amct_pytorch          → ERROR: No matching distribution found（repo.huaweicloud.com 镜像无此包）
find / -iname "*amct*"             → 无任何文件或离线 wheel
```
- AMCT（Advanced Model Compression Toolkit）是 CANN 单独发放的组件，不在启航营镜像里，也不是 PyPI 公共包；
- 第 6 章 `QmmCustom` 需要第 5 章导出的 W8A8 量化权重，因此连带阻塞；
- 解法：找助教要 `amct_pytorch-*.whl` 离线包 → `pip install` 后 05 可跑；或直接拿到老师已量化好的权重目录，跳过 05 只跑 06。

## 7. 产物位置（`/mnt/workspace` 持久化）

```
Sources/model_inference_optimization/qwen3_8b/run_outputs/
├── baseline_demo/metrics/baseline_demo_summary.json
├── baseline_profile/metrics/baseline_profile_summary.json
├── profiling_analysis_summary.json                 ← Top10 热点
├── benchmark_baseline|benchmark_add_rmsnorm/metrics/…
├── optimization_comparison.json                    ← +20.94%
├── benchmark_graph_mode_eager|_ge|_npugraph_ex/metrics/…
├── graph_backend_comparison.json                   ← ge_graph 最优
├── benchmark_ge_fused_o_proj_off|_16m|_32m/metrics/…
└── ge_fused_o_proj_prefetch_comparison.json        ← 16 MiB +1.06% / 32 MiB −1.00%

…/tutorials/llm_inference/qwen3_8b/0{2,3,4,7,8}_*.executed.ipynb   ← 带输出的 notebook 副本
…/models/qwen/res/20261121/qwen3_8b_baseline_profile/prof/{prefill,decode}/   ← profiler 原始产物
```

## 8. 图表（`assets/charts/`，可直接放进作业/PPT）

| 图 | 文件 | 说明 |
|---|---|---|
| 1 | `01-step4-ab-fusion.png` | 第 4 章 A/B：6.88 → 8.32 tok/s（+20.9%），标注精度与相似度 |
| 2 | `02-profiling-top-ops.png` | Profiling Top 8，橙色 MatMul 69%、浅蓝 RMSNorm 小算子链 |
| 3 | `03-step7-graph-modes.png` | 三种执行模式：ge_graph 31.6 tok/s（比 eager 快 354%） |
| 4 | `04-step8-prefetch.png` | o_proj 预取三组：16 MiB +1.06%、32 MiB −1.00% |
| 5 | `05-17b-vs-8b-amdahl.png` | 1.7B vs 8B：同样的融合，收益 43.4% vs 20.9%，右图用算子占比解释 Amdahl |

数据源：`run_outputs/*/metrics/*.json` 与 `optimization_comparison.json` / `graph_backend_comparison.json` / `ge_fused_o_proj_prefetch_comparison.json`，出图脚本逻辑（matplotlib，dpi=150）随图记录在本节。

## 8. 答辩/作业可直接用的一段话

> 在单卡 910B3（Atlas A2，64 GB HBM）上对 Qwen3-8B 做 bf16 离线推理优化。Profiling 显示 MatMul 占 69%，而 RMSNorm 被拆成 `Mul/Pow/Mean` 小算子链、合计 16.4%。仅打开 `enable_npu_rmsnorm` 与 `enable_add_rmsnorm` 两个融合开关、不改模型代码，吞吐从 6.88 提升到 8.32 tok/s（**+20.9%**），算子级精度误差 0.0、文本相似度 0.984。进一步切换到 GE 图模式，把 Host 下发折叠成图重放，吞吐升到 **31.6 tok/s（≈4.5× eager）**，优于 aclGraph（npugraph_ex，50.1 ms/token）。在 GE 图模式下追加 `o_proj` 权重预取（16 MiB，依赖 `value_cache_after_scatter` 与 Fused Attention 并行）仅再得 **+1.06%**，32 MiB 因 L2 争用反而 −1.0%，故不采用。三个优化叠加后，端到端 decode 时延从 143.5 ms/token 降到 31.6 ms/token。
