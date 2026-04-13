---
title: 開發流程與設定參數
tags: [architecture, devops, quality-gate, config, parameters]
sources:
  - docs/dev/README.md
  - docs/migration/README.md
  - docs/開機啟動.md
  - docs/config/PARAMETERS_INDEX.md
  - docs/config/gametext_and_prompts.md
date: 2026-04-11
---

# 開發流程與設定參數

## 唯一部署流程

```bash
./start
```

**`./start` 是改碼後的唯一正式流程：**
1. 完整品質閘門（clippy → test → checkrooms）
2. `cargo build --release`
3. Hex trunk（地圖資料驗證）
4. 服務重啟

**不得只改檔不啟動。**

## 品質閘門（Quality Gates）

| 指令 | 說明 | 必過條件 |
|---|---|---|
| `cargo clippy -- -D warnings` | 靜態分析 | 零警告 |
| `cargo test` | 單元測試 | 全過 |
| `cargo run --bin checkrooms` | 房間資料驗證 | 無錯誤 |
| `make verify` | 以上三者整合 | pre-push 前跑 |

## systemd 服務

- 部署：單機 Linux Mint
- 對外：Cloudflare Tunnel
- 遠端管理：AnyDesk
- PORT=1721

## 設定參數索引

### `data/config/server_defaults.json`

| 參數 | 用途 |
|---|---|
| `port` | 伺服器埠（預設 1721） |
| `tick_interval_ms` | 主迴圈間隔 |
| `ollama_model` | NPC Talk 用模型名稱 |
| `ollama_host` | Ollama 服務位址 |
| `npc_dialogue_score_threshold` | 對話品質門檻（≥35） |

### `data/config/simulation.json`

| 參數 | 用途 |
|---|---|
| `npc_npc_pair_pick` | NPC-NPC 配對抽樣數 |
| `npc_npc_social` | 社交觸發率 |
| `dialogue_anchors` | 對話錨點設定 |
| `idle_tick_interval` | 閒置輕量 tick 間隔 |

## 文案與 Prompt 設定

- `data/config/gametext.json` — 系統訊息模板
- `data/templates/llm_prompts.json` — LLM prompt 模板
- `data/templates/PLAYER_TALK_WEB_LLM.md` — 玩家 Talk 完整 prompt

## 歷史遷移紀錄

### SQLite → JSON → PostgreSQL
- 現行：PostgreSQL 為唯一權威，JSON 僅種子

### Go → Rust（ADR 010）
- 所有 Go 代碼已廢棄，無過渡期、無 FFI 橋接

## 相關頁面

- [[tech-stack]] — 技術選型背景
- [[architecture-refactor]] — 模組結構整頓
- [[observation-system]] — 算力配置策略