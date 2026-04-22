[README.md](https://github.com/user-attachments/files/26959929/README.md)
# NeuralStrike AI — 智能渗透测试平台

<p align="center">
  <strong>🧠 AI驱动的自动化渗透测试框架</strong>
</p>

<p align="center">
  <a href="#-快速部署">快速部署</a> •
  <a href="#-使用指南">使用指南</a> •
  <a href="#-工具支持">工具支持</a> •
  <a href="#-报告系统">报告系统</a> •
  <a href="#-免责声明">免责声明</a>
</p>

---

## 📖 项目简介

NeuralStrike 是一个基于 AI 的智能渗透测试平台，通过多Agent协作实现自动化安全评估。支持7大AI供应商，集成20+专业渗透测试工具，生成专业级审计报告。

### ✨ 核心特性

| 特性 | 说明 |
|------|------|
| 🧠 **AI驱动** | 智能规划、动态调整、实时交互，AI自主决策渗透策略 |
| 🤖 **多Agent协作** | 信息收集 → 漏洞扫描 → Web渗透 → 后渗透/提权 → 报告生成 |
| 💬 **AI实时对话** | 无需开始扫描也能与AI对话，执行工具命令，获取安全建议 |
| 🔧 **20+工具集成** | nmap/nuclei/sqlmap/masscan/hydra/john/hashcat 等，一键安装 |
| 📊 **SSE实时可视化** | AI思考过程、工具调用语法、扫描进度、解码结果实时展示 |
| 📋 **三级报告模板** | 简报/标准/专家，支持 MD/TXT/PDF/HTML 导出 |
| 🔓 **12种解码器** | Base64/Hex/URL/ROT13/MD5/SHA/JWT 等，自动检测并解码 |
| 🛡️ **15种漏洞POC** | 每种漏洞自动生成注入语句、利用代码、修复方案、WAF规则 |
| 🔐 **7大AI供应商** | OpenAI / DeepSeek / 硅基流动 / 通义千问 / 智谱 / Kimi / Claude |
| ⚡ **并发加速** | ThreadPool并发扫描，AI超时20s，漏洞+Web渗透并行执行 |
| 🎯 **自动提权** | 后渗透阶段自动尝试SUID/内核漏洞/凭据窃取/文件获取 |

---

## 🏗️ 系统架构

```
┌──────────────────────────────────────────────────────────┐
│                    Web 前端 (Flask + SSE)                 │
│  🧠 AI思考  │  🎛 控制面板  │  🔧 工具  │  🤖 API设置   │
└──────────────────────────┬───────────────────────────────┘
                           │
┌──────────────────────────▼───────────────────────────────┐
│                Workflow Orchestrator                      │
│  AI决策引擎 + 知识图谱降级 + 并行执行 + 用户交互 + SSE    │
└──┬───────┬───────┬───────┬───────┬───────┬────────────────┘
   │       │       │       │       │       │
┌──▼──┐ ┌──▼──┐ ┌──▼──┐ ┌──▼──┐ ┌──▼──┐ ┌──▼──┐
│Info │ │Vuln │ │Web  │ │Brute│ │Post │ │Report│
│Agent│ │Agent│ │Agent│ │Force│ │Explo│ │Agent │
└─────┘ └─────┘ └─────┘ └─────┘ └─────┘ └──────┘
```

---

## 🚀 快速部署

### 方式1: Kali Linux 一键部署（推荐）

```bash
# 克隆项目
cd /opt
git clone https://github.com/CaptainJW/nobody.git
cd NeuralStrike

# 一键部署（安装工具 + 依赖 + 权限 + 防火墙）
chmod +x deploy_kali.sh
sudo ./deploy_kali.sh

# 启动
python3 main.py

# 或使用快速启动脚本
./start_kali.sh
```

### 方式2: 手动安装

```bash
# 安装Python依赖
pip install flask requests beautifulsoup4 lxml

# 安装渗透测试工具（Kali）
sudo apt install nmap masscan nuclei whatweb nikto dirsearch gobuster \
    feroxbuster amass subfinder sqlmap dalfox testssl.sh dnsrecon \
    hydra john hashcat crowbar crackmapexec

# 启动
python3 main.py
```

启动后浏览器访问 `http://localhost:7777`

> 💡 从 Windows 访问 Kali：`http://<Kali-IP>:7777`

---

## 📱 使用指南

### 1. 配置AI供应商

打开网页 → 🤖 **API设置** → 选择供应商 → 输入API Key → 保存

| 供应商 | 默认模型 | Base URL |
|--------|----------|----------|
| 🟢 OpenAI | gpt-4 | api.openai.com/v1 |
| 🔵 DeepSeek | deepseek-chat | api.deepseek.com/v1 |
| 🟣 硅基流动 | Qwen2.5-72B | api.siliconflow.cn/v1 |
| 🟠 通义千问 | qwen-turbo | dashscope.aliyuncs.com |
| 🔷 智谱AI | glm-4-flash | open.bigmodel.cn |
| 🌙 Kimi | moonshot-v1-8k | api.moonshot.cn/v1 |
| 🟤 Claude | claude-3.5-sonnet | api.anthropic.com/v1 |

支持**在线获取模型列表**，选择供应商后自动拉取可用模型，可视化选择。

### 2. AI对话（无需开始扫描）

直接在页面底部输入框与AI交流，支持SSE流式实时反馈：

```
用户: 扫描局域网存活主机        → AI执行 arp-scan -l，实时展示结果
用户: nmap -sV 192.168.1.1     → AI执行端口扫描并分析
用户: base64: SGVsbG8gV29ybGQ= → AI自动Base64解码 → Hello World
用户: 解码 5ey354iK             → AI自动检测编码方式并解码
用户: 如何利用SQL注入？          → AI专业解答，无需执行工具
用户: sqlmap -u "http://..."    → AI执行sqlmap并展示结果
```

### 3. 开始渗透测试

🎛 **控制面板** → 输入目标URL/IP → 选择测试类型 → 🚀 开始测试

AI思考面板实时展示10种事件：
- 🧩 分析 — 目标类型判断、攻击面评估
- 📋 规划 — AI制定执行策略
- ⚡ 执行 — 开始执行具体动作
- 🔧 工具调用 — 检测到工具并准备执行
- 💻 命令执行 — 显示完整工具命令语法（绿色代码块）
- 📤 结果输出 — 端口/漏洞/路径/子域名/凭据/文件
- 🔍 评估 — AI评估结果可信度
- ➡️ 决策 — AI决定下一步操作
- 💬 用户消息 — 用户实时干预
- ❌ 错误 — 异常情况

渗透过程中可随时：
- ⏸ **暂停** → AI等待用户指令
- ▶️ **恢复** → 继续自动渗透
- 💬 **输入指令** → AI调整策略（如"重点测试SQL注入"）

### 4. 自动提权与文件获取

渗透测试后渗透阶段自动触发：
- 🔍 本地信息收集 → /etc/passwd, /etc/shadow, 用户组
- ⬆️ 权限提升 → SUID/GTFOBins/内核漏洞
- 🔑 凭据窃取 → .bash_history, 配置文件
- 📁 文件发现 → 敏感配置/数据库文件/备份文件
- 📂 目录枚举 → /var/www, /opt, /home下的关键目录

所有发现自动写入最终报告。

---

## 📋 报告系统

### 三级报告模板

| 模板 | 适用场景 | 包含内容 |
|------|----------|----------|
| 📄 **简报** | 快速概览 | 漏洞统计、Top5漏洞、端口列表、快速POC、注入语句 |
| ⚡ **标准** | 修复指导 | 完整漏洞描述、修复建议、代码示例、测试方法 |
| 🚀 **专家** | 审计报告 | 攻击模拟、Mermaid攻击链、CVSS评分、合规映射、WAF规则、修复脚本、ROI分析、邮件模板 |

支持导出: **Markdown / TXT / PDF / HTML**

### POC覆盖（15种漏洞类型）

| 漏洞类型 | 注入语句/POC | 自动化命令 | 修复代码 |
|----------|-------------|-----------|---------|
| SQL注入 | 7类注入语句 + UNION/盲注/时间盲注 | `sqlmap -u "..." --batch --dbs` | Python/PHP/Java/MyBatis |
| XSS | 7类payload + Cookie窃取/键盘记录 | `dalfox url "..."` | html.escape/CSP/Jinja2 |
| 命令注入 | 7类payload + 反弹Shell | `commix --url="..."` | subprocess安全API |
| 路径穿越 | 4类payload + 编码绕过 | `curl "...?file=../../../etc/passwd"` | os.path.normpath白名单 |
| LFI/RFI | PHP伪协议 + 日志注入 | `curl "...?page=../../../etc/passwd"` | allow_url_include |
| SSRF | 云元数据 + Redis/Gopher | `curl -d "url=http://127.0.0.1/"` | ipaddress内网过滤 |
| XXE | 外部实体 + Blind OOB | `curl -X POST` 含XML payload | defusedxml/lxml |
| 文件上传 | WebShell + .htaccess | `curl -F "file=@shell.php"` | uuid重命名+MIME验证 |
| CSRF | 自动提交隐藏表单 | `curl -X POST`含Origin头 | CSRF Token |
| 敏感文件 | 13条验证命令 | `curl -s ".../.env"` | Nginx/Apache配置 |
| 弱认证 | 默认凭据列表 | `hydra -l admin -P rockyou.txt` | 密码策略+MFA |
| SSL/TLS | testssl/sslscan命令 | `testssl.sh --full "..."` | Nginx SSL安全配置 |
| 反序列化 | ysoserial/pickle POC | `curl --data-binary @payload.bin` | 类型白名单 |
| 信息泄露 | curl验证9个端点 | `curl -s ".../phpinfo.php"` | 调试模式关闭 |
| 解码 | 12种编解码器 | 自动检测并解码 | — |

---

## 🔧 工具支持

### 自动检测与安装

🔧 **工具管理** 页面实时显示所有工具状态，支持一键安装（apt/pip/go/cargo/GitHub）。

| 工具 | 类型 | 安装方式 | 用途 |
|------|------|----------|------|
| nmap | CLI | apt | 端口扫描/服务检测 |
| masscan | CLI | apt | 高速端口扫描 |
| nuclei | CLI | apt/go | 漏洞扫描 |
| whatweb | CLI | apt | Web指纹识别 |
| nikto | CLI | apt | Web服务器扫描 |
| dirsearch | CLI | apt/pip | 目录扫描 |
| gobuster | CLI | apt/go | 目录爆破 |
| feroxbuster | CLI | apt/cargo | 目录爆破 |
| amass | CLI | apt/go | 子域名枚举 |
| subfinder | CLI | apt/go | 子域名发现 |
| sqlmap | CLI | apt/pip | SQL注入检测 |
| dalfox | CLI | apt/go | XSS扫描 |
| testssl.sh | CLI | apt/GitHub | SSL/TLS检测 |
| dnsrecon | CLI | apt | DNS枚举 |
| hydra | CLI | apt | 暴力破解 |
| john | CLI | apt | 密码破解 |
| hashcat | CLI | apt | GPU密码破解 |
| crowbar | CLI | apt | 暴力破解 |
| crackmapexec | CLI | apt/pip | 内网渗透 |
| bloodhound-ce | CLI | apt/pip | AD分析 |
| Hydra | CLI | apt | 暴力破解 || chisel | CLI | go | 隧道工具 |
| sslyze | Python | pip | SSL分析 |
| arp-scan | CLI | apt | 局域网扫描 |

### Go代理配置

国内环境自动配置 `GOPROXY=https://goproxy.cn,https://goproxy.io,direct`，解决Go工具编译问题。

---

## 🔓 解码工具

内置12种自动解码器，对话中自动检测编码数据：

| 解码器 | 关键词 | 示例 |
|--------|--------|------|
| Base64 | `base64:` | `base64: SGVsbG8=` → Hello |
| Base32 | `base32:` | `base32: JBSWY3DP` → Hello |
| Hex | `hex:` | `hex: 48656c6c6f` → Hello |
| URL | `url:` | `url: %E4%BD%A0%E5%A5%BD` → 你好 |
| HTML实体 | `html:` | `html: &amp;` → & |
| ROT13 | `rot13:` | `rot13: Hello` → Uryyb |
| Reverse | `reverse:` | `reverse: olleH` → Hello |
| MD5 | `md5:` | 显示不可逆哈希值 |
| SHA1 | `sha1:` | 显示不可逆哈希值 |
| SHA256 | `sha256:` | 显示不可逆哈希值 |
| JWT | `jwt:` | 解码JWT Token |
| 自动 | 无关键词 | 自动尝试所有方式 |

---

## 📁 项目结构

```
NeuralStrike/
├── main.py                    # 启动入口
├── web_server.py              # Flask Web服务器（1800+行）
├── chat_api.py                # 独立AI对话API (Blueprint + SSE)
├── config.py                  # 通用配置管理
├── setup_tools.py             # 工具安装脚本
├── deploy_kali.sh             # Kali一键部署
├── start_kali.sh              # 快速启动
├── requirements.txt           # Python依赖
├── .gitignore                 # Git忽略规则（含密钥保护）
├── agents/                    # AI Agent模块
│   ├── base_agent.py          # 基础Agent + SmartAgent（思考回调+工具调用）
│   ├── ai_client.py           # AI客户端（7供应商+速率限制+重试）
│   ├── workflow_orchestrator.py # 工作流编排器（AI决策+知识图谱+并行）
│   ├── info_gathering_agent.py  # 信息收集（端口/子域名/指纹/目录）
│   ├── vuln_scanner_agent.py    # 漏洞扫描（nuclei/敏感路径/NVD并发）
│   ├── web_pentest_agent.py     # Web渗透（SQLi/XSS/CMDI/SSRF并发）
│   ├── post_exploitation_agent.py # 后渗透（提权/凭据/文件获取）
│   ├── brute_force_agent.py     # 暴力破解（Hydra/默认凭据）
│   ├── report_agent.py          # 报告生成（15种POC+3级模板+解码）
│   ├── shared_memory.py         # Agent间共享上下文（深拷贝+锁）
│   ├── knowledge_graph.py       # 渗透测试知识图谱（30+节点）
│   ├── tool_manager.py          # 工具检测与管理（19+工具+别名）
│   └── utils.py                 # 公共工具函数（JSON解析/URL解析/消毒）
├── config/                    # 配置模块
│   ├── ai_manager.py          # AI配置管理器
│   └── ai_config.json         # AI供应商配置（API Key，受.gitignore保护）
├── web/                       # 前端
│   ├── index.html             # 主页面（会话管理+AI对话+报告模板）
│   └── static/
│       ├── app.js             # 前端逻辑（SSE流+多会话+聊天+报告）
│       └── style.css          # 样式（NeuralStrike品牌+暗色主题）
├── core/                      # 核心服务（MCP协议）
│   ├── hexstrike_server.py    # MCP服务端
│   └── hexstrike_mcp.py       # MCP客户端
└── data/                      # 数据目录（受.gitignore保护）
    └── wordlists/             # 默认字典
```

---

## ⚙️ 环境要求

| 组件 | 最低要求 | 推荐 |
|------|----------|------|
| 操作系统 | Ubuntu 20.04+ | Kali Linux 2024+ |
| Python | 3.9+ | 3.11+ |
| 内存 | 2GB | 4GB+ |
| 磁盘 | 1GB | 5GB+（含工具） |
| 网络 | 需要访问AI API | — |

---

## 🛡️ 安全说明

- API配置文件（`config/ai_config.json`）受 `.gitignore` 保护，不会被提交到代码仓库
- 工具安装接口仅允许局域网IP或携带访问令牌访问
- SSE流仅允许同源跨域访问
- 报告下载路径做了穿越校验，防止任意文件读取
- 所有工具调用前自动检测可用性，未安装工具自动降级

---

## ⚠️ 免责声明

本工具仅供**合法授权的安全测试**使用。使用者必须：

1. 获得目标系统所有者的**明确书面授权**
2. 遵守当地法律法规
3. 不得用于未授权的渗透测试
4. 对使用本工具产生的任何后果自行负责

**未经授权的渗透测试是违法行为。**



---

<p align="center">
  <strong>NeuralStrike AI</strong> — 让渗透测试更智能
</p>
