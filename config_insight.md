# AutoResearchClaw Config Insight

这份文档整理了 `config.yaml` 中主要可定制配置项的作用、常见修改方式和适用场景，方便在使用 AutoResearchClaw 时快速查阅。

配置定义来源：

- [researchclaw/config.py](/Users/lipeizhang/Downloads/code/vibe/AutoResearchClaw/researchclaw/config.py)
- [config.researchclaw.example.yaml](/Users/lipeizhang/Downloads/code/vibe/AutoResearchClaw/config.researchclaw.example.yaml)

## 一、最常改的 8 组配置

日常使用里，最常需要定制的是这几组：

- `project`
- `research`
- `llm`
- `experiment`
- `notifications`
- `openclaw_bridge`
- `export`
- `security`

其余如 `memory`、`skills`、`server`、`mcp`、`metaclaw_bridge`、`dashboard` 更偏高级扩展。

## 二、基础身份与运行模式

```yaml
project:
  name: "my-research"
  mode: "full-auto"
```

### `project.name`

- 用途：项目标识名，影响日志、产物归档、项目管理展示。
- 建议改法：使用稳定、简短的名字，例如 `gnn-drug-discovery`。

### `project.mode`

- 可选值：`docs-first` / `semi-auto` / `full-auto`
- 建议改法：
- 想全自动跑完：`full-auto`
- 想在关键节点停下来审阅：`semi-auto`
- 想更保守、强调文档和审批：`docs-first`

## 三、研究任务本身

```yaml
research:
  topic: "Graph neural networks for drug discovery"
  domains: ["machine-learning", "biology"]
  daily_paper_count: 12
  quality_threshold: 4.2
  graceful_degradation: true
```

### `research.topic`

- 用途：研究主题，最核心输入。
- 改法：直接写你的课题。

### `research.domains`

- 用途：限制文献检索范围，并给提示词更多领域上下文。
- 示例：
- AI 方向：`["machine-learning", "nlp"]`
- 交叉学科：`["biology", "chemistry", "machine-learning"]`

### `research.daily_paper_count`

- 用途：每轮目标收集论文数。
- 建议改法：
- 快速试跑：`6-10`
- 正式调研：`12-20`

### `research.quality_threshold`

- 用途：筛论文门槛。
- 建议改法：
- 更宽松：`3.5`
- 更严格：`4.5`

### `research.graceful_degradation`

- 用途：外部源失败时是否退化继续执行。
- 建议改法：
- 想尽量跑完：`true`
- 想严格失败即停：`false`

## 四、运行时控制

```yaml
runtime:
  timezone: "Asia/Shanghai"
  max_parallel_tasks: 3
  approval_timeout_hours: 12
  retry_limit: 2
```

### `runtime.timezone`

- 用途：日志、时间戳、提醒调度。
- 建议改法：按你的实际时区填写，例如 `Asia/Shanghai`。

### `runtime.max_parallel_tasks`

- 用途：并行度。
- 建议改法：
- 本机保守：`1-2`
- 机器资源充足：`3-6`

### `runtime.approval_timeout_hours`

- 用途：审批等待时间。
- 建议改法：如果靠机器人收消息，建议可以设成 `24`。

### `runtime.retry_limit`

- 用途：阶段失败后的重试次数。
- 建议改法：
- 调试期：`1-2`
- 正式长任务：`2-3`

## 五、通知与 OpenClaw 消息桥

```yaml
notifications:
  channel: "qq-bot"
  target: ""
  on_stage_start: true
  on_stage_fail: true
  on_gate_required: true

openclaw_bridge:
  use_cron: false
  use_message: true
  use_memory: true
  use_sessions_spawn: false
  use_web_fetch: false
  use_browser: false
```

### `notifications.channel`

- 用途：消息发往哪个通道。
- 改法：填你在 OpenClaw 中绑定的通道名，例如 `qq-bot`。

### `notifications.target`

- 用途：目标 ID，是否生效取决于 OpenClaw 的 message adapter 实现。
- 改法：如果机器人按群号或用户号路由，就填相应 ID。

