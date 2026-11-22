# 课程原始资料

本目录收录 CANN 启航营（两天）官方讲义与实践骨架。

## PPT

| 文件 | 讲次 | 页数 | 大小 |
|---|---|---|---|
| `01_artificial_intelligence_basics_light.pptx` | 第 1 讲 · 昇腾 AI 产业生态与 CANN 架构基础 | 41 | 7.5 MB |
| `02_llm_deployment_and_inference_with_cann_1h.pptx` | 第 2 讲 · 基于 CANN 部署和推理大模型 | 41 | 17 MB |
| `03_llm_training_and_inference_optimization_overview_with_cann_2h.pptx` | 第 3 讲 · 大模型推理优化与最佳实践 | 29 | 4.1 MB |
| `04_a2a3_ascend_c_simd_vector_operator_development_2h.pptx` | 第 4 讲 · Ascend C 矢量算子开发 | 54 | 5.3 MB |
| `05_a2a3_ascend_c_operator_pytorch_single_operator_call_0.5h.pptx` | 第 5 讲（上）· 算子接入 PyTorch | 29 | 2.8 MB |
| `06_a2a3_two_day_bootcamp_final_project.pptx` | 第 6 讲 · 结业作业 | 5 | 1.6 MB |

### 缺失的一份

**第 5 讲（下）· CANNBot 智能体**（`05_cannbot_highlights_open_source_community_edition_0.5h.pptx`，35 页）

原文件 **129 MB**，超过 GitHub 单文件 100 MB 限制，无法直接托管。
请从官方仓库获取：<https://gitcode.com/cann/cannbot-skills>

它的内容已在 [docs/d2-afternoon.html](../docs/d2-afternoon.html) 的 §4 做了完整精读。

## 大纲

`syllabus_two_days_bootcamp.md` —— 课程目标、教学安排、结业作业三档、考核方式。

## 实践骨架

`exercises/add_rms_norm/` —— 结业作业的工程结构：

```
add_rms_norm/
├── add_rms_norm_custom_bfloat16/
│   ├── add_rms_norm_custom.asc     # kernel 实现
│   ├── torch_binding.cpp           # PyTorch 接入
│   ├── CMakeLists.txt
│   └── test_add_rms_norm_custom.py # 测试
└── op_custom_test.py
```

## 版权

PPT 版权归 CANN 开源社区 / 华为所有，此处仅作学习资料转载。
如涉版权问题请提 Issue，会立即删除。
