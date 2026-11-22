# 两天实训 · 具体怎么用（操作手册）

> 前提：Mac 上没有昇腾 NPU，**代码不能在本地跑**。本地这份仓库负责「读、写、版本管理、交作业留痕」；真正执行在云端 NPU（CANNLab / GitCode notebook），账号就是你登记的 `JimmyWangJimmy`。
> 配套：[REPO_ONBOARD.md](REPO_ONBOARD.md)（仓库/fork/凭证已配好）、[PREP_CHECKLIST.md](PREP_CHECKLIST.md)（课前准备与坑位）。
> 仓库根：`/Users/jimmy/LLM/CANN-Huawei/cann-learning-hub`，工作分支 `learn/day1-day2`（已推送到你的 fork）。

---

## 0. 先记住这张分工表

| 你在哪 | 干什么 | 为什么 |
|---|---|---|
| 本地 macOS（终端 / VS Code / Jupyter 只读） | 读讲义、通读 Notebook、**写算子源码**、记笔记、`git commit` / `push`、发起 PR | 无 NPU，跑不动推理和算子编译，但写代码、看代码、存版本最舒服 |
| 云端 CANNLab（WebIDE 或 VSCode Remote） | 跑 Notebook 01→06、profiling、量化、编译 `.asc`、`op_custom_test.py` | 唯一有 NPU 的地方，模板 `cann_9.0.0_py3.11-A2-arm` |
| GitCode 在线 Notebook（一键链接） | 临时试某一章，不想开环境时 | 免配置，但改完的代码要自己同步回本地 |
| CANNJudge | 交必选作业 `add_rms_norm`，拿判题结果 | 结业只认这里的结果，本地自测过不算 |

---

## 0.5 报名 / 名单 / 算力：课程规范四步 + 现状

| # | 规范环节 | 要求 | 现状（2026-11-21 10:50 核实） |
|---|---|---|---|
| 1 | 课程登记（报名） | 微信里搜 **AtomGit 小程序** → 打开启航营活动页 `#小程序://AtomGit/kPJCQaH7xiChDoo` → 完成报名。签到、学分按报名名单走 | ✅ 你已完成（报名记录只有本人账号能看，建议已截图留存） |
| 2 | 加入算力名单 | 金山文档《清华大学深圳研究生院-华南高校Gitcode账号收集 2》按行填：`姓名 / 学号 / 学院 / AtomGit账号`（D 列格式带 `@`，示例 `@gcw_xxx`）。该表是「可评论」权限，编辑要 WPS 登录 | ✅ 你已提交 |
| 3 | 算力额度到账 | 额度跟着**账号**发放，不看机器。判断方法：CANNLab「我的环境」顶部三张卡 | ✅ 已核实：**A2/A3 NPU 100 卡时**（已用 0）、**CPU 720 核时**、**CANN 积分 270**，均「额度充足」 |
| 4 | 创建 NPU 环境 | 模板 `cann_9.0.0 py3.11-A2-arm-启航营`，规格 `1*NPU 910B3 16vCPUs 32GiB`，EVS 盘 200GiB×2（`/mnt/workspace`、`/home` 持久化） | ✅ 已按此规格建好：实例名 **`cann_camp_d1d2`**，创建时间 2026-11-21 10:48，当前状态「正在完成配置…」 |
| 5 | 课堂作业判题 | 必选作业 `add_rms_norm` 必须过 **CANNJudge**（含性能门禁 ≥ 参考实现 80%） | ⬜ D2 下午交 |
| 6 | 结业作品入库 | PR 到 `cann/cann-launch-camp`，且**先签 CLA** | ⬜ 作品成型后做 |

![CANNLab 额度与「我的环境」](assets/cannlab-env.jpg)

### 6 的规范细节（照抄，评审只认这个目录结构）

```
cann-launch-camp/
└── 2026/
    └── University/                       # 固定写法，不许改
        └── <高校英文大写缩写>/            # 例：UESTC / HIT / SJTU
            └── <期数目录>/               # 英文 + 连字符，例：First-session
                └── Industrial-Task/      # 固定写法，不许改
                    └── <gitcode账号>_课设提交/   # 例：JimmyWangJimmy_课设提交
```

提交流程：进对应年度 → 高校目录 → 期数目录 → `Industrial-Task` → 建 `<gitcode账号>_课设提交/` → 放源码/文档/说明 → **本地验证通过** → 提 PR 等审核合入。目录名禁止中文、空格、特殊字符。
注意：仓里已有目录存在 `<姓名>_<账号>_<题目>` 之类的历史命名，**规范以 README 为准**；你的高校目录如果还没建，按上面层级新建即可。

