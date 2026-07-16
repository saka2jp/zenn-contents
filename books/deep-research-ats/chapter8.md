---
title: "マルチテナントSaaS設計"
free: true
---

# マルチテナントSaaS設計

ATSはSaaS型が主流であり、複数の企業（テナント）が同一インフラを共有する。テナントデータの分離設計は、セキュリティ・スケーラビリティ・コンプライアンスに直結する重要な設計判断である（出典: [Daily.dev - Multi-Tenant Database Design Patterns](https://daily.dev/blog/multi-tenant-database-design-patterns-2024/)）。

## データ分離パターン4種

### パターン1: 共有DB・共有スキーマ（Pool）

```sql
-- 全テナントのデータが同一テーブルに存在
-- tenant_id カラムで分離
SELECT * FROM applications
WHERE tenant_id = 'acme-corp'
  AND status = 'ACTIVE';
```

| 評価軸 | 評価 |
|--------|------|
| データ分離性 | 低（アプリ層での制御に依存） |
| セキュリティ | 低〜中（Row Level Security導入で向上） |
| スケーラビリティ | 高 |
| コスト | 低 |
| カスタマイズ性 | 低 |

**PostgreSQL Row Level Security (RLS) を使った実装例:**

```sql
-- テナント分離ポリシー
ALTER TABLE applications ENABLE ROW LEVEL SECURITY;

CREATE POLICY tenant_isolation ON applications
  USING (tenant_id = current_setting('app.current_tenant')::UUID);

-- アプリケーション側でのテナント設定
SET app.current_tenant = 'acme-corp-uuid';
```

RLSはアプリケーション層ではなくDB層でのアクセス制御を実現し、プログラマーミスによるデータリークを防ぐ（出典: [Daily.dev - Multi-Tenant Database Design Patterns](https://daily.dev/blog/multi-tenant-database-design-patterns-2024/)）。

---

### パターン2: 共有DB・別スキーマ（Bridge）

```sql
-- テナントごとに独立したスキーマ
-- acme_corp.applications, startup_inc.applications のように分離

CREATE SCHEMA acme_corp;
CREATE TABLE acme_corp.applications (...);

CREATE SCHEMA startup_inc;
CREATE TABLE startup_inc.applications (...);
```

| 評価軸 | 評価 |
|--------|------|
| データ分離性 | 中（スキーマレベルで分離） |
| セキュリティ | 中 |
| スケーラビリティ | 中 |
| コスト | 中 |
| カスタマイズ性 | 中（テナント固有のカラム拡張が可能） |

課題: テナント数増加に伴うスキーマ管理の複雑化（マイグレーション管理が困難）

---

### パターン3: 別DB（Silo）

各テナントが完全に独立したデータベースインスタンスを持つ。

| 評価軸 | 評価 |
|--------|------|
| データ分離性 | 最高 |
| セキュリティ | 最高 |
| スケーラビリティ | 低（インスタンス数が線形増加） |
| コスト | 高 |
| カスタマイズ性 | 高 |

エンタープライズ向けの大型契約テナントに適用されることが多い。

---

### パターン4: ハイブリッドアプローチ

大多数のSMBテナントは共有スキーマ（Pool）に収容し、大規模エンタープライズテナントには専用DB（Silo）を割り当てる戦略。

```
テナント分類:
  ├── SMBテナント（99%) → 共有DB + RLS
  └── Enterpriseテナント（1%）→ 専用DBインスタンス
```

これが現代の主要ATSベンダーが採用する実際のパターン（推定）。

---

## テナント識別と認証

```
リクエスト処理フロー:

HTTPリクエスト
  ├── サブドメインから識別:  acme-corp.greenhouse.io
  ├── カスタムドメイン:     careers.acme-corp.com
  ├── HTTPヘッダー:         X-Tenant-ID: acme-corp
  └── JWTクレーム:          {"tenant_id": "acme-corp-uuid"}
         ↓
    テナントコンテキスト設定
         ↓
    DB接続プールからテナント対応接続取得
         ↓
    RLS or スキーマ切り替え
         ↓
    ビジネスロジック実行
```

## マルチテナント設計上の重要考慮点

### 1. テナント間クロス汚染の防止

- すべてのクエリに `WHERE tenant_id = ?` を強制（ORMレベルで実装）
- テスト環境で別テナントのデータが漏れないことを検証する専用テストスイート
- PostgreSQL RLS をバックストップとして採用

### 2. テナントごとのリソース分離

```
問題: あるテナントの大量バッチ処理が他テナントのパフォーマンスに影響（ノイジーネイバー問題）

対策:
  - キューワーカーのテナント別優先度管理
  - データベース接続プールのテナント別上限設定
  - APIレート制限のテナント別適用
  - Elasticsearch インデックスのテナント別分離
```

### 3. テナント固有のカスタマイズ

ATSは企業ごとに選考プロセスが異なるため、以下を可変にする必要がある。

- カスタム選考ステージ（Pipeline）の定義
- カスタムフォームフィールドの追加
- カスタムレポート・ダッシュボード
- SSO設定（SAML / OIDC）のテナント別設定

```json
// テナント設定テーブル例
{
  "tenant_id": "acme-corp-uuid",
  "custom_stages": ["技術試験", "カルチャーフィット面接"],
  "custom_fields": [
    {"name": "github_url", "type": "string", "required": false}
  ],
  "sso": {
    "provider": "okta",
    "entity_id": "https://acme.okta.com",
    "sso_url": "https://acme.okta.com/sso/saml"
  },
  "eeoc_enabled": true,
  "data_residency_region": "us-east-1"
}
```

### 4. データ主権・コンプライアンス

| 要件 | 対応 |
|------|------|
| GDPR（EU） | EU候補者データのEU内リージョン保存、忘れられる権利API |
| CCPA（California） | カリフォルニア州居住者の個人データ削除エンドポイント |
| SOC2 Type II | 監査ログ、アクセス制御、暗号化の証明 |
| EEOC（米国） | 雇用機会均等データの分離保存・任意入力 |

### 5. 2026年の新技術動向

**サーバーレスDB分離パターン**（出典: [Daily.dev](https://daily.dev/blog/multi-tenant-database-design-patterns-2024/)）:

- **Neon**: テナントごとにPostgresブランチを作成。各ブランチが独自のコンピュート・ストレージを持つ
- **Turso**: テナントごとに個別SQLiteデータベースをエッジに分散。超低レイテンシでの分離
