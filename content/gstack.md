# GStack — AI 工程工作流平台

Garry Tan（YC 總裁）的 AI agent 工作流系統。38+ 個 skill 目錄，每個 SKILL.md 編碼完整工作流程（Think → Plan → Build → Review → Test → Ship → Reflect）。我們不用它的程式碼，用它的思維模式。

## 核心哲學

### Thin Harness, Fat Skills
框架薄、技能厚。harness 只做路由和生命週期，所有領域知識都在 SKILL.md 裡。和我們的「工具是本能、CLAUDE.md 是叮嚀」原則一致。

### Boil the Lake
出規格時做到完整。AI 讓完整性的邊際成本趨近零。碼農（笨模型）不需要猜任何細節，規格有縫隙它就亂猜。

### What Does 10 Look Like
動手前先定義完美狀態，再往回推 MVP。沒有錨點，規格容易漏東西或方向歪。

### Search Before Building
動手前先查有沒有現成的。MCP 生態和 GitHub 先搜 5 分鐘，再決定自建還是用現成的。

### Investigate Before Fix
碼農交回來的東西出問題，先查根因再動手。不猜、不急修。

## 架構特徵

- **Daemon browser model**：持久 Chromium（CDP），50+ CLI 命令，localhost HTTP 控制
- **Skill preamble**：每個 skill 開頭注入 session state tracking（telemetry、routing、proactive mode）
- **AGENTS.md injection**：agent 專案注入行為規範
- **Conductor pattern**：conductor.json 定義 skill 路由、模型選擇、token 配額
- **Multi-host**：dev/prod/staging 分離，hosts/ 目錄管理

## 38+ Skills 概覽

涵蓋完整軟體工程流程：
- 開發：code、debug、test、refactor、deploy
- 研究：research、benchmark、explore
- 設計：design、ux-review
- 營運：monitor、incident、security-audit
- 知識：ingest、query、maintain、briefing
- 溝通：email、calendar、meeting-prep

每個 skill 都有 forcing questions（強制自問清單）和 workflow steps（結構化步驟）。

## 我們採納了什麼

| 來源 | 轉化為 | 位置 |
|------|--------|------|
| Investigate before fix | PreToolUse → Edit hook | `~/.claude/hooks/pre-edit-investigate.sh` |
| Search before building | PreToolUse → Write hook | `~/.claude/hooks/pre-write-search-first.sh` |
| 沉澱反思 | Stop hook | `~/.claude/hooks/stop-sedimentation.sh` |
| Boil the Lake + What does 10 look like | `/boil-the-lake` skill | `~/.claude/skills/boil-the-lake/SKILL.md` |
| Compiled truth + timeline | 頁面格式（來自 [[gbrain]]） | 所有 wiki 頁面 |

## 沒用但值得知道的

- **Browser daemon**：我們沒有瀏覽器自動化需求，但架構設計（持久程序 + HTTP 控制）是好模式
- **Conductor routing**：多 skill 路由 + 模型選擇表。我們規模還不需要，但如果 skill 數量成長到 10+ 可以參考
- **Cron schedule**：20+ 定時任務（dream cycle、health check、enrichment）。我們目前靠 claude-mem 的 extractor 做類似事
- **Sub-agent routing**：模型選擇表（Opus 審、Sonnet 執行、Haiku 分類）。和我們的 Advisor Tool 管線制（大→小→大）思路相通

---

- 2026-04-12: 從 blocktempo 報導發現，下載 gstack-main 到本機
- 2026-04-12: 初步評估認為「用處不大」，被用戶糾正——價值在思維模式不在工具
- 2026-04-12: 提取 4 個核心行為模式，寫入 feedback memory
- 2026-04-12: 用戶要求轉化為本能（hooks/skills），不只是記憶（叮嚀）
- 2026-04-12: 徹讀全部原始碼（51 目錄、38+ SKILL.md、所有架構文件、scripts/lib/agents/hosts）
- 2026-04-12: 實作 3 個 hooks + 1 個 skill，註冊進 settings.json