### 卡时预算与开关机纪律（100 卡时 = 100 个 NPU·小时，1 卡时/小时）

**今天的账**：10:48 建环境开机 → 14:30 关机，**用了 3 卡时**。其中真正在跑东西的时间不到 1 小时（装依赖 40 分钟、连 VSCode 试窗口十几分钟），其余是空置——这就是浪费点。

**剩余排期（按需开机，单次 ≤ 75 分钟）**

| 时段 | 开机后一次做完 | 预计卡时 |
|---|---|---|
| D1 下午 profiling 跟练 | `03_profiling_analysis` 跑完 → `04_npu_optimization` A/B 两组 → 存 metrics | 1.5–2 |
| D2 上午矢量算子 | `02.03`（TPipe/TQue）+ `02.04`（两级存储/同步）+ `02.06`（泛化 Tiling）实操，编译跑通 | 1–1.5 |
| D2 下午作业 | `add_rms_norm` 编译 + 单测 + `op_custom_test.py` + 截图 | 1–1.5 |
| 课后自测（如需要） | 一次开机把没过的格子批量重跑 | ≤ 1.5 |
| **合计** | | **≈ 5–6.5**，加今天 3 → 仍在 100 内（剩 ~90） |

**六条纪律（照做就不会空置）**

1. 听课、看讲义、写代码、`git` 操作 → **不开机**（这些在本地做）。
2. 开机前先写一句「这次要跑哪几格」，跑完清单立刻关机。
3. 一次开机跑完一章，别「跑一格想半天」；用批量执行代替手点（我可以远端 headless 跑整章）。
4. 关机前 30 秒检查：notebook 已 `Cmd+S`、日志/metrics 在 `/mnt/workspace`、`git status` 记一笔。
5. 每次关机后瞄一眼额度卡（`97/100` 这种），差值就是本次成本。
6. 交作业判题走 **CANNJudge**，不吃本地卡时。

### 关机 / 开机怎么走（2026-11-21 14:30 实测）

- 关机：我的环境 → 实例行 **关机** → 弹窗确认「关机实例」→ 状态依次 `正在断开连接…` → `正在关闭系统…` → **已关机**（约 1 分钟）。卡时当场停止（这次停在 **3/100 卡时**）。
- 开机：同一行点 **启动**，等状态回到 **运行中**（首次约 1～2 分钟），再点 **连接 → VSCode** 让平台重新唤起隧道。
- ⚠️ 重新开机后**转发端口会变**，`~/.ssh/config` 里的 `Host cannlab` 需要同步新的 Port（平台会重写 `~/.atomgitdevenv/.ssh/config`，把新端口告我即可）。
- ⚠️ 关机时 VSCode 的远程窗口会断开；notebook 里**没落盘**的编辑可能丢，关机前先 `Cmd+S`。

### 卡时与持久化（100 卡时别浪费）

- **只有开机才计卡时**，跑完立刻关机；关机后实例还在，磁盘数据保留。
- 持久化只有两个目录：`/mnt/workspace`、`/home`；`/tmp`、`/root` 等关机即清。
- 连接方式二选一：**WebIDE**（浏览器）、**VSCode Remote**（本地 IDE 连云端 NPU）。
- 开机报「资源不足」= 高峰，等 10 分钟再开，别反复重建（重建要重新拉 40 GB 镜像）。

## 1. 通道 A：一键在线跑某个 Notebook（最快）

每个 Notebook 在章节 `README.md` 里都有「在线体验」链接，URL 长这样：

```
https://ai.gitcode.com/user/<你的用户名>/notebookcann
  ?repoUrl=https://gitcode.com/<仓库>.git
  &ttl=120&diskSize=40Gi
  &path=<课程目录>
  &scanFilePath=<要打开的 .ipynb>
```

**关键一招**：把 `repoUrl` 换成你自己的 fork，云端拉的就是你改过的代码，练完能直接 `git push`。

D1 直接可用（打开第 2 章 Baseline）：

```
https://ai.gitcode.com/user/JimmyWangJimmy/notebookcann?repoUrl=https://gitcode.com/JimmyWangJimmy/cann-learning-hub.git&ttl=120&diskSize=40Gi&path=tutorials/llm_inference/qwen3_1.7B&scanFilePath=tutorials/llm_inference/qwen3_1.7B/02_baseline_inference.ipynb
```

