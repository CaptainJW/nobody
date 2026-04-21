[Uploading NeuralStrike_分析文档.md…]()
# NeuralStrike AI — 项目完整分析文档

> 生成时间：2026-04-20  
> 项目路径：`C:\Users\Lenovo\Desktop\NeuralStrike\NeuralStrike`

---

## 一、项目概述

NeuralStrike 是一个**基于 AI 的智能渗透测试自动化平台**，专为 Kali Linux 设计。

核心定位：
- 通过多 Agent 协作，实现从信息收集 → 漏洞扫描 → Web 渗透 → 后渗透 → 报告生成的全流程自动化
- 支持 7 大 AI 供应商（OpenAI / DeepSeek / 硅基流动 / 通义千问 / 智谱 / Kimi / Claude）
- 集成 20+ 专业渗透测试工具（nmap / nuclei / sqlmap / hydra 等）
- 提供 Web 界面（Flask + SSE 实时推流），浏览器访问 `http://localhost:7777`

---

## 二、技术栈

| 层次 | 技术 |
|------|------|
| 后端框架 | Python 3.9+，Flask 2.3+ |
| AI 接入 | OpenAI 兼容 REST API + Anthropic Messages API |
| 异步执行 | asyncio + ThreadPoolExecutor |
| 实时推流 | SSE（Server-Sent Events） |
| 前端 | 原生 HTML5 + Vanilla JS + CSS（暗色主题） |
| 工具调用 | subprocess（shlex 防注入） |
| 数据共享 | 内存中 SharedMemory（asyncio.Lock 保护） |
| 知识推理 | 自建渗透测试知识图谱（30+ 节点，20+ 边） |
| 报告导出 | Markdown / TXT / PDF（weasyprint）/ HTML |
| Python 依赖 | flask, requests, beautifulsoup4, lxml, weasyprint |

---

## 三、完整文件结构

```
NeuralStrike/
├── main.py                      # 启动入口，Flask 服务器，端口 7777
├── web_server.py                # Flask 路由层（1800+ 行），SSE 推流，扫描状态管理
├── chat_api.py                  # 独立 AI 对话 Blueprint（/api/chat）
├── config.py                    # 通用配置管理
├── setup_tools.py               # 工具安装脚本（apt/pip/go/cargo/GitHub）
├── deploy_kali.sh               # Kali 一键部署脚本
├── start_kali.sh                # 快速启动脚本
├── requirements.txt             # Python 依赖（flask/requests/bs4/lxml/weasyprint）
├── .gitignore                   # 含 API Key 保护规则
│
├── agents/                      # AI Agent 核心模块
│   ├── ai_client.py             # AI 客户端（7 供应商 + 速率限制 + 重试）
│   ├── base_agent.py            # BaseAgent（任务型）+ SmartAgent（AI 推理型）基类
│   ├── workflow_orchestrator.py # 工作流编排器（AI 决策 + 知识图谱 + 并行执行）
│   ├── info_gathering_agent.py  # 信息收集（端口/子域名/指纹/目录）
│   ├── vuln_scanner_agent.py    # 漏洞扫描（nuclei/敏感路径/NVD 并发）
│   ├── web_pentest_agent.py     # Web 渗透（SQLi/XSS/CMDI/SSRF 并发）
│   ├── post_exploitation_agent.py # 后渗透（提权/凭据/文件获取）
│   ├── brute_force_agent.py     # 暴力破解（Hydra/默认凭据）
│   ├── report_agent.py          # 报告生成（15 种 POC + 3 级模板 + 解码）
│   ├── shared_memory.py         # Agent 间共享上下文（深拷贝 + asyncio.Lock）
│   ├── knowledge_graph.py       # 渗透测试知识图谱（30+ 节点）
│   ├── tool_manager.py          # 工具检测与管理（19+ 工具 + 别名）
│   └── utils.py                 # 公共工具函数（JSON 解析/URL 解析/消毒）
│
├── config/
│   ├── ai_manager.py            # AI 配置管理器（供应商注册表 + 连接测试）
│   └── ai_config.json           # AI 供应商配置（API Key，受 .gitignore 保护）
│
├── core/
│   ├── hexstrike_server.py      # MCP 服务端
│   ├── hexstrike_mcp.py         # MCP 客户端
│   └── server_config.json       # MCP 服务配置
│
├── web/
│   ├── index.html               # 主页面（会话管理 + AI 对话 + 报告模板）
│   └── static/
│       ├── app.js               # 前端逻辑（SSE 流 + 多会话 + 聊天 + 报告）
│       └── style.css            # 样式（暗色主题）
│
├── data/
│   └── wordlists/common.txt     # 默认字典
├── logs/                        # 运行日志
├── results/                     # 扫描结果
└── pentest_results/             # 渗透测试报告
```