### `notifications.on_stage_start`

- 用途：阶段开始是否通知。
- 建议：需要阶段开始提醒时设为 `true`。

### `notifications.on_stage_fail`

- 用途：阶段失败是否通知。
- 建议：通常设为 `true`。

### `notifications.on_gate_required`

- 用途：需要人工审批时通知。
- 建议：通常设为 `true`。

### `openclaw_bridge.use_message`

- 用途：启用消息适配器。
- 建议：如果要走 QQ-bot 或其他消息机器人，必须设为 `true`。

### `openclaw_bridge.use_memory`

- 用途：把运行状态写入桥接记忆。
- 建议：通常设为 `true`。

### `openclaw_bridge.use_cron`

- 用途：定时恢复任务。
- 建议：一般先设为 `false`。

### `openclaw_bridge.use_sessions_spawn`

- 用途：并发子会话。
- 建议：先设为 `false`，除非确认 OpenClaw 侧支持稳定。

### `openclaw_bridge.use_web_fetch` / `openclaw_bridge.use_browser`

- 用途：借用 OpenClaw 提供网页抓取或浏览能力。
- 建议：默认先关闭，除非明确依赖 OpenClaw 侧 web 工具。

## 六、LLM 后端

### 1. 通过 OpenClaw OAuth / Agent 跑

```yaml
llm:
  provider: "acp"
  acp:
    agent: "codex"
    cwd: "."
    session_name: "researchclaw"
    timeout_sec: 1800
```

### `llm.provider`

- 常见值：`acp` / `openai-compatible`
- 建议改法：
- 走 OpenClaw 或 agent 认证：`acp`
- 自己直连 API：`openai-compatible`

### `llm.acp.agent`

- 用途：指定后端 agent 命令。
- 改法：你用哪个 agent 就填哪个，比如 `codex`。

### `llm.acp.cwd`

- 用途：agent 工作目录。
- 建议：一般填 `.`。

### `llm.acp.acpx_command`

- 用途：自定义 acpx 命令。
- 建议：通常留空。

### `llm.acp.session_name`

- 用途：持久会话名。
- 改法：想复用同一 agent 会话时可保持固定。

### `llm.acp.timeout_sec`

- 用途：agent 调用超时。
- 建议：长任务可设 `1800-3600`。

### 2. 直连 API

```yaml
llm:
  provider: "openai-compatible"
  base_url: "https://api.openai.com/v1"
  wire_api: "chat_completions"
  api_key_env: "OPENAI_API_KEY"
  api_key: ""
  primary_model: "gpt-4o"
  fallback_models: ["gpt-4.1", "gpt-4o-mini"]
  s2_api_key: ""
  notes: ""
```

### `llm.base_url`

- 用途：OpenAI 或代理 API 地址。

### `llm.wire_api`

- 可选值：`chat_completions` / `responses`
- 改法：看你的网关支持哪一种协议。

### `llm.api_key_env`

- 用途：从环境变量读取 API key。

### `llm.api_key`

- 用途：直接在配置中写死 key。
- 建议：不推荐，优先使用环境变量。

### `llm.primary_model`

- 用途：主模型名称。

### `llm.fallback_models`

- 用途：失败时的回退模型列表。

### `llm.s2_api_key`

- 用途：Semantic Scholar API key，用于提升速率限制。

### `llm.notes`

- 用途：备注，不影响逻辑。

## 七、安全与审批

```yaml
security:
  hitl_required_stages: [5, 9, 20]
  allow_publish_without_approval: false
  redact_sensitive_logs: true
```

### `security.hitl_required_stages`

- 用途：指定哪些 stage 必须人工确认。
- 建议改法：
- 更自动：`[]` 或配合 `--auto-approve`
- 更谨慎：保留 `[5, 9, 20]`

### `security.allow_publish_without_approval`

- 用途：未审批时是否允许发布。
- 建议：通常保持 `false`。

### `security.redact_sensitive_logs`

