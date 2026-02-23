# Digital Workers

基于 LLM 的仓库智能助手，自动监控 GitHub/GitCode 仓库，以你的风格回答 Issue 并审查 PR。

## 功能特性

- 🔔 **自动监控**: 定时检查指定仓库的新 Issue 和 PR
- 💬 **智能回复**: 使用 LLM 以你定义的风格自动回复 Issue
- 👀 **PR 审查**: 自动审查 PR，简单问题直接评论，复杂问题通知你
- 🌐 **多平台支持**: 支持 GitHub 和 GitCode（可扩展更多平台）
- 🎭 **风格定制**: 每个仓库可独立配置回复风格
- 💾 **状态追踪**: SQLite 存储已处理记录，避免重复回复
- 🖥️ **后台服务**: 支持 Windows 服务模式，24/7 运行

---

## 快速开始

### 1. 环境要求

- Python 3.11+
- Chrome 浏览器（已登录 GitHub/GitCode 账号）
- OpenRouter API Key

### 2. 安装

```bash
# 克隆项目
git clone https://github.com/yourname/DigitalWorkers.git
cd DigitalWorkers

# 方式一：使用 Conda（推荐）
conda env create -f environment.yml
conda activate digital-workers

# 方式二：使用 pip
pip install -r requirements.txt

# 安装 Playwright 浏览器
playwright install chromium
```

### 3. 配置

#### 3.1 设置 API Key

```bash
# Windows CMD
set OPENROUTER_API_KEY=sk-or-your-api-key-here

# Windows PowerShell
$env:OPENROUTER_API_KEY = "sk-or-your-api-key-here"

# Linux/Mac
export OPENROUTER_API_KEY=sk-or-your-api-key-here
```

#### 3.2 配置仓库

编辑 `config/repos/` 目录下的 JSON 文件：

**GitHub 仓库示例** (`config/repos/verl.json`):
```json
{
  "repo_url": "https://github.com/verl-project/verl",
  "platform": "github",
  "enabled": true,
  "style_prompt": "你是一个有帮助的助手。请专业、简洁、建设性地回答问题。",
  "max_replies_per_run": 5,
  "check_issues": true,
  "check_prs": true
}
```

**GitCode 仓库示例** (`config/repos/mindspeed-mm.json`):
```json
{
  "repo_url": "https://gitcode.com/Ascend/MindSpeed-MM",
  "platform": "gitcode",
  "enabled": true,
  "style_prompt": "你是一个有帮助的助手。请专业、简洁、建设性地回答问题。",
  "max_replies_per_run": 5,
  "check_issues": true,
  "check_prs": true
}
```

#### 3.3 主配置文件

编辑 `config/config.json`：

```json
{
  "schedule_interval_hours": 4,
  "max_replies_per_run": 5,
  "openrouter_api_key": "${OPENROUTER_API_KEY}",
  "default_model": "anthropic/claude-3-haiku",
  "chrome_user_data_dir": null,
  "log_level": "INFO",
  "repos_config_dir": "config/repos"
}
```

| 配置项 | 说明 | 默认值 |
|--------|------|--------|
| `schedule_interval_hours` | 检查间隔（小时） | 4 |
| `max_replies_per_run` | 每次最多回复数量 | 5 |
| `openrouter_api_key` | OpenRouter API Key | 从环境变量读取 |
| `default_model` | 使用的 LLM 模型 | anthropic/claude-3-haiku |
| `chrome_user_data_dir` | Chrome 用户数据目录 | 自动检测 |
| `log_level` | 日志级别 | INFO |

### 4. 运行

```bash
# 单次运行（测试用）
python -m src.main --once

# 启动定时服务
python -m src.main

# 指定配置文件
python -m src.main --config path/to/config.json

# 查看帮助
python -m src.main --help
```

---

## Windows 服务安装

### 安装服务

```powershell
# 以管理员身份运行 PowerShell
powershell -ExecutionPolicy Bypass -File scripts\install_service.ps1
```

### 管理服务

```powershell
# 启动服务
Start-Service DigitalWorkers

# 停止服务
Stop-Service DigitalWorkers

# 查看状态
Get-Service DigitalWorkers

# 查看日志
Get-Content logs\service.log -Tail 50
```

### 卸载服务

```powershell
powershell -ExecutionPolicy Bypass -File scripts\uninstall_service.ps1
```

---

## 项目结构

