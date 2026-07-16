---
title: "実装クイックスタート"
free: true
---

# 実装クイックスタート

## ケース1: Greenhouse Harvest API に接続する

### 前提環境

- Python 3.10+
- Greenhouse アカウント（API Key が必要）
- APIキーは「Greenhouse > Settings > Dev Center > API Credential Management」で取得

### 基本接続

```python
import requests
import base64

# 認証設定（APIキーをBase64エンコード: "apikey:"）
API_KEY = "your_greenhouse_api_key"
credentials = base64.b64encode(f"{API_KEY}:".encode()).decode()
headers = {
    "Authorization": f"Basic {credentials}",
    "Content-Type": "application/json"
}

BASE_URL = "https://harvest.greenhouse.io/v1"

# 求人一覧取得
def get_jobs(status="open"):
    resp = requests.get(
        f"{BASE_URL}/jobs",
        headers=headers,
        params={"status": status, "per_page": 100}
    )
    resp.raise_for_status()
    return resp.json()

# 候補者一覧取得
def get_candidates(job_id=None):
    params = {"per_page": 100}
    if job_id:
        params["job_id"] = job_id
    resp = requests.get(
        f"{BASE_URL}/candidates",
        headers=headers,
        params=params
    )
    resp.raise_for_status()
    return resp.json()

# 実行
jobs = get_jobs()
print(f"求人数: {len(jobs)}")
```

### Webhook受信エンドポイント（FastAPI）

```python
from fastapi import FastAPI, Request
import hmac, hashlib

app = FastAPI()

WEBHOOK_SECRET = "your_webhook_secret"

@app.post("/webhooks/greenhouse")
async def greenhouse_webhook(request: Request):
    body = await request.body()

    # 署名検証（Greenhouseは独自署名方式）
    # 詳細はGreenhouseドキュメント参照

    payload = await request.json()
    action = payload.get("action")

    if action == "candidate_hired":
        candidate = payload["payload"]["application"]["candidate"]
        print(f"採用確定: {candidate['first_name']} {candidate['last_name']}")
        # → HRISへの自動連携処理をここで呼び出す

    return {"status": "ok"}
```

---

## ケース2: LinkedIn Job Postings API で求人を配信する

### 基本的なCREATEリクエスト

```bash
curl -X POST "https://api.linkedin.com/rest/simpleJobPostings" \
  -H "Authorization: Bearer {access_token}" \
  -H "Content-Type: application/json" \
  -H "LinkedIn-Version: 202503" \
  -d '{
    "jobPostingOperationType": "CREATE",
    "externalJobPostingId": "job-001",
    "company": "urn:li:organization:12345",
    "companyApplyUrl": "https://careers.example.com/apply/001",
    "title": "Software Engineer",
    "description": "<p>We are looking for...</p>",
    "listedAt": 1718000000000,
    "location": "Tokyo, Japan",
    "workplaceTypes": ["Remote"],
    "employmentStatus": "FULL_TIME",
    "experienceLevel": "MID_SENIOR_LEVEL",
    "availability": "PUBLIC",
    "posterEmail": "recruiter@example.com"
  }'
```

### 求人のCLOSE

```bash
curl -X POST "https://api.linkedin.com/rest/simpleJobPostings" \
  -H "Authorization: Bearer {access_token}" \
  -H "Content-Type: application/json" \
  -d '{
    "jobPostingOperationType": "CLOSE",
    "externalJobPostingId": "job-001"
  }'
```

---

## ケース3: Google for Jobs 対応のJobPosting構造化データ実装

```html
<!-- キャリアページ（求人詳細）に埋め込む -->
<script type="application/ld+json">
{
  "@context": "https://schema.org/",
  "@type": "JobPosting",
  "title": "Software Engineer",
  "description": "<p>担当業務: バックエンドAPI開発...</p>",
  "hiringOrganization": {
    "@type": "Organization",
    "name": "株式会社サンプル",
    "sameAs": "https://example.co.jp"
  },
  "jobLocation": {
    "@type": "Place",
    "address": {
      "@type": "PostalAddress",
      "addressLocality": "渋谷区",
      "addressRegion": "東京都",
      "addressCountry": "JP"
    }
  },
  "datePosted": "2026-06-16",
  "validThrough": "2026-08-31T00:00",
  "employmentType": "FULL_TIME",
  "baseSalary": {
    "@type": "MonetaryAmount",
    "currency": "JPY",
    "value": {
      "@type": "QuantitativeValue",
      "minValue": 5000000,
      "maxValue": 10000000,
      "unitText": "YEAR"
    }
  }
}
</script>
```

検証: [Google Rich Results Test](https://search.google.com/test/rich-results) でエラー確認。

---

## よくある失敗と対処

| 失敗 | 原因 | 対処 |
|------|------|------|
| `401 Unauthorized` | APIキーの形式誤り | `Basic base64("apikey:")` の形式を確認（末尾のコロンを忘れない） |
| `429 Too Many Requests` | レート制限超過 | 指数バックオフ + キューで再試行。Greenhouse: 50req/10sec |
| Lever OAuthトークン失効 | 1時間で期限切れ | リフレッシュトークンによる自動更新ロジックを実装 |
| LinkedInの求人が表示されない | `availability: PUBLIC` の未設定 | 必須フィールドを確認 |
| Google for Jobs に掲載されない | `validThrough` が過去日 or 構造化データ構文エラー | Rich Results Testで検証 |
