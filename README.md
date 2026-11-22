# CANN 启航营 · 社区精读笔记

> 把昇腾 CANN 两天启航营的 7 讲 PPT，重写成**能读下去的**单文件网页讲义，每份都带一个能就地答疑的 AI 助教。

<p>
  <a href="https://jimmywangjimmy.github.io/cann-bootcamp-notes/"><img alt="Online" src="https://img.shields.io/badge/在线阅读-GitHub%20Pages-2ea44f.svg"></a>
  <img alt="License" src="https://img.shields.io/badge/license-MIT-blue.svg">
  <img alt="Pages" src="https://img.shields.io/badge/docs-4%20volumes-brightgreen.svg">
  <img alt="Offline" src="https://img.shields.io/badge/offline-100%25-success.svg">
  <img alt="Deps" src="https://img.shields.io/badge/dependencies-zero-lightgrey.svg">
</p>

---

## 这是什么

官方讲义是 PPT —— 信息密度高、但**不适合自读**：一页十几个要点，公式没有推导，代码没有逐行解释，概念之间没有过渡。

这个项目把 7 讲 PPT（共 200+ 页）重写成 4 份网页讲义，每份：

- **分层阅读**：每章开头「入门版」三条要点 →「深入浅出」动画示意 → 精读（公式推导、代码逐行）
- **公式可查**：编号连续（(1) 到 (63)），可以交叉引用
- **PPT 原页保留**：关键页以图片内联，配「该看哪里」的读图提示
- **可交互**：拖动滑块算 Tiling、切按钮看数据流、点播放看流水线动画
- **能答疑**：划词、点段落旁的 `?`、拖悬浮球到任意位置，就地提问

全部是**单文件 HTML**，双击即开，不联网也能读（除了 AI 助教需要 API key）。

> **在线读**：<https://jimmywangjimmy.github.io/cann-bootcamp-notes/>
> （含 AI 助教；本地打开则是完全离线版）

---

## 四份讲义

| 讲义 | 对应讲次 | 内容 | 规模 |
|---|---|---|---|
| [**D1 上午**](docs/d1-morning.html) | 第 1–2 讲 | 昇腾 AI 产业生态、CANN 架构、大模型部署与推理 | 9 章 · 42 公式 · 18 张 PPT 页 |
| [**D1 下午**](docs/d1-afternoon.html) | 第 3 讲 | 推理优化：融合算子、图模式、KV Cache、多卡并行 | 6 章 · 15 公式 · 13 张 PPT 页 |
| [**D2 上午**](docs/d2-morning.html) | 第 4 讲 | Ascend C 矢量算子开发：编程模型、C API、Tensor API | 6 章 · 8 公式 · 16 张 PPT 页 |
| [**D2 下午**](docs/d2-afternoon.html) | 第 5–6 讲 | 算子接入 PyTorch、CANNBot 智能体、结业作业 | 6 章 · 1 公式 · 13 张 PPT 页 |

还有一份[**总入口页**](docs/index.html)，把四份串成一条学习路径。

### 每份讲义里有什么

```
§0 导读          这一讲在整个课程里的位置、和前后讲的关系
§1..§N 正文章节   每章 = 入门版卡片 + 动态示意 + 精读（公式/代码/PPT 原页）
末章 小结与实践   一句话口诀、上机题、和结业作业的衔接
```

---

## 快速开始

### 只是读

```bash
git clone https://github.com/JimmyWangJimmy/cann-bootcamp-notes.git
cd cann-bootcamp-notes
open docs/index.html
```

`file://` 直接打开就能读。**但 AI 助教和语音输入需要本地服务**（浏览器对 `file://` 有跨域限制）：

```bash
python3 -m http.server 8000
# 然后访问 http://localhost:8000/docs/
```

### 启用 AI 助教

讲义内置了 DeepSeek 的调用逻辑，你只需要填自己的 key：

