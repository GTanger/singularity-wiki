# GBrain — 個人知識腦

Garry Tan（YC 總裁）開源的個人知識管理系統。Postgres-native，結合向量搜尋與關鍵字搜尋，設計給 AI agent 當長期記憶層。

## 核心架構

- **Contract-first**：37 個 operations 定義一次，CLI / MCP / tools-json 全自動生成，零重複
- **Pluggable engines**：PGLite（本地 WASM，零設定）或 Postgres（Supabase，可擴展）。工廠模式 + dynamic import，沒用到的引擎不載入
- **Markdown 為唯一真相源**：Git repo 是權威，DB 是索引。人可直接編輯，git history 保留
- **Hybrid RRF 搜尋**：向量 + 關鍵字 + multi-query expansion + reciprocal rank fusion。沒 API key 退化為純關鍵字，不會壞

## 搜尋實作

三層：
1. **Keyword**：tsvector + GIN 索引，加權（title > compiled_truth > timeline），pg_trgm 模糊比對
2. **Vector**：OpenAI text-embedding-3-large（1536 維），HNSW cosine distance
3. **Hybrid RRF**：多查詢展開（Claude Haiku 生成 2 個替代查詢），各結果列表用 RRF 公式（k=60）融合排序，去重

## Chunking

遞迴分割器，5 層分隔符（段落 → 行 → 句 → 子句 → 詞），貪婪合併至目標大小（300 詞），50 詞重疊。無損不變式：非重疊部分可重組為原文。

## Import 流程

1. 解析 markdown（frontmatter + compiled_truth + timeline）
2. SHA-256 hash 比對，相同跳過（冪等）
3. 分 chunk（compiled_truth 和 timeline 分開切）
4. 嵌入（transaction 外，因為是外部 API 呼叫）
5. Transaction 包住所有 DB 寫入（版本快照 + page + tags 調和 + chunks）

## 頁面格式

compiled truth + timeline 雙層結構：上半可重寫（當前認知），下半 append-only（證據軌跡）。我們已採用此格式。

## 我們的使用

- **MCP server** 已安裝，PGLite 本地引擎，61 頁 115 chunks 已匯入
- 主要用途：wiki 知識的向量搜尋補充（[[llm-wiki-kit]] 是主力，gbrain 是語意搜尋備援）
- 啟動腳本：`~/Projects/gbrain-master/gbrain-serve.sh`

## 值得學習的設計模式

| 模式 | 要點 |
|------|------|
| Contract-first | 定義一次行為契約，所有介面自動生成 |
| Content hash 冪等 | SHA-256 比對，重複 import 自動跳過 |
| Transaction 邊界 | 外部 API 呼叫放 transaction 外，DB 寫入包在裡面 |
| Tag 調和 | 明確兩步驟（刪舊、加新），不留殘留 |
| Cascade delete | Schema 層強制參照完整性，刪 page 連帶清 chunks/tags/links/versions |
| 優雅退化 | 沒 API key 就用關鍵字搜尋，不報錯不停擺 |

---

- 2026-04-12: 從 blocktempo 報導發現 gbrain 開源。下載 gbrain-master + gstack-main 到本機
- 2026-04-12: 安裝 Bun，編譯 gbrain，PGLite init，匯入 Obsidian vault 61 頁並嵌入 115 chunks
- 2026-04-12: 註冊為第 8 個 MCP server（~/.claude/.mcp.json）
- 2026-04-12: 採用 compiled truth + timeline 頁面格式
- 2026-04-12: 徹讀全部原始碼（32 TypeScript 檔、28 測試檔、7 skills、7 recipes、完整文件）
