# CLAUDE.md

## Repo 用途

英文(單字 + 文法)轉學考刷題單頁 app,純前端 + localStorage。

線上部署:**https://tuanzilee.github.io/eng-quiz/**(GitHub Pages,push 後 1-2 分鐘自動部署)

姊妹專案:`~/Desktop/psy/`(普心)、`~/Desktop/stats/`(心統)。**三個 repo 架構各不同**,別套錯。

## 檔案結構

```
eng/
├── english_quiz.html      ← 主程式(~1,310 行)。源 = 部署檔,直接改
├── vocab_data.json        ← 322 題單字(runtime fetch 載入)
├── grammar_data.json      ← 107 題文法(runtime fetch 載入)
├── README.md              ← GitHub repo 首頁
├── SOP.md                 ← 使用者操作手冊
└── CLAUDE.md              ← 本檔
```

### 跟 psy / stats 的關鍵差異

| | psy | stats | **eng** |
|---|---|---|---|
| Build pipeline | ✅ `build_html.py` + template | ❌ | **❌** |
| GitHub Actions | ✅ `build.yml` | ❌ | **❌** |
| PWA(離線 / 主畫面) | ❌ | ✅ `sw.js` + `manifest.json` | **❌** |
| AI hint(Anthropic API) | ❌ | ✅ | **❌** |
| 資料源數量 | 單(inline 進 index.html) | 單(`題庫.json`) | **雙**(`vocab` + `grammar`) |
| in-app 勘誤 | ❌ | 已砍(2026-05-16) | **從未有** |
| 個人筆記 | ❌ | 已砍(2026-05-16) | **從未有** |

→ eng 是**三個 repo 裡最瘦的**。**不要**為了「跟 stats 對齊」就加 PWA / AI hint / build pipeline——精瘦是設計選擇。

## 題庫 schema

### vocab_data.json(322 題)

```
{
  "id":              "NTU_107_V01",   ← 不要改;格式 學校_年份_V兩位數
  "source":          "NTU",            ← NTU / NCKU / SCU / NCCU / PSYCH / MOCK
  "year":            107,
  "year_ad":         2018,
  "exam_section":    "...",
  "difficulty":      "easy / medium / hard",
  "sentence":        "...",            ← 含 ___ 空格
  "options":         [...],
  "answer":          "B",              ← 常勘誤這個
  "target_word":     "...",
  "chinese":         "...",
  "definition":      "...",
  "example":         "...",
  "distractor_notes": { "A": "...", "C": "...", "D": "..." },  ← 誘答選項解析
  "sentence_zh":     "..."
}
```

### grammar_data.json(107 題)

```
{
  "id":              "GR_TENSE_001",   ← 不要改;格式 GR_主題縮寫_三位數
  "grammar_topic":   "時態",            ← 中文主題名
  "difficulty":      "...",
  "question":        "...",
  "options":         [...],
  "answer":          "B",
  "explanation":     "...",
  "common_mistakes": "..."
}
```

→ **改 `english_quiz.html` 不會改變題目資料**;題目資料只在 `vocab_data.json` / `grammar_data.json`。

## 勘誤工作流

```
1. claude.ai 出 prompt:「NTU_107_V01 答案應該是 C 不是 B,因為 ...」
2. Claude Code 在 vocab_data.json 或 grammar_data.json 找到對應 id,改欄位
3. git add 對應 json
4. git commit -m "修正 NTU_107_V01 答案"
5. git push origin main
6. GitHub Pages 1-2 分鐘自動部署 → 強制重整即可拿到新版
```

**不需要 bump 任何版本號**(eng 沒 PWA,沒 sw.js cache 問題,push + 強制重整就能看到新版)。

## 保留的子系統(46 個 function)