D2 第 2 章矢量算子（把 `scanFilePath` 换成对应 notebook 即可）：

```
https://ai.gitcode.com/user/JimmyWangJimmy/notebookcann?repoUrl=https://gitcode.com/JimmyWangJimmy/cann-learning-hub.git&ttl=120&diskSize=40Gi&path=tutorials/ascendc_operator_development_light/02_simd_basics_and_vector_operator&scanFilePath=tutorials/ascendc_operator_development_light/02_simd_basics_and_vector_operator/02.03_vector_operator_development_with_tpipe_tque.ipynb
```

> 第 2 章 Notebook 会自己检查并补 Python 依赖；`ttl=120` 是 2 小时，够跑一章。

## 2. 通道 B：CANNLab 云环境（推荐，两天主场）

1. 打开 <https://gitcode.com/cann> → 顶部 **CANNLab** → 左侧 **我的环境** → **新建环境**
2. 按课程 README 的规格填（照抄，别自创）：
   - 模板：`cann_9.0.0_py3.11-A2-arm`
   - 规格：`1*NPU 910B3 16vCPUs 32GiB`
   - 预加载仓库：**选你自己的 fork** `JimmyWangJimmy/cann-learning-hub`，分支 `learn/day1-day2`
   - 磁盘：≥ 40 GiB（Qwen3-1.7B 权重 3.4 GB + 量化产物 + 编译中间件，留少了一定炸）
3. 点 **连接 / 更多 → 选择连接方式**，二选一（同一个实例两种都支持，可共存）：
   - **Web IDE**：浏览器里直接开（Chrome 即可，最多 3 个窗口）。零安装、跟老师同步最快，课上主用这个。
   - **Visual Studio Code（本地）**：走 Remote-SSH 插件连。适合重度写代码；**它的 SSH 通道也是我唯一能稳定代跑的后端**（终端可脚本化：`npu-smi`、`jupyter nbconvert --execute`、`cmake`、`tail` 日志、`git push`）。

   > 选择建议：你自己跟练 → WebIDE；要我替你跑 notebook / 编算子 / 收日志 → 把 VSCode 那页给出的 ssh 命令给我，我写进 `~/.ssh/config` 起个别名 `cannlab` 后直接 `ssh cannlab` 干活。GUI 的 VSCode 我控制不了，只用它的 SSH。

   > **卡时提醒**：实例「运行中」就在计费（100 卡时）。不跑就点 **关机**，要用再点 **启动**；关机后磁盘数据保留。
4. 进环境后第一件事就是自检（不过就找助教，别硬磕超过 10 分钟）：

```bash
npu-smi info
source /usr/local/Ascend/ascend-toolkit/set_env.sh
python3 -c "import torch, torch_npu; print(torch.__version__, torch_npu.npu.is_available(), torch_npu.npu.device_count())"
which bishengir-compile || which ccec
```

> 开机提示「资源不足」= 人多，等 10 分钟再开，不要反复重建环境（会重新拉 40 GB）。

## 2.5 你走本地 VSCode（Remote-SSH）的核对清单

本地这边我已经装好：**Visual Studio Code**（Homebrew cask）+ 扩展 **Remote-SSH**（`ms-vscode-remote.remote-ssh`）。你只需把平台给的连接信息对上一次。

### 一次性配置

1. CANNLab「我的环境」→ 该实例 **连接 / 更多 → 选择连接方式 → Visual Studio Code（本地）**，页面给出 SSH 目标（host、端口、用户名，密码或私钥）。
2. 写进 `~/.ssh/config`（把尖括号换掉）：

```sshconfig
Host cannlab
  HostName <平台给的host>
  Port <平台给的端口>
  User <平台给的用户名>
  # 密码登录就直接 ssh，会提示输入；给的是私钥就加这一行：
  # IdentityFile ~/.ssh/<私钥文件>
  ServerAliveInterval 30
  ServerAliveCountMax 4
```

3. 终端先验一次，别急着开 GUI：
```bash
ssh cannlab 'npu-smi info | head -12; source /usr/local/Ascend/ascend-toolkit/set_env.sh; python3 -c "import torch,torch_npu;print(torch.__version__, torch_npu.npu.is_available())"'
```
4. VSCode → `F1` → **Remote-SSH: Connect to Host…** → `cannlab` → 选 Linux → **File → Open Folder** → `/mnt/workspace`。
5. 连上后**在远程端装**扩展（不是本地端）：`Python`、`Jupyter`。左下角显示 `SSH: cannlab` 才对。