---

## 四、系统架构

### 4.1 整体分层架构

```
┌─────────────────────────────────────────────────────────────┐
│                    Web 前端（浏览器）                         │
│   AI 思考面板 │ 控制面板 │ 工具管理 │ API 设置 │ 报告下载    │
└──────────────────────────┬──────────────────────────────────┘
                           │ HTTP / SSE
┌──────────────────────────▼──────────────────────────────────┐
│              Flask Web 服务器（web_server.py）                │
│   /api/scan/start  /api/scan/stream  /api/chat              │
│   /api/report/*    /api/tools/*      /api/ai/*              │
└──────────────────────────┬──────────────────────────────────┘
                           │
┌──────────────────────────▼──────────────────────────────────┐
│           WorkflowOrchestrator（工作流编排器）                │
│   AI 决策引擎 + 知识图谱降级 + 并行执行 + 用户交互 + SSE     │
└──┬──────┬──────┬──────┬──────┬──────┬────────────────────────┘
   │      │      │      │      │      │
┌──▼──┐┌──▼──┐┌──▼──┐┌──▼──┐┌──▼──┐┌──▼────┐
│Info ││Vuln ││Web  ││Brute││Post ││Report │
│Agent││Agent││Agent││Force││Explo││Agent  │
└──┬──┘└──┬──┘└──┬──┘└──┬──┘└──┬──┘└──┬────┘
   └──────┴──────┴──────┴──────┴──────┘
                    │
         ┌──────────▼──────────┐
         │    SharedMemory      │
         │  findings/vulns/     │
         │  services/techs/     │
         │  credentials         │
         └─────────────────────┘
                    │
         ┌──────────▼──────────┐
         │   AI Client Layer    │
         │  OpenAI / DeepSeek / │
         │  Qwen / Claude / ... │
         └─────────────────────┘
```

### 4.2 Agent 双体系

| 体系 | 基类 | 特点 | 用途 |
|------|------|------|------|
| SmartAgent | `SmartAgent(ABC)` | AI 推理驱动，动态决策，带思考回调 | 主要 Agent（信息收集/漏洞/Web/后渗透/报告） |
| BaseAgent | `BaseAgent(ABC)` | 传统任务型，固定流程 | 兼容旧版 Agent |

---

## 五、核心模块详解

### 5.1 AI 客户端（agents/ai_client.py）

**供应商注册表（PROVIDER_REGISTRY）**：

| 供应商 ID | 类 | 默认模型 | Base URL |
|-----------|-----|---------|----------|
| openai | OpenAIClient | gpt-4 | api.openai.com/v1 |
| deepseek | DeepSeekClient | deepseek-chat | api.deepseek.com/v1 |
| siliconflow | SiliconFlowClient | Qwen2.5-72B | api.siliconflow.cn/v1 |
| qwen | QwenClient | qwen-turbo | dashscope.aliyuncs.com |
| zhipu | ZhipuClient | glm-4-flash | open.bigmodel.cn |
| kimi | KimiClient | moonshot-v1-8k | api.moonshot.cn/v1 |
| claude | ClaudeClient | claude-3-5-sonnet | api.anthropic.com/v1 |

