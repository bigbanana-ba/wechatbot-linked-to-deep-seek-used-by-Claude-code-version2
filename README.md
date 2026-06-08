# 🤖 WeChat AI Agent

> 把 DeepSeek 变成微信多模块聊天机器人 — 支持工具调用、联网搜索、图片识别、文件操作、记忆系统

## ✨ 功能

| 能力 | 说明 |
|------|------|
| 💬 智能对话 | DeepSeek V3/V4/R1 多模型自由切换 |
| 🌐 联网搜索 | Chrome 浏览器 + Bing HTTP 双保险 |
| 🖼️ 图片识别 | Qwen3-VL-Plus 最新模型 |
| 🔍 GitHub 搜索 | 搜索开源项目和代码 |
| 📁 文件操作 | 读写文件、生成 Excel/Word |
| 💻 命令执行 | Python 脚本、Shell、系统工具 |
| 🧠 记忆系统 | 人类可读 preferences.txt |
| 📤 发送文件 | cc-connect send 原生支持 |
| 🎬 视频分析 | ffmpeg 抽帧 + 视觉识别 |
| 💰 Token 优化 | 智能截断、分级上下文、省 60-90% |
| 🔌 MCP 支持 | 可扩展连接任意 MCP Server |

## 🏗️ 架构

```
微信 → cc-connect → fake-claude.py → DeepSeek API
                     ├── 图片识别 → Qwen-VL API
                     ├── 联网搜索 → Chrome CDP (puppeteer)
                     └── 记忆系统 → preferences.txt + memory.json
```

## 🚀 快速开始

### 1. 安装依赖

```powershell
# Node.js 22+
# Python 3.12+
# Chrome 浏览器

npm install -g cc-connect@beta puppeteer-core
pip install openpyxl python-docx pillow imageio-ffmpeg

# 安装并配置 web-access skill（可选，增强联网能力）
npx skills add eze-is/web-access@web-access -g -y
```

### 2. 配置 API Key

打开 `fake-claude.py`，填入你的 API Key：

```python
DEEPSEEK_API_KEY = "sk-你的DeepSeek-Key"
QWEN_API_KEY = "sk-你的千问-Key"  # 阿里云百炼，用于图片识别
```

### 3. 扫码登录微信

```powershell
cc-connect weixin setup --project wechat-bot
```

用微信扫描终端显示的二维码。

### 4. 启动机器人

双击 `启动微信机器人.bat`，或者：

```powershell
cc-connect --force
```

## 📁 文件结构

```
wechat-bot/
├── fake-claude.py          # AI 核心（工具调用、记忆、搜索）
├── chrome-browse.js        # Chrome 浏览器自动化脚本
├── claude.cmd              # cc-connect 启动桥接
├── preferences.txt         # 用户偏好（人类可读）
├── memory.json             # 对话历史 + 事实记忆
├── 启动微信机器人.bat       # 一键启动
└── README.md
```

## 🛠️ 可用工具

机器人可以自主选择使用以下工具：

- `web_search` - 用 Chrome 搜索网页
- `bash` - 执行 Shell 命令和 Python 脚本
- `read` / `write` / `list_dir` / `grep` - 文件操作
- `send_to_wechat` - 发送文件/图片到微信
- `ask_model` - 调用更强的模型（deepseek-v4-pro / deepseek-reasoner）
- `remember_preference` / `forget_preferences` - 记忆偏好
- `read_docx` - 读取 Word 文档
- `analyze_video` - 分析视频内容

## 🗣️ 微信聊天命令

| 命令 | 作用 |
|------|------|
| `/reset` | 重置对话历史和偏好 |







## Tokens节省对比
📸 场景：发一张世界杯足球赛图片
优化前

Qwen-VL 识别图片:  ~600 tokens (图片 + prompt)

System Prompt:    ~300 tokens (大段工具说明)
对话历史:         ~400条 × ~50字/条 = ~20,000 tokens
用户消息+图片描述: ~100 tokens
max_tokens:       16,384 tokens (API 回复上限)

总计 input:       ~21,000 tokens
总计 output:      ~200 tokens
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
单次费用:         约 ¥0.03
优化后

Qwen-VL 识别图片:  ~600 tokens (不变)

System Prompt:    ~150 tokens (精简 50%)
对话历史:         40条 × ~50字 = ~2,000 tokens (省 90%)
用户消息+图片描述: ~100 tokens
max_tokens:       4,096 tokens (回复上限)

总计 input:       ~2,850 tokens
总计 output:      ~200 tokens
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
单次费用:         约 ¥0.005
📊 对比
优化前	优化后	省
Input tokens	21,000	2,850	86%
单次费用	¥0.03	¥0.005	83%
回复速度	~3s	~1.5s	快一倍
1000次图片	¥30	¥5	省 ¥25/月
关键是历史消息——闲聊场景下没必要把 400 条旧聊天全发给 AI，40 条就够了。这才是真正吃 token 的大头。


## ⚠️ 注意事项

- 需要一台 Windows 电脑保持开机运行
- Chrome 需要保持打开（或开启 CDP 调试端口）
- DeepSeek 和千问 API 都按量计费，日常聊天用 deepseek-chat 很便宜（月均几块钱）
- 微信 iLink 协议仅供个人学习和研究使用


## 🙏 致谢

- [cc-connect](https://github.com/chenhg5/cc-connect) - 微信桥接
- [DeepSeek](https://platform.deepseek.com) - AI 模型
- [Qwen-VL](https://dashscope.aliyun.com) - 图片识别
- [web-access](https://github.com/eze-is/web-access) - 浏览器自动化 Skill



## 致自己
   一切的一切开始于一个群文件的“处理”，再由自己对ai的思考，对ai工作流的理解，自己内心深处对智能体研究的热爱，和日日夜夜的坚持并且解决困难与独特对ai的见解，最后才诞生此物。愿所有人身体健康，万事如意，学业有成。谢谢