- 用途：日志脱敏。
- 建议：通常保持 `true`。

## 八、实验总控

```yaml
experiment:
  mode: "sandbox"
  time_budget_sec: 300
  max_iterations: 10
  max_refine_duration_sec: 0
  metric_key: "val_loss"
  metric_direction: "minimize"
  keep_threshold: 0.0
```

### `experiment.mode`

- 可选值：`simulated` / `sandbox` / `docker` / `ssh_remote` / `colab_drive` / `agentic`
- 建议改法：
- 本地真实执行：`sandbox`
- 最稳妥隔离：`docker`
- 远程 GPU 机器：`ssh_remote`
- 只调试流程：`simulated`
- 让 coding agent 在容器里完成实验：`agentic`

### `experiment.time_budget_sec`

- 用途：单次实验时长预算。
- 建议：小试跑 `180-300`，正式实验 `600+`。

### `experiment.max_iterations`

- 用途：迭代修复或优化次数。
- 建议：一般 `5-10`。

### `experiment.max_refine_duration_sec`

- 用途：Stage 13 总时长上限。
- 建议：`0` 表示自动。

### `experiment.metric_key`

- 用途：主优化指标名，例如 `accuracy`、`f1`、`val_loss`。

### `experiment.metric_direction`

- 可选值：`minimize` / `maximize`

### `experiment.keep_threshold`

- 用途：结果保留阈值。
- 建议：通常默认即可。

## 九、sandbox 本地执行

```yaml
sandbox:
  python_path: ".venv/bin/python3"
  gpu_required: false
  allowed_imports: [math, random, json, csv, numpy, torch, sklearn]
  max_memory_mb: 4096
```

### `sandbox.python_path`

- 用途：本地实验执行使用的 Python。
- 建议：这是最需要确认的一项，务必指向实际可用解释器。

### `sandbox.gpu_required`

- 用途：是否要求 GPU。
- 建议：没有 GPU 时不要设为 `true`。

### `sandbox.allowed_imports`

- 用途：沙箱允许导入的库。
- 建议：做更复杂实验时可以补充需要的包名。

### `sandbox.max_memory_mb`

- 用途：内存限制。
- 建议：内存紧张时调低，复杂任务时适度调高。

## 十、docker 实验

```yaml
docker:
  image: "researchclaw/experiment:latest"
  gpu_enabled: true
  gpu_device_ids: []
  memory_limit_mb: 8192
  network_policy: "setup_only"
  pip_pre_install: []
  auto_install_deps: true
  shm_size_mb: 2048
  container_python: "/usr/bin/python3"
  keep_containers: false
```

### `docker.image`

- 用途：实验容器镜像名。

### `docker.gpu_enabled`

- 用途：容器是否透传 GPU。

### `docker.gpu_device_ids`

- 用途：指定只用哪些 GPU，例如 `[0]`。

### `docker.memory_limit_mb`

- 用途：容器内存限制。

### `docker.network_policy`

- 可选值：`none` / `setup_only` / `pip_only` / `full`
- 建议改法：
- 安全优先：`none`
- 只在安装依赖时联网：`setup_only`

### `docker.pip_pre_install`

- 用途：预装依赖列表。

### `docker.auto_install_deps`

- 用途：自动从代码中推断依赖并安装。

### `docker.shm_size_mb`

- 用途：共享内存大小。
- 建议：数据加载较多时调大。

### `docker.container_python`

- 用途：容器中的 Python 路径。

### `docker.keep_containers`

- 用途：是否保留容器。
- 建议：调试时可设为 `true`。

## 十一、ssh_remote 远程 GPU

```yaml
ssh_remote:
  host: "gpu-server"
  user: "yourname"
  port: 22
  key_path: "~/.ssh/id_rsa"
  gpu_ids: [0]
  remote_workdir: "/tmp/researchclaw_experiments"
  remote_python: "python3"
  setup_commands: ["source ~/venv/bin/activate"]
  use_docker: false
  docker_image: "researchclaw/experiment:latest"
  docker_network_policy: "none"
  docker_memory_limit_mb: 8192
  docker_shm_size_mb: 2048
  timeout_sec: 600
  scp_timeout_sec: 300
  setup_timeout_sec: 300
```

