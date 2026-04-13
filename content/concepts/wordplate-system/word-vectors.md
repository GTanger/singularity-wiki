---
title: 詞元與向量
aliases: [Word Elements, 詞元, Word Vectors]
tags: [奇點世界, 詞盤, 詞元, 向量, embedding, pgvector, bge-m3]
sources:
  - docs/reference/詞盤彙整.md
  - docs/reference/詞盤系統收斂規格.md
  - docs/implementation/016_詞元embedding生成—碼農工單.md
date: 2026-04-11
status: 已完成
parent: "[[index]]"
---

# 詞元與向量

> 500 顆詞元構成詞盤的基本粒子。每顆詞元攜帶 1030 維混合向量，十類語義決定句法位置，五類來源決定採集管道。

## 詞元池

**已完成**——`data/config/word_elements.json`，共 500 顆。

每顆詞元的資料結構：

| 欄位 | 說明 |
|------|------|
| `id` | 唯一 ID（如 `"zhan"`） |
| `char` | 單字（如「斬」） |
| `semantic` | 語義類（十類之一） |
| `sources` | 可能來源陣列（獸/植/礦/水/氣） |
| `desc` | 世界觀語義定義（如「利刃橫過的一瞬」） |
| `embedding_desc` | 豐富描述，供 bge-m3 生成 embedding |

## 十類語義

| 語義類 | 角色 | 數量 | 句法位置 |
|--------|------|------|----------|
| 主（本命） | 主詞 | 20 | 句首，決定「誰」 |
| 屬 | 形容詞 | 80 | 修飾語 |
| 動 | 動詞 | 70 | 核心語 |
| 受 | 受詞 | 60 | 目標語，決定作用對象 |
| 構 | 名詞/框架 | 60 | 承載語 |
| 態 | 副詞/狀態 | 70 | 效果語 |
| 補 | 補語 | 70 | 結果/程度語 |
| 助 | 語氣詞 | 30 | 改變句性 |
| 否 | 否定詞 | 20 | 反轉語義 |
| 玄 | 元參數 | 20 | **不入句，改擲骰本身** |

**合計**：20+80+70+60+60+70+70+30+20+20 = **500 顆**

### 玄類詞元的特殊性

玄類詞元不參與[[sentence-crafting|造句]]，而是入竅後修改[[three-fork|疊煉]]系統的擲骰參數本身——成功率偏高、漲幅偏大。玄類不入句，入骰。

## 五類來源

獸、植、礦、水、氣。同一詞元可有多來源，來源決定該詞元從哪類資源點的[[five-layer-pipeline|五層管線]]中產出。

## 詞元屬性與加成對照

詞元入竅後依其屬性產生特殊加成：

| 詞元屬性（可擴充） | 特殊加成效果 |
|-------------------|------------|
| 火 | 火系傷害＋、火抗＋、火系技能增幅 |
| 冰 | 冰系傷害＋、冰抗＋、遲緩/凍結相關 |
| 堅韌 | 物防＋、破防抗性 |
| 疾速 | 行動序、跑速、攻速＋ |
| 靈敏 | 命中、閃避＋ |
| 記憶 | 技能冷卻、敘事相關（可選） |

後端維護「允許的詞元屬性」清單。剝名/蒐集得到的詞元帶其中一類或多類標籤。同竅多詞元的規則需另定（僅取最高或疊加上限）。

## Embedding 向量

**已完成**

每顆詞元擁有 **1030 維混合向量**：

| 維度區間 | 來源 | 說明 |
|----------|------|------|
| 1–1024 維 | 本地 Ollama bge-m3 對 `embedding_desc` 跑一次 embedding | 語義維度，一次性成本 |
| 1025–1030 維 | 手動標註 | 遊戲維度：`atk/def/spd/dot/aoe/heal` |

### 為什麼用 desc 而不是字本身

「斬」一個字丟進 embedding 太短、語義模糊；「利刃橫過的一瞬」精準描述了這個字在本遊戲世界中的含義。`desc` 是人工撰寫的語義定義，embedding 將其轉為數學表達——**文字定義 → 向量 → 遊戲數值，全鏈自動化**。

### 儲存：word_element_embeddings 表

向量存入 PostgreSQL pgvector。表結構：

```sql
CREATE TABLE IF NOT EXISTS word_element_embeddings (
    id TEXT PRIMARY KEY,         -- 詞元 ID，對應 word_elements.json
    char TEXT NOT NULL,          -- 單字，如「斬」
    semantic TEXT NOT NULL,      -- 語義類，如「動」
    desc_text TEXT NOT NULL,     -- desc 原文
    embedding vector(1024) NOT NULL  -- 1024 維向量（bge-m3 輸出）
);
```

> **注意**：表中存的是 1024 維（bge-m3 原始輸出）。6 維手標遊戲維度（gamedims）在查詢時拼接，不在此表中。

### 生成腳本

獨立 binary：`src/bin/generate_embeddings.rs`

執行方式：

```bash
cd ~/Projects/singularity_world
cargo run --bin generate_embeddings
```

流程：
1. 讀取 `data/config/word_elements.json`（500 顆）
2. 逐筆呼叫 Ollama bge-m3 API（`http://localhost:11434/api/embed`）
3. 取得 1024 維向量，以 pgvector 字串格式寫入 PG
4. 使用 `ON CONFLICT DO UPDATE` 確保冪等（重跑不炸）
5. 不引入新依賴（reqwest、serde、postgres 專案皆已有）

**驗證 SQL**：

```sql
-- 確認數量（應為 500）
SELECT COUNT(*) FROM word_element_embeddings;

-- 確認維度（應為 1024）
SELECT id, char, vector_dims(embedding) FROM word_element_embeddings LIMIT 5;

-- 語義相似度測試
SELECT a.char AS char_a, b.char AS char_b,
       1 - (a.embedding <=> b.embedding) AS cosine_similarity
FROM word_element_embeddings a, word_element_embeddings b
WHERE a.id = 'zhan' AND b.id IN ('pi', 'yu')
ORDER BY cosine_similarity DESC;
```

### 新增詞元流程

1. 寫一句 `desc` 與 `embedding_desc`
2. 跑一次 bge-m3 embedding
3. 填寫 6 維 `gamedims`（atk/def/spd/dot/aoe/heal）
4. 零公式調整，即可投入[[sentence-crafting|造句引擎]]

### 向量距離 = 技能差異

- 兩個句向量距離近 → 類似的技能
- 換一顆詞元 → 句向量偏移 → 效果**連續滑動**，不是離散跳變
- 「銳·斬·弧」和「堅·盾·殼」在向量空間裡自然落在攻擊區和防禦區的對角

這是連續光譜，非離散分類。詳見[[sentence-crafting|造句系統]]。

## 相關頁面

- [[index]] — 詞盤系統總覽
- [[five-layer-pipeline]] — 詞元如何從世界中產出
- [[sentence-crafting]] — 詞元向量如何合成技能
- [[three-fork]] — 裸詞元的四條去路

---

*編譯自設計文檔群，2026-04-11*
