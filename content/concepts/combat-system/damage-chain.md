---
title: 傷害鏈 — 核心公式與結算
tags:
  - system
  - combat
  - formula
  - singularity-world
sources:
  - docs/reference/戰鬥系統概念—三版融合.md
  - docs/reference/戰鬥系統概念公式—提案版.md
  - docs/reference/戰鬥系統概念設計—提取自gemini概念檔.md
  - docs/decisions/001_combat_unified_rules.md
  - docs/implementation/011_戰鬥接線三件套—碼農工單.md
date: 2026-04-11
---

# 傷害鏈 — 核心公式與結算

> 三版融合之單一戰鬥公式參考。所有已知概念元素皆納入公式，未實裝時以 `1`（乘性中性）或 `0`（加性中性）代入，公式仍成立。

## 概念元素一覽

### 外顯與基礎屬性

| 符號 | 語義 | 來源 | 未實裝預設 |
|------|------|------|-----------|
| EVit | 有效體質（生命、減傷、姿態上限） | 裝備修正後 | EVit := Vit |
| EQi | 有效氣脈（技能燃料、虛弱判定） | 裝備修正後 | EQi := Qi |
| EDex | 有效靈敏（先手、命中、行動頻率） | 裝備修正後 | EDex := Dex |

**有效屬性計算**（已實作 `db::effective_stats`）：遍歷角色 `equipment_slots`，加總各裝備的 `vit_bonus`、`dex_bonus`、`atk_bonus`。

### 內在三軸（[[concepts/character-system/soulseed|SoulSeed]]）

| 符號 | 語義 | 展開公式 | 未實裝預設 |
|------|------|---------|-----------|
| α（能階） | 傷害峰值、爆發 | `0.5 + (amp - AMP_MIN) / (AMP_MAX - AMP_MIN) * 1.5` | α := 1 |
| β（時脈） | 資源利用率、破甲弱化 | `0.5 + (freq - FREQ_MIN) / (FREQ_MAX - FREQ_MIN) * 1.0` | β := 1 |
| γ（相位） | 暴擊/偏轉/反彈機率 | `(phase - PHASE_MIN) / (PHASE_MAX - PHASE_MIN)` | γ := 0 |

### 行動槽與時序

| 符號 | 語義 | 未實裝預設 |
|------|------|-----------|
| Tick | 戰鬥時基 | 100ms |
| AP_per_tick | 每 Tick 恢復量 | `EDex × (0.5 + 0.5×β)`，未實裝 := 1 |
| T_base | 基礎攻擊門檻 | 100 AP → 未實裝視為每輪可出手 |
| T_skill | 技能門檻 | `100 + Skill_Complexity_AP` → 未實裝 := 1 |
| C_combo | 連擊門檻倍率 | 連發時 ×1.2 → 未實裝 := 1 |

### 技能與[[concepts/wordplate-system/index|詞盤]]

| 符號 | 語義 | 未實裝預設 |
|------|------|-----------|
| R | 共鳴階級倍率（一階 1.0 / 二階 1.5 / 三階 2.5） | R := 1 |
| Atk_Power | 技能/詞元基礎威力 | 1 |
| Atk_Gear | 裝備加成 | 0 |
| Skill_Bonus | 技能加成 | 0 |
| Skill_Cost | 氣脈消耗基礎值 | 0 |
| Atk_Penetration | 破甲值 | 0 |
| Skill_γ_match | γ 與技能屬性契合之命中加成 | 0 |

### 地形修正（環境共鳴）

| 標籤 | 符號 | 效果 | 未實裝預設 |
|------|------|------|-----------|
| [靜謐] | M_Silent | γ 波動縮小，可預測 | 1 |
| [富態] | M_Lush | α 放大、消耗略增 | 1（已實裝：lush=1.15） |
| [咬合] | M_Grip | 姿態穩定、僵直降 | 1 |
| [混沌] | M_Chaos | γ 波動放大 | 1（已實裝：chaos=1.08） |

### 元素/狀態/環境（gemma3 融入）

| 符號 | 語義 | 未實裝預設 |
|------|------|-----------|
| Elem_mult | 元素傷害加成 | 1 |
| Status_mult | 狀態效果相互作用 | 1 |
| Env_mult | 環境影響 | 1 |

### 狀態異常與 CT 閾值

| 符號 | 語義 | 未實裝預設 |
|------|------|-----------|
| CT_中毒 | 基礎傷害 × 0.2 | CT := ∞（不觸發） |
| CT_麻痺 | 基礎傷害 × 0.15 | CT := ∞ |
| CT_沉默 | 基礎傷害 × 0.1 | CT := ∞ |

若 `D_Final > CT_類型` 則施加對應異常，持續時間依傷害與抗性決定。

---

## 核心公式（單式鏈）

