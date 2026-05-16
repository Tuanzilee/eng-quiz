# eng-quiz SOP

> 線上版:**https://tuanzilee.github.io/eng-quiz/**
> 工程化 Repo:`~/Desktop/eng/`
> 工程化說明(給 Claude 看):見同目錄 `CLAUDE.md`

---

## 一、日常使用

### Mac 本機刷題

```bash
cd ~/Desktop/eng
python3 -m http.server 8080
```

打開瀏覽器:`http://localhost:8080/english_quiz.html`

> ⚠ 不能直接點兩下 `english_quiz.html` 開啟(`fetch()` 在 `file://` 會失敗)

### iPad / 手機刷題

Safari 進線上版即可:
```
https://tuanzilee.github.io/eng-quiz/
```

⚠ **目前無 PWA**(沒有 sw.js / manifest)。可以「加入主畫面」做成 icon,但**沒有離線能力**,每次開都要連網。

→ 若以後想要 iPad 離線刷題,跟 Claude 講「幫 eng 加 PWA」,他會仿 stats 的 sw.js 補上。

---

## 二、勘誤 / 補題流程

### 流程總覽

```
看到題目錯 / 想補題
   ↓
打開 claude.ai (網頁版),貼題目截圖 + 描述「哪題哪欄錯」
   ↓
讓 Claude 生 prompt(包含:題目 id、要改的欄位、新值、依據)
   ↓
把 prompt 丟給本機的 Claude Code(在 ~/Desktop/eng 目錄)
   ↓
Claude Code 改 vocab_data.json 或 grammar_data.json + commit + push
   ↓
等 1-2 分鐘 GitHub Pages 部署
   ↓
強制重整(下方第三節)→ 拿到新版
```

### 為什麼直接改 JSON、不在 app 內勘誤

跟 stats 同一套理由:
- **直接改源頭**:題目資料只在 `*_data.json`,改它就好
- **可追蹤**:每次勘誤都是一個 git commit,有歷史
- **eng 沒 PWA**:重整就拿新版,不像 stats 還要管 sw.js cache 版本

### 對 claude.ai 出 prompt 的範例

> 截圖一張 NTU 107 單字考題,告訴 Claude:「`NTU_107_V05` 標準答案是 C,但 vocab_data.json 寫 B,幫我生 prompt 給 Claude Code 改。」

它會回類似:

```
修正 NTU_107_V05 答案

在 ~/Desktop/eng/vocab_data.json:
  找到 id="NTU_107_V05"
  把 answer 從 "B" 改成 "C"
  在 distractor_notes 補上對 B 的誤答解析

commit 「修正 NTU_107_V05 答案」並 push。
(注意:eng 沒 PWA,不需要 bump 任何版本)
```

把這段貼進 `cd ~/Desktop/eng` 後啟動的 Claude Code,等它做完。

---

## 三、題庫更新後如何刷新

### Mac
```
Cmd + Shift + R(強制重新整理)
```

### iPad Safari
1. 下拉頁面強制重整
2. 或關閉 tab 再打開

> eng 沒有 PWA,所以**沒有快取卡舊版的問題**,普通重整就行(這比 stats 簡單)。

---

## 四、答題紀錄說明

| localStorage key | 存什麼 |
|---|---|
| `vocab_progress_v1` | 每題 vocab 的對 / 錯次數 |
| `vocab_known_v1` | 標記為「已認識」的 vocab id 集合 |
| `grammar_progress_v1` | 每題 grammar 的對 / 錯次數 |
| `mock_history_v1` | 模考歷史紀錄(時間 / 分數 / 答題明細) |

- 全部存在**該裝置的瀏覽器 localStorage**
- 清除瀏覽器快取會消失
- Mac 跟 iPad **不互通**(沒做帳號同步)
- 建議主要在一台裝置刷題

---

## 五、題目 ID 規則

### vocab(322 題)

`學校_年份_V兩位數`

| 學校 | 範例 |
|---|---|
| 台大 | `NTU_107_V01` |
| 台綜成大 | `NCKU_113_V15` |
| 東吳 | `SCU_109_V08` |
| 政大 | `NCCU_111_V22` |
| 心理學專業英文 | `PSYCH_xxx_V01` |
| 自製模擬題 | `MOCK_xxx_V01` |

### grammar(107 題)

`GR_主題縮寫_三位數`,例:

- `GR_TENSE_001`(時態)
- `GR_SUBJ_VERB_005`(主詞動詞一致)
- `GR_INVERSION_003`(倒裝句)

主題涵蓋:時態、主詞動詞一致、介系詞慣用、代名詞、倒裝句、假設語氣、分詞構句、助動詞、動名詞與不定詞、句型結構、名詞子句 ...

---

## 六、沒裝的功能(誠實告知)

| 沒裝 | 為什麼 | 想要的話 |
|---|---|---|
| PWA / 離線 | 主要在 Mac 刷,線上版加主畫面就夠 | 跟 Claude 講「加 PWA」 |
| AI 對話 / 引導 | 真考題 + 解析 + 誘答選項解析已經夠用 | 暫不規劃 |
| in-app 勘誤面板 | stats 教訓:勘誤該在源頭改、不該臨時改 client | **不會加**(設計選擇) |
| 個人筆記 | 同上 | **不會加**(設計選擇) |
| 跨裝置同步 | 沒有後端,純前端 | 暫不規劃 |

---

## 七、常見問題

**Q:打開網頁顯示「載入失敗」 / 空白**
A:確認用 `python3 -m http.server 8080` 啟動,不能直接點兩下 html。網址要進 `/english_quiz.html`。

**Q:GitHub Pages 網址打不開**
A:等多幾分鐘,或到 repo Settings → Pages 確認部署狀態。

**Q:題庫更新後沒變化**
A:強制重整(Cmd+Shift+R)。eng 沒 PWA,不會卡 sw.js cache,單純是瀏覽器自己的 HTTP cache。

**Q:Claude Code 改完 push 了,但網頁還是舊版**
A:同上,強制重整。或檢查 GitHub Actions / Pages 是否還在部署中(repo 頁面 commit 旁邊的綠勾)。

**Q:答題紀錄不見了**
A:大概是清了瀏覽器快取 / localStorage,沒救。建議定期不要清。換裝置也不會同步。