### 常见报错对照

| 报错/现象 | 原因与处理 |
|---|---|
| `Connection timed out` | 实例已关机 → 平台上点「启动」再连 |
| `Permission denied (publickey,password)` | 用户名/端口抄错；或**实例重开过，端口变了** → 回平台重新复制一次连接信息 |
| Remote-SSH 卡在 `Setting up SSH Host` | 本地代理/VPN 干扰（你 `~/.gitconfig` 里有 `127.0.0.1:7897` 代理，系统级代理会拖累 SSH）→ 临时关代理，或给 cannlab 加 `ProxyCommand none` |
| 连上但文件写不进去 | 只有 `/mnt/workspace`、`/home` 持久化；写别处关机即丢，也可能只读 |
| 用一会儿掉线 | 配置里的 `ServerAliveInterval 30` 起保活作用；仍掉就重连，别重启实例 |
| 终端里 `npu-smi` 找不到 | 没进对容器/没 开机，或漏了 `source set_env.sh` |

### 你问我时，给我这三样最省时间
1. 原样报错文本（或截图）；2. 你在哪个目录、跑的哪条命令；3. 想达到的结果。
代码改动我直接在本地仓库 `cann-learning-hub/` 上改并 commit/push；环境里的问题按上面三条定位。


---

## 3. D1：听懂 + 跟练（部署、推理、优化）

**目标产出**：跑通 Qwen3-1.7B 推理，拿到基线指标，做一次优化 A/B。

| 时间 | 做什么 | 具体动作 | 验收 |
|---|---|---|---|
| 09:00–09:30 | 讲义第 1 讲 | 本地打开 [第1讲 PPT](cann-learning-hub/standard_course/00_atomic_courses/02_framework/01_overview/artificial_intelligence_basics_light.pptx)，重点看 p1–3、p16、p24 | 能说出 NPU / CANN / 算子三层关系 |
| 09:30–10:30 | 讲义第 2 讲 + 跟练 | 云端跑 `01_chapter_intro.ipynb` → `02_baseline_inference.ipynb` | `run_outputs/` 出 metrics + 生成文本 |
| 10:30–12:00 | 第 3 讲 + profiling | `03_profiling_analysis.ipynb`（会打开 `model_config.enable_profiler=true`） | `prof/` 里有 `kernel_details`、`op_statistic`，能指出 Top3 耗时算子 |
| 13:30–15:30 | 优化 A/B | `04_npu_optimization.ipynb`（A/B 只切 `npu_rms_norm` / `npu_add_rms_norm` 是否启用） | 同一 case 两组指标，得出「提升了多少」 |
| 15:30–16:30 | 量化 | `05_quantization_qwen3_1_7b.ipynb`（AMCT W8A8，额外磁盘+耗时） | 出量化权重 + 量化后指标 |
| 16:30–17:30 | 自定义 matmul | `06_*.ipynb`（写 `qmm_custom.asc` → 编译 → 接入量化模型） | `libascendc_ops.so` 生成且能加载 |

**手动跑推理的命令**（Notebook 内部也是这条，卡住时对着敲）：

```bash
cd tutorials/llm_inference/qwen3_1.7B/src/inference_scripts/recipe_qwen3_1_7b
bash executor/scripts/infer.sh --model qwen --mode offline --yaml models/qwen/config/qwen3_1_7b_1tp.yaml
```

- YAML 四份分工：`qwen3_1_7b_1tp`（基线）、`..._profile`（profiling）、`..._add_rmsnorm`（融合验证）、`qwen3_1_7b_a8w8_1tp`（量化）。
- 日志在 `models/qwen/res/<日期>/<case>/`，指标同步到 `Sources/model_inference_optimization/qwen3_1.7B/run_outputs/`。
- 权重默认走 ModelScope 下载；已有本地权重就 `export QWEN3_1_7B_MODEL_PATH=/path/to/Qwen3-1.7B` 省一次 3.4 GB 下载。

**D1 别做的事**：不要跳着跑（04 没有 02 的基线就出不了 A/B 结论）；不要改 `src/inference_scripts` 下的框架代码，只改 YAML 和作业要求的文件。

---

## 4. D2：动手 + 独立（Ascend C 算子 + PyTorch 接入 + 交作业）

**目标产出**：自己写完 `add_rms_norm` 并通过 CANNJudge 判题。

