# Odoo Enquiry Form to GitHub Issues

This project connects a **Google Form** to a **GitHub repository** to automatically create GitHub Issues based on form responses. It's perfect for collecting structured inquiries (e.g., Odoo support) and managing them in GitHub with automation.

## Features

- Collect form submissions from Google Forms
- Automatically create GitHub Issues using Google Apps Script
- Attach timestamp, title, and description
- Support for Japanese and English fields
- Logs status and API responses

---

## 📋 Setup Guide

### 1. Create a Google Form

Create a Google Form with the following fields:

1. **Title** (`お問い合わせタイトル`)
2. **Description** (`説明`)
3. **Image Upload**(`Optional`)
4. (Optional) **Timestamp** is automatically collected by Google Forms

### 2. Open Apps Script

1. In your Form, click the three-dot menu (︙) → **Script Editor**
2. Replace the default script with the following code:

```javascript
var GITHUB_TOKEN = ;
var REPO_OWNER = ;
var REPO_NAME = "Odoo-Enquiry-Form";

function onFormSubmit(e) {
  const responses = e.values;
  Logger.log("Responses: " + JSON.stringify(responses));

  const timestamp = responses[0];
  const title = responses[1];       // お問い合わせタイトル
  const description = responses[2]; // 説明

  if (!title || !description) {
    Logger.log("Missing required fields: title or description");
    return;
  }

  const issueTitle = `[Odoo Inquiry] ${title}`;
  const issueBody = `
**Timestamp**: ${timestamp}

**Description**:
${description}
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
