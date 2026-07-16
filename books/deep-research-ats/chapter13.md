---
title: "学習ロードマップ"
free: true
---

# 学習ロードマップ

## 3段階ロードマップ

### Stage 1: 基礎理解（1〜2週間）

**目標**: ATSの全体構造とデータモデルを理解する

学習項目:
- [ ] ATSの5層アーキテクチャを説明できる
- [ ] Job / Candidate / Application / Stage / Interview / Offer の関係を図示できる
- [ ] Greenhouse vs Lever のデータモデルの違い（Candidate vs Opportunity）を説明できる
- [ ] Webhook vs ポーリングの使い分けを説明できる

推奨リソース:
1. [Greenhouse API Overview](https://support.greenhouse.io/hc/en-us/articles/10568627186203-Greenhouse-API-overview)
2. [Lever Developer Documentation](https://hire.lever.co/developer/documentation)
3. [How ATS Work - CrawlJobs](https://www.crawljobs.com/blog/how-applicant-tracking-systems-work)

---

### Stage 2: API実装（2〜4週間）

**目標**: 実際にATSのAPIと接続し、データを読み書きできる

学習項目:
- [ ] Greenhouse Harvest API v3 で求人・候補者データを取得できる
- [ ] Lever Data API で Opportunity を操作できる
- [ ] Webhookを受信・検証するエンドポイントを実装できる
- [ ] レート制限に対応した再試行ロジック（指数バックオフ）を実装できる
- [ ] schema.org JobPostingをキャリアページに実装し、Google Rich Results Testで検証できる
- [ ] LinkedIn Job Postings APIで求人のCREATE/CLOSE操作ができる

推奨リソース:
1. [GitHub - lever/postings-api](https://github.com/lever/postings-api)（Lever Postings APIの公式仕様）
2. [Google Search Central - Job Posting](https://developers.google.com/search/docs/appearance/structured-data/job-posting)
3. [LinkedIn Job Posting Schema](https://learn.microsoft.com/en-us/linkedin/talent/job-postings/api/job-posting-api-schema)
4. [Unified.to ATS API Overview](https://docs.unified.to/ats/overview)（複数ATS統合の参考）

---

### Stage 3: アーキテクチャ設計（実務投入フェーズ）

**目標**: ATSを中心としたHR技術基盤の設計判断ができる

学習項目:
- [ ] マルチテナント設計（Silo/Pool/Bridge）の使い分けを判断できる
- [ ] PostgreSQL Row Level SecurityによるテナントデータIsolationを実装できる
- [ ] ATS → HRIS の自動データ連携フロー（オファー承諾トリガー）を設計できる
- [ ] レジュメパーシングのNLP/AIパイプラインを設計できる
- [ ] GDPR/CCPA/EEOC対応のデータモデルを設計できる
- [ ] Unified API（Merge.dev等）の採用判断基準を説明できる

推奨リソース:
1. [Multi-Tenant Database Design Patterns - Daily.dev](https://daily.dev/blog/multi-tenant-database-design-patterns-2024/)
2. [ATS Integrations Architecture Guide - Truto](https://truto.one/blog/what-are-ats-integrations-2026-architecture-strategy-guide/)
3. [Merge.dev HRIS Unified API Documentation](https://docs.merge.dev/)
4. [Resume2Vec: Transforming ATS with Intelligent Resume Embeddings - MDPI](https://www.mdpi.com/2079-9292/14/4/794)
5. [HR Open Standards - JDX](https://www.hropenstandards.org/news/guest-post-jdx-a-schema-for-job-data-exchange)

---

## 重要な設計課題と推奨プラクティス

### 課題1: 複数ATSへの同時対応

**推奨**: Unified API層（Merge.dev / Unified.to / Kombo）の採用。自前で10種類のATS対応を実装するコストと比較して判断。

### 課題2: OAuthトークンライフサイクル管理

**推奨**: トークン有効期限の1分前に自動リフレッシュするバックグラウンドジョブを実装。Leverの1時間期限は特に注意が必要。

### 課題3: Webhook配信の信頼性

**推奨**: 受信側での冪等性（Idempotency Key）管理。同一イベントの重複処理を防ぐ。

```python
# 冪等性確保の例
def handle_webhook(event_id: str, payload: dict):
    if ProcessedEvent.exists(event_id):
        return  # 重複処理をスキップ
    with transaction():
        ProcessedEvent.create(event_id)
        process_payload(payload)
```

### 課題4: GDPR「忘れられる権利」への対応

**推奨**: 候補者削除APIエンドポイントの実装と、関連するすべてのシステム（検索インデックス・バックアップ等）への削除伝播設計。

### 課題5: AIスクリーニングのバイアス

**推奨**: 保護属性をモデルから除外 + 定期的な公平性評価（Fairness Audit）+ Human in the Loop設計。
