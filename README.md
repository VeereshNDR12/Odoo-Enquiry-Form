# Odoo お問い合わせフォーム → GitHub Issues 自動作成

このプロジェクトでは、**Googleフォーム**から送信された内容をもとに、**GitHubのIssue**を自動で作成します。Odooのサポート受付など、問い合わせ管理をGitHubで行いたい場合に便利です。

---

## ✅ 主な機能

* Googleフォームで問い合わせを受付
* Google Apps Scriptを使って、GitHubのIssueを自動作成
* タイトル・説明・送信日時をIssueに追加
* 日本語と英語のフォームに対応
* 処理ログやGitHubの返答を記録

---

## 🛠️ セットアップ方法

### 1. Googleフォームを作る

次のようなフォーム項目を作成してください：

1. **お問い合わせタイトル**
2. **説明**
3. **画像のアップロード**（※現時点では対応していません）

---

### 2. フォームとスプレッドシートを連携する

1. フォームの「回答」タブから、**スプレッドシートを作成**します（緑色のアイコンをクリック）。
2. 作成されたスプレッドシートを開きます。

---

### 3. Apps Script を設定する

1. スプレッドシートのメニューから、**拡張機能 → Apps Script** を選択。
2. 初期コードを削除し、下記のコードを貼り付けます：

```javascript
var GITHUB_TOKEN = ""; // GitHubトークンをここに入力
var REPO_OWNER = "";   // GitHubのユーザー名や組織名
var REPO_NAME = "Odoo-Enquiry-Form"; // リポジトリ名

function onFormSubmit(e) {
  const responses = e.values;
  Logger.log("Responses: " + JSON.stringify(responses));

  const timestamp = responses[0];
  const title = responses[1];              // お問い合わせタイトル
  const description = responses[2];        // 説明
  const imageUpload = responses[3] || "なし"; // サポート画像（Google Drive リンク）
  const email = responses[4] || "未記入";      // メールアドレス

  if (!title || !description) {
    Logger.log("Missing required fields: title or description");
    return;
  }

  const issueTitle = `[Odoo Inquiry] ${title}`;
  const issueBody = `
**Timestamp**: ${timestamp}

**Email**: ${email}

**Description**:
${description}

**Support Image**:
${imageUpload !== "なし" ? imageUpload : "（画像はアップロードされていません）"}
`;

  const payload = {
    title: issueTitle,
    body: issueBody,
    labels: ['Odoo']
  };

  const options = {
    method: 'post',
    contentType: 'application/json',
    headers: {
      Authorization: `Bearer ${GITHUB_TOKEN}`
    },
    payload: JSON.stringify(payload),
    muteHttpExceptions: true
  };

  const url = `https://api.github.com/repos/${REPO_OWNER}/${REPO_NAME}/issues`;
  const response = UrlFetchApp.fetch(url, options);
  Logger.log("Status Code: " + response.getResponseCode());
  Logger.log("GitHub Response: " + response.getContentText());
}

```

---

### 4. GitHubのトークンを取得する

1. GitHubにログイン → プロフィール →「**Settings**」→「**Developer settings**」を開く。
2. 「**Personal Access Tokens**」→「**Fine-grained tokens**」→「新しいトークンを作成」。
3. 次を設定します：

   * 対象の**ユーザー**と**リポジトリ**を選ぶ
   * **Issues に「Read & Write」権限**をつける
4. トークンをコピーして、スクリプトの `GITHUB_TOKEN` に貼り付けます。
5. `REPO_OWNER`（GitHubユーザー名）と `REPO_NAME`（リポジトリ名）も入力してください。

---

### 5. トリガーを追加する

1. Apps Script画面で左側の「時計マーク（⏰）」をクリック。
2. 「＋トリガーを追加」をクリック。
3. 以下の設定をします：

   * 関数：`onFormSubmit`
   * イベントのソース：「スプレッドシートから」
   * イベントの種類：「フォーム送信時」
4. 保存します。

---

### ✔️ テストしてみよう！

* Googleフォームからテスト送信をしてみましょう。
* GitHubのリポジトリを確認して、Issueが作成されたかチェックします。
* Apps Scriptの「ログ」でエラーがないか確認できます。


