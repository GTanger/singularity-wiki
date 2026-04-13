---
title: 對話池組合機制
tags: [npc, dialogue, template, pool, combination]
sources:
  - docs/reference/對話池模擬運作機制.md
  - docs/implementation/對話池—從固定句到組合句.md
date: 2026-04-11
---

# 對話池組合機制

對話池是 **非 LLM 的模板組合引擎**。不需要 GPU，萬人同時跑不吃算力。設計目標：50 條模板 × 多槽位 × 大詞表 = 遠超 50 種不重複輸出。

## 完整呼叫流程

```
client: do_action("Talk", target_npc_id)
  → server: handleDoAction
  → server: buildTalkNarrative
  → db: PickFromDialogue(occupationID, key, personality)
  → db: FillPlaceholders(line, entity, room, time)
  → return NPC 回覆文字
```

## 佔位符系統

| 佔位符 | 來源 | 範例值 |
|--------|------|--------|
| `{name}` | NPC 個體名 | 王五 |
| `{room}` | 當前房間名 | 城隍廟前 |
| `{time}` | 遊戲時段 | 清晨 / 正午 / 傍晚 |
| `{mood}` | disposition | 沒好氣 / 懶洋洋 |
| `{verb}` | 動作小詞表 | 擦了擦汗 / 嘆了口氣 |
| `{thing}` | 物件小詞表 | 這把劍 / 今天的生意 |
| `{goods}` | 職業販賣品類 | 草藥 / 鐵器 |

## 八種延伸機制

| 機制 | 效果 |
|------|------|
| 多槽位 × 大詞表 | 1 條 → 數百種組合 |
| 情境注入 | 同句不同地點/時段語感不同 |
| 片段組合 | 前/中/後段各抽一項拼接 |
| 池子混用 | talk 時 12% 機率抽 greet |
| 情境權重 | 依時段/房間提高含關鍵字句權重 |
| 微變體 | 句尾啦/呀、口語化置換 |
| 子槽位 | 40% 機率組合成複合動詞 |
| seed 機制 | hash(entityID + roomID + key) 可重現 |

## 性格影響

- Boldness 高：偏向池後半（強勢句）
- Boldness 低：偏向池前半（謙遜句）
- Sensitivity 高：偏長句、熱絡語氣
- Sensitivity 低：偏短句、冷淡語氣

## 與 LLM Talk 的關係

對話池是 **LLM 失敗時的 fallback**，不是替代品。也獨立用於 NPC 進房/離房反應句、Wander/Work/Idle 出發文案。

## 相關頁面

- [[npc-system/dialogue]] — 對話系統總覽
- [[npc-system/interaction]] — NPC 間交互行為
- [[npc-system/identity-emergence]] — 職業影響對話池選擇