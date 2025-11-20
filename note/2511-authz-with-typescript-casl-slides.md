---
marp: true
theme: default
paginate: true
header: TypeScript×CASLでつくるSaaSの認可
footer: @PeopleX, Inc.
---

# TypeScript×CASLでつくるSaaSの認可

###### 株式会社PeopleX 坂津 潤平 / 芹澤 和也

---

# はじめに

BtoB SaaS開発において「認可 (Authorization)」は複雑でセキュリティリスクの高い領域

- **UI** では、権限がないボタンを「見せない」
- **API** では、権限がないリソース操作を「させない」
- **DB** では、権限がないデータを「引かない」

これらがバラバラに実装されると、セキュリティホールやバグの温床に。

-> PeopleX AI面接における**CASL** を中心に据えた認可アーキテクチャの実践を紹介

---

# BtoB SaaSの代表認可パターン

| パターン | 概要 | 具体例 |
| ---- | ---- | ---- |
| **ロールベース** | 役割（Role）に基づく大枠の制御 | 「管理者」は設定変更可、「一般」は閲覧のみ |
| **属性ベース** | 所属に基づく動的な制御 | 自分の部署のリソースのみ閲覧可 |
| **関係性ベース** | 所有に基づく動的な制御 | 自分の所有・担当するリソースのみ閲覧可 |
| **フィールドレベル** | 特定の項目（列）ごとの制御 | 給与額や評価コメントなど、センシティブな情報のマスク |

---

# 認可の代表的なモデルのおさらい

SaaS要件に応じて適切なモデルを選択する必要がある

| モデル | 判定基準 | 特徴 |
| :--- | :--- | :--- |
| **RBAC** | **役割** | ・シンプルで直感的<br>・「誰が何をできるか」を定義 |
| **ABAC** | **属性** | ・「AかつBなら許可」のような複雑な条件を表現できる<br>・きめ細かな制御が可能 |
| **ReBAC** | **関係性** | ・グラフ構造や階層構造に強い<br>・親リソースの権限を子へ継承させやすい |
| **PBAC** | **ポリシー** | ・認可ロジックを分離・コード管理できる (Policy as Code)<br>・RBAC/ABAC/ReBACを包括的に表現可能 |

---

# デモ

---

# 題材: PeopleX AI面接

| 代表的な要件 | 認可モデルの適用 | 内容 |
| ---- | ---- | ---- |
| テナント管理者はテナント設定の操作のみ可能 | **RBAC** | 「テナント管理者」というロールに適切な権限を定義 |
| 評価担当者は「自部署」の応募者のみ閲覧可能 | **ABAC** | 企業ユーザーの属性（部署ID）とリソースの属性を比較 |
| 評価担当者には個人情報（氏名・生年月日など）を見せない | **Field-Level** | 特定のフィールド（列）へのアクセス制御 |

要求に合わせてどの認可パターンで実装するかを決定
コストパフォーマンスやメンテナンス性などを考慮

---

# 技術選定: SaaS vs OSS

## 「AaaS（Authorization as a Service）」

Auth0 のような IDaaS は広く普及している
BtoB SaaS における認可機能の提供
SaaSとOSSどちらを選定するべき？

---

# 主要な認可サービス