**关键设计**：
- `OpenAICompatibleClient`：统一处理 6 个 OpenAI 兼容供应商，消除 200+ 行重复代码
- `ClaudeClient`：独立实现（Anthropic Messages API 格式不同）
- `MockAIClient`：无 API Key 时的降级模拟
- `RateLimiter`：线程安全速率限制（20次/分钟，200次/小时）
- `AISession`：多轮对话历史管理（最近 N 轮，防 token 超限）
- `AIAgentFactory`：工厂模式创建客户端

### 5.2 SmartAgent 基类（agents/base_agent.py）

SmartAgent 的 AI 推理执行流程（`execute_with_reasoning`）：

```
1. _analyze_situation()   → 分析目标类型、可用工具、隐蔽模式
2. _ai_plan() / _build_plan_from_tasks()  → AI 制定或编排器指定执行计划
3. _execute_plan()        → 按步骤执行（调用 _do_xxx 方法）
4. _ai_evaluate()         → AI 评估结果（可信度、安全问题、关键发现）
5. _ai_decide_next()      → AI 决定下一步（推荐 Agent、停止条件）
6. _extract_context()     → 提取上下文更新（端口/漏洞/技术/子域名）
```

**思考回调系统**（`_emit_thinking`）：
- 类型：`analyze / plan / action / evaluate / decide / command / error / info`
- 通过回调函数实时推送到前端 SSE 流

**工具执行**（`_run_tool` / `_run_tool_async`）：
- 自动检查工具是否安装（`shutil.which`）
- 自动发出工具调用 + 命令语法 + 结果三类思考事件
- 支持同步和异步两种执行方式

**AI 降级策略**（`_generate_default_response`）：
- AI 不可用时，使用知识图谱（`PentestKnowledgeGraph`）推理默认响应

### 5.3 工作流编排器（agents/workflow_orchestrator.py）

**执行流程**：

```
execute_intelligent_workflow(target, options)
    │
    ├─ _ai_initial_analysis()     → 分析目标类型/技术栈/攻击面
    │      └─ 降级: _knowledge_graph_initial_analysis()
    │
    ├─ _ai_create_strategy()      → 制定多阶段执行策略
    │      └─ 降级: _knowledge_graph_strategy()
    │
    ├─ _execute_dynamic_workflow() → 动态执行各阶段
    │      ├─ 每阶段: _execute_phase()
    │      │      ├─ 并行 Agent: asyncio.gather()
    │      │      └─ 顺序 Agent: 逐个执行
    │      ├─ _ai_should_continue() → AI 决定是否继续
    │      └─ _ai_adjust_strategy() → AI 动态调整策略
    │
    └─ _ai_generate_report()      → 生成报告摘要
```

**默认执行阶段（Web 应用目标）**：

| 阶段 | Agent | 并行 | 超时 |
|------|-------|------|------|
| 信息收集 | info_gathering | 否 | 300s |
| 并行扫描 | vuln_scanner + web_pentest + brute_force | 是 | 600s |
| 后渗透 | post_exploitation | 否 | 600s |
| 报告生成 | report | 否 | 120s |

**多目标并发**：`asyncio.Semaphore` 控制最大并发数（默认 3），每个目标独立 SharedMemory。

**用户交互**：
- 暂停/恢复：`_wait_if_paused()` 轮询 SCANS 状态
- 实时对话：`_handle_user_message()` 让 AI 分析用户指令并调整策略

### 5.4 知识图谱（agents/knowledge_graph.py）

`PentestKnowledgeGraph` 包含 30+ 节点，分为：

| 类别 | 节点示例 |
|------|---------|
| 端口/服务 | open_port_80/443/22/21/3306/6379/27017/3389/8080 |
| 技术栈 | wordpress/apache/nginx/iis/php/asp_net/nodejs/spring |
| 漏洞 | sql_injection/xss/lfi/rfi/ssrf/rce/xxe/file_upload/csrf/weak_credential |
| 配置问题 | git_exposure/env_exposure/debug_mode/cors_misconfig/directory_listing |

