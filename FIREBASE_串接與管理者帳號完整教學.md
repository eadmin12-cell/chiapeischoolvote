# 嘉北國小投票系統 Firebase 串接與管理者帳號完整教學

## 1. 目標與現況

本文件對應目前專案檔案：
- `/Users/chunsheng/Desktop/市長投票/嘉北國小自治市小市長線上投票.html`
- `/Users/chunsheng/Desktop/市長投票/candidates.config.js`

目前程式已包含：
1. 使用 Firebase Web SDK（`firebase-app`, `firebase-auth`, `firebase-firestore`）
2. 學生投票後寫入 Firestore `votes` collection
3. 右上角管理者登入（Email/Password）
4. 管理者可查各候選人累積票數

你現在需要完成的是：
1. 在 Firebase Console 建立專案
2. 取得 `firebaseConfig` 並填入 HTML
3. 開啟 Authentication（Email/Password）
4. 建立管理者帳號
5. 開啟 Firestore Database
6. 設定安全規則（投票可寫入、票數只允許管理者登入查看）

---

## 2. 前置準備

請先確認：
1. 你可以登入 Google 帳號
2. 可連到 [Firebase Console](https://console.firebase.google.com/)
3. 你的投票網頁是由 HTTPS 網站或受信任網域提供（正式上線時建議）

補充：
- 目前你在本機 `file://` 測試，某些瀏覽器會對模組/權限有限制；上線建議部署到 Firebase Hosting。

---

## 3. 建立 Firebase 專案（一步一步）

1. 進入 [Firebase Console](https://console.firebase.google.com/)
2. 點選「新增專案」
3. 專案名稱建議：`school-vote-2026`（可自行命名）
4. Google Analytics 可先關閉（本系統非必要）
5. 建立完成後進入專案首頁

---

## 4. 建立 Web App 並取得 `firebaseConfig`

1. 在專案首頁點「</>」(Web)
2. App 暱稱例如：`vote-web`
3. 先不勾「Firebase Hosting」也可以（後續再加）
4. 建立後會看到類似：

```js
const firebaseConfig = {
  apiKey: "...",
  authDomain: "...",
  projectId: "...",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "..."
};
```

5. 複製這段，貼到：
- `/Users/chunsheng/Desktop/市長投票/嘉北國小自治市小市長線上投票.html`
- 找到 `const firebaseConfig = { ... }` 區塊，逐欄替換

---

## 5. 啟用 Authentication（管理者登入）

1. Firebase Console 左側選單：`Build > Authentication`
2. 點「Get started」
3. 進入 `Sign-in method`
4. 開啟 `Email/Password`
5. 儲存

### 建立管理者帳號

方法 A（建議）：
1. Authentication > Users
2. 點「Add user」
3. 輸入 Email（例如 `admin@school.local`）
4. 設定強密碼（至少 12 字元，含大小寫、數字、符號）
5. 建立完成

方法 B：
- 做一個獨立管理後台註冊流程（本專案目前未提供，不建議開放）

管理者登入即是你建立的 Email/Password。

---

## 6. 啟用 Firestore Database

1. 左側選單：`Build > Firestore Database`
2. 點「Create database」
3. 選擇地區（建議離你近，例如 `asia-east1`）
4. 可先用 `Production mode`，規則再手動改（推薦）
5. 建立完成

---

## 7. 目前資料結構（程式已使用）

### Collection: `votes`

每一筆文件由投票時新增，欄位：
- `candidateNumber`：數字（候選號次）
- `mayorName`：字串（正候選人姓名）
- `viceName`：字串（副候選人姓名）
- `className`：字串（班級）
- `createdAt`：Firestore server timestamp

範例文件：

```json
{
  "candidateNumber": 8,
  "mayorName": "侯杏蓁",
  "viceName": "蔡宗宏",
  "className": "501",
  "createdAt": "serverTimestamp"
}
```

---

## 8. Firestore 安全規則（重點）

你有兩種需求：
1. 投票頁需要可新增票
2. 管理者登入後才能讀取票數

可先使用以下規則：

```text
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // 允許任何人新增投票（如果要更嚴格，需加入防機器人或校內登入）
    match /votes/{voteId} {
      allow create: if request.resource.data.keys().hasOnly([
        'candidateNumber', 'mayorName', 'viceName', 'className', 'createdAt'
      ])
      && request.resource.data.candidateNumber is int
      && request.resource.data.mayorName is string
      && request.resource.data.viceName is string
      && request.resource.data.className is string;

      // 只有登入者可讀（你可以再限縮成特定管理者 email）
      allow read: if request.auth != null;

      // 不允許前端直接修改或刪除投票資料
      allow update, delete: if false;
    }
  }
}
```

### 限制成只有特定管理者 email 可讀（進階）

```text
allow read: if request.auth != null
  && request.auth.token.email in [
    'admin@school.local',
    'principal@school.local'
  ];
```

注意：email 名單寫死在規則裡，改名單就要再發布規則。

---

## 9. 管理者帳號最佳實務

1. 不要共用同一組帳密給多人
2. 每位管理者一組帳號
3. 密碼至少 12 碼
4. 定期輪替密碼（例如每學期）
5. 人員異動立即停權（Authentication > Users 刪除或禁用）

---

## 10. 你的程式需要填哪些值

檔案：`/Users/chunsheng/Desktop/市長投票/嘉北國小自治市小市長線上投票.html`

找到：

```js
const firebaseConfig = {
  apiKey: "請填入你的 apiKey",
  authDomain: "請填入你的 authDomain",
  projectId: "請填入你的 projectId",
  storageBucket: "請填入你的 storageBucket",
  messagingSenderId: "請填入你的 messagingSenderId",
  appId: "請填入你的 appId"
};
```

逐欄替換成 Firebase Console 給你的值，不要多空白、不要少引號。

---

## 11. 測試流程（建議照順序）

1. 打開網頁
2. 任選候選人投票一次
3. 到 Firestore 看 `votes` 是否新增一筆
4. 點右上「管理者」
5. 用你建立的管理者 Email/Password 登入
6. 確認票數表格能正常顯示
7. 再投 1 票，按「重新整理票數」，數字應增加

---

## 12. 常見錯誤與排查

### A. `Firebase: Error (auth/invalid-login-credentials)`
原因：帳密錯誤
處理：
1. 確認 Email 無打錯
2. 到 Authentication > Users 重設密碼

### B. `Missing or insufficient permissions`
原因：Firestore 規則不允許
處理：
1. 檢查你是否有登入管理者
2. 檢查 rules 是否已發佈
3. 確認 `votes` 的 read/create 條件

### C. `Firebase App named '[DEFAULT]' already exists`
原因：重複初始化
處理：
- 現行程式只初始化一次，不要重複貼第二份 SDK 初始化碼

### D. 網頁無法載入 Firebase 模組
原因：網路/瀏覽器限制/企業防火牆
處理：
1. 用正常網路測試
2. 檢查 `https://www.gstatic.com/firebasejs/` 是否可連

---

## 13. 正式上線建議（強烈）

1. 使用 Firebase Hosting 發布，不用長期 `file://`
2. 設定 App Check（防濫用）
3. 增加每位學生唯一識別（學號/座號）與防重複投票
4. 後台統計改為 Cloud Function 彙整，降低前端直接讀全量資料
5. 為管理者加入第二層驗證（若組織政策允許）

---

## 14. 管理者帳號是什麼？

在本系統中，「管理者帳號」就是你在 Firebase Authentication 裡建立的 Email/Password 使用者。

建立位置：
- Firebase Console > Authentication > Users > Add user

登入位置：
- 投票頁右上角「管理者」按鈕

---

## 15. 你現在最少要做的 5 件事

1. 建立 Firebase 專案
2. 啟用 Authentication Email/Password
3. 建立至少 1 組管理者帳號
4. 啟用 Firestore 並發布規則
5. 把 `firebaseConfig` 寫入 HTML

完成後這個投票系統就能實際記票與查票。
