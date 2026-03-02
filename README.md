# html-input-file-type-test

測試 HTML `<input type="file">` 在不同 `accept` 屬性設定下的行為差異。

## 目的

探討瀏覽器在開啟檔案選擇器時，不同 `accept` 寫法對可選取檔案類型的篩選效果，尤其針對音訊檔案（如 `.m4a`）的相容性問題進行驗證。

## 測試項目

| 項目 | `accept` 值 | 說明 |
|------|-------------|------|
| 上傳圖片 | `image/*` | 允許所有圖片格式，選取後顯示預覽縮圖 |
| 上傳聲音 | `audio/*` | 廣泛 MIME 類型，理論上涵蓋所有音訊格式（含 `.m4a`） |
| 上傳聲音 2.0 | `.mp3,.wav,.ogg,.m4a` | 指定副檔名，明確允許 `.m4a` |
| 上傳聲音 3.0 | `audio/mpeg,audio/wav,audio/ogg,audio/mp4,audio/x-m4a` | 指定具體 MIME 類型，含 `audio/x-m4a` |
| 上傳任意檔案 | _(無限制)_ | 不設 `accept`，可選取任何檔案 |

## 使用方式

直接用瀏覽器開啟 [index.html](index.html) 即可，無需任何建構步驟或相依套件。

選取檔案後，頁面會顯示該檔案的 `name`、`size`、`type` 等 metadata，方便比對各種 `accept` 設定實際過濾到的 MIME 類型。

## 觀察重點

- `audio/*` 與明確指定副檔名 / MIME 類型，在不同作業系統或瀏覽器上的篩選結果是否一致
- `.m4a` 檔案的 MIME 類型可能為 `audio/mp4` 或 `audio/x-m4a`，不同環境回報值有所差異
