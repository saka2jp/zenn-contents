---
title: "データモデル設計 — コアエンティティと関係"
free: true
---

# データモデル設計 — コアエンティティと関係

ATSのコアデータモデルは「組織構造」「候補者パイプライン」「評価・結果」「運用メタデータ」の4ドメインで構成される（出典: [Truto Blog - ATS Integrations Architecture](https://truto.one/blog/what-are-ats-integrations-2026-architecture-strategy-guide/)）。

## 主要エンティティ一覧

### Job（求人・ジョブレクイジション）

```
Job
├── id: UUID
├── title: String               # 職位名
├── requisition_id: String      # 要件管理番号
├── department_id: FK           # 部門
├── location: String
├── workplace_type: Enum        # ON_SITE / HYBRID / REMOTE
├── employment_type: Enum       # FULL_TIME / PART_TIME / CONTRACT
├── experience_level: Enum      # ENTRY / MID / SENIOR
├── description: Text
├── status: Enum                # OPEN / CLOSED / DRAFT / PAUSED
├── pipeline_id: FK             # 選考パイプライン
├── hiring_team: [User]         # 採用担当者・面接官
├── job_postings: [JobPosting]  # 各媒体への掲載情報
└── created_at, updated_at
```

### Candidate（候補者）

```
Candidate
├── id: UUID
├── first_name, last_name: String
├── email: String               # 主キーとして機能する場合も
├── phone: String
├── location: String
├── tags: [String]
├── source: String              # LinkedIn / Indeed / Referral
├── applications: [Application]
├── documents: [Document]       # 履歴書・職務経歴書
├── notes: [Note]
├── eeoc_data: Object           # 任意: 雇用機会均等データ
└── created_at, updated_at
```

### Application（応募）

```
Application
├── id: UUID
├── candidate_id: FK
├── job_id: FK
├── status: Enum                # ACTIVE / REJECTED / HIRED / WITHDRAWN
├── current_stage_id: FK
├── stage_history: [StageHistory]
├── scorecards: [Scorecard]
├── interviews: [Interview]
├── offer: Offer
├── rejection_reason: String
└── applied_at, updated_at
```

### Stage（選考ステージ）

```
Stage
├── id: UUID
├── pipeline_id: FK
├── name: String                # 書類選考 / 一次面接 / 最終面接 etc
├── order: Integer
├── stage_type: Enum            # SOURCED / APPLIED / SCREENING / INTERVIEW / OFFER / HIRED
├── requires_interview: Boolean
└── sla_days: Integer           # SLA（推奨処理日数）
```

### Interview（面接）

```
Interview
├── id: UUID
├── application_id: FK
├── interviewers: [User]
├── scheduled_at: DateTime
├── duration_minutes: Integer
├── interview_type: Enum        # PHONE / VIDEO / ON_SITE / TECHNICAL
├── scorecard: Scorecard
├── video_conference_url: String
└── status: Enum                # SCHEDULED / COMPLETED / CANCELLED
```

### Offer（内定・オファー）

```
Offer
├── id: UUID
├── application_id: FK
├── base_salary: Decimal
├── currency: String
├── start_date: Date
├── offer_status: Enum          # DRAFT / SENT / ACCEPTED / DECLINED / RESCINDED
├── offer_document_url: String
├── approved_by: [User]
└── sent_at, responded_at
```

### Scorecard（評価票）

```
Scorecard
├── id: UUID
├── interview_id: FK
├── reviewer_id: FK
├── overall_rating: Enum        # STRONG_YES / YES / NO / STRONG_NO
├── attributes: [AttributeScore]
│   └── AttributeScore
│       ├── attribute: String   # コミュニケーション / 技術力 etc
│       └── rating: Integer
├── private_notes: Text
└── submitted_at
```

## エンティティ関係図

```
Tenant
  └── Department ─── Job ──── JobPosting (LinkedIn / Indeed / Google Jobs)
                      │
                      ├── Pipeline ─── Stage (順序付き)
                      │
                      └── Application ──── Candidate
                             │
                             ├── Interview ──── Scorecard
                             │                    └── (面接官User)
                             │
                             ├── Offer
                             │
                             └── StageHistory
```

## ベンダーによるモデルの違い

ATSベンダーは同じ概念を異なる方法でモデル化している（出典: [Truto Blog](https://truto.one/blog/what-are-ats-integrations-2026-architecture-strategy-guide/)）。

| 概念 | Greenhouse | Lever |
|------|-----------|-------|
| 候補者の主要エンティティ | Candidate（人）+ Application（応募行為）を分離 | Opportunity（候補者×ポジション）に統合 |
| 設計思想 | 1人の候補者は複数応募を持てる | CRM志向で機会ベース管理 |
| 廃止 | — | Candidateエンドポイントは2020年廃止。Opportunityを使用 |

この違いがUnified API（統一API）の必要性を生む根本原因となっている（後述の章7参照）。
