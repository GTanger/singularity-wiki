---
title: Claude Code 工具鏈與技能全覽
tags: [architecture, tools, mcp, plugins, skills, infrastructure]
sources:
  - .claude/.mcp.json
  - .claude/plugins/installed_plugins.json
  - .claude/plugins/marketplaces/thedotmack/CHANGELOG.md
date: 2026-04-12
---

# Claude Code 工具鏈與技能全覽

Claude Code（Opus 4.6）作為審局者的完整能力清單。分為四層：內建工具、MCP 伺服器、雲端插件、Skills。

**Claude Code 版本**：2.1.100

## 一、內建工具（Core Tools）

Claude Code 本體自帶，不需額外設定。

| 工具 | 用途 | 備註 |
|------|------|------|
| **Read** | 讀取檔案（含圖片、PDF、Jupyter） | 支援 offset/limit 分段讀 |
| **Write** | 建立新檔案或完整覆寫 | 既有檔必須先 Read |
| **Edit** | 精確字串替換（diff 級修改） | 不動其餘內容，支援 replace_all |
| **Bash** | 執行 shell 指令 | 有 timeout、背景執行、沙箱 |
| **Glob** | 檔案名稱模式搜尋 | 取代 find/ls |
| **Grep** | 檔案內容正規式搜尋 | 基於 ripgrep，取代 grep/rg |
| **Agent** | 啟動子代理（並行/獨立工作樹） | 支援 Explore/Plan/general-purpose 型態 |
| **TodoWrite** | 任務追蹤（當前對話） | 不跨對話持久 |
| **Skill** | 呼叫已安裝的 Skill | `/commit`、`/review-pr` 等斜線指令 |
| **WebFetch** | 抓取網頁內容 | URL 必須由用戶提供或程式碼相關 |
| **WebSearch** | 網頁搜尋 | 一般問題用 |
| **LSP** | Language Server Protocol 操作 | 程式碼跳轉、補全、診斷 |
| **NotebookEdit** | 編輯 Jupyter notebook 儲存格 | .ipynb 專用 |

### 子代理型態（Agent subtypes）

| 型態 | 用途 | 工具權限 |
|------|------|---------|
| **general-purpose** | 複雜多步驟任務 | 全部工具 |
| **Explore** | 快速搜尋程式碼庫 | 唯讀（不能 Edit/Write） |
| **Plan** | 設計實作方案 | 唯讀（不能 Edit/Write） |

## 二、本機 MCP 伺服器

設定於 `~/.claude/.mcp.json`，每次啟動 Claude Code 時載入。

### 版次總覽

