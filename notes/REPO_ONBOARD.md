# cann-learning-hub 本地落地与两天实训导航

> 执行时间：2026-11-21 10:25（10:40 完成 git 全链路配置）　｜　浏览器：Chrome + browser-skill（bsk 0.5.0 / protocol 1.9，已登录 AtomGit）
> 状态：**Fork 完成 → Clone 完成 → LFS 补齐 → 学习分支已切出并推送 → SSH 推送链路实测通过**，开箱可写可推。

## 一、仓库现状（已核验）

| 项 | 值 |
|---|---|
| 上游 | <https://gitcode.com/cann/cann-learning-hub>（`cann/cann-learning-hub`） |
| 我的 fork | <https://gitcode.com/JimmyWangJimmy/cann-learning-hub>（命名空间 `JimmyWangJimmy`，仅 master） |
| 本地路径 | `cann-learning-hub/`（当前工作目录下） |
| fork 与上游一致性 | 均为 `9d4497ea` `docs(git_basics): refresh lesson 06 Fork & PR notebook with credential guide` ✅ |
| remote | `origin` = 我的 fork（**SSH**：`git@gitcode.com:JimmyWangJimmy/cann-learning-hub.git`）；`upstream` = cann 官方（HTTPS，本地已加 `insteadOf` 自动转 SSH） |
| 工作分支 | `learn/day1-day2`（已推送，跟踪 `origin/learn/day1-day2`）；master 保持干净跟踪 origin/master |
| 体量 | 2.6 GB（`.git` 1.4 GB，含 LFS 对象）；Notebook 833 个 |
| LFS | 已 `brew install git-lfs` + `git lfs install --local` + `git lfs pull`；84 个 pptx 指针全部拉成实体（剩余 0 个占位文件） |

fork 页面当时显示「项目 Fork 中，请稍后」，等 ~30 秒后 `git ls-remote` 已能拿到同一 commit，说明复制完成；刷新后 fork 首页已正常显示 `master` / 639 次提交 / `9d4497ea`，语言构成与上游一致（Notebook 75.82%），「下载使用量 2」即这次 clone 的计数。

## 二、已配置好的东西（不用再动）

### 2.1 git 身份（提交可正常归到你账号）

```
user.name  = JimmyWangJimmy
user.email = JimmyWangJimmy@noreply.gitcode.com
```
邮箱取自 AtomGit「设置 → 电子邮件」里的**默认邮箱**（平台自带的 noreply 地址），提交会自动关联到 @JimmyWangJimmy 的贡献图，不会暴露真实邮箱。

### 2.2 推送凭证：SSH 密钥（已加到账号，实测可推）

| 项 | 值 |
|---|---|
| 私钥 | `~/.ssh/id_ed25519_atomgit`（ed25519，无口令，仅本机） |
| 公钥 | `~/.ssh/id_ed25519_atomgit.pub`，注释 `jimmy@atomgit-cann` |
| AtomGit 侧 | 设置 → SSH 公钥 里已建 `mac-jimmy-cann-20261121`，**永不过期**，创建于 2026-11-21 10:39 |
| SSH 配置 | `~/.ssh/config` 增加 `Host gitcode.com`（IdentityFile 指向该私钥、IdentitiesOnly yes、accept-new）；`gitcode.com` 主机指纹已写入 `~/.ssh/known_hosts` |
| 连通性 | `ssh -T git@gitcode.com` → `remote: Welcome to GitCode, JimmyWangJimmy` ✅ |
| 推送实测 | `git push -u origin learn/day1-day2` 成功，服务端钩子 `[PASSED]`，分支已建在 fork 上 ✅ |

![AtomGit SSH 公钥已添加](assets/atomgit-ssh-key-added.jpg)

### 2.3 附加的省事配置

- `url.git@gitcode.com:.insteadOf=https://gitcode.com/`：**复制官网 HTTPS 地址也能推**，自动走 SSH，不用每次想协议。
- `core.quotepath=false`：中文文件名不再显示成 `\344\270\255` 转义。
- `pull.ff=only`：只允许快进合并，避免练习分支被隐式 merge commit 污染。
- `git lfs install`（全局）：以后换任何仓库 clone 都自动装 LFS 过滤器。
- 不需要 Personal Access Token：日常 commit / push / 建 MR 全靠 SSH，令牌只在你手动用 HTTPS 调 API 时才需要。