```
K_Hit     = Atk_EDex / max(Atk_EDex + Def_EDex, 1)
P_Hit     = clamp(0.05, 0.95, K_Hit × 1.2 × (1 + Skill_γ_match))
命中      = random() < P_Hit

綜合傷害_raw = (Atk_Power + Atk_Gear + Skill_Bonus) × R × M_Lush
K_Impact  = 0.8 + 0.4 × α
D_Impact  = 綜合傷害_raw × K_Impact
Pen       = Atk_Penetration + (1 - Def_β) × Const_pen
mit       = Def_EVit / max(Def_EVit + Pen × 20, 1)
D_after_mit = D_Impact × (1 - mit)   [若未命中則 0]

D_Final_base = D_after_mit × Elem_mult × Status_mult × Env_mult

[相位] 若 γ 觸發 Deflect → D_Final = 0
       若 γ 觸發 Crit → D_Final = D_Final_base × 1.5
       否則 D_Final = D_Final_base

Qi_Drain  = Skill_Cost × (2 - β) × M_Lush
[姿態] Posture -= g(D_Final)；若 Posture ≤ 0 → Stagger
[狀態異常] 若 D_Final > CT_類型 → 施加對應異常
```

**未實裝時代入**：Skill_γ_match=0, R=1, M_Lush=1, α=1, β=1, γ=0（不觸發 Crit/Deflect），Atk_Power=1, Atk_Gear=0, Skill_Bonus=0, Pen=0, Elem/Status/Env=1, CT=∞。

---

## 先手與行動資格

- **先手**：`EDex_Atk ≥ EDex_Def` 則攻方先攻；相等時攻方先
- **可出手**：`AP ≥ T_base`（基礎）或 `AP ≥ T_skill × C_combo`（技能/連擊）
- **可放技能**：`EQi ≥ Qi_Drain`

## 命中判定（已實作）

```rust
fn hit_check(rng, atk_dex, def_dex) -> bool {
    let denom = max(atk_dex + def_dex, 1);
    let k_hit = atk_dex / denom;
    let p_hit = clamp(0.05, 0.95, k_hit * 1.2);
    rng.random() < p_hit
}
```

實作位置：`src/combat/mod.rs` `hit_check` 函式。

## 相位干涉

結算最後引入 γ 的隨機擾動：
- **正向 (γ > 0)**：高機率「暴擊 (Critical)」× 1.5 或「破甲 (ArmorBreak)」
- **逆向 (γ < 0)**：高機率「偏轉 (Deflect)」D=0 或「反彈 (Reverse)」
- γ 極端（趨近 ±1）時變數增大
- M_Silent 縮小波動、M_Chaos 放大波動

## 狀態歸零語義

| 歸零 | 語義 |
|------|------|
| HP ≤ 0 | 死亡，需重組（僅**送行**允許至此） |
| EQi ≤ 0 | 虛弱（減傷/命中降） |
| Posture ≤ 0 | 僵直（AP 暫停 1 秒） |

## 名詞對照（gemma3 ↔ 本文件）

| gemma3 | 本文件 |
|--------|--------|
| 基礎傷害 + 裝備 + 技能 | 綜合傷害_raw |
| 怪物防禦/實際傷害 | (1 - mit) |
| 敵方反擊 | 併入 mit 或獨立 Counter |
| 命中率 | P_Hit |
| 元素/狀態/環境 | Elem_mult / Status_mult / Env_mult |
| 暴擊 | γ 觸發 Crit |
| 狀態異常 CT | D_Final > CT_類型 |

## 常數一覽

| 常數 | 建議值 | 說明 |
|------|--------|------|
| 命中率上下限 | 0.05～0.95 | 避免必中/必閃 |
| 減傷分母 | 20 | `Def_EVit/(Def_EVit+20)`，效益遞減 |
| 暴擊倍率 | 1.5 | γ 觸發時 |
| α/β/γ 預設 | 1, 1, 0 | 未傳 SoulSeed 時 |
| 地形 lush 倍率 | 1.15 | 已實裝 |
| 地形 chaos 倍率 | 1.08 | 已實裝 |

## 文字戰報標籤

| 標籤 | 用途 |
|------|------|
| `[Initiative]` | 先手方 |
| `[Action:Skill]` | 命中 + 傷害 |
| `[Action:Miss]` | 揮空 |
| `[Conflict:Crit]` | γ 暴擊（待實裝標籤） |
| `[Conflict:Deflect]` | γ 偏轉 |
| `[Result]` | 剩餘 HP / 勝負 |

## 相關頁面

- [[concepts/combat-system/index|戰鬥系統]] — 總覽、ADR-001、三版融合、實作狀態
- [[concepts/combat-system/philosophy|戰鬥哲學]] — 留人/送行、借物、死亡規則
- [[concepts/character-system/soulseed|SoulSeed]] — α/β/γ 三軸展開
- [[concepts/character-system/three-dimensions|外顯三維]] — Vit/Qi/Dex
- [[concepts/wordplate-system/sentence-crafting|造句系統]] — 共鳴階級 R
