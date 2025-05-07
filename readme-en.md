# Odoo Enquiry Form to GitHub Issues

This project connects a **Google Form** to a **GitHub repository**, automatically creating GitHub Issues from form submissions. It's ideal for collecting structured inquiries (e.g., for Odoo support) and managing them with GitHub automation.

---

## ✨ Features

* Collect submissions using **Google Forms**
* Automatically create **GitHub Issues** via **Google Apps Script**
* Includes **timestamp**, **title**, and **description**
* Supports both **Japanese** and **English** fields
* Logs API responses and submission status for troubleshooting

---

## 📋 Setup Guide

### 1. Create a Google Form

Set up a Google Form with the following fields:

1. **Title** (`お問い合わせタイトル`)
2. **Description** (`説明`)
3. **Image Upload** (Optional — currently not handled by the script)

---

### 2. Link Form to Google Sheet

1. In the **Responses** tab of your Google Form, click on the green Sheets icon to create a linked Google Spreadsheet.
2. Open the linked **Google Sheet**.

---

### 3. Add Google Apps Script

1. In the Google Sheet, go to **Extensions → Apps Script**.
2. Delete the default code and **paste the following script**:

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

### 4. Generate a GitHub Token

1. Go to your [GitHub Developer Settings](https://github.com/settings/tokens).
2. Select **"Fine-grained tokens"** and click **"Generate new token"**.
3. Fill in the required information:

   * **Repository access**: Select the owner and repository where you want to create issues.
   * **Permissions**: Grant **Read and Write** access to Issues.
4. Generate the token and **copy it**.
5. Paste the token into `GITHUB_TOKEN` in your script.
6. Also update `REPO_OWNER` and `REPO_NAME` as needed.

---

### 5. Add a Trigger

1. In Apps Script, click the clock icon ⏰ (Triggers).
2. Click **"+ Add Trigger"**.
3. Choose:

   * **Function**: `onFormSubmit`
   * **Event source**: "From spreadsheet"
   * **Event type**: "On form submit"
4. Save the trigger.

---

### ✅ Test It

* Submit a test response through your Google Form.
* Check your GitHub repository to confirm the issue was created.
* Review the **Apps Script Logs** for API response status if needed.
