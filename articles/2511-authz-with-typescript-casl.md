---
title: "TypeScript×CASLでつくるSaaSの認可"
emoji: "🛡️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["TypeScript", "CASL", "React", "Prisma"]
published: false
---

# はじめに

BtoB SaaS の開発において、「認可（Authorization）」 は複雑でセキュリティリスクの高い領域です。
ただし、認可ロジックとビジネスロジックが絡み合うと、アプリは複雑化してしまいます。
そのため、認可ロジックを如何に疎結合に独立して保てるかが重要と考えています。

- **UI** では、権限がないボタンを「見せない」
- **API** では、権限がないリソース操作を「させない」
- **DB** では、権限がないデータを「引かない」

これらがバラバラに実装されると、セキュリティホールやバグの温床になります。

本記事では、フロントエンド/バックエンドで TypeScript を採用している PeopleX AI面接を題材に、**CASL** を中心に据えた認可アーキテクチャの実践を紹介します。 

# BtoB SaaS における認可と技術選定

BtoB SaaSにおける認可は、サービスによって大きく要求が変わらないため、主流なデザインパターンがいくつか存在します。

まずはどんな認可パターンがあるのか紹介した上で、我々が採用した認可パターンについて解説します。

## 認可モデルのおさらい

本題に入る前に、代表的な4つの認可モデルについて整理します。SaaS開発では、これらを要件に応じて選択することが一般的です。

| モデル | 判定基準 | 特徴・メリット | デメリット | 典型的なユースケース |
| :--- | :--- | :--- | :--- | :--- |
| **RBAC**<br>(Role Based Access Control) | **役割 (Role)**<br>例: Admin, Manager, Guest | ・シンプルで直感的<br>・「誰が何をできるか」が明確 | ・ロールの数が増えすぎると管理が困難になる (Role Explosion)<br>・「自分の作成した記事」のような文脈依存の制御が苦手 | 管理画面、ブログの投稿権限 |
| **ABAC**<br>(Attribute Based Access Control) | **属性 (Attribute)**<br>例: 部署, 作成日, 公開ステータス | ・「AかつBなら許可」のような複雑な条件を表現できる<br>・きめ細かな制御が可能 | ・ポリシー定義が複雑になりがち<br>・パフォーマンスへの影響が出やすい | 部署ごとのデータ閲覧制限、時間帯制限 |
| **PBAC**<br>(Policy Based Access Control) | **ポリシー (Policy)**<br>例: ルール言語での条件定義 | ・認可ロジックを分離・コード管理できる (Policy as Code)<br>・RBAC/ABAC/ReBACを包括的に表現可能 | ・専用言語や評価エンジンの学習・導入コスト<br>・デバッグが難しくなる場合がある | AWS IAM, OPA, 複雑なコンプライアンス要件 |
| **ReBAC**<br>(Relationship Based Access Control) | **関係性 (Relationship)**<br>例: 所有者(Owner), 所属(Member) | ・グラフ構造や階層構造に強い<br>・親リソースの権限を子へ継承させやすい | ・リレーション探索のコストが高い<br>・実装難易度が高い（Google Zanzibarなど） | Google Drive (フォルダ権限の継承)、SNSのフレンド限定公開 |

ほとんどのSaaSではロール管理だけで十分な場合が多いですが、組織外の個人情報を扱うケースや導入企業のセキュリティ要求レベルが高いとそれ以外の要求が発生する可能性があります。これらを「if 文」や「ロールのハードコード」で実装しようとすると、ビジネスロジックと認可ロジックが密結合し、メンテナンス不能なスパゲッティコードが生まれてしまいます。

視覚的・直感的に違いが理解しやすいデモアプリを用意したので自由に触ってみてください！

https://access-control-visualizer.vercel.app

https://github.com/saka2jp/access-control-visualizer

## 今回の題材：[PeopleX AI面接](https://peoplex.jp/)の場合

PeopleX AI面接は、ATS（採用管理システム）としての側面もあり、まだ組織に所属していない外部の方の個人情報を取り扱うサービスのため、認可の要件レベルは一般的な BtoB SaaS と比べると高いと感じています。

要求を整理するとざっくりと以下のような形になります。