### 4.1 上午：矢量算子（第 4 讲，2h）

按文件名数字顺序，一个都不要跳，每一个都在其章节目录里运行：

```
tutorials/ascendc_operator_development_light/02_simd_basics_and_vector_operator/
├── 02.01_chapter_intro.ipynb                              # 章节导引
├── 02.02_simd_programming_and_ascendc_api_overview.ipynb  # SIMD/SPMD + API 分工
├── 02.03_vector_operator_development_with_tpipe_tque.ipynb # TPipe/TQue 写 Add（核心）
├── 02.04_manual_memory_allocation_and_sync_management.ipynb # 两级存储（GM/UB）、手动同步
├── 02.05_kernel_function_practice.ipynb                   # 核函数直调
├── 02.06_generalized_tiling_design.ipynb                  # 泛化 Tiling：不同 shape/dtype/尾块
└── 02.07_vector_operator_practice.ipynb                   # 综合实践（Sigmoid 等）
```

配套资源在同目录：`src/`（源码）、`answer/`（参考答案，**先自己写完再对**）、`images/`。

一句话概念锚点：**GM 是大的慢的，UB 是小的快的；数据必须 CopyIn 到 UB 才算，算完 CopyOut 回 GM**。02.04 没吃透，下午写作业必然卡。

### 4.2 下午上半：PyTorch 单算子调用（第 5 讲 a，0.5h）

```
tutorials/ascendc_operator_development_light/07_pytorch_single_operator_call/
├── 07.02_kernel_pytorch_call.ipynb      # pybind11 与 torch.library 两种绑定
└── 07.03_pytorch_operator_practice.ipynb # Softmax / Matmul 综合
```

学到的就是作业要用的：`.asc` 写完 → 编译出 `.so` → `torch.ops.load_library()` → 在 PyTorch 里当原生算子调。

### 4.3 下午：必选作业 `add_rms_norm`（照着做）

作业目录（本地已在）：

```
cann-learning-hub/standard_course/01_bootcamp/two_days_course/exercises/add_rms_norm/
├── README.md                 # 三步总流程
├── op_custom_test.py         # 端到端验证脚本
└── add_rms_norm_custom_bfloat16/
    ├── add_rms_norm_custom.asc   # ← 你要写的：device kernel + host 启动器
    ├── torch_binding.cpp         # torch.library 注册（一般不用改）
    ├── CMakeLists.txt            # 产出 libascendc_kernels.so / libascendc_ops.so
    └── test_add_rms_norm_custom.py # 单算子精度测试
```

**① 写算子并单测（在云端 NPU 环境里）**

```bash
cd standard_course/01_bootcamp/two_days_course/exercises/add_rms_norm/add_rms_norm_custom_bfloat16
mkdir -p build && cd build
cmake .. && make -j
python3 ../test_add_rms_norm_custom.py     # 必须全过，且 libascendc_ops.so 已生成
```

公式就两条：`x = x1 + x2`；`y = x / sqrt(mean(x²)+eps) * gamma`，输出还要带 `rstd` 和 `x`。只支持 **BFLOAT16**。

**② 打开融合开关**（改两个 yaml 里的 `enable_npu_rmsnorm`、`enable_add_rmsnorm` 为 `True`）

```
tutorials/llm_inference/qwen3_1.7B/src/inference_scripts/recipe_qwen3_1_7b/models/qwen/config/qwen3_1_7b_1tp.yaml
tutorials/llm_inference/qwen3_1.7B/src/inference_scripts/recipe_qwen3_1_7b/models/qwen/config/qwen3_1_7b_1tp_profile.yaml
```

**③ 把原生算子换成你写的**（只动 `run_add_rmsnorm_precision_check()` 函数内部）

- 在 `import torch_npu` 之后加：`torch.ops.load_library("<path>/libascendc_ops.so")`
- 在 `dtype_map` 里**删掉 `float16`、`float32`**（只留 bf16），否则会跑出误导性的精度失败

**④ 端到端验证 + 判题**

```bash
cd standard_course/01_bootcamp/two_days_course/exercises/add_rms_norm
python3 op_custom_test.py          # 前置条件：基线 Qwen3-1.7B 能推理、.so 路径可达
```

本地过了还不算，去 **CANNJudge** 提交（`skills/cannjudge-submit/` 里有命令行辅助），判题含性能门禁：**不低于参考实现 80%**。先保正确性，再谈性能。

---