**核心方法**：
- `get_next_actions(findings)` → 根据发现推荐下一步动作（按优先级排序）
- `get_chain_actions(keys, depth=2)` → 深度推理链式动作
- `classify_and_recommend(findings)` → 综合分类 + 推荐接口
- `_calculate_priority()` → 基于严重程度 + 动作类型计算优先级

### 5.5 共享内存（agents/shared_memory.py）

`SharedMemory` 是 Agent 间的统一数据总线：

**存储结构**：
```python
_context = {
    'findings': [],          # 所有发现
    'vulnerabilities': [],   # 漏洞列表
    'attack_surface': {},    # 攻击面（按端口索引）
    'services': {},          # 服务信息（按端口索引）
    'technologies': [],      # 技术栈
    'credentials': [],       # 凭据
    'execution_history': [], # 执行历史
    'key_findings': [],      # 关键发现（critical/high）
    'config_issues': [],     # 配置问题
}
```

**关键特性**：
- `asyncio.Lock` 保护所有写操作
- `get_context()` 返回深拷贝，隔离修改
- 发布-订阅模式：`subscribe(event_type, callback)` 支持事件通知
- `update_from_agent_output()` 智能解析 Agent 输出，自动分类写入
- `get_summary()` 同步方法，供 AI 决策使用

### 5.6 AI 配置管理（config/ai_manager.py）

`AIConfigManager` 功能：
- 加载/保存 `config/ai_config.json`
- `get_active_provider()` 自动找到有效 API Key 的供应商
- `test_connection()` 测试 AI 服务连通性（含延迟测量）
- `get_all_provider_status()` 返回所有供应商配置状态
- `save_provider()` 保存供应商配置（API Key / 模型 / Base URL）

---

## 六、Web API 接口

| 端点 | 方法 | 说明 |
|------|------|------|
| `/` | GET | 主页面（index.html） |
| `/api/scan/start` | POST | 启动渗透测试扫描 |
| `/api/scan/stream/<scan_id>` | GET | SSE 实时推流（思考事件） |
| `/api/scan/pause/<scan_id>` | POST | 暂停扫描 |
| `/api/scan/resume/<scan_id>` | POST | 恢复扫描 |
| `/api/scan/stop/<scan_id>` | POST | 停止扫描 |
| `/api/chat` | POST | AI 对话（Blueprint，SSE 流式） |
| `/api/report/generate` | POST | 生成报告 |
| `/api/report/download/<id>` | GET | 下载报告（路径穿越校验） |
| `/api/tools/status` | GET | 工具可用状态 |
| `/api/tools/install` | POST | 安装工具（需认证） |
| `/api/ai/providers` | GET | 获取供应商列表 |
| `/api/ai/config` | GET/POST | 获取/保存 AI 配置 |
| `/api/ai/test` | POST | 测试 AI 连接 |
| `/api/ai/models` | GET | 获取供应商模型列表 |

**安全机制**：
- 工具安装接口需要令牌认证（`_check_auth()`）
- 本地/局域网 IP 免认证
- 启动时生成随机 `_ACCESS_TOKEN`（可通过环境变量 `NEURALSTRIKE_TOKEN` 自定义）

---

## 七、数据流程图

### 7.1 渗透测试完整流程