常改的是：

- `host`
- `user`
- `key_path`
- `gpu_ids`
- `setup_commands`

如果远程环境比较脏，建议优先考虑 `use_docker: true`。

## 十二、colab_drive

```yaml
colab_drive:
  drive_root: ""
  poll_interval_sec: 30
  timeout_sec: 3600
  setup_script: ""
```

- 用途：把任务通过 Google Drive 方式交给 Colab 执行。
- 建议：除非你确实这样用，否则通常不用配置。

## 十三、agentic / code_agent / cli_agent / opencode

这几组都和“代码生成有多激进”有关。

### `agentic`

- 用途：让 coding agent 在容器内完成实验开发。
- 常改项：
- `agent_cli`
- `timeout_sec`
- `gpu_enabled`
- `max_turns`

### `code_agent`

- 用途：仓库自带的高级代码生成流水线。
- 常改项：
- `enabled`
- `architecture_planning`
- `sequential_generation`
- `hard_validation`
- `exec_fix_max_iterations`

建议：

- 想更稳：保留默认，大多开着
- 想更省 token 和更快：适当关掉 `architecture_planning`，减少 repair 次数

### `cli_agent`

```yaml
cli_agent:
  provider: "llm"
  binary_path: ""
  model: ""
  max_budget_usd: 5.0
  timeout_sec: 600
  extra_args: []
```

- 用途：把 Stage 10 和 13 交给外部 CLI agent。
- `provider` 可选：`llm` / `claude_code` / `codex`
- 如果你未来要配合 OpenClaw + Codex，比较值得关注这组。

### `opencode`

- 用途：外部 OpenCode agent，用于复杂代码生成。
- 常改项：
- `enabled`
- `auto`
- `complexity_threshold`
- `timeout_sec`

建议：

- 想少触发：提高 `complexity_threshold`
- 想更积极调用：降低到 `0.1-0.2`

## 十四、benchmark_agent / figure_agent / repair

### `benchmark_agent`

- 用途：自动找数据集和基线。
- 常改项：
- `enable_hf_search`
- `enable_web_search`
- `tier_limit`
- `min_benchmarks`
- `min_baselines`

### `figure_agent`

- 用途：自动生成论文图表和插图。
- 常改项：
- `enabled`
- `min_figures` / `max_figures`
- `output_format`: `python` 或 `latex`
- `nano_banana_enabled`
- `gemini_api_key`
- `dpi`

### `repair`

- 用途：实验失败后的修复循环。
- 常改项：
- `enabled`
- `max_cycles`
- `min_completion_rate`
- `use_opencode`

如果你追求更稳妥的论文产出，建议保留开启。

## 十五、导出

```yaml
export:
  target_conference: "neurips_2025"
  authors: "Anonymous"
  bib_file: "references"
```

### `export.target_conference`

- 用途：决定 LaTeX 模板风格。
- 改法：想投哪个 venue 就配哪个模板。

### `export.authors`

- 用途：论文作者名。

### `export.bib_file`

- 用途：BibTeX 文件名，不带 `.bib`。

## 十六、prompts

```yaml
prompts:
  custom_file: "my_prompts.yaml"
```

- 用途：通过自定义 prompt 文件深度定制 agent 行为。
- 适合：你想让它更保守、更工程化、或更偏理论时。

## 十七、web_search

```yaml
web_search:
  enabled: true
  tavily_api_key: ""
  tavily_api_key_env: "TAVILY_API_KEY"
  enable_scholar: true
  enable_crawling: true
  enable_pdf_extraction: true
  max_web_results: 10
  max_scholar_results: 10
  max_crawl_urls: 5
```

### 常见改法

- 想增强开放网页搜索：配置 `tavily_api_key`
- 想严格减少外部抓取：关掉 `enable_crawling`
- 想省时间和成本：降低结果数量