| 子系統 | 主要 function | localStorage key |
|---|---|---|
| vocab 練習 + 篩選 + 智慧出題 | `renderVList` / `openVocab` / `renderVPractice` / `vSmartNext` / `vFiltered` / `renderVFilters` | `vocab_progress_v1`(對/錯次數) |
| vocab「已認識」標記 | (內嵌在 vocab 系統) | `vocab_known_v1` |
| grammar 練習 + 篩選 + 智慧出題 | `renderGList` / `openGrammar` / `renderGPractice` / `gSmartNext` / `gFiltered` / `renderGFilters` | `grammar_progress_v1` |
| 錯題本(vocab + grammar 合在一頁) | `renderNotebook` / `getVWrongWords` / `getGWrongQs` / `updateNBBadge` | (從 vProg / gProg 推導,無獨立 key) |
| 弱點追蹤 | `renderWeakness` / `computeTopicStats` | (推導) |
| 模考(抽題 / 計時 / 提交 / 歷史) | `startMock` / `submitMock` / `updateMockTimer` / `renderMockSetup` / `renderMockHistory` / `toggleReview` / `resetMock` | `mock_history_v1` |
| Tab + 分頁切換 | `switchTab` / `switchVPhase` / `switchGPhase` | — |

**命名 pattern**:vocab 邏輯前綴 `v`(`vProg` / `vFiltered` / `vSmartNext`)、grammar 用 `g`(`gProg` / `gFiltered` / `gSmartNext`)、兩者通用的 mock 用全名。改/加功能要遵守這個約定。

## 沒有 / 從未有的子系統

明確聲明,**不要自作主張加**:

- ❌ in-app 勘誤(`openEditPanel` / `corrections` 之類) — 勘誤該直接改源頭 JSON,不該 client 端臨時改
- ❌ 個人筆記(`personalNotes` / `renderNoteSection`) — 同上理由
- ❌ AI hint / Anthropic API 對話(`saveApiKey` / `callAI`) — 真考題 + `explanation` + `distractor_notes` 已足夠,刻意不加 AI 對話
- ❌ 匯出題庫鈕(`exportJSON`) — 沒勘誤系統就沒匯出需求
- ❌ Service Worker / PWA — 想要 iPad 離線再加,目前不需要
- ❌ Build pipeline / Actions / template — 架構簡單到不需要

要加上面任何一個之前,**先問**為什麼非加不可。

## 與 Claude 的協作風格

- **直接動手,做完再報**。不需要逐步請示;只在「會動到大量檔案 / 不可逆操作 / 需求模糊」時才停下確認
- **改檔優先用 Edit(str_replace),少用 Write 整檔覆蓋**
- **不要重複問同樣的事**。已經寫進這份 CLAUDE.md 的事,直接照辦
- 改 `english_quiz.html` 後本機開 `python3 -m http.server 8080` 點一下確認沒事
- 勘誤改完 push 後,提醒使用者強制重整(Cmd+Shift+R)

## 已知狀態(2026-05-16)

### 題庫分布

| 類型 | 題數 | 來源分布 |
|---|---|---|
| vocab | 322 | NTU / NCKU / SCU / NCCU / PSYCH / MOCK |
| grammar | 107 | 依文法主題分(時態 / 主詞動詞一致 / 倒裝句 / 假設語氣 / ...等) |

### 題目 ID 規則

- **vocab**:`學校_年份_V兩位數`,例 `NTU_107_V01`、`NCKU_113_V15`
- **grammar**:`GR_主題縮寫_三位數`,例 `GR_TENSE_001`、`GR_SUBJ_VERB_005`

### 相關但不在 repo 內

`~/Desktop/eng-local-backup/eng-read-quiz/english_reading.html` — 936 行 React-style 英文閱讀測驗 app,從未上線、不在任何 repo。**先擱置**,未來若要做閱讀測驗 app 再評估獨立 repo 或併入 eng。

### 待勘誤

(尚未盤點;之前無 in-app 勘誤系統,所有勘誤紀錄都靠 git commit 追)