```
用户输入目标 URL/IP
        │
        ▼
  POST /api/scan/start
        │
        ▼
  WorkflowOrchestrator.execute_intelligent_workflow()
        │
        ├──► AI 初始分析（目标类型/技术栈/攻击面）
        │         └── 降级：知识图谱推理
        │
        ├──► AI 制定策略（多阶段 + 并行规划）
        │         └── 降级：知识图谱默认策略
        │
        ├──► 阶段1：信息收集（SmartInfoGatheringAgent）
        │         ├── nmap 端口扫描
        │         ├── whatweb/curl 指纹识别
        │         ├── gobuster/dirsearch 目录扫描
        │         └── amass/subfinder 子域名枚举
        │         └──► 写入 SharedMemory
        │
        ├──► 阶段2：并行扫描（asyncio.gather）
        │         ├── VulnScannerAgent（nuclei/CVE/SSL）
        │         ├── WebPentestAgent（SQLi/XSS/SSRF/XXE/CSRF/文件上传）
        │         └── BruteForceAgent（hydra/默认凭据）
        │         └──► 写入 SharedMemory
        │
        ├──► 阶段3：后渗透（SmartPostExploitationAgent）
        │         ├── 本地信息收集（/etc/passwd, /etc/shadow）
        │         ├── 权限提升（SUID/GTFOBins/内核漏洞）
        │         ├── 凭据窃取（.bash_history/配置文件）
        │         └── 文件发现（敏感配置/数据库/备份）
        │         └──► 写入 SharedMemory
        │
        └──► 阶段4：报告生成（SmartReportAgent）
                  ├── 读取 SharedMemory 全量数据
                  ├── 生成 15 种漏洞 POC
                  ├── 三级模板（简报/标准/专家）
                  └── 导出 MD/TXT/PDF/HTML
```

### 7.2 SSE 实时推流流程

```
Agent._emit_thinking(type, content)
        │
        ▼
WorkflowOrchestrator._thinking_callback()
        │
        ▼
web_server: SSE Queue.put(event)
        │
        ▼
GET /api/scan/stream/<scan_id>  (text/event-stream)
        │
        ▼
前端 app.js EventSource 接收
        │
        ▼
渲染到 AI 思考面板（10 种事件类型）
```

### 7.3 AI 决策降级链

```
AI 调用（带 20s 超时）
    │
    ├── 成功 → 解析 JSON 响应 → 执行
    │
    └── 失败/超时
            │
            ├── 知识图谱推理（PentestKnowledgeGraph）
            │       └── 基于发现类型推荐动作
            │
            └── 硬编码默认策略（Web/IP 两种模板）
```

---

## 八、Agent 能力矩阵

| Agent | 主要工具 | 核心任务 |
|-------|---------|---------|
| InfoGatheringAgent | nmap, masscan, whatweb, gobuster, dirsearch, amass, subfinder, dnsrecon | 端口扫描、服务检测、Web 指纹、目录枚举、子域名发现 |
| VulnScannerAgent | nuclei, nikto, testssl.sh, curl | CVE 检测、Web 漏洞扫描、SSL/TLS 检测、敏感路径探测 |
| WebPentestAgent | sqlmap, dalfox, commix, curl | SQL 注入、XSS、命令注入、SSRF、XXE、CSRF、文件上传、路径穿越 |
| BruteForceAgent | hydra, crowbar, crackmapexec | SSH/FTP/HTTP/RDP/MySQL 暴力破解、默认凭据测试 |
| PostExploitationAgent | 系统命令、find、cat | 本地信息收集、SUID 提权、凭据窃取、敏感文件发现 |
| ReportAgent | weasyprint, markdown | 漏洞 POC 生成、三级报告模板、多格式导出、12 种解码器 |

---

## 九、安全设计

| 机制 | 实现 |
|------|------|
| 命令注入防护 | `shlex.split()` 替代 `shell=True` |
| API Key 保护 | `.gitignore` 排除 `config/ai_config.json` |
| 访问控制 | 随机令牌 + 局域网 IP 白名单 |
| 路径穿越防护 | 报告下载路径校验 |
| 速率限制 | `RateLimiter`（20次/分钟，200次/小时） |
| 工具安装认证 | 高危接口需 Bearer Token |
| SSE 跨域 | 仅允许同源访问 |

---

## 十、部署方式

### 方式1：Kali Linux 一键部署（推荐）
```bash
cd /opt
git clone <repo>
cd NeuralStrike
chmod +x deploy_kali.sh
sudo ./deploy_kali.sh
python3 main.py
```

### 方式2：Docker
```bash
docker build -t neuralstrike .
docker run -p 7777:7777 --cap-add=NET_RAW neuralstrike
```

