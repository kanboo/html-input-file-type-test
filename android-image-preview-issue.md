# Android 圖片預覽問題：URL.createObjectURL vs FileReader

## 問題描述

在 Android 裝置上透過 `<input type="file" accept="image/*">` 選取圖片後，
使用 `URL.createObjectURL(file)` 產生圖片預覽時，遇到部分情況圖片無法正常顯示。

## Android 的兩種檔案選擇器

Android 系統提供兩種選取檔案的方式：

| 選擇器 | Package | 說明 |
|---|---|---|
| 新版相片選擇器 | `com.android.providers.media.photopicker` | Android 13+ 新增，隱私導向設計，授予**暫時性** content URI 存取權限 |
| 傳統文件選擇器 | `com.android.providers.media.documents` | SAF (Storage Access Framework)，權限相對寬鬆 |

## 根本原因分析

### `URL.createObjectURL(file)` 的問題

```js
const url = URL.createObjectURL(file); // 建立 blob: URL
img.src = url;                          // 之後才去讀取
```

- `createObjectURL` 建立的是一個 **lazy reference（延遲引用）**，即 `blob:` URL
- 它**不會立刻讀取**檔案內容，而是等到 `<img>` 實際渲染時才回去存取原始 content URI
- `photopicker` 授予的是**暫時性**存取權限，當瀏覽器（WebView）實際去讀取時，該權限可能已經過期
- 結果：`photopicker` 選出的圖片無法顯示；`documents` 選出的圖片可以正常顯示

### `FileReader.readAsDataURL(file)` 的解法

```js
const reader = new FileReader();
reader.onload = function (e) {
  img.src = e.target.result; // data:image/jpeg;base64,...
};
reader.readAsDataURL(file); // 立刻讀取，在 change 事件觸發當下完成
```

- `FileReader` 在 `change` 事件觸發的**當下就立刻把整個檔案讀進記憶體**
- 轉換成 `data:image/...;base64,...` 格式，是一個**自包含字串**
- 後續不再需要存取原始 content URI，因此不受 URI 權限生命週期影響
- 結果：`photopicker` 和 `documents` 兩種方式選出的圖片，均可正常顯示 ✅

### 比喻

| 方式 | 比喻 |
|---|---|
| `URL.createObjectURL` | 給一張「借條」，晚點再來拿東西 ——  等你來拿時借條可能已過期 |
| `FileReader` | 當場把東西搬走，不依賴憑證 |

## 結論

在 Android WebView / 行動裝置瀏覽器環境中，若需要預覽使用者上傳的圖片，
建議優先使用 **`FileReader.readAsDataURL`** 而非 `URL.createObjectURL`，
以確保相容兩種檔案選擇器。

## 測試結果

| 方法 | `photopicker` | `documents` |
|---|---|---|
| `URL.createObjectURL` | ❌ 無法顯示 | ✅ 正常 |
| `FileReader.readAsDataURL` | ✅ 正常 | ✅ 正常 |

## 參考程式碼

```html
<input type="file" id="input-image" accept="image/*" />
```

```js
document.getElementById('input-image').addEventListener('change', function () {
  const file = this.files[0];
  if (!file) return;

  const reader = new FileReader();
  reader.onload = function (e) {
    const img = document.createElement('img');
    img.src = e.target.result; // base64 data URL，不依賴 content URI
    document.getElementById('preview').appendChild(img);
  };
  reader.readAsDataURL(file);
});
```

## 注意事項

- `FileReader` 會將整個檔案載入記憶體，對於大尺寸圖片需注意記憶體使用量
- 若需釋放 `URL.createObjectURL` 產生的 URL，須手動呼叫 `URL.revokeObjectURL(url)`；`FileReader` 的 data URL 則由 GC 自動回收
- 此問題特別容易出現在 Android WebView 情境（如 React Native WebView、Flutter WebView 等）
