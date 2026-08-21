# hungshinlee.github.io

李鴻欣 (Hung-Shin Lee) 的個人網站 — 國立臺灣師範大學跨域科技產業創新研究學院 AI 跨域應用研究所。

正式網址：<https://web.ntnu.edu.tw/~hslee/>
鏡像：<https://hungshinlee.github.io/>

雙語（繁中／英文），單一自足的靜態頁面，無建置流程、無相依套件。

## 檔案

| 檔案 | 說明 |
|---|---|
| `index.html` | 整個網站 — 版面、樣式、內容資料全在這一個檔案裡 |
| `support.js` | Claude Design 的 `<x-dc>` runtime，負責樣板插值與中英切換 |
| `photo.jpg` | 個人照，同時作為連結預覽圖 (`og:image`) |

## 本機預覽

```sh
python3 -m http.server 8000
```

然後開啟 <http://localhost:8000/>。直接用 `file://` 開啟不行 — runtime 需要透過 HTTP 載入。

## 部署

推送到 `main` 只完成一半：

1. **GitHub Pages** — `git push` 後自動部署到 `hungshinlee.github.io`
2. **NTNU（正式網址）** — 沒有自動化，需手動用 FTPS 上傳 `index.html`

詳細步驟與注意事項見 [CLAUDE.md](CLAUDE.md)。
