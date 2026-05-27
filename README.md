# eng-quiz

英文(單字 + 文法)轉學考刷題,純前端 + localStorage,共 429 題(vocab 322 / grammar 107)。

線上版:**https://tuanzilee.github.io/eng-quiz/**

## 本機刷題(Mac)

```bash
cd ~/Desktop/eng
python3 -m http.server 8080
```

打開瀏覽器:`http://localhost:8080/`

> ⚠ 不能直接點兩下 `index.html` 開啟(`fetch('./*.json')` 在 `file://` 會失敗)。

## 詳細操作 / 勘誤流程

→ 見 [SOP.md](SOP.md)

## 工程說明(給協作的 Claude 看)

→ 見 [CLAUDE.md](CLAUDE.md)