1. 打开任意一份讲义
2. 点右下角悬浮球 → 输入 API key（只存在浏览器 localStorage，不上传）
3. 之后可以：划词提问、点段落旁 `?` 就地解释、拖悬浮球到任意位置问「这一段」

> key 申请：<https://platform.deepseek.com/api_keys>
> 一次提问约 2500 token（含上下文），成本约半分钱。

详细实现说明见 [tools/ai-tutor.md](tools/ai-tutor.md)。

---

## 课程原始资料

`materials/` 下有大纲与实践骨架；PPT 原件体积较大（共 38 MB），未纳入 Git，请从官方渠道获取后放入 `materials/ppt/`：

| 文件 | 讲次 | 页数 | 大小 |
|---|---|---|---|
| `01_artificial_intelligence_basics_light.pptx` | 第 1 讲 | 41 | 7.5 MB |
| `02_llm_deployment_and_inference_with_cann_1h.pptx` | 第 2 讲 | 41 | 17 MB |
| `03_llm_training_and_inference_optimization_overview_with_cann_2h.pptx` | 第 3 讲 | 29 | 4.1 MB |
| `04_a2a3_ascend_c_simd_vector_operator_development_2h.pptx` | 第 4 讲 | 54 | 5.3 MB |
| `05_a2a3_ascend_c_operator_pytorch_single_operator_call_0.5h.pptx` | 第 5 讲（上） | 29 | 2.8 MB |
| `06_a2a3_two_day_bootcamp_final_project.pptx` | 第 6 讲 | 5 | 1.6 MB |

> 讲义里已经内联了 **60 张关键页**（`assets/slides/` 也有一份独立 WebP），所以**不下载 PPT 也能完整读**。
>
> 需要原件时：<https://gitcode.com/cann/> 下的课程仓库；第 5 讲（下）CANNBot 的 PPT（129 MB）见 <https://gitcode.com/cann/cannbot-skills>。

`materials/exercises/add_rms_norm/` 是结业作业的工程骨架（kernel、torch binding、测试）。

---

## 官方资源（GitCode / 昇腾社区）

| 资源 | 链接 | 用途 |
|---|---|---|
| Ascend C API 与样例 | <https://gitcode.com/cann/asc-devkit> | API 源码、使用示例、参考实现 |
| Ascend C 编程指南 | <https://asc.gitcode.com> | 编程模型、API 用法与最佳实践 |
| 算子领域样例仓库 | <https://gitcode.com/cann/cann-samples/tree/master/Samples/> | 各领域算子实现样例 |
| CANN Learning Hub | <https://gitcode.com/cann/cann-learning-hub/> | 全栈教程与 Notebook 练习 |
| CANNBot Skills | <https://gitcode.com/cann/cannbot-skills> | 算子开发智能体 |
| 教材《Ascend C 异构并行程序设计》 | <https://gitcode.com/HIT1920/AscendCBook> | 系统教材 |
| 硬件架构可视化 | <https://cann-vision.gitcode.com/> | 在线拆解 A2/A3 架构 |
| CANN 社区主页 | <https://gitcode.com/cann/> | 全部开源仓库 |
| 昇腾社区 | <https://www.hiascend.com/> | 文档、论坛、课程 |
| CANN 开发者论坛 | <https://bbs.huaweicloud.com/forum/forum-1109-1.html> | 问题求助、技术交流 |

---

## 目录结构

```
cann-bootcamp-notes/
├── docs/                      四份讲义 + 总入口（单文件 HTML，可离线）
│   ├── index.html
│   ├── d1-morning.html        1.8 MB
│   ├── d1-afternoon.html      1.7 MB
│   ├── d2-morning.html        1.9 MB
│   └── d2-afternoon.html      1.4 MB
├── materials/
│   ├── syllabus_two_days_bootcamp.md
│   ├── ppt/                   官方 PPT（6 份）
│   └── exercises/             add_rms_norm 工程骨架
├── assets/
│   └── slides/                从 PPT 导出的关键页（WebP）
├── notes/
│   ├── PREP_CHECKLIST.md      课前准备核对
│   ├── TRAINING_RUNBOOK.md    两天实训操作手册（CANNLab / 本地分工）
│   ├── D1-afternoon-results.md  Qwen3-8B 全链路实验结果
│   └── REPO_ONBOARD.md        本地仓库 fork / LFS / SSH 落地记录
└── tools/
    └── ai-tutor.md            AI 助教的架构与配置说明
```

