# 大一学生零成本搭建微信 AI 机器人：从 Hello World 到多模态 Agent

> 一个不需要 Claude Code、不需要 ChatGPT Plus、只靠 DeepSeek API 就能跑的微信 AI 机器人。

---

## 起因

大一下学期，我想给微信装一个 AI。不是那种聊天机器人 SaaS，而是**我自己的 AI**——能读我的文件、能搜网页、能识别图片、能记住我的偏好。

市面上的方案有两个问题：要么要钱（ChatGPT Plus $20/月），要么要技术（Claude Code 的配置劝退了周围的同学）。

于是我决定自己搭一个。最终跑通了：**微信 ↔ cc-connect ↔ 自写 Python Agent ↔ DeepSeek API**，总成本几乎为零。

---

## 能做什么

用了一个月后，这个机器人的能力清单：

- 💬 **聊天**：DeepSeek V3 日常对话，遇到复杂问题自动切 V4 Pro
- 🌐 **上网**：调用 Chrome 浏览器搜索，国内外网站都能访问
- 🖼️ **识图**：收到微信图片 → 千问 VL 识别 → 告诉你图片里有什么
- 📁 **文件**：读写文件、生成 Excel/Word
- 💻 **命令**：执行 Python 脚本、Shell 命令
- 🧠 **记忆**：记住你的偏好（"回复简洁点"），重启不丢
- 📤 **发文件**：生成的报表截图直接发回微信

---

## 怎么搭

### 你需要什么

| 东西 | 说明 |
|------|------|
| Windows 电脑 | 需要一直开着 |
| DeepSeek API Key | [platform.deepseek.com](https://platform.deepseek.com) 注册，充值 10 块钱能用很久 |
| 千问 API Key | [dashscope.aliyun.com](https://dashscope.aliyun.com) 注册，有免费额度 |
| Chrome 浏览器 | 自带的最好 |
| 微信 | 扫个码就行 |

### 核心依赖

```bash
npm install -g cc-connect@beta puppeteer-core
pip install pillow openpyxl python-docx imageio-ffmpeg
```

### 三步跑起来

1. **配 Key**：把 DeepSeek 和千问的 Key 填进 `fake-claude.py`
2. **扫码**：`cc-connect weixin setup --project wechat-bot`
3. **启动**：双击 `启动微信机器人.bat`

---

## 遇到的问题和解决

### 1. Claude Code 在 Windows 上崩溃

cc-connect 官方推荐用 Claude Code 做 Agent 后端。但 `claude.exe` 在 Windows 上的 stream-json 交互模式有 .NET CLR 崩溃 Bug（`0xe0434352`），试了 CC Switch、CCR 中转都没绕过去。

**解决**：用 Python 重写了一个 Claude Code 兼容的 Agent（fake-claude.py），用纯 HTTP API 调 DeepSeek，不依赖 Claude Code 本体。

### 2. DeepSeek 不支持图片输入

DeepSeek 的 `/chat/completions` 接口只支持文本，微信发来的图片无法直接处理。

**解决**：引入 Qwen-VL（阿里千问）做视觉层。收到图片 → 转 base64 → Qwen-VL 识别 → 把文字描述喂给 DeepSeek。

### 3. 联网搜索在墙内不可用

DuckDuckGo 不稳定，Google 需要梯子。

**解决**：用 puppeteer-core 连接 Chrome CDP，调用 Bing 搜索。Chrome 可以用用户的登录态，天然支持带验证码的网站。额外加了 HTTP 兜底方案，Chrome 不在时也能搜。

### 4. DeepSeek function calling 不稳定

有时 AI 就是不调工具，直接编造答案。

**解决**：精简工具列表（从 13 个砍到 8 个），在系统提示里明确优先级，加了自动搜索检测作为兜底。后来发现最好的方案是让 AI 少做选择——给更少的工具，让每个工具职责更清晰。

### 5. 记忆系统多次翻车

JSON 编码损坏、AI 记不住偏好、重启就忘...

**解决**：除了 memory.json 存对话历史，增加了人类可读的 `preferences.txt`。AI 每次对话都会读取偏好文件，用户可以用自然语言修改（"以后回复再简洁点"）。

---

## 踩坑心得

1. **Windows 上能跑的 Linux 上不一定**：Node.js 子进程、文件路径、编码全是坑
2. **别迷信大厂的 CLI 工具**：Claude Code 在 Windows 上的体验真的很差
3. **prompt engineering 比模型更重要**：一个 20 行的系统提示能改变整个行为
4. **API Key 管理是最大安全隐患**：开源前一定要清理干净
5. **DuckDuckGo 在国内真的很慢**：搜索引擎选 Bing 或者直接用浏览器

---

## 下一步

- [ ] 托盘图标 + 开机自启（告别命令行窗口）
- [ ] 一键安装包（NSIS）
- [ ] 中转 API 服务（让完全不懂技术的人也能用）
- [ ] 多用户支持

---

## 代码

GitHub：[你的仓库地址]

---

*作者：大一生，2026 年 6 月*