| 代表的な要件 | 認可モデルの適用 | 内容 |
| ---- | ---- | ---- |
| テナント管理者はテナント設定の操作のみ可能 | **RBAC** | 「テナント管理者」というロールに適切な権限を定義 |
| 評価担当者は「自部署」の応募者のみ閲覧可能 | **ABAC** | 企業ユーザーの属性（部署ID）とリソースの属性を比較 |
| 評価担当者には個人情報（氏名・生年月日など）を見せない | **Field-Level** | 特定のフィールド（列）へのアクセス制御 |

AI面接では要求に合わせてどの認可パターンで実現するかを決めています。
どの認可モデルを選択しても実現できることが多いですが、コストパフォーマンスやメンテナンス性などを考慮し、オーバーエンジニアリングにならないような設計を意識しています。

# 技術選定：認可 SaaS vs OSS ライブラリ

Auth0 のような IDaaS は広く普及していますが、BtoB SaaS における認可の複雑化が課題となっており、これを解決する「Authorization as a Service」が注目されています。

AaaSの主要なサービスを比較・調査しました。

## 1. 主要な認可サービス

1.  **[Auth0 FGA](https://auth0.com/fine-grained-authorization)**
    - Auth0 が提供する、Google Zanzibar モデルに基づいた認可サービス。
    - **特徴:** Auth0 (Okta) エコシステムの一部であり、多機能。

2.  **[Oso Cloud](https://www.osohq.com/application-authorization)**
    - 開発者体験 (DX) を最優先に設計された、BtoB SaaS 開発で人気のあるサービスの一つ。
    - **特徴:** 独自のポリシー言語「Polar」を使用。

3.  **[Permit.io](https://www.permit.io/)**
    - OPA (Open Policy Agent) をベースにしつつ、UIベースの管理に強みを持つサービス。
    - **特徴:** 「ローコード」での権限管理を重視。権限設定を変更できる 管理UI を提供。

## 2. OSS ライブラリの採用

これら外部サービスは強力ですが、外部通信によるレイテンシや、ローカル開発・CI 環境の複雑化といったオーバーヘッドも伴います。

今回のシステムでは、以下の要件を重視し、**Node.js プロセス内で完結するライブラリベースのアプローチ**を採用しました。

- **コスト:** 従量課金される SaaS モデルを避け、スケーリングしてもコストが増加しないようにしたい。
- **開発体験:** 外部依存を減らし、`npm install` だけで開発環境が整うシンプルさを保ちたい。
- **DB連携:** ビジネスロジックと認可ロジックのWhere句を分離しつつ、効率的なクエリを発行したい。
- **低レイテンシ:** 認可判定のたびにネットワーク通信を発生させたくない。

# なぜ CASL なのか？

Node.js エコシステムの認可ライブラリには [Casbin](https://casbin.org/), [AccessControl](https://onury.io/accesscontrol/) などもありますが、中でも **CASL** が既存のアーキテクチャや要件にマッチしていました。

1.  **Isomorphic:** 定義したポリシーをフロントエンドとバックエンドの両方で再利用できます。
2.  **Type Safety:** ポリシーをTypeScript で宣言的に記述でき、型推論が効きます。
3.  **API Integration:** デコレータによるAPIレベルでの宣言的な認可設定が可能です。
4.  **Database Integration:** `@casl/prisma` を使うことで、クエリをポリシーとして宣言的に定義できます。これは他ライブラリにはない強みです。
5.  **Frontend Integration:** `@casl/react` を使うことで、直感的・宣言的な認可判定が可能です。

# CASLによる認可実装

## 1. TypeScript によるポリシー定義

### Ability とは？

CASL における **Ability** は、認可ルールの心臓部となるクラスです。
これは「ユーザーがどのような操作（Action）を、どのリソース（Subject）に対して行えるか」というルールを詰め込んだオブジェクトです。

例えば、「記事（Article）を読む（read）ことができる」「自分の書いた記事なら更新（update）できる」といったルール定義を `Ability` インスタンスとして定義します。
アプリケーション側では、この Ability に対して `ability.can('update', article)` と問いかけるだけで、複雑な条件分岐なしに権限判定を行うことができます。

ポリシーのような認可ルールを宣言的にTypeScriptで定義することができます。

### 型定義とAbilityの構築

アプリケーション固有のアクションとリソース（Subject）を型定義します。
Prisma の生成した型をそのまま Subject として利用可能です。

```typescript
import { Candidate, JobPost } from '@prisma/client'; // Prismaの型
import { PureAbility } from '@casl/ability';
import { PrismaQuery, Subjects } from '@casl/prisma';

// アクションの定義
export type Action = 'manage' | 'create' | 'read' | 'update' | 'delete';

export type AppSubjects = 
  | Subjects<{
      Candidate: Candidate;
      JobPost: JobPost;
    }>
  | 'all';

// Prisma向けのAbility型定義
export type AppAbility = PureAbility<[Action, AppSubjects], PrismaQuery>;
```

### ポリシーファクトリの実装

次に、ユーザーコンテキストを受け取り `AppAbility` を生成する関数を実装します。ここで `AbilityBuilder` を使用しますが、型安全性を確保するために `createPrismaAbility` を使用します。

```typescript
import { AbilityBuilder } from '@casl/ability';
import { createPrismaAbility } from '@casl/prisma';

export function defineAbilityFor(user: UserContext): AppAbility {
  const { can, cannot, build } = new AbilityBuilder(createPrismaAbility);

  if (user.role === 'Recruiter') {
    // ABAC: 自部署の求人のみ閲覧・操作可能
    can(['read', 'update'], 'JobPost', {
      departmentId: { in: user.departmentIds },　// Prismaのクエリ構文で条件を記述
    });
  }
  if (user.role === 'Evaluator') {
    // Field-Level: 個人情報は見せない
    cannot('read', 'Candidate', ['name', 'email']);
  }
  return build();
}
```

ここで重要なのは、`conditions` オブジェクトの中に Prisma のクエリ構文（`in`, `some` など）を直接書いている点です。これが後の Prisma 連携で効いてきます。
また、例のように愚直に条件分岐で `can`, `cannot` を呼び出しているのではなく、ロールと権限のペアをJSONで宣言的に管理しています。

## 2. バックエンド実装: API レベルの制御

NestJS では、Guard とカスタムデコレータを組み合わせることで、コントローラーのメソッドごとに宣言的な認可チェックを実装できます。
公式ドキュメントの [Authorization](https://docs.nestjs.com/security/authorization) セクションでも紹介されているパターンです。

```typescript
@Get()
@UseGuards(PoliciesGuard)
@CheckPolicies((ability) => ability.can('read', 'JobPost'))
async findAll() {
  return this.jobPostService.findAll();
}
```

このように記述することで、`findAll` メソッドが実行される前に `PoliciesGuard` がポリシーを検証し、権限がない場合は `403 Forbidden` を返します。

## 3. バックエンド実装：Prisma との連携

API で「権限のあるデータだけを取得する」ために、全件取得してからプログラムで filter するのはパフォーマンス的に望ましくはありません。DB クエリの段階で絞り込む必要があります。
CASL 公式の `@casl/prisma` プラグインを使用することで、定義した Ability を Prisma の `where` 句として呼び出すだけで済みます。

### accessibleBy ヘルパーの利用

```typescript
import { accessibleBy } from '@casl/prisma';

export async function findJobPosts(user: UserContext) {
  const jobPosts = await prisma.candidate.findMany({
    where: {
      AND: [
        // AbilityからWhere句を参照
        accessibleBy(ability, 'read').JobPost,
      ]
    }
  });

  return jobPosts;
}
```

Abilityにて定義したクエリ構文が参照されます。

これを自前で愚直に実装してしまうと、条件分岐地獄に陥ります。CASL に任せることで、**「ポリシー定義（Ability）」を変更するだけで、発行される SQL も自動的に追従する** 状態を作ることができます。


## 4. フィールドレベルの認可

個人情報は見せないという要件に対しては、`permittedFieldsOf` ヘルパーが利用可能です。

```typescript
import pick from 'lodash/pick';
import { permittedFieldsOf } from '@casl/ability/extra';

// 取得したオブジェクトから、見せてはいけないフィールドを除外する
const candidate = await prisma.candidate.findUnique({ ... });
const fields = permittedFieldsOf(
  ability, 'read', candidate,
  { fieldsFrom: rule => rule.fields || [] },
);

// Lodashのpickなどでフィルタリング
const sanitizedCandidate = pick(candidate, fields);
res.json(sanitizedCandidate);
```

ただし、Prisma の `select` 句に変換するのは複雑度が高いため、API レスポンスを返す直前でフィルタリング（DTOへのマッピング時など）を行うのが現実的なケースもあると考えています。

## 5. フロントエンド実装：React との連携

フロントエンドでも同じポリシーを共有します。
ポリシーをAPI経由で取得し、JSON オブジェクトから `AppAbility` を生成しています。
CASL の React バインディング `@casl/react` を使用します。

### UI での利用

Canコンポーネントか、関数での判定が可能です。

```tsx
export function CandidateCard({ candidate }) {
  const ability = useAppAbility() // AppAbilityを参照するカスタムフック

  <div className="card">
    <h1>{candidate.name}</h1>
    
    {/* 編集権限がある場合のみボタンを表示 */}
    <Can I="update" this={candidate}>
      <button onClick={handleEdit}>編集</button>
    </Can>

    {/* 削除権限がない場合はボタンを非活性化 */}
    <button
      className="danger"
      onClick={handleDelete}
      disabled={!ability.can('delete', 'CompanyInterview')}
    >
      削除
    </button>
  </div>
};
```

# 実際に導入・運用してみた感想

最後に、CASL をプロダクション環境に導入してみて感じたメリット・デメリットをまとめます。

## よかったポイント

1.  **TypeScriptによる認可ロジックの宣言的定義**
    認可のロジックが `Ability`（ポリシー定義）として一箇所に集約できる点です。仕様変更があっても、ここだけ直せば全レイヤーに反映される安心感があります。

2.  **Prisma 連携が強力**
    開発者は認可のロジックとビジネスロジックを混合せず、独立してクエリを書くことができる点です。ロジックが混合せず、メンテナンス性の高いコードを維持することができています。

3.  **開発者体験の向上**
    実装が宣言的・直感的にできる点です。利用者は認可基盤の具体まで詳細に理解できていなくとも、認可ロジックを適切に適用できる基盤を整えることができました。

## 工夫が必要なポイント

1.  **複雑な条件の制御**
    Prisma の `WhereInput` に変換できる条件しか書けないため、あまりに複雑になるケースにおいては、無理に CASL に押し込まずサービス層で補完するなどの割り切りも必要と感じました。

2.  **型定義の複雑さ**
    TypeScript の型推論をフル活用しようとすると、ジェネリクスや型定義がやや複雑になりがちです。
    例えば、`PureAbility<[Action, Subjects<{ Candidate: Candidate }>], PrismaQuery>` のような定義は、初見では直感的に理解しづらいかもしれません。
    しかし、この複雑さを受け入れることで、「リソース名のタイポ」や「存在しないフィールドでの絞り込み」をコンパイルエラーとして検知できる強力な安全性が手に入ります。何より補完が効きます。長期的な保守性を考えると十分なリターンがあると感じています。

3.  **SQLの複雑化**
    `accessibleBy` は便利な反面、生成される SQL が暗黙的で意図せず複雑で冗長になってしまうことがあります。インデックスが効きにくいクエリが生成されていないか、定期的なスロークエリの監視や開発時のデバッグは必要と感じています。

# まとめ

BtoB SaaS の認可は複雑になりがちですが、CASL を導入することで、型安全かつ堅牢で疎結合な認可基盤をすばやく構築できました。
外部の認可 SaaS を使う選択肢もありますが、TypeScript / Next.js / NestJS / Prisma を採用しているアーキテクチャにおいては、CASLが有力候補なのではないかと感じています。
認可のような一般化されている技術はうまくSaaSやOSSを利用することで、プロダクトのコアバリュー開発に集中したいですね。
そのためにも正しい知識を元に、状況に沿った最適な技術選定を行える力を今後も磨いていきたいです。

※ すべてのコードはサンプルであり、実際のプロダクションでのコードではありません。