---

## 技术实现

**为什么是单文件 HTML？**

- 拿到就能用：不需要 npm install、不需要构建、不需要服务器
- 离线可读：KaTeX（含 20 个字体）、PPT 图（base64 WebP）全部内联
- 好分享：一个文件发微信/邮件都行

**里面用了什么**

| 组件 | 做法 |
|---|---|
| 公式 | KaTeX 0.16.11 全内联（CSS + JS + 字体 base64），约 640 KB |
| 公式写法 | 正文用 `<span data-tex="...">`（加载时一次性渲染）；AI 回答用 `$...$`（流式解析） |
| PPT 原页 | Keynote 导出 → WebP（quality 88）→ base64 内联 |
| 版式 | GitBook 风格：左侧章节树 + 中间正文 + 右侧目录，深浅双主题 |
| 交互 | 原生 JS，无框架。Pointer Events 拖动、MediaRecorder 录音、手写 WAV 编码 |
| 响应式 | ≥781px 三栏 / 781–1000px 两栏 / <780px 抽屉 |

**AI 助教的上下文设计**（这是最花时间的一部分）

一次提问会带上五层上下文，让模型知道「你在读哪里」：

1. 全文大纲（章节结构）
2. 当前位置（精确到 h3）
3. 当前小节正文（约 3000 字）
4. 相邻小节的首尾（各约 400 字）
5. 你选中的片段

累计约 2500 token。另外每份讲义内置一张「约定数字表」，把关键数值（8192/256/1、I\*≈200、1.21 GB 等）写进 system prompt，防止模型自己编数字。

细节见 [tools/ai-tutor.md](tools/ai-tutor.md)。

---

## 这几份讲义是怎么做出来的

1. 用 `python-pptx` 提取每页文本，通读一遍确定章节骨架
2. Keynote 导出关键页为图片，转 WebP
3. 按「说人话 / 严谨说 / 工程含义」三段式重写正文
4. 给每章的抽象概念配一个**动画**（不是静态图）和一个**可调参数的演示**
5. 公式统一编号，跨讲义连续（D1 上午 (1)–(42)，D1 下午 (43)–(57)，D2 上午 (58)–(62)，D2 下午 (63)）
6. 内联 KaTeX、图片，产出单文件
7. 用 Chrome headless + CDP 验证：公式渲染数、图片加载、标签配对、布局无重叠

---

## 贡献

欢迎提 Issue 或 PR：

- **内容勘误**：公式推导、代码注释、PPT 页码对应关系
- **补充讲义**：第 5 讲下半（CANNBot）目前只有提纲，没有精读版
- **交互改进**：现有的动画/演示都在 `docs/*.html` 的 `<script>` 里，改起来很直接
- **翻译**：英文版（面向国际社区）

提交前请确认：改完的 HTML 仍然是**单文件、离线可用**（不要把外链 CDN 加进来）。

---

## License

代码与笔记文本：[MIT](LICENSE)

`materials/ppt/` 下的 PPT 版权归 **CANN 开源社区 / 华为** 所有，此处仅作学习资料转载，遵循其原始许可。如涉版权问题请联系删除。

---

## 致谢

- 课程与讲义：CANN 开源社区（黄金华、梁冰龙等讲师）
- 公式排版：[KaTeX](https://katex.org/)
- AI 助教：DeepSeek API
- 灵感：两天课程里所有「这一页没讲清楚」的瞬间

---

<p align="center">
  <sub>如果这几份讲义帮你省了几个小时的啃 PPT 时间，欢迎点个 ⭐</sub>
</p>