### 方式3：手动安装
```bash
pip install flask requests beautifulsoup4 lxml weasyprint
sudo apt install nmap nuclei sqlmap hydra gobuster ...
python3 main.py
```

启动后访问：`http://localhost:7777`  
从 Windows 访问 Kali：`http://<Kali-IP>:7777`

### 启动参数
```bash
python3 main.py --port=8080   # 自定义端口
python3 main.py --host=0.0.0.0  # 监听所有网卡
```

---

## 十一、环境要求

| 组件 | 最低 | 推荐 |
|------|------|------|
| 操作系统 | Ubuntu 20.04+ | Kali Linux 2024+ |
| Python | 3.9+ | 3.11+ |
| 内存 | 2GB | 4GB+ |
| 磁盘 | 1GB | 5GB+（含工具） |
| 网络 | 需访问 AI API | — |

---

## 十二、全部可调用工具清单

### 12.1 端口扫描（port_scan）

| 工具 | 安装方式 | 说明 |
|------|---------|------|
| nmap | apt install nmap | 端口扫描 / 服务版本检测 / OS 识别 |
| masscan | apt install masscan | 高速大规模端口扫描 |
| python-nmap | pip install python-nmap | nmap Python 绑定库 |

### 12.2 漏洞扫描（vuln_scan）

| 工具 | 安装方式 | 说明 |
|------|---------|------|
| nuclei | apt / go install | 基于模板的漏洞扫描（CVE/配置/Web） |
| nikto | apt install nikto | Web 服务器漏洞扫描 |
| wpscan | apt install wpscan | WordPress 专项扫描（插件/主题/用户枚举） |
| openvas | apt install openvas | 综合漏洞扫描平台 |

### 12.3 指纹识别（fingerprint）

| 工具 | 安装方式 | 说明 |
|------|---------|------|
| whatweb | apt install whatweb | Web 技术栈指纹识别 |

### 12.4 目录扫描（dir_scan）

| 工具 | 安装方式 | 说明 | 优先级 |
|------|---------|------|--------|
| feroxbuster | apt install feroxbuster | 高速递归目录爆破（Rust） | 1 |
| gobuster | apt / go install | 目录/DNS/VHost 爆破 | 2 |
| dirsearch | apt install dirsearch | 目录扫描（Python） | 3 |
| dirb | apt install dirb | 经典目录扫描 | 4 |
| ffuf | apt / go install | 高速 Web Fuzzer | 5 |

### 12.5 子域名枚举（subdomain）

| 工具 | 安装方式 | 说明 |
|------|---------|------|
| subfinder | apt / go install | 被动子域名发现 |
| amass | apt install amass | 主动+被动子域名枚举 |

### 12.6 SQL 注入（sqli_scan）

| 工具 | 安装方式 | 说明 |
|------|---------|------|
| sqlmap | apt install sqlmap | 自动化 SQL 注入检测与利用 |

### 12.7 XSS 扫描（xss_scan）

| 工具 | 安装方式 | 说明 |
|------|---------|------|
| dalfox | apt / go install | 参数驱动 XSS 扫描与验证 |

### 12.8 SSL/TLS 扫描（ssl_scan）

| 工具 | 安装方式 | 说明 |
|------|---------|------|
| sslyze | pip install sslyze | SSL/TLS 配置分析（Python） |
| testssl / testssl.sh | apt install testssl.sh | 全面 SSL/TLS 检测脚本 |

### 12.9 DNS 工具（dns）

| 工具 | 安装方式 | 说明 |
|------|---------|------|
| dig | apt install dnsutils | DNS 查询工具 |
| dnsrecon | apt install dnsrecon | DNS 枚举（区域传输/子域名/反查） |

### 12.10 暴力破解（brute_force）

