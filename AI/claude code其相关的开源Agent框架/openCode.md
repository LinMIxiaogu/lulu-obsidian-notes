<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1773976808969-7fadff52-a9f3-4c1c-b313-7fc7ccc7d3ad.png" width="548" title="" crop="0,0,1,1" id="ud5ac6e48" class="ne-image">

  

OpenCode 与 Claude Code 能力相近，核心区别：

  

- 不绑定任何特定 LLM 提供商（支持 Claude、OpenAI、Google、本地模型）
    

## Agents设计

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1773976848493-a1b8054c-704b-483c-a1ee-0e28f74ba2d9.png" width="528" title="" crop="0,0,1,1" id="uc593955b" class="ne-image">

  

有两类 Agent：Primary Agent 是用户直接对话的主 Agent，通过 Tab 键切换；

  

Subagent 是由主 Agent 自动派发或用 `@` 手动调用的专用 Agent。

  

内置 `build`（全权限默认主 Agent）、`plan`（只读规划模式）、`general`（通用多步任务子 Agent，可并行）、`explore`（只读代码探索子 Agent），以及三个用户不可见的系统 Agent：`compaction`（自动压缩长 context）、`title`、`summary`

  

  

  

## 工具

<img src="https://cdn.nlark.com/yuque/0/2026/png/26453565/1773976866295-7dd624c6-0b08-40ce-ab62-0008b7a7f10f.png" width="561" title="" crop="0,0,1,1" id="u355127b5" class="ne-image">

  

内置 15 个工具，按用途分五组：

  

- 文件操作（`read/write/edit/patch/glob/grep/list`）
    
- 执行（`bash/lsp`）
    
- 网络（`webfetch/websearch`）
    
- 任务管理（`todowrite/todoread/question`）
    
- 扩展（`skill/MCP/custom tools`）
    

所有工具默认开启，通过 `permission` 字段控制每个工具的行为（`allow/deny/ask`），支持通配符批量配置。

  

  

  

配置与规则

项目根目录的 `AGENTS.md`（由 `/init` 命令自动生成）告诉 Agent 项目结构和编码风格，建议提交到 Git。自定义 Agent 可以用 Markdown 文件（放在 `~/.config/opencode/agents/` 或 `.opencode/agents/`）或 `opencode.json` 中的 `agent` 字段配置，支持指定模型、温度、工具权限、系统提示词等。