# Browser Use 三大方案完整使用教程

> **面向**: 想立刻上手的开发者  
> **前置阅读**: [对比报告](BROWSER_USE_COMPARISON_REPORT.md)（可选）  
> **日期**: 2026-03-31

---

## 目录

- [Part 1: Playwright MCP 使用教程](#part-1-playwright-mcp-使用教程)
- [Part 2: browser-use 使用教程](#part-2-browser-use-使用教程)
- [Part 3: Stagehand 使用教程](#part-3-stagehand-使用教程)
- [Part 4: 场景速查表](#part-4-场景速查表)

---

# Part 1: Playwright MCP 使用教程

> 微软官方 MCP Server，通过无障碍树（非截图）让 AI 操控浏览器。零 LLM 额外成本。

## 1.1 安装

### Claude Code（推荐，一行命令）
```bash
claude mcp add playwright npx @playwright/mcp@latest
```

### Claude Desktop
编辑 `~/.claude/claude_desktop_config.json`：
```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest"]
    }
  }
}
```

### VS Code
```bash
code --add-mcp '{"name":"playwright","command":"npx","args":["@playwright/mcp@latest"]}'
```

### Cursor
在 Cursor Settings → MCP → Add new MCP Server：
```json
{ "command": "npx @playwright/mcp@latest" }
```

### Docker 部署
```json
{
  "mcpServers": {
    "playwright": {
      "command": "docker",
      "args": ["run", "-i", "--rm", "--init", "--pull=always", "mcr.microsoft.com/playwright/mcp"]
    }
  }
}
```

### HTTP 远程模式
```bash
# 服务器端
npx @playwright/mcp@latest --port 8931 --headless

# 客户端配置
{ "mcpServers": { "playwright": { "url": "http://your-server:8931/mcp" } } }
```

## 1.2 配置选项

### CLI 参数速查

| 参数 | 说明 | 默认值 |
|------|------|--------|
| `--browser <name>` | chromium / firefox / webkit / chrome / msedge | chromium |
| `--headless` | 无头模式（无界面） | false |
| `--caps <list>` | 额外能力: vision, pdf, storage, network, devtools, testing | core |
| `--user-data-dir <path>` | 用户数据目录（持久化登录状态） | 临时目录 |
| `--isolated` | 隔离会话（不保存到磁盘） | false |
| `--port <port>` | HTTP/SSE 监听端口 | 无（stdio 模式） |
| `--extension` | Chrome 扩展模式（连接已打开的浏览器） | false |
| `--proxy-server <url>` | 代理服务器 | 无 |
| `--device <name>` | 设备模拟，如 "iPhone 15" | 无 |
| `--viewport-size <WxH>` | 视口大小，如 "1280x720" | 浏览器默认 |
| `--timeout-action <ms>` | 操作超时 | 5000 |
| `--timeout-navigation <ms>` | 导航超时 | 60000 |
| `--config <path>` | JSON 配置文件路径 | 无 |
| `--no-sandbox` | 禁用沙箱（Docker/CI 必需） | false |
| `--init-script <path>` | 页面初始化脚本 | 无 |

### 配置文件示例 (config.json)

```json
{
  "browser": {
    "browserName": "chromium",
    "launchOptions": { "headless": true, "channel": "chrome" },
    "contextOptions": { "viewport": { "width": 1280, "height": 720 } },
    "userDataDir": "~/.pw-profile"
  },
  "capabilities": ["core", "pdf", "storage", "network"],
  "network": {
    "allowedOrigins": ["https://example.com"],
    "blockedOrigins": ["https://ads.example.com"]
  },
  "secrets": { "api_key": "sk-***", "password": "***" },
  "timeouts": { "action": 5000, "navigation": 60000 }
}
```

使用: `npx @playwright/mcp@latest --config config.json`

## 1.3 核心工具使用

### 导航

| 工具 | 参数 | 说明 |
|------|------|------|
| `browser_navigate` | `{ url }` | 导航到 URL |
| `browser_navigate_back` | 无 | 返回上一页 |
| `browser_wait_for` | `{ time?, text?, textGone? }` | 等待条件 |

### 元素交互

| 工具 | 参数 | 说明 |
|------|------|------|
| `browser_click` | `{ ref, element?, button?, modifiers? }` | 点击元素 |
| `browser_type` | `{ ref, text, submit?, slowly? }` | 输入文本 |
| `browser_select_option` | `{ ref, values }` | 选择下拉项 |
| `browser_fill_form` | `{ fields: [{ref, value}] }` | 批量填充表单 |
| `browser_hover` | `{ ref }` | 悬停 |
| `browser_drag` | `{ startRef, endRef }` | 拖拽 |
| `browser_press_key` | `{ key }` | 按键（Enter/Tab/...） |
| `browser_file_upload` | `{ paths }` | 上传文件 |
| `browser_handle_dialog` | `{ accept, promptText? }` | 处理对话框 |

> **ref** 值从 `browser_snapshot` 返回的页面快照中获取，如 `ref=e2`。

### 页面信息

| 工具 | 参数 | 说明 |
|------|------|------|
| `browser_snapshot` | `{ selector?, depth? }` | 页面无障碍树快照 |
| `browser_take_screenshot` | `{ fullPage?, type?, ref? }` | 截图 |
| `browser_console_messages` | `{ level? }` | 获取控制台消息 |
| `browser_network_requests` | `{ filter?, requestBody? }` | 列出网络请求 |
| `browser_evaluate` | `{ function }` | 执行页面 JS |

### 标签页

| 工具 | 参数 | 说明 |
|------|------|------|
| `browser_tabs` | `{ action: "list"/"create"/"close"/"select" }` | 标签页管理 |

### 存储（需 `--caps=storage`）

| 工具 | 说明 |
|------|------|
| `browser_cookie_get/set/delete/list/clear` | Cookie CRUD |
| `browser_localstorage_get/set/delete/list/clear` | localStorage CRUD |
| `browser_sessionstorage_get/set/delete/list/clear` | sessionStorage CRUD |
| `browser_storage_state` | 导出存储快照 |
| `browser_set_storage_state` | 导入存储快照 |

### 网络模拟（需 `--caps=network`）

| 工具 | 说明 |
|------|------|
| `browser_route` | Mock 网络请求（拦截并返回自定义响应） |
| `browser_route_list` | 列出所有 mock 路由 |
| `browser_unroute` | 移除 mock 路由 |
| `browser_network_state_set` | 设置离线/在线模式 |

### Vision 坐标交互（需 `--caps=vision`）

| 工具 | 说明 |
|------|------|
| `browser_mouse_click_xy` | XY 坐标点击 |
| `browser_mouse_move_xy` | 移动鼠标 |
| `browser_mouse_drag_xy` | XY 坐标拖拽 |
| `browser_mouse_wheel` | 鼠标滚轮 |

### DevTools（需 `--caps=devtools`）

| 工具 | 说明 |
|------|------|
| `browser_start_video` / `browser_stop_video` | 视频录制 |
| `browser_start_tracing` / `browser_stop_tracing` | 性能追踪 |

## 1.4 典型场景

### 场景 1：在 Claude Code 中搜索网页

```
你: "打开 GitHub 搜索 playwright-mcp 项目"

Claude 自动执行:
→ browser_navigate("https://github.com")
→ browser_snapshot() → 获取页面结构，找到搜索框 ref=e5
→ browser_click(ref="e5")
→ browser_type(ref="e5", text="playwright-mcp")
→ browser_press_key(key="Enter")
→ browser_snapshot() → 返回搜索结果给你
```

### 场景 2：登录网站并保持状态

```bash
# 使用持久化 profile，首次登录后自动保存
npx @playwright/mcp@latest --user-data-dir ~/.pw-profile
```

```
你: "登录到 example.com"

Claude:
→ browser_navigate("https://example.com/login")
→ browser_snapshot() → 找到用户名(e1)、密码(e2)、登录按钮(e3)
→ browser_type(ref="e1", text="user@example.com")
→ browser_type(ref="e2", text="password123")
→ browser_click(ref="e3")

下次启动时，登录状态自动恢复（因为用了 --user-data-dir）
```

### 场景 3：Chrome 扩展模式（复用已登录浏览器）

```bash
# 1. 安装 Chrome 扩展: Playwright MCP Bridge
# 2. 配置
npx @playwright/mcp@latest --extension
```

```
你: "帮我在已打开的 Gmail 里搜索来自 boss 的邮件"

Claude 直接操作你的浏览器（无需重新登录）
→ browser_snapshot() → 获取当前 Gmail 页面
→ browser_click(ref="搜索框")
→ browser_type(text="from:boss")
→ browser_press_key(key="Enter")
→ browser_snapshot() → 返回搜索结果
```

### 场景 4：Mock API + 截图

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest", "--caps=core,network,pdf"]
    }
  }
}
```

```
你: "Mock 掉 /api/users 返回测试数据，然后截图保存"

Claude:
→ browser_route(pattern="**/api/users", status=200, body='[{"name":"Alice"}]')
→ browser_navigate("https://app.example.com/users")
→ browser_take_screenshot(fullPage=true, filename="users.png")
→ browser_pdf_save(filename="users.pdf")
```

---

# Part 2: browser-use 使用教程

> Python 自主浏览器 Agent，让 AI 像人一样自主操控浏览器完成多步骤任务。

## 2.1 安装

### 方式一：uv（推荐）
```bash
uv init my-project && cd my-project
uv add browser-use python-dotenv
uv sync
# 安装浏览器
uvx browser-use install
```

### 方式二：pip
```bash
pip install browser-use
playwright install chromium
```

### 方式三：快速模板
```bash
uvx browser-use init --template advanced
```

## 2.2 环境变量配置 (.env)

```bash
# === 必需：至少选一个 LLM ===
BROWSER_USE_API_KEY=sk-...            # ChatBrowserUse 专用模型（推荐）
OPENAI_API_KEY=sk-...                 # OpenAI GPT-4o 等
ANTHROPIC_API_KEY=sk-ant-...          # Claude Sonnet/Haiku
GOOGLE_API_KEY=...                    # Gemini
DEEPSEEK_API_KEY=...                  # DeepSeek

# === 可选：浏览器配置 ===
BROWSER_USE_HEADLESS=false            # 是否无头模式
CHROME_PATH=/path/to/chrome           # 自定义 Chrome 路径
CHROME_USER_DATA=~/Library/Application Support/Google/Chrome
CHROME_PROFILE_DIRECTORY=Default      # Chrome 配置文件名

# === 可选：Cloud 模式 ===
BROWSER_USE_CLOUD_API_KEY=...         # Cloud 浏览器 API Key

# === 可选：日志 ===
BROWSER_USE_LOGGING_LEVEL=INFO        # DEBUG/INFO/WARNING/ERROR
```

> API Key 获取: https://cloud.browser-use.com/new-api-key

## 2.3 Python API

### 最简代码（5 行）

```python
from browser_use import Agent, ChatBrowserUse
from dotenv import load_dotenv
load_dotenv()

agent = Agent(task="Find the price of MacBook Pro on Apple.com", llm=ChatBrowserUse())
agent.run_sync()
```

### 异步运行

```python
import asyncio
from browser_use import Agent, ChatBrowserUse
from dotenv import load_dotenv
load_dotenv()

async def main():
    agent = Agent(
        task="Search for flights from NYC to LA on Google Flights",
        llm=ChatBrowserUse(),
        max_steps=50,
    )
    result = await agent.run()
    print(f"最终结果: {result.final_result()}")
    print(f"执行了 {len(result.history)} 步")

asyncio.run(main())
```

### Agent 完整参数

```python
agent = Agent(
    task="任务描述",                    # 必需
    llm=ChatBrowserUse(),              # 必需，LLM 实例
    browser=browser,                    # 可选，自定义浏览器
    browser_profile=profile,            # 可选，浏览器配置
    max_steps=100,                      # 最大步数（默认 100）
    flash_mode=False,                   # 快速模式（跳过思考，2-3 倍速）
    calculate_cost=True,                # 计算 API 成本
    page_extraction_llm=cheap_llm,      # 页面提取用的廉价模型
    vision_detail_level="low",          # 截图质量: low/high/auto
    max_history_items=10,               # 历史记录数量（降低 token）
    tools=custom_tools,                 # 自定义工具
    output_model_schema=MyModel,        # 结构化输出 Schema
    initial_actions=[                   # 初始操作
        {"open_url": "https://example.com"}
    ],
)
```

### Browser 配置

```python
from browser_use import Browser

# 基础配置
browser = Browser(headless=True)

# 使用本地 Chrome（复用登录状态）
browser = Browser(
    executable_path="/Applications/Google Chrome.app/Contents/MacOS/Google Chrome",
    user_data_dir="~/Library/Application Support/Google/Chrome",
    profile_directory="Default",
    headless=False
)

# 云浏览器
browser = Browser(
    use_cloud=True,
    cloud_proxy_country_code="us",  # 使用美国 IP
    cloud_timeout=120
)

# 代理
browser = Browser(
    proxy="http://user:pass@proxy.example.com:8080"
)
```

### 自定义工具

```python
from browser_use import Tools, ActionResult

tools = Tools()

@tools.action("Save data to file")
async def save_to_file(data: str, filename: str) -> ActionResult:
    with open(filename, "w") as f:
        f.write(data)
    return ActionResult(extracted_content=f"Saved to {filename}")

@tools.action("Send email notification")
async def send_email(to: str, subject: str) -> ActionResult:
    # 你的发邮件逻辑
    return ActionResult(extracted_content=f"Email sent to {to}")

agent = Agent(task="...", llm=ChatBrowserUse(), tools=tools)
```

### 结构化输出

```python
from pydantic import BaseModel

class Product(BaseModel):
    name: str
    price: float
    rating: float

agent = Agent(
    task="Find the top 3 laptops on Amazon",
    llm=ChatBrowserUse(),
    output_model_schema=Product
)
result = await agent.run()
products = result.structured_output  # List[Product]
```

## 2.4 CLI 命令

```bash
# 启动浏览器
browser-use open https://example.com
browser-use open https://example.com --headless  # 无头模式

# 查看页面状态
browser-use state                    # 列出所有可交互元素
browser-use screenshot               # 截图
browser-use title                    # 页面标题
browser-use html                     # 页面 HTML

# 交互
browser-use click 5                  # 点击第 5 个元素
browser-use type "搜索关键词"         # 输入文本
browser-use keys Enter               # 按键
browser-use scroll down              # 滚动

# 导航
browser-use back                     # 后退
browser-use scroll up                # 向上滚动

# Tab 管理
browser-use tabs                     # 列出标签页
browser-use switch 1                 # 切换到第 1 个标签

# Cookie 管理
browser-use cookies                  # 列出所有 cookie
browser-use cookie-set name=value    # 设置 cookie
browser-use cookies-export file.json # 导出
browser-use cookies-import file.json # 导入

# 等待
browser-use wait-for "#element"      # 等待元素出现
browser-use wait-for-text "Loading"  # 等待文本出现

# JavaScript
browser-use eval "document.title"    # 执行 JS

# Agent 任务
browser-use agent "Find today's weather in Beijing"  # 直接运行 Agent
```

## 2.5 MCP Server 集成 Claude Code

在 `~/.claude/app.json` 中添加：

```json
{
  "mcpServers": {
    "browser-use": {
      "command": "uvx",
      "args": ["browser-use[cli]", "--mcp"],
      "env": {
        "BROWSER_USE_API_KEY": "sk-your-key",
        "OPENAI_API_KEY": "sk-..."
      }
    }
  }
}
```

**集成后 Claude Code 自动获得的工具**:
- `browser_navigate` / `browser_click` / `browser_type` / `browser_scroll`
- `browser_extract_content` / `browser_get_state` / `browser_screenshot`
- `retry_with_browser_use_agent` — 完整 Agent 任务执行

## 2.6 典型场景

### 场景 1：自主数据提取

```python
agent = Agent(
    task="""Go to Hacker News, extract the top 10 stories 
    with their titles, scores, and comment counts""",
    llm=ChatBrowserUse()
)
result = await agent.run()
```

### 场景 2：表单填写

```python
agent = Agent(
    task="""Go to https://example.com/register and fill out:
    - Name: John Doe
    - Email: john@example.com
    - Password: SecurePass123
    Then click Register""",
    llm=ChatBrowserUse()
)
await agent.run()
```

### 场景 3：使用 ChatBrowserUse 专用模型（推荐）

```python
from browser_use import ChatBrowserUse

# 比通用 LLM 快 3-5 倍，便宜 10 倍
llm = ChatBrowserUse(model="bu-2-0")

agent = Agent(task="Check my GitHub notifications", llm=llm)
await agent.run()
```

### 场景 4：Cloud + 代理 + 成本追踪

```python
browser = Browser(
    use_cloud=True,
    cloud_proxy_country_code="jp"  # 日本 IP
)

agent = Agent(
    task="Check product prices on Amazon Japan",
    llm=ChatBrowserUse(),
    browser=browser,
    calculate_cost=True
)

result = await agent.run()
print(f"Total API cost: ${result.cost():.4f}")
```

### 场景 5：快速模式（跳过思考）

```python
agent = Agent(
    task="Get the weather in San Francisco",
    llm=ChatBrowserUse(),
    flash_mode=True,       # 跳过思考过程
    max_steps=20,          # 限制步数
    max_history_items=5    # 减少 token 消耗
)
await agent.run()
```

### 场景 6：并行多 Agent

```python
import asyncio

tasks = [
    "Find MacBook Pro price on Apple.com",
    "Find Galaxy S25 price on Samsung.com",
    "Find Pixel 9 price on Google Store"
]

agents = [Agent(task=t, llm=ChatBrowserUse()) for t in tasks]
results = await asyncio.gather(*[a.run() for a in agents])

for task, result in zip(tasks, results):
    print(f"{task}: {result.final_result()}")
```

---

# Part 3: Stagehand 使用教程

> AI + 代码混合模式的浏览器自动化框架，提供 act/extract/observe/agent 四层 API。

## 3.1 安装

### 要求
- **Node.js**: >=20.19.0 或 >=22.12.0（不支持 21.x）
- **包管理器**: npm / pnpm / yarn

### 安装步骤

```bash
# 方式一：创建新项目（推荐）
npx create-browser-app my-project
cd my-project

# 方式二：添加到现有项目
npm install @browserbasehq/stagehand zod dotenv
```

### 环境变量 (.env)

```bash
# === LLM API（至少选一个）===
OPENAI_API_KEY=sk-...                  # OpenAI（推荐 gpt-4o-mini）
ANTHROPIC_API_KEY=sk-ant-...           # Claude
GOOGLE_API_KEY=...                     # Gemini

# === Browserbase（可选，云端部署）===
BROWSERBASE_API_KEY=bbpk_...
BROWSERBASE_PROJECT_ID=...

# === 配置（可选）===
HEADLESS=false                         # 是否无头模式
ENABLE_CACHING=true                    # 启用操作缓存
VERBOSE=1                              # 日志级别 0-2
```

## 3.2 初始化 Stagehand

```typescript
import { Stagehand } from "@browserbasehq/stagehand";
import "dotenv/config";

// 本地模式
const stagehand = new Stagehand({
  env: "LOCAL",                        // "LOCAL" 或 "BROWSERBASE"
  model: "openai/gpt-4o-mini",        // provider/model 格式
  verbose: 1,                          // 0=静默, 1=信息, 2=调试
  headless: false,                     // 是否无头
  enableCaching: true,                 // 启用 act() 缓存
  experimental: false,                 // 启用实验特性（Shadow DOM 等）
});

await stagehand.init();
const page = stagehand.context.pages()[0];

// ... 使用完毕后关闭
await stagehand.close();
```

### 模型选择（15+ 提供商）

```typescript
// 格式: "provider/model"
model: "openai/gpt-4o-mini"          // 快速便宜（推荐）
model: "openai/gpt-4.1"              // 最新最强
model: "anthropic/claude-sonnet-4-5"  // Claude
model: "google/gemini-2.0-flash"      // Gemini
model: "groq/llama-3.3-70b"          // Groq
model: "cerebras/llama-3.3-70b"      // Cerebras
model: "ollama/llama3"               // 本地 Ollama
```

## 3.3 四大核心 API

### act() — 执行操作

自然语言 → DOM 定位 → 执行。支持自愈（失败时自动重新定位）。

```typescript
// 基础用法
await stagehand.act("Click the login button");
await stagehand.act("Type 'hello world' into the search box");
await stagehand.act("Select 'United States' from the country dropdown");
await stagehand.act("Scroll down to the footer");
await stagehand.act("Press Enter");

// 使用变量（敏感数据安全）
await stagehand.act("Type '{email}' into the email field", {
  variables: { email: "user@example.com" }
});

await stagehand.act("Type '{password}' into the password field and press Enter", {
  variables: { password: "s3cret!" }
});

// 指定模型
await stagehand.act("Click submit", {
  model: "anthropic/claude-sonnet-4-5"
});

// 超时控制
await stagehand.act("Wait for the loading spinner to disappear", {
  timeout: 30000  // 30 秒
});
```

### extract() — 提取数据

使用 Zod Schema 确保类型安全的结构化数据提取。

```typescript
import { z } from "zod";

// 提取简单数据
const { price } = await stagehand.extract(
  "Extract the total price",
  z.object({ price: z.string() })
);

// 提取列表
const { items } = await stagehand.extract(
  "Extract all product listings",
  z.object({
    items: z.array(z.object({
      name: z.string().describe("Product name"),
      price: z.number().describe("Price in USD"),
      rating: z.number().describe("Star rating out of 5"),
      url: z.string().url().describe("Product link"),
    }))
  })
);

console.log(items);
// [{ name: "MacBook Pro", price: 1999, rating: 4.8, url: "https://..." }, ...]

// 提取带分页检测
const { products, hasNextPage } = await stagehand.extract(
  "Extract products and check if there's a next page",
  z.object({
    products: z.array(z.object({ name: z.string(), price: z.number() })),
    hasNextPage: z.boolean().describe("Whether pagination has a next page")
  })
);
```

### observe() — 观察页面

返回页面上可执行的操作列表，**不执行**任何操作。用于人工审查或缓存操作。

```typescript
// 获取所有可交互元素
const actions = await stagehand.observe();
console.log(actions);
// [
//   { description: "Click 'Sign In' button", selector: "xpath=...", method: "click" },
//   { description: "Type in search box", selector: "xpath=...", method: "fill" },
//   ...
// ]

// 带指令的观察
const loginActions = await stagehand.observe("Find the login form elements");

// observe → 审查 → act 工作流
const actions = await stagehand.observe("Find the delete button");
console.log("AI 建议:", actions[0].description);
// 确认无误后执行
await stagehand.act(actions[0]);
```

### agent() — 自主 Agent

创建可执行多步骤复杂任务的自主 Agent，支持三种模式和自定义工具。

```typescript
// 基础用法
const agent = stagehand.agent({
  model: "openai/gpt-4o",
  systemPrompt: "You are a helpful shopping assistant.",
  maxSteps: 20,
});

const result = await agent.execute({
  instruction: "Go to Amazon, search for wireless headphones, and find the best-rated one under $100"
});

console.log(result.message);   // Agent 的最终回答
console.log(result.steps);     // 执行的步骤列表
```

#### 三种 Agent 模式

```typescript
// DOM 模式（默认）— 基于语义的 DOM 操作，任何模型都支持
const agent = stagehand.agent({ mode: "dom" });

// Hybrid 模式 — DOM + 坐标混合，需要视觉模型（Gemini/Claude Sonnet）
const agent = stagehand.agent({ mode: "hybrid" });

// CUA 模式 — Computer Use Agent，需要 Computer Use API
const agent = stagehand.agent({ mode: "cua" });
```

#### Agent + 自定义工具

```typescript
import { tool } from "ai";

const searchTool = tool({
  description: "Search the web for information",
  inputSchema: z.object({ query: z.string() }),
  execute: async ({ query }) => {
    // 调用搜索 API
    return { results: ["result1", "result2"] };
  }
});

const agent = stagehand.agent({
  tools: { search: searchTool },
  systemPrompt: "Use the search tool to find information before browsing."
});
```

#### Agent + MCP 集成

```typescript
// 使用 Exa 搜索 MCP
const agent = stagehand.agent({
  integrations: [
    `https://mcp.exa.ai/mcp?exaApiKey=${process.env.EXA_API_KEY}`
  ],
  systemPrompt: "You have access to Exa search for web research."
});

await agent.execute({
  instruction: "Research the latest AI news and summarize the top 3 stories"
});
```

## 3.4 页面操作

```typescript
const page = stagehand.context.pages()[0];

// 导航
await page.goto("https://example.com");
await page.goBack();
await page.reload();

// Playwright 原生操作（可以和 Stagehand API 混用）
await page.click("#button");
await page.fill("#input", "text");
await page.waitForSelector(".loading", { state: "hidden" });
await page.screenshot({ path: "screenshot.png" });
```

## 3.5 典型场景

### 场景 1：提取新闻标题

```typescript
import { Stagehand } from "@browserbasehq/stagehand";
import { z } from "zod";

const stagehand = new Stagehand({ env: "LOCAL", model: "openai/gpt-4o-mini" });
await stagehand.init();
const page = stagehand.context.pages()[0];

await page.goto("https://news.ycombinator.com");

const { stories } = await stagehand.extract(
  "Extract the top 5 stories with titles and scores",
  z.object({
    stories: z.array(z.object({
      title: z.string(),
      score: z.number(),
      url: z.string().url()
    }))
  })
);

stories.forEach(s => console.log(`${s.score} - ${s.title}`));
await stagehand.close();
```

### 场景 2：表单填写（变量系统）

```typescript
await page.goto("https://example.com/register");

await stagehand.act("Type '{name}' into the name field", {
  variables: { name: "Alice Zhang" }
});
await stagehand.act("Type '{email}' into the email field", {
  variables: { email: "alice@example.com" }
});
await stagehand.act("Type '{password}' into the password field", {
  variables: { password: "Str0ngP@ss!" }
});
await stagehand.act("Click the Register button");
```

### 场景 3：observe → 审查 → act

```typescript
await page.goto("https://admin.example.com/users");

// AI 建议可用操作
const deleteActions = await stagehand.observe("Find all delete user buttons");
console.log(`Found ${deleteActions.length} delete buttons:`);
deleteActions.forEach((a, i) => console.log(`  ${i}: ${a.description}`));

// 人工确认后执行第一个
if (deleteActions.length > 0) {
  console.log(`Executing: ${deleteActions[0].description}`);
  await stagehand.act(deleteActions[0]);
}
```

### 场景 4：Browserbase 云端模式

```typescript
const stagehand = new Stagehand({
  env: "BROWSERBASE",
  model: "openai/gpt-4o",
  browserbaseSessionCreateParams: {
    browserSettings: {
      solveCaptchas: true,     // 自动解验证码
      advancedStealth: true,   // 反爬隐身
      viewport: { width: 1288, height: 711 }
    }
  }
});
await stagehand.init();

// 100+ 并发，自动处理验证码和反爬
await page.goto("https://difficult-site.com");
const data = await stagehand.extract("Extract all data", schema);
```

### 场景 5：Agent 完成复杂购物任务

```typescript
const agent = stagehand.agent({
  mode: "dom",
  model: "openai/gpt-4o",
  systemPrompt: "You are a shopping assistant. Be thorough in comparing prices.",
  maxSteps: 30
});

const result = await agent.execute({
  instruction: `
    1. Go to amazon.com
    2. Search for "wireless noise-cancelling headphones"
    3. Filter by 4+ stars and price under $100
    4. Find the top 3 results and compare their features
    5. Report which one is the best value
  `
});

console.log("Recommendation:", result.message);
```

## 3.6 高级配置

### 缓存控制

```typescript
const stagehand = new Stagehand({
  enableCaching: true,   // 启用 act() 结果缓存
  cacheDir: "./cache",   // 缓存目录
  // 同一指令+URL+变量键 → 跳过 LLM，直接回放操作
});

// 第一次: LLM 推理 → 执行 → 缓存
await stagehand.act("Click login");  // ~3 秒

// 第二次: 缓存命中 → 直接执行
await stagehand.act("Click login");  // ~200ms

// 禁用缓存（动态页面推荐）
const stagehand = new Stagehand({ enableCaching: false });
```

### 日志级别

```typescript
verbose: 0  // 静默
verbose: 1  // 关键信息（推荐）
verbose: 2  // 完整调试日志
```

### experimental 标志

```typescript
const stagehand = new Stagehand({
  experimental: true  // 启用以下特性:
  // - Shadow DOM 穿透
  // - DeepLocator 跨 iframe 定位
  // - Hybrid 模式坐标工具
  // - 高级 A11y Tree 优化
});
```

---

# Part 4: 场景速查表

> "我想做 X，该用哪个方案？"

| 我想... | 推荐方案 | 原因 |
|---------|---------|------|
| **给 Claude Code 加浏览器** | Playwright MCP | 一行命令，零成本 |
| **搜索网页、查信息** | Playwright MCP | 最快，纯文本交互 |
| **填表单、登录网站** | Playwright MCP | 确定性最高 |
| **生成 PDF** | Playwright MCP | 内置 `--caps=pdf` |
| **截图保存** | Playwright MCP | `browser_take_screenshot` |
| **Mock API 响应** | Playwright MCP | `--caps=network` |
| **让 AI 自主完成复杂任务** | browser-use | Agent 模式最强 |
| **多步骤自动化（购物/订票）** | browser-use | Watchdog 防死循环 |
| **处理验证码** | browser-use (Cloud) | CAPTCHA 自动等待 |
| **Python 项目集成** | browser-use | 原生 Python |
| **使用命令行操作浏览器** | browser-use CLI | `browser-use open/click/type` |
| **提取结构化数据（类型安全）** | Stagehand | Zod Schema 强类型 |
| **高频重复任务（日常报表）** | Stagehand | 缓存后零边际成本 |
| **先观察再操作（安全流程）** | Stagehand | observe → act |
| **跨 iframe/Shadow DOM** | Stagehand | DeepLocator |
| **100+ 并发浏览器** | Stagehand + Browserbase | 云端弹性伸缩 |
| **TypeScript 项目集成** | Stagehand | 原生 TypeScript |
| **使用 15+ 种 LLM** | Stagehand | AI SDK 统一路由 |
| **企业安全合规** | browser-use | Docker 纵深防御 |
| **零额外 LLM 成本** | Playwright MCP | 纯结构化数据 |

### 推荐组合

```
日常使用:
  Playwright MCP (默认) + browser-use (复杂任务时手动指定)

生产环境:
  Playwright MCP (简单操作) + Stagehand (重复任务/数据提取)

全覆盖:
  三个都配置，按场景自动/手动选择
```

### 三合一 MCP 配置

```json
{
  "mcpServers": {
    "playwright": {
      "command": "npx",
      "args": ["@playwright/mcp@latest", "--headless", "--caps=core,pdf,storage"]
    },
    "browser-use": {
      "command": "uvx",
      "args": ["browser-use[cli]", "--mcp"],
      "env": { "BROWSER_USE_API_KEY": "sk-..." }
    }
  }
}
```