| 工具 | 安装方式 | 说明 |
|------|---------|------|
| hydra | apt install hydra | 多协议在线暴力破解（SSH/FTP/HTTP/RDP/MySQL） |
| medusa | apt install medusa | 并行暴力破解 |
| ncrack | apt install ncrack | 网络认证破解 |
| john | apt install john | 离线密码破解（John the Ripper） |
| hashcat | apt install hashcat | GPU 加速密码破解 |
| crowbar | apt install crowbar | RDP/SSH/VNC 暴力破解 |
| patator | apt install patator | 多用途暴力破解框架 |
| pymysql | pip install pymysql | MySQL Python 库（用于数据库暴破） |
| psycopg2 | pip install psycopg2-binary | PostgreSQL Python 库 |
| pymssql | pip install pymssql | MSSQL Python 库 |
| paramiko | pip install paramiko | SSH Python 库（用于 SSH 暴破） |

### 12.11 后渗透（post_exploit）

| 工具 | 安装方式 | 说明 |
|------|---------|------|
| linpeas | apt install peass-ng | Linux 本地提权信息收集脚本 |
| winpeas | apt install peass-ng | Windows 本地提权信息收集脚本 |
| crackmapexec / nxc | apt / pip | 内网横向移动（SMB/WMI/LDAP） |
| chisel | go install | TCP/UDP 隧道工具（内网穿透） |
| socat | apt install socat | 多功能网络中继/反弹 Shell |
| proxychains / proxychains4 | apt install proxychains4 | 代理链（SOCKS/HTTP） |
| msfvenom / msfconsole | apt install metasploit-framework | Metasploit 载荷生成与利用框架 |
| impacket | pip install impacket | Windows 协议套件（SMB/LDAP/Kerberos） |
| impacket-smbexec | apt install impacket-scripts | SMB 远程执行 |
| impacket-wmiexec | apt install impacket-scripts | WMI 远程执行 |
| impacket-psexec | apt install impacket-scripts | PsExec 风格远程执行 |
| bloodhound / bloodhound-ce | apt / pip | AD 域攻击路径分析 |
| mimikatz | 手动安装 | Windows 凭据提取（需手动部署） |
| responder | apt install responder | LLMNR/NBT-NS 毒化 + 凭据捕获 |
| smbclient | apt install samba | SMB 文件共享访问 |
| enum4linux | apt install enum4linux | Windows/Samba 信息枚举 |

### 12.12 网络工具（network）

| 工具 | 安装方式 | 说明 |
|------|---------|------|
| curl | apt install curl | HTTP 请求工具（Web 测试核心） |
| wget | apt install wget | 文件下载 |
| arp-scan | apt install arp-scan | 局域网主机发现 |
| traceroute | apt install traceroute | 路由追踪 |
| whois | apt install whois | 域名/IP 注册信息查询 |
| netcat / nc | apt install netcat | 网络瑞士军刀（反弹 Shell/端口监听） |
| nbtscan | apt install nbtscan | NetBIOS 名称扫描 |
| scapy | pip install scapy | Python 网络包构造与分析库 |

### 12.13 Web 扫描（web_scan）

| 工具 | 安装方式 | 说明 |
|------|---------|------|
| beautifulsoup4 | pip install beautifulsoup4 | HTML 解析库（用于 Web 内容分析） |

---

### 工具优先级选择策略

各类别中，系统按以下优先级自动选择最佳可用工具：

| 类别 | 优先顺序 |
|------|---------|
| 端口扫描 | nmap → masscan → python-nmap |
| 漏洞扫描 | nuclei → nikto → wpscan → openvas |
| 目录扫描 | feroxbuster → gobuster → dirsearch → dirb → ffuf |
| 子域名 | subfinder → amass |
| SSL 扫描 | sslyze → testssl |
| 暴力破解 | hydra → medusa → ncrack → john → hashcat → crowbar → patator |
| 后渗透 | msfvenom → crackmapexec → impacket → bloodhound → chisel → responder → socat |
| 网络工具 | curl → wget → arp-scan → traceroute → whois → netcat → nbtscan → scapy |

---

## 十三、免责声明

本工具仅供**合法授权的安全测试**使用。使用者必须获得目标系统所有者的明确书面授权，遵守当地法律法规。未经授权的渗透测试是违法行为。