## 十八、knowledge_base / memory / skills / knowledge_graph

这组偏向长期知识积累。

### `knowledge_base`

- `backend`：`markdown` / `obsidian`
- `root`：知识库存放目录
- `obsidian_vault`：如果你用 Obsidian，就填 vault 路径

### `memory`

- 用途：长期记忆系统
- 常改项：
- `enabled`
- `store_dir`
- `embedding_model`
- `inject_at_stages`

如果你不想跨 run 学习，可以关闭。

### `skills`

- 用途：技能库加载
- 常改项：
- `custom_dirs`
- `external_dirs`
- `max_skills_per_stage`

如果你将来想注入自己的领域技能，这组会很重要。

### `knowledge_graph`

- 用途：知识图谱
- 建议：除非你明确需要复杂知识沉淀，否则先不要开启。

## 十九、MetaClaw

```yaml
metaclaw_bridge:
  enabled: false
  proxy_url: "http://localhost:30000"
  skills_dir: "~/.metaclaw/skills"
  fallback_url: ""
  fallback_api_key: ""
  prm:
    enabled: false
    api_base: ""
    api_key_env: "PRM_API_KEY"
    api_key: ""
    model: "gpt-5.4"
    votes: 3
    temperature: 0.6
    gate_stages: [5, 9, 15, 20]
  lesson_to_skill:
    enabled: true
    min_severity: "warning"
    max_skills_per_run: 3
```

- 用途：更高级的跨运行学习和 PRM 质量门禁。
- 建议：如果你当前目标只是先稳定跑起来，先不要开启。

## 二十、Web 平台与附加服务

### `server`

- 用途：开启 FastAPI 服务
- 常改项：
- `enabled`
- `host`
- `port`
- `auth_token`

### `dashboard`

- 用途：dashboard 刷新和浏览器通知
- 常改项：
- `enabled`
- `refresh_interval_sec`

### `mcp`

- 用途：MCP server 集成
- 常改项：
- `server_enabled`
- `server_port`

### `overleaf`

- 用途：与 Overleaf Git 同步
- 常改项：
- `enabled`
- `git_url`
- `branch`

## 二十一、研究增强模块

### `trends`

- 用途：每日研究趋势跟踪。
- 建议：长期追踪领域热点时再开启。

### `copilot`

- 用途：人机协作模式。
- `mode` 可选：`co-pilot` / `auto-pilot` / `zero-touch`
- 如果你想每个阶段都征求你的意见，这组值得调。

### `quality_assessor`

- 用途：论文质量打分。
- 建议：大多数情况下默认即可。

### `calendar`

- 用途：会议 deadline 规划。
- 建议：围绕目标 venue 做时间提醒时再开启。

## 二十二、三套常用配置思路

### 1. OpenClaw OAuth + 最省心

```yaml
project:
  mode: "full-auto"

llm:
  provider: "acp"
  acp:
    agent: "codex"
    cwd: "."

openclaw_bridge:
  use_message: true
  use_memory: true

notifications:
  channel: "qq-bot"
  on_stage_start: true
  on_gate_required: true

experiment:
  mode: "sandbox"
  sandbox:
    python_path: ".venv/bin/python3"
```

### 2. 本地保守试跑

```yaml
project:
  mode: "semi-auto"

experiment:
  mode: "sandbox"
  time_budget_sec: 180
  max_iterations: 3

research:
  daily_paper_count: 6
```

### 3. 远程 GPU 正式跑

```yaml
project:
  mode: "full-auto"

experiment:
  mode: "ssh_remote"
  time_budget_sec: 1200
  ssh_remote:
    host: "your-gpu-host"
    user: "your-user"
    key_path: "~/.ssh/id_rsa"
    gpu_ids: [0]
```

## 二十三、两个重要提醒

- `acp` 模式下，`llm.base_url` 和 `llm.api_key_env` 可以不配；直连 API 才需要。
- `knowledge_base.root` 是校验必填路径，目录最好先建好，否则 `researchclaw validate` 可能报路径错误。