1.  **[Auth0 FGA](https://auth0.com/fine-grained-authorization)**
    - Auth0 が提供する、Google Zanzibar モデルに基づいた認可サービス。
    - **特徴:** Auth0 (Okta) エコシステムの一部であり、多機能。

2.  **[Oso Cloud](https://www.osohq.com/application-authorization)**
    - 開発者体験 (DX) を最優先に設計された、BtoB SaaS 開発で人気のあるサービスの一つ。
    - **特徴:** 独自のポリシー言語「Polar」を使用。

3.  **[Permit.io](https://www.permit.io/)**
    - OPA (Open Policy Agent) をベースにしつつ、UIベースの管理に強みを持つサービス。
    - **特徴:** 「ローコード」での権限管理を重視。権限設定を変更できる 管理UI を提供。

---

# Why CASL?

1. **Isomorphic** 🔄
   - ポリシーを FE/BE 両方で再利用可能
2. **Type Safety** 🛡️
   - TypeScript 完全対応
3. **Database Integration** 🗄️
   - Prisma の Where 句へ自動変換 (`@casl/prisma`)
4. **Declarative** 📝
   - 宣言的なポリシー定義

---

# 実装: Abilityの定義

Prismaの型をそのままSubjectとして利用可能

```typescript
import { Candidate } from '@prisma/client';
import { PureAbility } from '@casl/ability';
import { PrismaQuery, Subjects } from '@casl/prisma';

// アクションとリソースの定義
export type Action = 'manage' | 'create' | 'read' | 'update' | 'delete';

// Prismaモデルと紐付け
export type AppSubjects = Subjects<{ Candidate: Candidate }> | 'all';

export type AppAbility = PureAbility<[Action, AppSubjects], PrismaQuery>;
```

---

# 実装: ポリシーファクトリ

ユーザーコンテキストから動的に権限を生成

```typescript
export function defineAbilityFor(user: UserContext): AppAbility {
  const { can, cannot, build } = new AbilityBuilder(createPrismaAbility);

  if (user.role === 'Recruiter') {
    // ABAC: 自部署の候補者のみ操作可能
    can(['read', 'update'], 'Candidate', {
      departmentId: { in: user.departmentIds },　// Prismaのクエリ構文で条件を記述
    });
  }
  if (user.role === 'Interviewer') {
    // Field-Level: 個人情報は見せない
    cannot('read', 'Candidate', ['salary', 'address']);
  }
  return build();
}
```

---

# BE実装: APIレベルの制御 (NestJS)

Guardとデコレータで宣言的に記述

```typescript
@Get()
@UseGuards(PoliciesGuard)
// 読む権限があるかチェック
@CheckPolicies((ability) => ability.can('read', 'Candidate'))
async findAll() {
  return this.candidatesService.findAll();
}
```

---

# BE実装: Prisma統合 (重要!)

`accessibleBy` で **権限があるデータだけ** をDBから引く

```typescript
import { accessibleBy } from '@casl/prisma';

// Service層
const candidates = await prisma.candidate.findMany({
  where: {
    AND: [
      // ✨ Magic: AbilityからWhere句を自動生成
      accessibleBy(ability, 'read').Candidate,
    ]
  }
});
```
✅ `departmentId: { in: [...] }` が自動的にSQLに適用される

---

# FE実装: Reactコンポーネント

`<Can>` コンポーネントで宣言的にUI制御

```tsx
export function CandidateCard({ candidate }) {
  return (
    <div className="card">
      <h1>{candidate.name}</h1>
      
      {/* 権限がある場合のみボタンを表示 */}
      <Can I="update" this={candidate}>
        <button onClick={handleEdit}>編集</button>
      </Can>
    </div>
  );
}
```

---

# FE実装: Reactコンポーネント

カスタムフック＆関数呼び出し

```tsx
export function CandidateCard({ candidate }) {
  const ability = useAppAbility();

  return (
    <div className="card">
      <h1>{candidate.name}</h1>
      
      {/* 権限がない場合はボタンを非活性 */}
      <button onClick={handleEdit} disabled={ability.cannot('update', candidate)}>編集</button>
    </div>
  );
}
```

---

# 導入してよかったポイント

## 👍
- **一貫性:** FE/BEで同じロジックを使用可能
- **堅牢性:** DBレベルでのフィルタリングで漏洩防止
- **安全性:** TypeScriptによる型補完

---

# 工夫が必要なポイント

## 🤔
- **型定義の複雑さ:** ジェネリクス等の理解が必要
- **クエリの複雑化:** `accessibleBy` が生成するクエリのパフォーマンス監視は必要

---

# まとめ

BtoB SaaSの認可は複雑だが、**CASL + Prisma** で解決できる

- 認可ロジックを **「ポリシー定義」** に集約
- **Isomorphic** に FE/BE で共有
- ビジネスロジックと認可ロジックを **疎結合** に保つ

**堅牢でメンテナンス性の高い認可基盤を構築しましょう！**

