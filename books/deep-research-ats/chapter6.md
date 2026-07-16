---
title: "標準仕様・連携規格 — HR Open Standards / Google for Jobs / LinkedIn"
free: true
---

# 標準仕様・連携規格

## 概要マップ

ATSが関与する標準仕様は複数の層に分かれる。

```
採用ツール間データ交換層
├── HR Open Standards（HRXML）    → ATS間・HRIS間のデータ交換標準
└── JDX（Job Data Exchange）      → 求人データの意味論的標準化

求人掲載・SEO層
├── schema.org JobPosting        → 検索エンジン向け構造化データ
├── Google for Jobs              → Googleの求人検索への掲載
└── LinkedIn Job Postings API    → LinkedIn への求人連携

候補者ソーシング層
├── LinkedIn RSC                 → Recruiter System Connect
└── Indeed Employer API          → Indeed連携
```

---

## 1. schema.org JobPosting（Google for Jobs）

Google for Jobsへ掲載するためにはJSONLDまたはMicrodata形式でJobPosting構造化データをページに埋め込む（出典: [Google Search Central - Job Posting](https://developers.google.com/search/docs/appearance/structured-data/job-posting)）。

### 必須フィールド

| フィールド | 説明 | 例 |
|-----------|------|-----|
| `title` | 職位名 | `"ソフトウェアエンジニア"` |
| `description` | 職務内容（HTML可） | `"<p>担当業務...</p>"` |
| `hiringOrganization` | 採用企業情報 | `{"@type": "Organization", "name": "株式会社XX"}` |
| `jobLocation` | 勤務地 | `{"@type": "Place", "address": {...}}` |
| `datePosted` | 掲載日 | `"2026-06-16"` |

### 推奨フィールド

```json
{
  "@context": "https://schema.org/",
  "@type": "JobPosting",
  "title": "Software Engineer",
  "description": "<p>We are looking for...</p>",
  "hiringOrganization": {
    "@type": "Organization",
    "name": "Example Corp",
    "sameAs": "https://example.com"
  },
  "jobLocation": {
    "@type": "Place",
    "address": {
      "@type": "PostalAddress",
      "streetAddress": "東京都渋谷区",
      "addressLocality": "渋谷",
      "addressRegion": "東京都",
      "postalCode": "150-0002",
      "addressCountry": "JP"
    }
  },
  "datePosted": "2026-06-16",
  "validThrough": "2026-08-31",
  "employmentType": "FULL_TIME",
  "jobLocationType": "TELECOMMUTE",
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
```

Google for Jobsは「採用企業のキャリアページ、求人ボード、スタッフィング会社、有効なJobPosting構造化データを持つアグリゲーター」からジョブカードを収集する（出典: Google Search Central）。

:::message
`validThrough` を過去日に設定するか、ページを削除することで求人を終了とマークできる。期限切れ求人の放置はペナルティ対象になる可能性がある。
:::

---

## 2. LinkedIn Job Postings API

LinkedIn Job Postings APIは、ATSやジョブディストリビューターがLinkedIn上に求人を自動配信するためのAPIである（出典: [LinkedIn Job Posting Schema - Microsoft Learn](https://learn.microsoft.com/en-us/linkedin/talent/job-postings/api/job-posting-api-schema)）。

### Foundation Schema（必須フィールド）

| フィールド | 型 | 必須 | 説明 |
|-----------|-----|------|------|
| `jobPostingOperationType` | String | Yes | `CREATE`, `UPDATE`, `RENEW`, `CLOSE`, `UPGRADE`, `DOWNGRADE` |
| `externalJobPostingId` | String | Yes | ATS側の一意ID（最大75文字） |
| `company` | URN | Yes* | `urn:li:organization:{company_id}` 形式 |
| `companyApplyUrl` | String | Yes | 応募ページURL |
| `title` | String | Yes | 職位名（最大200文字） |
| `description` | String | Yes | 職務内容（100〜25,000文字、一部HTMLタグ可） |
| `listedAt` | Integer | Yes | ATS登録日時（epoch ms、UTC） |
| `location` | String | Yes | 勤務地（"CITY, STATE, COUNTRY"等の形式） |
| `availability` | String | Yes | `PUBLIC` 固定 |
| `posterEmail` | String | Yes | 求人掲載者メールアドレス |

### 推奨フィールド

| フィールド | 説明 |
|-----------|------|
| `workplaceTypes` | `On-site` / `Hybrid` / `Remote` |
| `employmentStatus` | `FULL_TIME` / `PART_TIME` / `CONTRACT` 等 |
| `experienceLevel` | `ENTRY_LEVEL` / `MID_SENIOR_LEVEL` / `DIRECTOR` 等 |
| `categories` | LinkedInの定義する職種カテゴリ（最大3つ） |
| `compensation` | 給与情報（期間・タイプ・金額レンジ） |
| `expireAt` | 有効期限（デフォルト: 180日） |

### 給与情報のスキーマ

```json
"compensation": {
  "compensations": [
    {
      "period": "YEARLY",
      "type": "BASE_SALARY",
      "value": {
        "range": {
          "start": {"amount": "5000000", "currencyCode": "JPY"},
          "end": {"amount": "10000000", "currencyCode": "JPY"}
        }
      }
    }
  ]
}
```

### LinkedIn RSC（Recruiter System Connect）

RSCは採用担当者がLinkedIn Recruiter画面からATS内の応募状況をリアルタイムに確認できる深い統合機能。`partnerRequisitionId` でATSのJob RequisitionとLinkedInを紐づける（出典: [LinkedIn Job Posting Schema](https://learn.microsoft.com/en-us/linkedin/talent/job-postings/api/job-posting-api-schema)）。

---

## 3. HR Open Standards（旧HRXML）

HR Open Standardsコンソーシアムは、HRシステム間のデータ交換標準を策定している業界団体。主要な標準として以下が存在する。

| 標準 | 内容 |
|------|------|
| **Recruiting標準** | 求人・応募者データの交換フォーマット |
| **Onboarding標準** | 内定者のオンボーディングデータ |
| **Payroll標準** | 給与関連データ |

### JDX（Job Data Exchange）

JDXはHR Open StandardsとSchema.org JobPostingをベースに、米国商工会議所財団が提案した求人データ交換のための新しいスキーマ（出典: [HR Open Standards - JDX紹介](https://www.hropenstandards.org/news/guest-post-jdx-a-schema-for-job-data-exchange)）。

- RDF形式でクラス・プロパティ・概念スキームのボキャブラリを構築
- GitHubでデータモデルを公開
- JSON-Schema形式で部分的にHR Open Standardsに統合済み
- 地理的エリア別の求人情報・職務要件・報酬・キャリアパスの標準化を目指す

:::message
JDXはまだ普及段階にある。現時点での一次情報として扱うことを推奨（出典: [HR Open Standards](https://www.hropenstandards.org/)）。
:::

---

## 4. 各標準の関係整理

```
求人情報の流れ:

ATS内の Job エンティティ
         │
         ├──[LinkedIn Job Postings API]──→ LinkedIn掲載
         │
         ├──[schema.org JobPosting]──→ 自社キャリアページ → Google for Jobs
         │
         ├──[Indeed Employer API]──→ Indeed掲載
         │
         └──[HR Open Standards]──→ HRIS・バックグラウンドチェックツール等との連携
```
