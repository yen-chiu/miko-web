# 异次元通讯 · 网页版

《异次元通讯》2016 原版与「次元复苏」重置版的非官方网页移植版：纯静态、无构建、无后端。

- 线上地址：<https://yenchiu.com/miko-web/>
- 数据来源：重置版 APK 的明文剧情 CSV（`assets/story/*.csv`）+ 原版 APK 的节奏与素材
- 剧情编译、素材整理见主站仓库里的逆向工作区（`reverse/`）

## 目录

```
index.html     单页应用入口
app.js         剧情引擎：分块跳转、选项、等待、词典/图鉴/动态解锁、存档
style.css      全部样式
data/          story.json（剧情）与 meta.json（词典/图鉴/动态/头像/节奏）
assets/        章节图、照片、头像、音乐、界面图
```

## 本地预览

```bash
python -m http.server 8000
# 打开 http://127.0.0.1:8000/
```

## 后日谈：接 AI（可选）

主线 14,401 行剧情完全走本地 `data/story.json`，不联网。只有菜单里的「后日谈」——
结局之后继续和她说话——会调用一次接口：

```
POST https://api.yenchiu.com/chat
{ "persona": "miko", "text": "今天下雨了，你带伞了吗？",
  "history": [{ "role": "me", "text": "…" }, { "role": "her", "text": "…" }] }
→ { "reply": "……才、才不是因为你说下雨才这样的。" }
```

- 每次调用还会带上 `memory`：**已经读完的章节标题 + 最近 36 条真实剧情**（来自每章的聊天记录），
  这样她不会凭空编出没发生过的校园琐事；读过的章节越多，她记得的越多。
- 人设、模型、密钥全在那个 Worker 里（见 `yen-chiu/yen-chiu.github.io` 的 `tools/worker`），
  本仓库不含任何密钥；`persona` 目前有 `miko` / `iris` / `lily`。
- 接口没配好（503）或网络不通时，页面不会报错：「后日谈」只记录你说的话，顶部提示「信号有点差，先记下了」。
- 想换成自己的后端：在 `app.js` 里覆盖 `window.MIKO_AI = async (history, text) => '…'` 即可；
  调试时也可以先设 `window.MIKO_API_BASE = 'http://127.0.0.1:xxxx'` 指到本地桩。
- 防滥用靠 Worker 的 Origin 白名单 + 每 IP 每分钟 `RATE_LIMIT_CHAT` 次限流 + 文本/上下文长度上限。

## 说明

剧情脚本、插画、语音与音乐版权归原作者所有；本仓库只做个人研究与怀旧用途，不商用。