| MCP | 使用版次 | 最新版次 | 專案頁面 | 更新方式 |
|-----|---------|---------|----------|---------|
| graphthulhu | dev (0.4.0) | v0.4.0 | [GitHub](https://github.com/skridlevsky/graphthulhu) | `go install` 手動 |
| llm-wiki-kit | 0.1.0 | 0.1.0 (非 PyPI) | [GitHub](https://github.com/iamsashank09/llm-wiki-kit) | `pipx upgrade` 手動 |
| brave-search | 0.6.2 | 0.6.2 | [GitHub](https://github.com/modelcontextprotocol/servers/tree/main/src/brave-search) | `npx -y` 自動 |
| postgres | 0.6.2 | 0.6.2 | [GitHub](https://github.com/modelcontextprotocol/servers/tree/main/src/postgres) | `npx -y` 自動 |
| claude-historian | 1.0.3 | 1.0.3 | [GitHub](https://github.com/Vvkmnn/claude-historian-mcp) | `npx -y` 自動 |
| shodh-memory | 0.2.0 | 0.2.0 | [GitHub](https://github.com/varun29ankuS/shodh-memory) | npm 全域手動 |
| gbrain | 本地開發 | — | `~/Projects/gbrain-master/` | 自行維護 |
| GitHub MCP | 雲端 | — | Anthropic 官方 | 自動 |

### 2.1 graphthulhu — Obsidian 知識圖譜引擎

- **版次**：dev build (基於 v0.4.0)
- **專案**：https://github.com/skridlevsky/graphthulhu
- **來源**：Go 二進位（`~/go/bin/graphthulhu`）
- **後端**：讀取 Obsidian vault（`~/Projects/obsidian-vault`）
- **功能**：頁面 CRUD、區塊操作、圖譜分析（拓撲、孤兒、關連、知識缺口）、日誌搜尋、決策追蹤
- **核心工具**：
  - `create_page` / `get_page` / `rename_page` / `delete_page` — 頁面管理
  - `append_blocks` / `update_block` / `move_block` — 區塊級編輯
  - `search` / `find_by_tag` / `find_connections` — 搜尋與關連
  - `graph_overview` / `topic_clusters` / `knowledge_gaps` / `list_orphans` — 圖譜分析
  - `traverse` — 從某頁出發走 N 層連結
  - `decision_create` / `decision_resolve` / `decision_defer` / `decision_check` — 決策追蹤
  - `journal_range` / `journal_search` — 日誌查詢
- **更新方式**：`go install github.com/skridlevsky/graphthulhu@latest` 手動
- **定位**：Obsidian vault 的結構化操作介面，gbrain 知識腦的底層之一

### 2.2 llm-wiki-kit — LLM 維護的結構化 Wiki

- **版次**：0.1.0（pipx 安裝）
- **專案**：https://github.com/iamsashank09/llm-wiki-kit
- **來源**：Python 二進位（`~/.local/bin/llm-wiki-kit`），不在 PyPI 上
- **後端**：同一個 Obsidian vault 的 `wiki/` 子目錄
- **功能**：從原始文件（docs/、對話 log）編譯出結構化 wiki 頁面，含 FTS 全文搜尋
- **核心工具**：
  - `wiki_write_page` / `wiki_read_page` — 頁面讀寫
  - `wiki_search` — FTS5 全文搜尋
  - `wiki_ingest` — 攝入原始文件為 source
  - `wiki_status` — 總覽（頁數、source 數、最近活動）
  - `wiki_lint` — 檢查交叉引用完整性
  - `wiki_graph` — 頁面關係圖
  - `wiki_log` — 操作日誌
  - `wiki_init` — 初始化新 wiki
- **更新方式**：`pipx upgrade llm-wiki-kit` 或從 GitHub 重裝
- **定位**：知識編譯層。散落的設計文件經過這裡變成可查詢的結構化知識。目前 63 頁，8 大系統

### 2.3 brave-search — 網路搜尋

- **版次**：0.6.2（npx 自動拉取）
- **專案**：https://github.com/modelcontextprotocol/servers/tree/main/src/brave-search
- **來源**：`npx -y @modelcontextprotocol/server-brave-search`（每次啟動拉最新）
- **功能**：
  - `brave_web_search` — 網頁搜尋
  - `brave_local_search` — 本地商家搜尋
- **需要**：`BRAVE_API_KEY` 環境變數
- **定位**：補充 WebSearch 的另一個搜尋管道

### 2.4 postgres — 直接查詢資料庫

- **版次**：0.6.2（npx 自動拉取）
- **專案**：https://github.com/modelcontextprotocol/servers/tree/main/src/postgres
- **來源**：`npx -y @modelcontextprotocol/server-postgres`（每次啟動拉最新）
- **連線**：`postgresql://postgres@localhost:5432/singularity`
- **功能**：
  - `query` — 執行 SQL 查詢（SELECT/INSERT/UPDATE/DELETE）
- **定位**：直接操作 Singularity World 的 PostgreSQL 資料庫，繞過後端 API

### 2.5 claude-historian — 對話歷史搜尋

- **版次**：1.0.3（npx 自動拉取）
- **專案**：https://github.com/Vvkmnn/claude-historian-mcp
- **來源**：`npx -y claude-historian-mcp`（每次啟動拉最新）
- **功能**：搜尋過去的 Claude Code 對話記錄
- **核心工具**：
  - `search` — 按 scope（conversations/files/errors/plans/config/tasks/memories）搜尋
  - `inspect` — 取得特定 session 的摘要
  - 支援時間範圍過濾（today/week/month）
- **定位**：回溯過去對話中的決策、錯誤解法、工具使用模式

### 2.6 shodh-memory — 持久化任務與記憶管理

- **版次**：0.2.0（npm 全域安裝）
- **專案**：https://github.com/varun29ankuS/shodh-memory
- **來源**：`@shodh/memory-mcp`（Node 二進位，安裝於 nvm node v22）
- **更新方式**：`npm install -g @shodh/memory-mcp@latest` 手動
- **定位**：跨對話的持久化記憶 + 任務管理系統。功能範圍廣，涵蓋其他系統沒有的領域
- **核心工具**（36 個）：

  **記憶管理**（其他系統有類似功能）：
  - `remember` / `recall` / `forget` / `read_memory` — 記憶 CRUD
  - `recall_by_tags` — 標籤搜尋（**獨有**：MEMORY.md 和 claude-mem 都沒有標籤搜尋）
  - `list_memories` / `recent_memories` / `quick_recall` — 記憶瀏覽
  - `context_summary` / `session_summary` / `what_i_know` — 上下文摘要
  - `memory_stats` / `memory_health` / `count` — 統計與健康檢查

  **待辦與專案管理**（**獨有**：內建 TodoWrite 不跨對話、其他 MCP 都沒有）：
  - `add_todo` / `list_todos` / `update_todo` / `complete_todo` / `delete_todo` / `reorder_todo` — 待辦 CRUD + 排序
  - `list_subtasks` — 子任務
  - `add_todo_comment` / `list_todo_comments` / `update_todo_comment` / `delete_todo_comment` — 待辦評論
  - `add_project` / `list_projects` / `archive_project` / `delete_project` — 專案管理
  - `todo_stats` — 待辦統計
  - `pending_work` — 未完成工作總覽

  **提醒系統**（**獨有**）：
  - `set_reminder` / `list_reminders` / `dismiss_reminder` — 時間提醒

  **備份與索引維護**（**獨有**）：
  - `backup_create` / `backup_list` / `backup_verify` / `backup_purge` / `backup_restore` — 完整備份生命週期
  - `verify_index` / `repair_index` — 索引健康與修復
  - `consolidation_report` — 記憶合併報告

  **Token 管理**：
  - `token_status` / `reset_token_session` — 用量追蹤

### 2.7 gbrain — 知識腦核心

- **版次**：本地開發版
- **來源**：自寫腳本（`~/Projects/gbrain-master/gbrain-serve.sh`），Bun 執行
- **定位**：統合 graphthulhu + llm-wiki-kit + Obsidian vault 的上層調度。知識腦的中控
- **更新方式**：自行維護

### 2.8 GitHub MCP（雲端）

- **來源**：Anthropic 官方提供，透過 Claude Code 雲端連接
- **功能**：GitHub API 操作（PR、issue、code search、release 等）
- **核心工具**：issue_read/write、pull_request_read/review、search_code/issues/repositories、create_branch、push_files 等完整 GitHub 操作
- **更新方式**：Anthropic 自動更新

## 三、雲端 MCP（Claude.ai 側）

透過 Claude.ai 帳號連接的第三方服務，工具名稱前綴 `mcp__claude_ai_*`。

| 服務 | 工具數 | 用途 |
|------|--------|------|
| **Canva** | 30+ | 設計稿建立/編輯/匯出、品牌管理、評論 |
| **Context7** | 2 | 即時查詢任何程式庫/框架的最新文件（resolve-library-id → query-docs） |
| **Gmail** | 7 | 讀取/搜尋郵件、建立草稿、標籤管理 |
| **Google Calendar** | 9 | 行事曆事件 CRUD、空閒時段查詢、回覆邀請 |

### Context7 特別說明

即使自認知道某個程式庫的用法，也應該用 Context7 查一次——訓練資料可能過時。適用場景：API 語法、版本遷移、設定方式、CLI 用法。

## 四、Plugin 系統

### 版次總覽

| Plugin | 使用版次 | 最新版次 | 專案頁面 | 更新指令 |
|--------|---------|---------|----------|---------|
| claude-mem | 12.1.0 (2026-04-09) | 12.1.0 | [GitHub](https://github.com/thedotmack/claude-mem) | `claude plugins update claude-mem@thedotmack` |
| rust-analyzer-lsp | 1.0.0 (2026-03-23) | 1.0.0 | Anthropic 官方 | `claude plugins update rust-analyzer-lsp@claude-plugins-official` |

### 4.1 claude-mem@thedotmack（v12.1.0）

第三方記憶插件。提供 MCP 工具 + Skills + Hooks。

**更新歷程**（近期）：
- v12.1.0 (4/9) — Knowledge Agent 系統（記憶編譯成可對話語料庫）
- v12.0.1 (4/8) — 緊急修復：v12.0.0 的 MCP server 在 Node 下 crash（bun:sqlite 錯誤引入）
- v12.0.0 (4/7) — 架構大改
- v11.0.1 (4/6) — hotfix
- v11.0.0 (4/5) — 大版本

**MCP 工具（前綴 `mcp__plugin_claude-mem_mcp-search__`）**：
- `search` — 語義搜尋記憶
- `get_observations` — 取得特定觀察記錄全文
- `timeline` — 時間軸瀏覽
- `smart_search` / `smart_outline` / `smart_unfold` — 智慧搜尋（大綱→展開特定符號）
- `build_corpus` / `prime_corpus` / `query_corpus` — Knowledge Agent（12.1.0 新功能：編譯記憶為可對話語料庫）
- `list_corpora` / `rebuild_corpus` / `reprime_corpus` — 語料庫管理

**Skills**：
- `/mem-search` — 引導式記憶搜尋
- `/smart-explore` — 智慧程式碼探索
- `/timeline-report` — 時間軸報告
- `/knowledge-agent` — 建立知識語料庫
- `/make-plan` — 規劃
- `/version-bump` — 版本遞增
- `/do` — 執行任務

**目前狀態**：MCP 工具因 PATH 問題無法使用（`uvx` 不在 Claude Code 繼承的 PATH 中）。從終端機啟動 Claude Code 時正常。

**更新**：`claude plugins update claude-mem@thedotmack`（手動，無自動更新）

### 4.2 rust-analyzer-lsp@claude-plugins-official（v1.0.0）

官方 Rust LSP 插件，提供 Rust 程式碼的跳轉、補全、診斷功能。

## 五、Skills（斜線指令）

除 claude-mem 提供的 skills 外，`~/.claude/skills/` 下有約 90 個預裝 skill，涵蓋：

| 類別 | 範例 |
|------|------|
| 前端框架 | nextjs, react-native-expo, sveltia-cms, tailwind-v4-shadcn |
| 後端框架 | fastapi, flask, hono-routing |
| AI/LLM | claude-agent-sdk, claude-api, openai-agents, google-gemini-api |
| 雲平台 | cloudflare-* 系列（12 個）、vercel-*、neon-vercel-postgres |
| 開發工具 | playwright-local, mcp-cli-scripts, skill-creator, project-planning |
| 搜尋/爬蟲 | tavily-* 系列（7 個）、firecrawl-scraper |
| 設計 | color-palette, icon-design, image-gen, favicon-gen |
| 辦公 | office, google-workspace |
| 工作流 | project-workflow, project-session-management, docs-workflow |
| 專案特有 | llm-wiki（LLM Wiki 操作引導） |

大部分為通用 skill，與 Singularity World 專案無直接關係。實際常用的是 llm-wiki 和 claude-mem 提供的 skills。

## 六、持久化記憶系統

Claude Code 沒有跨對話記憶，靠外部系統補足：

| 層 | 機制 | 用途 |
|---|------|------|
| **CLAUDE.md** | 每次對話自動載入 | 專案規則、角色定位、硬規則 |
| **MEMORY.md + 記憶檔** | `~/.claude/projects/*/memory/` | 跨對話事實：用戶偏好、回饋、專案狀態 |
| **shodh-memory** | MCP 持久化記憶 + 待辦 + 專案 | 跨對話任務管理、提醒、備份（獨有功能最多） |
| **claude-historian** | MCP 搜尋對話歷史 | 回溯過去決策、錯誤解法 |
| **llm-wiki-kit** | 結構化 wiki（63 頁） | 設計知識庫，人工編譯維護 |
| **graphthulhu** | Obsidian 圖譜操作 | wiki 的底層結構化存取 |
| **claude-mem** | 自動觀察 + 語義搜尋 | 被動記錄（目前 PATH 問題待修） |

### 記憶優先級

1. **CLAUDE.md** — 最高權威，每次對話必讀
2. **MEMORY.md** — 人工維護的跨對話記憶，主動讀寫
3. **wiki** — 設計知識的編譯產物，查閱用
4. **shodh-memory** — 持久化任務與提醒，管理用
5. **claude-historian** — 歷史回溯，驗證用
6. **claude-mem** — 被動觀察，補充用

## 七、更新與維護

| 項目 | 使用版次 | 更新方式 | 風險 |
|------|---------|---------|------|
| Claude Code 本體 | 2.1.100 | Anthropic 推送 | 低（官方控制） |
| brave-search | 0.6.2 | `npx -y` 每次啟動自動拉最新 | 中（npm 供應鏈風險） |
| postgres | 0.6.2 | `npx -y` 每次啟動自動拉最新 | 中（npm 供應鏈風險） |
| claude-historian | 1.0.3 | `npx -y` 每次啟動自動拉最新 | 中（npm 供應鏈風險） |
| graphthulhu | dev (0.4.0) | `go install` 手動 | 低（自己控制） |
| llm-wiki-kit | 0.1.0 | `pipx upgrade` 手動 | 低（自己控制） |
| shodh-memory | 0.2.0 | `npm -g` 手動 | 低（自己控制） |
| gbrain | 本地開發 | 自行維護 | 低（自己控制） |
| claude-mem plugin | 12.1.0 | `claude plugins update` 手動 | 中（第三方，高頻更新） |
| rust-analyzer-lsp | 1.0.0 | `claude plugins update` 手動 | 低（官方） |

**注意**：`npx -y` 的 MCP 沒有版本鎖定，每次啟動都拉 latest。如果 npm 套件被 compromise，會自動吃到惡意版本。可考慮改為固定版次（如 `npx @modelcontextprotocol/server-brave-search@0.6.2`）降低風險。

## 相關頁面

- [[tech-stack]] — Singularity World 技術選型
- [[collaboration]] — 協作約定
- [[implementation-pipeline]] — 實作管線
