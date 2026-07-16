---
title: "システム連携パターン"
free: true
---

# システム連携パターン

ATSは単体では完結せず、多様な外部システムと連携して機能する（出典: [Knit - ATS Integration Guide](https://www.getknit.dev/blog/ats-integration-guide)）。

## 連携マップ全体図

```
                     ┌──────────────────────┐
                     │         ATS          │
                     │  (Greenhouse / Lever) │
                     └──────────┬───────────┘
                                │
       ┌───────────┬────────────┼────────────┬───────────────┐
       ↓           ↓            ↓            ↓               ↓
  ジョブボード   HRIS/HCM    評価ツール  バックグラウンド  コミュニケーション
  連携         統合         連携        チェック連携      ツール連携
  LinkedIn      Workday      HackerRank  Sterling          Slack
  Indeed        BambooHR     Codility    Checkr            Teams
  Glassdoor     Rippling      Pymetrics   GoodHire          Gmail
  Google Jobs   SAP SF        Mercer      Certn             Zoom
```

---

## 1. ジョブボード連携

### 求人配信（Job Distribution）

ATSからLinkedIn・Indeed等への求人配信は、各社の求人掲載APIを通じて行われる。

**フロー:**
```
ATS (Job作成)
  → Job Board API呼び出し（CREATE操作）
  → ジョブボード側で求人を公開
  → 応募者がジョブボード経由で応募
  → ATS側のApplication APIへPOST
  → Candidate + Applicationが自動生成
```

### Apply Connect（LinkedIn）

LinkedIn上で完結する応募体験を提供する機能。ATSの `onsiteApplyConfiguration` を設定することで、候補者がLinkedIn上で直接応募できる（出典: [LinkedIn Job Posting Schema](https://learn.microsoft.com/en-us/linkedin/talent/job-postings/api/job-posting-api-schema)）。

### Track Pixel

LinkedIn求人ページに埋め込むトラッキングピクセルURLを指定することで、閲覧・クリック等の行動データをATSに送信できる（`trackingPixelUrl` フィールド）。

---

## 2. HRIS / HCM統合

ATSとHRISの統合は「採用→入社」のデータ引き継ぎが主目的。内定承諾後に候補者データをHRISへ転送し、従業員レコードを自動生成する（出典: [Knit - ATS Integration Guide](https://www.getknit.dev/blog/ats-integration-guide)）。

### 主要連携パターン

| パターン | 説明 | 代表例 |
|---------|------|--------|
| **オファー承諾トリガー** | 候補者がオファーを承認したタイミングでHRISへPOST | Workday + Greenhouse |
| **新入社員レコード自動生成** | ATSの候補者データを変換してHRISの従業員エンティティを作成 | BambooHR + Lever |
| **オンボーディング連携** | ATSからオンボーディングSaaSへ内定者データを転送 | SAP SF Onboarding 2.0 |
| **給与データ連携** | 給与情報をATSオファーデータからPayrollシステムへ | ADP + iCIMS |

### SAP SuccessFactors統合事例

SAP SuccessFactors HCMを使用する企業が外部ATSを利用する場合、SuccessFactors Onboardingとの統合が必要になる。候補者データをATS→SuccessFactors Onboarding 2.0へ渡す設定が公式に文書化されている（出典: [SAP Learning - Integrating SF Onboarding with External ATS](https://learning.sap.com/courses/sap-successfactors-onboarding-academy/integrating-sap-successfactors-onboarding-with-an-external-ats_cfb7138b-a5d9-49f1-9656-c7e259960329)）。

### Workday統合の特徴

Workday HCMはネイティブ統合クラウドと「Workday Extend」フレームワークを持ち、コアデータへのカスタム統合を構築できる（出典: [Althire.ai - HRIS Integration Platforms](https://althire.ai/feeds/blog/platforms-in-product-integrations-hris)）。

---

## 3. 評価ツール連携

コーディングテスト・アセスメントツールとATSの連携パターン。

```
フロー:
ATS（特定Stageに到達）
  → 評価ツールへ招待リクエスト送信（候補者情報 + Job情報）
  → 評価ツールが候補者へ招待メール送信
  → 候補者がテスト完了
  → 評価ツールが結果をATSのWebhook/APIへPOST
  → ATSのScorecardまたはカスタムフィールドに結果が反映
```

| ツール | 連携方式 | 対応ATS |
|--------|---------|--------|
| HackerRank | REST API + Webhook | Greenhouse, Lever, Workday等 |
| Codility | API連携 | Greenhouse, Lever |
| Pymetrics | API連携 | 主要ATS各種 |

---

## 4. バックグラウンドチェック連携

オファー承諾後または内定前に実施するバックグラウンドチェック（身元確認・犯罪歴調査等）との連携。

**フロー:**
```
ATS（Offer承諾またはBGC Stageに到達）
  → BGCベンダーAPIへ候補者情報送信
  → BGCベンダーが調査実施
  → 結果をATSへWebhookで返送
  → ATSの候補者プロファイルにBGCステータスを更新
```

| ベンダー | 対応ATS |
|---------|--------|
| Sterling | Greenhouse, Workday |
| Checkr | Greenhouse, Lever, Workable |
| GoodHire | Greenhouse, BambooHR |
| Certn | Greenhouse, Lever |

---

## 5. コミュニケーションツール連携

面接スケジューリング・チーム通知のための連携。

| ツール | 連携内容 |
|--------|---------|
| Google Calendar | 面接スケジュール自動カレンダー登録 |
| Zoom / Teams | ビデオ面接URLの自動生成・挿入 |
| Slack | 候補者ステージ変更通知・内定通知 |
| Docusign / HelloSign | オファーレターの電子署名 |

---

## 連携実装の技術的課題

1. **OAuth管理**: Leverトークンの1時間期限切れ対策（リフレッシュトークン自動更新ロジック必須）
2. **レート制限**: Greenhouse 50req/10sec、Lever 10req/sec の超過対策（バックオフ+キュー）
3. **ページネーション**: ベンダーごとに異なる実装（cursor/offset/page方式）
4. **データモデル不一致**: Greenhouse Candidate ≠ Lever Opportunity の正規化（出典: [Truto Blog](https://truto.one/blog/what-are-ats-integrations-2026-architecture-strategy-guide/)）
5. **Webhookの冪等性**: 重複イベント配信への対応（Idempotency Key管理）
