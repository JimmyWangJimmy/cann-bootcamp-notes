# 课前准备核对 · CANN 启航营（两天）

> 通知要求 **11 月 17 日前** 完成三项准备；今天是 **2026-11-21（周五）上课日**，两项表单类事项已过期，需今天开场前补办。材料部分已全部下载整理到 `materials/`。

## 一、三项准备状态

| # | 事项 | 状态 | 你要做的事 |
|---|---|---|---|
| 1️⃣ | 课程登记（AtomGit 小程序 `#小程序://AtomGit/kPJCQaH7xiChDoo`） | ⬜ 待你确认 | 微信里搜「AtomGit」小程序 → 打开该活动页完成登记。登记记录只有你自己账号能看，我这边无法代查。**开课前截图留存**，签到/学分一般按登记名单走。 |
| 2️⃣ | 算力申请（金山文档表单 <https://www.kdocs.cn/l/cmRqvRS8S7pR?f=301>） | ⬜ 待你提交 | 表单要填 **AtomGit 账号**（不是手机号/邮箱，是 AtomGit 用户名）。链接跳转 `account.kdocs.cn` 登录后才能填，必须你本人操作。**这是今天最关键的卡点**：两天课程 7 小时实践全在 NPU 上，账号没进名单就拿不到算力，只能干看。填完把账号名发给助教/群里 @ 老师确认已录入。 |
| 3️⃣ | 课前预习（learning hub） | ✅ 已就绪 | 大纲原文 + 7 份讲义已下载到本地，见 `materials/README.md`。今天时间紧，预习按下方「30 分钟速览」走。 |

## 二、开课前 30 分钟：速览路径

按“今天要听懂什么”倒序看，别从头翻 PPT：

1. `materials/ppt/01_...light.pptx` p1–3、p16、p24 — 建立昇腾 / CANN 分层认知与 NPU 并行架构（D1 上午全程靠这个）。
2. `materials/ppt/02_..._1h.pptx` p8、p24 — token 切分与 Qwen 端到端路径（下午实践要用）。
3. `materials/ppt/04_..._2h.pptx` p40「两级存储，两种 Tensor」— 这是 D2 算子开发唯一必须先懂的概念，今天不看，明天上午会卡住。
4. `materials/exercises/add_rms_norm/README.md` — 扫一眼结业作业要交什么（必选作业 = `add_rms_norm` 通过判题）。

## 三、两天议程（浓缩版）

| 时段 | 内容 | 产出 |
|---|---|---|
| D1 上午 | 第 1 讲 生态与架构基础（1h）→ 第 2 讲 基于 CANN 部署推理大模型（1h） | 跑通大模型推理；`qwen3_1.7B` Notebook 01–02 |
| D1 下午 | 第 3 讲 推理优化与最佳实践（2h） | 用 profiling 找瓶颈、做优化 A/B 对比；Notebook 03–05 |
| D2 上午 | 第 4 讲 Ascend C 矢量算子开发（2h） | 亲手写 Add 算子并通过判题 |
| D2 下午 | 第 5 讲 PyTorch 接入 + CANNBot（1h）→ 第 6 讲 作业辅导与展示 | 交必选作业 `add_rms_norm`，冲作业 2/3 加分 |

学习节奏（大纲原话）：**D1 听懂 + 跟练，D2 动手 + 独立**。D1 不追求自己写代码，D2 必须自己敲。

## 四、上课前的环境自检（拿到算力后第一件事）

```bash
# 1. 卡是否可见
npu-smi info

# 2. CANN 环境变量（路径按实际安装位置，通常是 latest 软链）
source /usr/local/Ascend/ascend-toolkit/set_env.sh

# 3. torch_npu 是否可用
python3 -c "import torch, torch_npu; print(torch.__version__, torch_npu.npu.is_available(), torch_npu.npu.device_count())"

# 4. 算子编译工具链
which bishengir-compile || which ccec   # 缺了就是 toolkit 没 source 对
```

四项有一条不过，立刻找助教，不要自己折腾超过 10 分钟——上午的实践是连着后面的作业走的。

## 五、容易踩的坑（提前知道能省半天）

1. **算力按账号发放，不是按机器**。换终端/换容器不丢，但账号没登记就全程没卡。表单里 AtomGit 用户名写错（大小写、下划线）是最常见的返工原因。
2. **Notebook 顺序不能跳**。`qwen3_1.7B` 的 01→06 是基线→剖析→优化的递进，跳着跑后面 `04_npu_optimization` 没有基线数据对比，A/B 结论出不来。
3. **结业作业只认 CANNJudge 判题通过**，本地自测过了不算。判题含性能门禁（不低于参考实现 80%），先保正确性再谈性能。
4. **`op_custom_test.py` 的前置条件**：基线 Qwen3-1.7B 能正常推理、`libascendc_ops.so` 已生成且路径可访问，否则脚本必然报错，别误判成算子写错了。
5. **自定义算子只实现了 BFLOAT16**。做精度替换时要把 `dtype_map` 里的 `float16`、`float32` 删掉，否则会跑出误导性的精度失败。
6. 第 4 讲 PPT 有 54 页、2 小时，是最密的一讲；课上跟不上的部分，课后回看 `tutorials/ascendc_operator_development_light/02_simd_basics_and_vector_operator` 补。

## 六、今天开场就要做的 3 件事（按顺序）

1. 微信小程序完成课程登记 → 截图。
2. 金山文档提交 AtomGit 账号 → 群里 @ 老师/助教确认算力已开。
3. `npu-smi info` 自检通过 → 打开 `qwen3_1.7B` Notebook 01 跟着跑。

> 材料索引与讲义速查表见 [materials/README.md](materials/README.md)。
