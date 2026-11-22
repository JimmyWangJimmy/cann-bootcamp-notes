# AI 助教：架构与配置

四份讲义里内置的 AI 答疑功能，不依赖任何后端 —— 纯前端直连 DeepSeek API。

---

## 为什么是「就地」答疑

一开始我做的是「打开侧边面板提问」，试了两天就推翻了。原因是：

> 用户读到某一段卡住时，他的目标是 **5 秒内解开、然后继续读**。
> 任何需要「打开面板 → 描述问题 → 等回答 → 关掉面板」的交互，都在打断阅读。

所以现在的设计是：**答案出现在你卡住的地方**。

| 入口 | 触发方式 | 适用 |
|---|---|---|
| 段落 `?` | 点段落旁的小圆点 | 这一段没看懂，要一句话解释 |
| 节末「测一下」 | 点按钮 | 自测这一节记住了没有 |
| 划词工具条 | 选中文字后浮出 | 这个词/这句公式什么意思 |
| 悬浮球 | 拖到任意段落，点一下 | 想追问，但不想输入太多 |
| `⌘K` | 快捷键 | 手边没有具体位置，直接问 |

---

## 五层上下文

模型不知道你在读哪一页，就会答得很泛。所以每次提问都带上五层：

```
① 全文大纲        buildOutline() 生成的章节树（约 300 字）
② 当前位置        ctx.where —— 精确到 h3
③ 当前小节正文    blockText(sec, h3)，截断到 3000 字
④ 相邻首尾        上一节结尾 420 字 + 下一节开头 320 字
⑤ 选中片段        划词时的 selection
```

加起来约 **2500 token**，一次提问成本约半分钱。

### 为什么还要「约定数字表」

光给上下文不够 —— 模型会自己发明数字。比如问「Decode 的算术强度是多少」，它可能答「大约 5~10」，而讲义里算过是 `I ≈ 0.25`。

所以每份讲义的 system prompt 里都塞了一张表：

```js
SYS += '\n\n【本讲义的约定数字（引用时必须一致，不要自己另编）】\n' +
'- 一次 Add+RMSNorm 的搬运字节数，融合前 24nd、融合后 6nd\n' +
'- 8B 模型 Decode 的 TPOT 下界 ≈ 10.8 ms/token\n' +
...
```

模型引用数字时会优先用这些，答错率明显下降。

---

## 关键实现

### 1. 公式两套语法

| 位置 | 写法 | 渲染时机 |
|---|---|---|
| 静态正文 | `<span data-tex="...">` | 页面加载时一次性 KaTeX 渲染 |
| AI 回答 | `$...$` / `$$...$$` | `md()` 流式解析 |

**不能混用** —— 这是我自己踩过的坑：在正文里写 `$...$` 不会被渲染，会原样显示。

### 2. 流式回答

```js
const res = await fetch('https://api.deepseek.com/chat/completions', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json', Authorization: 'Bearer ' + key },
  body: JSON.stringify({
    model: 'deepseek-flash',
    messages: buildPopMessages(),
    stream: true,
    stream_options: { include_usage: true },
    thinking: { type: 'enabled' }   // 思考模式
  })
});
```

逐块读 `data:` 行，增量调 `md()` 重绘。

### 3. 为什么能纯前端直连

实测 DeepSeek 的两个端点都会回显 CORS 头：

```
access-control-allow-origin: *
```

所以不需要自建代理。key 存在 `localStorage`，只在本机。

> 如果你 fork 后要改成自己的模型服务，注意检查该服务是否允许浏览器直连。

### 4. 语音输入

```
MediaRecorder 录音
  → decodeAudioData 解码
  → OfflineAudioContext 重采样到 16 kHz
  → 手写 WAV 头（16-bit PCM）
  → POST https://api.deepseek.com/v1/tools/asr
```

只接受 16-bit PCM WAV 或 Ogg Opus，≤120 秒、≤8 MiB。

### 5. 就地卡片插在哪

这个花了不少时间。最初把卡片插在 `.card p` 后面，结果被挤成 200px 宽。

正确做法是**向上找到最外层的内容容器**再插：

```js
const WRAP = '.card,.cal,.readout,.tbl-wrap,.grid,.rows,.layer,.opt';
let host = el;
while (host.parentElement && !host.matches(WRAP)) host = host.parentElement;
host.insertAdjacentElement('afterend', card);
```

同时把 `?` 按钮只加到 `article section[id] > p` 上（从 138 个减到 85 个），避免满屏都是圆点。

---

## 配置自己的 key

打开任意一份讲义，点右下角悬浮球，按提示填 key。

或者直接在控制台：

```js
localStorage.setItem('dsk', 'sk-xxxxxxxx');
location.reload();
```

清掉：

```js
localStorage.removeItem('dsk');
```

---

## 模型选择

讲义默认用 `deepseek-flash`（1M 上下文，便宜）。如果某天它不可用，改 `docs/*.html` 里搜 `MODEL`：

```js
const MODEL = 'deepseek-flash';   // 可换成 deepseek-v4-pro
```

---

## 已知限制

| 限制 | 原因 |
|---|---|
| `file://` 下语音不可用 | 浏览器限制 `file://` 的 MediaRecorder |
| `file://` 下 KaTeX 字体可能缺失 | 跨域字体加载限制，用 `http://localhost` 就好 |
| 长文档在部分内嵌浏览器里加载不全 | 超过约 1.5 MB 有截断风险，用真实浏览器 |
| 无对话记忆持久化 | 刷新即清空，只有单次会话的上下文 |
