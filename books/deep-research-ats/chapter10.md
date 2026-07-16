---
title: "類似技術・ベンダー比較"
free: true
---

# 類似技術・ベンダー比較

## ATSカテゴリの全体マップ

```
採用管理ソリューションの分類

Pure ATS（採用特化）
├── エンタープライズ向け: iCIMS, Oracle Taleo, SAP SuccessFactors RCM
├── 成長企業向け:        Greenhouse, Lever, Jobvite
└── SMB向け:             Workable, BambooHR Recruiting, Zoho Recruit

HCM統合型（人事全般の中の採用モジュール）
├── エンタープライズ: Workday Recruiting, SAP SuccessFactors, Oracle HCM
└── 中堅企業:         BambooHR, Rippling, ADP Workforce Now

専門特化型
├── 技術系採用:    Lever（CRM機能重視）, Ashby
├── 派遣・人材紹介: Bullhorn
└── ボリューム採用: Paradox（会話型AI）

Unified API（ATS統合レイヤー）
├── Merge.dev
├── Unified.to
├── Kombo
├── Truto
└── Knit（旧GetKnit）
```

---

## 主要ベンダー詳細比較

| 評価軸 | Greenhouse | Lever | Workday Recruiting | iCIMS | SAP SuccessFactors |
|--------|-----------|-------|--------------------|-------|-------------------|
| **対象規模** | 中堅〜大企業 | 中堅〜大企業 | 大企業 | 大企業 | 大企業 |
| **APIの充実度** | 高（Harvest v3） | 高（Data API + Postings） | 中（Workday Extend） | 中 | 中〜高 |
| **カスタマイズ性** | 高 | 中 | 中（Workday Extend使用） | 高 | 中 |
| **HRIS統合** | API連携が必要 | API連携が必要 | ネイティブ統合 | API連携 | ネイティブ統合 |
| **採用CRM機能** | 別途 | 組み込み（強み） | 別途 | 別途 | 別途 |
| **データモデルの独自性** | Candidate/Application分離 | Opportunity統合 | 独自HCMモデル | 標準的 | SAP固有 |
| **OAuth対応** | ○ | ○（1時間期限） | ○ | ○ | ○ |
| **Webhook** | ○ | ○（HMAC署名） | ○ | ○ | ○ |
| **市場シェア** | 上位 | 上位 | 大企業市場上位 | **10.7%（業界1位）** | 大企業市場上位 |
| **2025年の動き** | Harvest API v3標準化 | EU向けOAuth対応 | — | — | SmartRecruiters買収 |

出典: [Apps Run The World - ATS Market](https://www.appsruntheworld.com/top-10-hcm-software-vendors-in-applicant-tracking-market-segment/), [Unified.to - 15 ATS APIs](https://unified.to/blog/15_ats_apis_to_integrate_with_in_2026_greenhouse_lever_workable)

---

## Unified API（ATS統合レイヤー）の比較

複数のATSに接続するサービス（RPAアグリゲーター層）を提供するベンダーの比較。

| ベンダー | 対応ATS数 | データモデル | 特徴 |
|---------|---------|------------|------|
| **Merge.dev** | 70+ | 統一スキーマ | HRIS・会計・CRMも対応する総合Unified API |
| **Unified.to** | 多数 | 9オブジェクト標準化 | リアルタイム直接アクセス、キャッシュなし |
| **Kombo** | 多数 | 統一スキーマ | 欧州市場に強い |
| **Truto** | 多数 | 設定駆動マッピング | カスタムAPI quirks対応 |
| **Knit** | 多数 | 統一スキーマ | ノーコード統合ビルダー |

出典: [Unified.to ATS Overview](https://docs.unified.to/ats/overview), [Kombo - ATS API](https://www.kombo.dev/use-cases/ats-api.html)

---

## 選択の判断基準

```
どのATSを選ぶべきか？

採用規模・組織タイプ別:
├── エンタープライズ（1000+名）
│   ├── 既存がWorkday/SAP → そのまま利用
│   └── 採用特化が必要 → iCIMS, Greenhouse, Jobvite
│
├── 成長企業（100〜1000名）
│   ├── 採用CRM機能重視 → Lever
│   ├── 面接プロセス厳格化 → Greenhouse
│   └── コスト重視 → Workable
│
└── スタートアップ（〜100名）
    ├── HR全体をまとめたい → BambooHR, Rippling
    └── 採用特化でAPI重視 → Lever（Postings APIが無料公開）

Unified API採用タイミング:
  - 複数のATSに対応するB2B SaaSを開発する場合
  - ATS毎の独自対応コストが開発リソースを圧迫する場合
  → Merge.dev / Unified.to を検討
```

---

## ATSのロックインリスクと対策

ATSのデータは候補者の個人情報・採用判断履歴を含む重要資産。ベンダー切り替えリスクを最小化するには：

1. **データエクスポートAPI確認**: Harvest APIでの全データGET可能かを事前確認
2. **標準フォーマット対応**: HRXML / JDXへのエクスポート対応
3. **Webhook/イベント駆動**: リアルタイムミラーリングで内部データレイク構築
4. **候補者同意管理**: GDPR/CCPA対応で移行時の同意再取得コストを把握
