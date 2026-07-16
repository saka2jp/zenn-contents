---
title: "API設計 — Greenhouse・Lever実装詳解"
free: true
---

# API設計 — Greenhouse・Lever実装詳解

## Greenhouse API全体構成

Greenhouseは5種類のAPIを提供する（出典: [Greenhouse API Overview](https://support.greenhouse.io/hc/en-us/articles/10568627186203-Greenhouse-API-overview)）。

| API名 | 用途 | 認証方式 |
|-------|------|---------|
| **Harvest API v3** | 内部データの読み書き（メイン統合API） | APIキー（Basic認証） |
| **Job Board API** | 求人公開・キャリアページ埋め込み | 公開（一部APIキー） |
| **Application API** | 外部からの応募受付 | APIキー |
| **Partner API** | 承認済みパートナー向け | OAuth 2.0 |
| **Reports API** | 採用分析データ取得 | APIキー |

:::message
**Harvest API v1/v2は2026年8月31日廃止予定**。カスタム統合は v3 へ移行が必要（出典: [Greenhouse Support](https://support.greenhouse.io/hc/en-us/articles/10568627186203-Greenhouse-API-overview)）。
:::

### Harvest API v3 主要エンドポイント

```
# 求人（Jobs）
GET    /v1/jobs                    # 求人一覧
GET    /v1/jobs/{id}               # 求人詳細
POST   /v1/jobs                    # 求人作成
PATCH  /v1/jobs/{id}               # 求人更新

# 候補者（Candidates）
GET    /v1/candidates              # 候補者一覧
GET    /v1/candidates/{id}         # 候補者詳細
POST   /v1/candidates              # 候補者作成
PATCH  /v1/candidates/{id}         # 候補者更新
DELETE /v1/candidates/{id}         # 候補者削除

# 応募（Applications）
GET    /v1/applications            # 応募一覧
GET    /v1/applications/{id}       # 応募詳細
DELETE /v1/applications/{id}       # 応募削除

# 面接（Interviews）
GET    /v1/scheduled_interviews    # 面接一覧
POST   /v1/applications/{id}/move  # ステージ変更

# オファー（Offers）
GET    /v1/offers                  # オファー一覧
GET    /v1/applications/{id}/offers # 特定応募のオファー

# 評価（Scorecards）
GET    /v1/scorecards              # スコアカード一覧
POST   /v1/applications/{id}/scorecards # 評価提出
```

### Greenhouse レート制限

**10秒あたり50リクエスト**（出典: [Truto Blog](https://truto.one/blog/what-are-ats-integrations-2026-architecture-strategy-guide/)）。超過時は `429 Too Many Requests` を返す。

### Greenhouse Webhook

Webhookはアプリケーション・候補者・面接のイベントをHTTPS POSTで送信する。

```json
// candidateHired イベントのペイロード例
{
  "action": "candidate_hired",
  "payload": {
    "application": {
      "id": 12345,
      "candidate": {
        "id": 67890,
        "first_name": "Taro",
        "last_name": "Yamada",
        "email": "taro@example.com"
      },
      "job": {
        "id": 11111,
        "name": "Software Engineer"
      },
      "offer": {
        "id": 99999,
        "status": "accepted",
        "start_date": "2026-08-01"
      }
    }
  }
}
```

## Lever API全体構成

Leverは2種類のAPIを公開する（出典: [Lever Developer Documentation](https://hire.lever.co/developer/documentation)）。

| API名 | 用途 |
|-------|------|
| **Postings API** | 公開求人情報の取得（キャリアページ向け） |
| **Data API** | Opportunity・Stage・Offer等の全データ操作 |

### Lever Data API 主要エンドポイント

```
# Opportunities（Leverの主要エンティティ）
GET    /v1/opportunities           # 機会一覧
GET    /v1/opportunities/{id}      # 機会詳細
POST   /v1/opportunities           # 機会作成
PUT    /v1/opportunities/{id}/stage # ステージ更新

# Postings（職種）
GET    /v1/postings                # 職種一覧
POST   /v1/postings                # 職種作成

# Offers（オファー）
GET    /v1/opportunities/{id}/offers # オファー一覧

# Feedback（フィードバック）
GET    /v1/opportunities/{id}/feedback # フィードバック一覧
POST   /v1/opportunities/{id}/feedback # フィードバック提出

# Tags
GET    /v1/tags                    # タグ一覧
POST   /v1/opportunities/{id}/addTags # タグ付与
```

### Lever 認証方式

```
# APIキー認証（内部利用向け）
Authorization: Basic {base64(apiKey:)}

# OAuth 2.0（パートナー統合向け）
1. ユーザーを認可URLへリダイレクト
2. 認可コード取得
3. アクセストークンに交換（有効期限: 1時間）
4. リフレッシュトークンで更新（有効期限: 1年または90日不活動）
```

:::message alert
**LeverのOAuthトークンは1時間で期限切れ**。リフレッシュトークンの自動更新ロジックが必須（出典: [Truto Blog](https://truto.one/blog/what-are-ats-integrations-2026-architecture-strategy-guide/)）。
:::

### Lever レート制限

- デフォルト: **10リクエスト/秒**
- バースト: 最大 **20リクエスト/秒**（トークンバケットアルゴリズム）（出典: [Lever Developer Documentation](https://hire.lever.co/developer/documentation)）

### Lever Webhook イベント種別

```
applicationCreated       # 応募作成
candidateHired           # 採用決定
candidateStageChange     # ステージ変更
candidateArchiveChange   # アーカイブ状態変更
interviewCreated         # 面接作成
interviewUpdated         # 面接更新
interviewDeleted         # 面接削除
contactCreated           # コンタクト作成
contactUpdated           # コンタクト更新
```

セキュリティ: HMAC-SHA256署名（出典: [Lever Developer Documentation](https://hire.lever.co/developer/documentation)）。再試行は5回（指数バックオフ）、2xx応答で受信確認。

### Lever Webhook ペイロード構造

```json
{
  "triggeredAt": 1718000000000,
  "event": "candidateStageChange",
  "data": {
    "opportunityId": "abc-123",
    "candidateId": "xyz-456",
    "toStageId": "stage-789",
    "toStageName": "Final Interview"
  },
  "signature": {
    "token": "xxx",
    "timestamp": "1718000000",
    "signature": "hmac-sha256-signature"
  }
}
```

## Lever Postings API（公開求人）

Postings APIはGitHubで公開されているオープンAPI仕様である（出典: [lever/postings-api on GitHub](https://github.com/lever/postings-api)）。

```
GET https://api.lever.co/v0/postings/{site}
# パラメータ例
?team=Engineering&location=Remote&commitment=Full-time
```

レスポンス例:
```json
[
  {
    "id": "posting-uuid",
    "text": "Software Engineer",
    "categories": {
      "team": "Engineering",
      "location": "Remote",
      "commitment": "Full-time"
    },
    "applyUrl": "https://jobs.lever.co/company/posting-uuid/apply",
    "hostedUrl": "https://jobs.lever.co/company/posting-uuid",
    "description": "<p>Job description HTML...</p>",
    "createdAt": 1718000000000
  }
]
```

## ページネーション設計

両ATSとも `cursor` ベースまたは `offset/limit` ベースのページネーションを採用。

| ベンダー | 方式 | デフォルト件数 | 最大件数 |
|---------|------|------------|---------|
| Greenhouse | `page` + `per_page` | 100 | 500 |
| Lever | `offset` + `limit` + `hasNext` | 100 | 100 |