### 2.4 还剩一件不是我能代做

```bash
# 拿到算力后跑环境自检（PREP_CHECKLIST.md 第四节）
npu-smi info
source /usr/local/Ascend/ascend-toolkit/set_env.sh
python3 -c "import torch, torch_npu; print(torch.__version__, torch_npu.npu.is_available())"
```
前三项任何一项不过，直接找助教，别自己折腾超过 10 分钟。

## 三、D1 实训路径（听懂 + 跟练）

### 3.1 讲义（本地已实体化，可直接打开）

| 讲次 | 文件 |
|---|---|
| 第 2 讲 大模型部署与推理 | [02_llm_deployment_and_inference_with_cann_1h.pptx](cann-learning-hub/standard_course/01_bootcamp/two_days_course/02_llm_deployment_and_inference_with_cann_1h.pptx) |
| 第 3 讲 推理优化与最佳实践 | [03_llm_training_and_inference_optimization_overview_with_cann_2h.pptx](cann-learning-hub/standard_course/01_bootcamp/two_days_course/03_llm_training_and_inference_optimization_overview_with_cann_2h.pptx) |
| 第 1 讲 生态与架构（在原子课程区） | `standard_course/00_atomic_courses/02_framework/01_overview/` |

### 3.2 六个 Notebook，顺序不能跳

目录 `cann-learning-hub/tutorials/llm_inference/qwen3_1.7B/`（760 KB，纯代码无 LFS）：

1. `01_chapter_intro.ipynb` — 章节导引与依赖
2. `02_baseline_inference.ipynb` — 基线推理，**必须先跑出基线**
3. `03_profiling_analysis.ipynb` — profiling 找瓶颈
4. `04_npu_optimization.ipynb` — 优化 + 与 02 的 A/B 对比
5. `05_quantization_qwen3_1_7b.ipynb` — 量化
6. `06_custom_matmul_operator_development_and_integration_with_qwen3_1_7b.ipynb` — 自定义 MatMul 接入

跑法：在 NPU 容器里 `jupyter lab`，或 `jupyter nbconvert --to notebook --execute`；`requirements.txt` 与 `src/` 同目录。

### 3.3 概念补课（D1 上午听不懂时回看）

`cann-learning-hub/quick_start/cann_basics/`：`01_ai_basics` → `02_what_is_npu` → `03_what_is_cann` → `04_npu_practice`。

## 四、D2 实训路径（动手 + 独立）

### 4.1 Ascend C 矢量算子（上午 2h）

主目录 `cann-learning-hub/tutorials/ascendc_operator_development_light/`（11 MB）：

- `02_simd_basics_and_vector_operator/` — 对应第 4 讲「两级存储、两种 Tensor」，**今天没懂明天必卡**
- `05_debug_and_profile/` — 调试与性能
- `07_pytorch_single_operator_call/` — 第 5 讲 a：torch.ops 单算子调用

完整版在同级 `ascendc_operator_development/`：`03_intermediate_vector_operator_development`、`07_Troubleshooting`、`08_performance_optimization`、`09_course_practice`。

### 4.2 结业作业 add_rms_norm（D2 下午必交）

`cann-learning-hub/standard_course/01_bootcamp/two_days_course/exercises/add_rms_norm/`

```
README.md                     # 三步流程：算子开发 → 打开 enable_add_rmsnorm → 替换原生算子做精度验证
op_custom_test.py             # 端到端验证（前置：基线能推理 + libascendc_ops.so 已生成）
add_rms_norm_custom_bfloat16/ # .asc 算子源码 + CMakeLists + torch_binding.cpp + 精度测试
```

三个改动点（README 原文）：

1. `src/qwen3_npu_adaptation.py` 里 `run_add_rmsnorm_precision_check()` 内 `torch.ops.load_library(".../libascendc_ops.so")`；
2. 同函数 `dtype_map` 删掉 `float16`、`float32`（自定义算子只实现 BFLOAT16）；
3. 两个 yaml 的 `enable_npu_rmsnorm`、`enable_add_rmsnorm` 改 `True`：
   `tutorials/llm_inference/qwen3_1.7B/src/inference_scripts/recipe_qwen3_1_7b/models/qwen/config/qwen3_1_7b_1tp.yaml`、`..._profile.yaml`

