---
title: Hex 視覺調整
tags:
  - hex
  - visual
  - frontend
  - canvas
  - pwa
  - singularity-world
source: project--hex-visual.md
date: 2026-04-10
---

# Hex 視覺調整

**日期：** 2026-04-10

## 完成項

- **草原格改用甲骨文字型**：display_char「茻」→「屮屮」，套用 FZJiaGuWen（方正甲骨文）
- **HEX_R 114 → 202**：格寬 ≈ 350px ≈ 現實 10m。移動動畫 duration 基準同步調整為 350px

## 技術事實

- Canvas 不觸發 CSS @font-face 下載：必須用 `document.fonts.load()` 主動預載
- SW cache 必須跟 index.html 版次一起 bump（手機 PWA 拿不到新版）
- FZJiaGuWen.ttf 2.6MB 未子集化：目前只用「屮屮」兩字，後續可用 pyftsubset 壓縮

---

## 相關頁面

- [[concepts/subcell-movement|格內連續移動]]
- [[concepts/hex-map/index|Hex 地圖]]