```
DigitalWorkers/
├── src/                          # 源代码
│   ├── main.py                   # 主入口点
│   ├── config.py                 # 配置加载
│   ├── scheduler.py              # APScheduler 调度
│   ├── logging_config.py         # 日志配置
│   ├── browser/                  # 浏览器自动化
│   │   └── chrome.py             # Playwright + Chrome
│   ├── llm/                      # LLM 集成
│   │   ├── base.py               # 抽象接口
│   │   ├── openrouter.py         # OpenRouter 客户端
│   │   └── prompts.py            # Prompt 模板
│   ├── platforms/                # 平台处理器
│   │   ├── base.py               # 抽象基类
│   │   ├── github.py             # GitHub 实现
│   │   └── gitcode.py            # GitCode 实现
│   ├── handlers/                 # 业务逻辑
│   │   ├── issue.py              # Issue 处理
│   │   └── pr.py                 # PR 处理
│   ├── notifications/            # 通知系统（占位）
│   │   └── base.py               # 接口定义
│   ├── storage/                  # 数据存储
│   │   └── sqlite.py             # SQLite 操作
│   └── utils/                    # 工具函数
│       └── retry.py              # 重试逻辑
├── config/                       # 配置文件
│   ├── config.json               # 主配置
│   ├── config.example.json       # 示例配置
│   └── repos/                    # 仓库配置
├── scripts/                      # 脚本
│   ├── install_service.ps1       # 安装 Windows 服务
│   ├── uninstall_service.ps1     # 卸载服务
│   └── run_service.py            # 服务入口
├── tests/                        # 测试文件
├── data/                         # SQLite 数据库
├── logs/                         # 日志文件
├── pyproject.toml                # 项目配置
├── environment.yml               # Conda 环境
└── requirements.txt              # pip 依赖
```

---

## 自定义回复风格

在仓库配置中修改 `style_prompt` 字段：

**专业风格**:
```json
"style_prompt": "你是一个专业的开发者。回答时使用技术术语，提供代码示例，并引用相关文档。"
```

**友好风格**:
```json
"style_prompt": "你是一个友好的社区助手。用简单易懂的语言解释问题，鼓励提问者。"
```

**中文风格**:
```json
"style_prompt": "你是一个有帮助的助手。用中文回答问题，保持专业和礼貌。"
```

---

## PR 审查策略

系统会自动判断 PR 是否需要通知你：

**自动评论** (不通知):
- 修改行数 < 500
- 修改文件 < 20
- 不涉及安全相关文件

**通知你** (不自动评论):
- 修改行数 > 500
- 修改文件 > 20
- 涉及安全相关文件（auth, password, secret, key 等）

---

## 添加新仓库

1. 在 `config/repos/` 目录创建新的 JSON 文件
2. 配置仓库 URL、平台和风格
3. 重启服务或等待下次调度

**示例**:
```json
{
  "repo_url": "https://github.com/your-org/your-repo",
  "platform": "github",
  "enabled": true,
  "style_prompt": "你的风格描述",
  "max_replies_per_run": 5,
  "check_issues": true,
  "check_prs": true
}
```

---

## 支持的 LLM 模型

通过 OpenRouter 支持多种模型：

| 模型 | 说明 |
|------|------|
| `anthropic/claude-3-haiku` | 快速、经济（默认） |
| `anthropic/claude-3-sonnet` | 平衡性能和成本 |
| `openai/gpt-4-turbo` | OpenAI GPT-4 |
| `google/gemini-pro` | Google Gemini |
| `meta-llama/llama-3-70b` | Meta LLaMA |

修改 `config/config.json` 中的 `default_model` 即可切换。

---

## 开发

### 运行测试

```bash
# 运行所有测试
pytest tests/ -v

# 运行特定测试
pytest tests/test_config.py -v
```

### 代码检查

```bash
# 类型检查
mypy src/

# 代码风格
ruff check src/

# 格式化
ruff format src/
```

---

## 故障排除

### Chrome 未登录

确保 Chrome 浏览器已登录 GitHub/GitCode 账号。程序会使用你的登录会话。

### API Key 无效

检查环境变量是否正确设置：
```bash
echo %OPENROUTER_API_KEY%  # Windows CMD
echo $env:OPENROUTER_API_KEY  # PowerShell
```

### 浏览器自动化失败

1. 确保安装了 Playwright：
```bash
playwright install chromium
```

2. 关闭所有 Chrome 窗口后再运行

### 日志查看

```bash
# 实时查看日志
Get-Content logs\assistant.log -Tail 50 -Wait

# 或直接打开
notepad logs\assistant.log
```

---

## 许可证

MIT License

---

## 背景

在试用了 OpenClaw 之后，发现其 token 消耗特别多，除此之外在调用工具时（比如控制 chrome），不是插件有问题，就是 windows 权限不足。

作为一个普通人，可能用不起顶尖的模型，也不允许 agents 这么大量的烧 token。

但是作为程序员，我能利用免费的普通模型 + OpenCode 编程 来穿刺我真正需要的场景，并通过软件开发的文档和测试（UT/CI）来看护功能。


## TODO

1. AI 评论 gitee，github
2. 帮我每日监控和收集信息：
		1. 我想做的开源项目：opencode+飞书，
		2. 金融消息？