判题只认 CANNJudge（含性能门禁 ≥ 参考实现 80%），本地自测通过不算过。

### 4.3 CANNBot（第 5 讲 b）

`cann-learning-hub/tutorials/CANNBot/`；提交侧辅助脚本在 `skills/cannjudge-submit/`。

## 五、日常操作

fork 上已有分支 `learn/day1-day2`，直接 `git push` 就会更新它；要提 PR 用这条现成入口：
<https://gitcode.com/JimmyWangJimmy/cann-learning-hub/merge_requests/new?source_branch=learn/day1-day2>

```bash
# 同步官方更新（fork 落后时）
git fetch upstream && git checkout master && git merge --ff-only upstream/master
git push origin master        # 顺带把 fork 追上

# 继续实训分支
git checkout learn/day1-day2 && git rebase master

# 提交练习成果（commit → push 到 fork → 在 fork 页发起 PR 到 cann/cann-learning-hub）
git add -A && git commit -m "feat(exercise): add_rms_norm 实现与验证日志"
git push -u origin learn/day1-day2
```

fork 页面上「同步源项目」按钮可一键同步 master（浏览器里已确认存在，链接 `CANN / cann-learning-hub`＋`同步源项目`）。

## 六、坑位提醒

1. **push 已能直接成功**（SSH 已配好，`git push` 就行）。只有当你把 remote 手动改回 HTTPS 且机器没存令牌时才会要密码——别删 `~/.ssh/id_ed25519_atomgit`，删了就得重新加公钥。第 06 课 `quick_start/git_basics/06_gitcode_fork_pr.ipynb` 讲的是 HTTPS+令牌那条路，正好补上两种方式的差异。
2. **LFS 已处理好**：以后换机器 clone 记得先 `git lfs install`，否则 pptx 全是 1 KB 指针文件。
3. `.git` 1.4 GB，磁盘别塞满；只需代码时可用 `git clone --filter=blob:none`。
4. 官方仓库 639 次提交、每天都有合入，两天里 `git fetch upstream` 建议一天一次，别在练习中途 rebase 到别人刚改的 notebook 上。
5. 课堂 Notebook 走 CANNLab 在线环境时，本地这份只作为对照与离线备份；两边的 `src/` 路径以在线环境实际为准。

## 七、我这边做过的浏览器动作（可复现）

1. bsk 打开 <https://gitcode.com/cann/cann-learning-hub>，观察到 Star 2101 / Fork 790，已登录（右上头像，命名空间 `JimmyWangJimmy`）；
2. 点 `Fork` → 弹窗默认名 `cann-learning-hub`、路径 `JimmyWangJimmy/cann-learning-hub`、分支 `master`、介绍沿用上游 → 点「创建Fork项目」；
3. 页面转「项目Fork中，请稍后」，约 30 s 后自动跳到 fork 仓库首页；
4. 终端 `git clone` 上游 → 改名 `upstream`、加 `origin=fork`、`fetch origin`、切 `learn/day1-day2`、`git lfs pull`；
5. 进 fork 的个人主页 → 「编辑个人资料」→ 用户设置里读「电子邮件」拿默认邮箱 `JimmyWangJimmy@noreply.gitcode.com`；
6. 设置 →「SSH 公钥」→ 右上「+ SSH 公钥」→ 名称 `mac-jimmy-cann-20261121` + 粘贴本机 ed25519 公钥 → 新建 → 页面回「新建密钥成功」，列表出现该密钥；
7. 终端 `ssh -T git@gitcode.com` 拿到 `Welcome to GitCode, JimmyWangJimmy`，`git push -u origin learn/day1-day2` 通过服务端钩子检查，推送成功。

> 小坑（下次自己加公钥会遇到）：AtomGit 设置页那个「+ SSH 公钥」按钮和弹窗里的「新建」，普通点击会被一层透明元素挡住，坐标点击也无效；改成点按钮内的图标容器（`div.devui-button__icon-fix`）或聚焦后用 Tab→Enter 才能提交。