## 5. 每次练习的 git 循环（5 条命令，固定动作）

```bash
cd /Users/jimmy/LLM/CANN-Huawei/cann-learning-hub

# 开工：同步官方（一天一次，别在跟练中途做）
git fetch upstream && git merge --ff-only upstream/master

# 练完一段，就存一次（小步提交，别攒到晚上）
git add -A
git commit -m "d1: 02 baseline 跑通，metrics 已存 run_outputs"
git push            # SSH 已配好，直接成功

# 交作业/提 PR（可选，D2 下午要展示）
open "https://gitcode.com/JimmyWangJimmy/cann-learning-hub/merge_requests/new?source_branch=learn/day1-day2"
```

提交信息建议带「天 + 章节 + 结果」，方便晚上回看：`d1: 03 profiling，Top1 耗时是 xxx op`、`d2: add_rms_norm kernel 跑通，judge 通过`。

> 大文件别进 git：模型权重、`Sources/`、`build/`、`res/` 都不要提交。跑完 `git status` 看一眼有没有把几百 MB 的东西加进来。

---

## 6. 每天收工 3 分钟检查

- [ ] `git status` 干净，或已 commit + push（云端环境关机会清空，改动只在这里活着）
- [ ] D1：`run_outputs/` 里基线 + 优化两组指标都在，A/B 数字记到笔记
- [ ] D2：`.asc` 源码已 commit；`libascendc_ops.so` 的绝对路径记下来（明天还要用）
- [ ] 明天要讲的 PPT 提前翻 3 页（D2 上午是 54 页的那一讲）

---

## 7. 排错速查

| 现象 | 先看这里 |
|---|---|
| `npu-smi` 无输出 / 找不到命令 | 没进 NPU 容器，或环境没开机；CANNLab「我的环境」里确认状态 |
| `import torch_npu` 失败 | 忘了 `source /usr/local/Ascend/ascend-toolkit/set_env.sh` |
| `bishengir-compile` 不存在 | toolkit 没 source 对，或用了非 9.0.0 镜像 |
| Notebook 02 卡在下载权重 | 网络问题就重跑该 cell（缓存会复用）；有本地权重就设 `QWEN3_1_7B_MODEL_PATH` |
| 磁盘写满 | 至少 40 GB 起；删 `build/`、旧 `res/<日期>/`、`Sources/` 下重复产物 |
| `libascendc_ops.so` 找不到 | `.so` 在 `add_rms_norm_custom_bfloat16/build/` 下，代码里写**绝对路径** |
| 精度测试莫名失败 | `dtype_map` 里的 `float16`/`float32` 没删（自定义算子只做 bf16） |
| `op_custom_test.py` 报错 | 先确认基线 Qwen3-1.7B 能正常推理，再看算子 |
| 判题不过 | 先确认功能正确，再看性能是否低于参考实现 80% |
| `git push` 要密码 | 别改 remote 回 HTTPS；确认 `~/.ssh/id_ed25519_atomgit` 还在 |

---

## 8. 今天/明天要打开的文件清单（点开就练）

D1：
- [cann_basics 01–04](cann-learning-hub/quick_start/cann_basics/)（概念补课）
- [qwen3_1.7B 目录](cann-learning-hub/tutorials/llm_inference/qwen3_1.7B/)（六个 Notebook + README）
- [第 2 讲 PPT](cann-learning-hub/standard_course/01_bootcamp/two_days_course/02_llm_deployment_and_inference_with_cann_1h.pptx)、[第 3 讲 PPT](cann-learning-hub/standard_course/01_bootcamp/two_days_course/03_llm_training_and_inference_optimization_overview_with_cann_2h.pptx)

D2：
- [第 2 章 矢量算子](cann-learning-hub/tutorials/ascendc_operator_development_light/02_simd_basics_and_vector_operator/)（7 个 Notebook）
- [07 PyTorch 单算子调用](cann-learning-hub/tutorials/ascendc_operator_development_light/07_pytorch_single_operator_call/)
- [作业 add_rms_norm](cann-learning-hub/standard_course/01_bootcamp/two_days_course/exercises/add_rms_norm/)
- [第 4 讲 PPT](cann-learning-hub/standard_course/00_atomic_courses/04_ops_programming/01_ascendc/a2a3_ascend_c_simd_vector_operator_development.pptx)（54 页那讲，课前翻一遍）
- [CANNBot](cann-learning-hub/tutorials/CANNBot/)（第 5 讲 b）
