---
title: "[登壇資料原案] TypeScriptエンジニアのための「認可」再入門 - CASLで実現するセキュアなBtoB SaaS開発"
emoji: "🛡️"
type: "idea"
topics: ["typescript", "security", "authorization", "casl", "prisma"]
published: false
---

# はじめに

この記事は、TSKaigi などの技術カンファレンスでの登壇を想定した発表スライドの原案兼台本です。
バックエンドエンジニア（BE）とフロントエンドエンジニア（FE）の2名による掛け合い形式の発表を想定しています。
発表時間は40分です。

---

## 0. オープニング (5分)

### スライド 1: タイトル
**TypeScript×CASLでつくるSaaSの認可**

- Speaker:
    - Backend Engineer (BE)
    - Frontend Engineer (FE)

**(Talk Script)**
**BE:** 皆さんこんにちは。「TypeScript×CASLでつくるSaaSの認可」というタイトルで発表させていただきます。
**FE:** 今回は、バックエンドエンジニアとフロントエンドエンジニアのペアプロ形式でお送りします。よろしくお願いします！
**BE:** よろしくお願いします。さて、今日のテーマは「認可 (Authorization)」です。
**FE:** 認証 (Authentication) ではなく、認可 (Authorization) ですね。
**BE:** そうです。「誰が」を確認するのが認証で、「何ができるか」を確認するのが認可ですね。地味ですが、SaaS開発においては絶対に避けて通れない、かつ最も事故りやすい機能の一つです。
**FE:** 今日は、私たちが開発しているサービスで直面した「認可の闇」と、それをどうやってTypeScriptのエコシステムで乗り越えたか、という実録をお話しできればと思います。

### スライド 2: 認可、つらくないですか？
**Authorization is Hard.**

- **Frontend:**
    - 「管理者は表示、一般は見せない」
    - 「自分のデータなら編集ボタンを出す」
    - 「あ、でも課長代理は見れるようにして…」
- **Backend:**
    - 「APIで他人のデータ引けないようにしなきゃ」
    - 「if (user.role === 'admin') ... else if ...」
    - 「あれ、ここチェック漏れてない？」

**(Talk Script)**
**FE:** いきなりですが皆さん、認可の実装ってつらくないですか？ 
**BE:** つらいですねぇ。
**FE:** フロントエンドだと「このボタン、誰に出すんだっけ？」みたいな条件分岐がコンポーネント中に散らかりがちですよね。
**BE:** 「管理者は出る」「一般ユーザーは出ない」くらいならいいんですけどね。
**FE:** そうそう。「自分のデータなら編集ボタンを出す」とか。「下書き状態なら作成者だけが見れる」とか。
**BE:** さらに途中から「あ、課長代理という役職が増えたので、課長と同じ権限で見れるようにして」とか言われるともう…。
**FE:** うわぁ…。全画面の `if` 文を見直して回る未来が見えますね。
**BE:** バックエンドも大変ですよ。APIでデータを返すとき、絶対に「他人のデータ」を返してはいけない。
**FE:** セキュリティインシデント、一発退場ですからね。
**BE:** だから、APIのあちこちに `if (user.role === 'admin')` みたいな分岐を書くわけですが、これが増えてくると「あれ、このAPI、チェック漏れてない？」って不安で夜も眠れなくなります。
**FE:** 今日は、そんな「認可のつらみ」を、TypeScriptのエコシステムを使ってどう解決したか、という話をします。具体的には **CASL** というライブラリを使います。
**BE:** 「キャッスル」と読みます。これを使うと、フロントエンドとバックエンドで認可ロジックを共有できて、しかもDBクエリまで安全になるという、夢のような話です。

## 1. BtoB SaaSにおける認可の複雑性 (10分)

### スライド 3: 認可の5つのレイヤー
**Authorization Layers in BtoB SaaS**

1.  **L1. テナント分離** (絶対的な境界)
2.  **L2. ロールベース (RBAC)** (Admin / Member)
3.  **L3. 属性ベース (ABAC)** (部署、ステータス)
4.  **L4. 関係性ベース (ReBAC)** (所有者、継承)
5.  **L5. フィールドレベル** (個人情報マスク)

**(Talk Script)**
**BE:** まず、なぜSaaSの認可はこんなにも難しいのか、整理してみましょう。僕たちはこれを5つのレイヤーで分類して考えています。
**FE:** L1の「テナント分離」は基本中の基本ですね。A社のユーザーはB社のデータに絶対にアクセスできない。これは絶対防衛ラインです。
**BE:** 次にL2の「ロールベース (RBAC)」。Adminなら設定変更できるけど、Memberは閲覧のみ、みたいなやつですね。
**FE:** ここまではよくあるんです。でもBtoB SaaS、特にエンタープライズ向けだと、その先が求められますよね。
**BE:** そうなんです。L3の「属性ベース (ABAC)」。例えば「自分の部署のリソースのみ閲覧可」とか。これには「ユーザーの部署ID」と「データの部署ID」を比較する必要があります。
**FE:** ロールだけじゃ決まらないやつですね。さらにL4の「関係性ベース (ReBAC)」。
**BE:** 「自分が作成した記事」とか「自分がオーナーであるプロジェクト」とかですね。Google Driveのフォルダ権限継承なんかもこれに近いです。
**FE:** そして極めつけがL5、「フィールドレベル」。
**BE:** 「評価担当者は候補者のデータを見れるけど、年収（Salary）カラムだけは見ちゃダメ」みたいなやつです。
**FE:** これらを全部、愚直に `if` 文で書くとどうなるか。
**BE:** ビジネスロジックと認可ロジックが密結合して、メンテナンス不能なスパゲッティコードが一丁上がりです。
**FE:** L3より上の要件が出てきた瞬間に、単純なロール管理では破綻するんですよね。

### スライド 4: 今回の題材：AI面接サービス
**Case Study: AI Interview Service (ATS)**

- **要件:**
    - 採用担当者: 自部署の応募者のみ閲覧・編集可能
    - 評価担当者: 担当する面接のみ閲覧、個人情報は閲覧不可
- **課題:**
    - 複雑な条件の組み合わせ
    - BE/FEでのロジック二重管理

**(Talk Script)**
**FE:** 私たちが開発しているAI面接サービスでも、まさにこの課題に直面しました。
**BE:** 具体的にはこんな要件です。「採用担当者は、自部署の応募者のみ閲覧・編集可能」。これは ABAC ですね。
**FE:** そして「評価担当者は、自分が担当する面接のみ閲覧できて、かつ個人情報（年収など）は見ちゃダメ」。これは ReBAC とフィールドレベル制御の組み合わせです。
**BE:** ロールだけで制御できないんですよね。「自部署である」とか「担当である」といった動的な条件が絡んでくる。
**FE:** これをフロントとバックエンドで別々に実装すると、絶対に整合性が取れなくなります。「画面では編集ボタンが出てるのに、押すとAPIから403エラーが返ってくる」とか。
**BE:** ユーザーからすると「バグですか？」ってなりますよね。
**FE:** 逆に「画面では隠してるけど、APIを直接叩いたら見えちゃった」なんてことになったら…。
**BE:** それはもう始末書ものです。なので、フロントとバックエンドで認可ロジックを「一元管理」したい。これが今回の最大のモチベーションでした。

## 2. 技術選定：SaaS vs OSS (5分)

### スライド 5: 認可のアプローチ：SaaSかライブラリか
**Buy vs Build?**

- **Authorization-as-a-Service (SaaS)**
    - Auth0 FGA, Oso Cloud, Permit.io
    - メリット: スケールする、管理画面がある
    - デメリット: 外部通信のレイテンシ、コスト、学習コスト
- **Library (In-Process)**
    - CASL, Casbin, AccessControl
    - メリット: 低レイテンシ、DB統合、型安全性
    - デメリット: 自前実装が必要

**(Talk Script)**
**BE:** そこで技術選定です。最近は「Authorization-as-a-Service（認可サービス）」も流行っていますよね。
**FE:** 有名どころだと、Auth0 FGA とか Oso Cloud、Permit.io なんかがありますね。
**BE:** Auth0 FGA は Google Zanzibar モデルベースでスケーラビリティが売りですし、Oso Cloud は BtoB SaaS のモデリングが得意です。Permit.io は管理画面がリッチで、非エンジニアでも権限設定ができるのが魅力です。
**FE:** 便利そうですよね。管理画面でポチポチ設定できたり。
**BE:** 確かに強力なんですが、今回は見送りました。理由は大きく2つ、「レイテンシ」と「DB統合」です。
**FE:** レイテンシ、ですか。
**BE:** 認可判定って、ほぼすべてのリクエストで発生するじゃないですか。一覧画面で50件データを出すなら、50回判定が必要かもしれない。
**FE:** 確かに。そのたびに外部APIを叩いていたら、塵も積もって山となりますね。
**BE:** そうです。ネットワーク通信のオーバーヘッドは避けたかった。それと、コスト面も気になります。Check API のコール数課金だと、スケールしたときに怖い。
**FE:** もう一つの「DB統合」というのは？
**BE:** 今回はバックエンドに Prisma を使っているんですが、認可ルールをそのまま DB クエリ（Where句）に変換したかったんです。外部サービスだと、どうしても「データ同期」が必要になったり、フィルタリングが複雑になったりします。
**FE:** なるほど。それで、Node.js プロセス内で完結するライブラリベースのアプローチを選んだわけですね。

### スライド 6: Why CASL?
**The Choice: CASL**

1.  **Isomorphic:** UI/APIでロジックを共有できる
2.  **Type Safety:** TypeScript製で型推論が強力
3.  **API Integration:** 宣言的なAPI実装ができる
4.  **Database Integration:** 宣言的なクエリ発行ができる (`@casl/prisma`)
5.  **Frontend Integration:** 宣言的なコンポーネント実装ができる (`@casl/react`)

**(Talk Script)**
**BE:** そこで選んだのが **CASL** です。
**FE:** CASL、初めて聞く方もいるかもしれませんが、JS/TS界隈では結構老舗の認可ライブラリですよね。
**BE:** そうですね。CASLの何が良いって、一番は **Isomorphic** なところです。一度定義したルールを、Node.js（バックエンド）でもReact（フロントエンド）でも使い回せる。
**FE:** これです！ 私たちフロントエンドエンジニアからすると、バックエンドと同じルール定義ファイルを `import` して使うだけでいい。
**BE:** 二重管理の撲滅ですね。
**FE:** しかも TypeScript 製なので型推論が強力です。`can('update', ...)` って書くときに、第二引数に何が来るか補完が出る。`Candidate` なのか `User` なのか、型が教えてくれます。
**BE:** そして僕の推し機能は **Database Integration** です。CASLで定義したポリシーを、Prisma の Where 句として出力できる `@casl/prisma` という公式パッケージがあるんです。
**FE:** これが決め手でしたね。
**BE:** はい。これで「宣言的な認可」が可能になります。それでは、実際の実装を見ていきましょう。

## 3. 実装詳細：共通定義 (5分)

### スライド 7: Abilityの定義 (Types)
**Defining the "Ability"**

```typescript
// アクションとリソース（Subject）の定義
export type Action = 'manage' | 'create' | 'read' | 'update' | 'delete';

// PrismaのモデルをそのままSubjectとして利用
export type AppSubjects = Subjects<{ Candidate: Candidate }> | 'all';

export type AppAbility = PureAbility<[Action, AppSubjects], PrismaQuery>;
```

**(Talk Script)**
**BE:** 実装を見ていきましょう。まずは型定義です。CASLでは、何ができるかというルールセットを `Ability` というクラスで表現します。
**FE:** コードを見てください。`AppSubjects` という型定義がありますね。ここで `Subjects<{ Candidate: Candidate }>` と書いています。
**BE:** `Candidate` は Prisma Client から生成された型定義そのものです。これを CASL に渡すことで、「Candidate モデルに対する操作」であることを型システムに教えます。
**FE:** これのおかげで、リソース名のタイポ（例えば `Kandidate` とか書いちゃうとか）はコンパイルエラーになります。
**BE:** Action も `read`, `update` などリテラル型で定義します。これらを組み合わせて `AppAbility` 型を作ります。これが我々のアプリケーションにおける「認可の辞書」になります。

### スライド 8: ポリシーファクトリ (Logic)
**Policy Factory**

```typescript
export function defineAbilityFor(user: UserContext): AppAbility {
  const { can, cannot, build } = new AbilityBuilder(createPrismaAbility);

  if (user.role === 'Recruiter') {
    // "自部署" の候補者のみ操作可能
    can(['read', 'update'], 'Candidate', {
      departmentId: { in: user.departmentIds },
    });
  }
  
  // 面接官は個人情報を見れない
  if (user.role === 'Interviewer') {
    cannot('read', 'Candidate', ['salary', 'address']);
  }

  return build();
}
```

**(Talk Script)**
**BE:** そしてこれが認可ロジックの本体、ポリシーファクトリです。`defineAbilityFor` 関数を見てください。
**FE:** ユーザー情報（`UserContext`）を受け取って、そのユーザーができること（`Ability`）を生成して返す関数ですね。
**BE:** そうです。ここで `AbilityBuilder` を使ってルールを記述していきます。例えば真ん中の `if (user.role === 'Recruiter')` のブロック。
**FE:** 「採用担当者」の場合ですね。
**BE:** `can(['read', 'update'], 'Candidate', ...)` とあります。「Candidate を読んだり更新したりできる」という定義ですが、その第3引数に注目してください。
**FE:** `{ departmentId: { in: user.departmentIds } }` ...これ、Prisma のクエリそのものじゃないですか？
**BE:** 鋭いですね！ ここで Prisma の構文（MongoDB風のクエリ記法）を使って条件を書くことで、後でこの条件をそのまま DB クエリとして再利用できるんです。
**FE:** なるほど。ここで「自部署のIDに含まれるものだけ」という条件を定義しているわけですね。
**BE:** 下の方には `cannot('read', 'Candidate', ['salary', ...])` もあります。これがフィールドレベルの制御。「salary カラムは読めない」という定義です。
**FE:** これが一箇所にまとまっているのがいいですね。仕様変更があったら、このファイルだけ直せばいい。

## 4. 実装詳細：Backend Deep Dive (10分)

### スライド 9: APIでの利用 (Guard)
**NestJS Guard Implementation**

```typescript
@Get()
@UseGuards(PoliciesGuard)
@CheckPolicies((ability) => ability.can('read', 'Candidate'))
async findAll() {
  return this.candidatesService.findAll();
}
```

**(Talk Script)**
**BE:** では、定義した Ability をバックエンド（NestJS）でどう使うか見てみましょう。
**BE:** NestJS には Guard という仕組みがあります。ここで `PoliciesGuard` という自作のガードを挟んでいます。
**FE:** `@CheckPolicies` デコレータが見えますね。`(ability) => ability.can('read', 'Candidate')` と書いてある。
**BE:** はい。これは「この API を叩くには、Candidate を read する権限が必要だよ」と宣言しています。
**FE:** リクエストが来たら、まず Guard が実行されて、さっきの `defineAbilityFor` で生成された ability を使ってこの条件をチェックするわけですね。
**BE:** その通り。権限がなければ、コントローラーの処理が走る前に `403 Forbidden` が返されます。
**FE:** シンプルですね。でもこれだと、「Candidate一覧を見る権限があるか」という 0 か 1 かのチェックしかできないのでは？
**BE:** お、いい質問ですね。「権限はあるけど、全件じゃなくて自分の担当分だけ返したい」というケースですね。
**FE:** そうです。「自部署のデータだけ返す」はどうするんですか？ 全件取得してからメモリ上でフィルタリングするのは嫌ですよ？

### スライド 10: Prisma連携 (accessibleBy) - ここがポイント
**Magical "accessibleBy" helper**

```typescript
// Service層での実装
async findAll(ability: AppAbility) {
  return prisma.candidate.findMany({
    where: {
      AND: [
        // ⬇️ AbilityからWhere句を取得
        accessibleBy(ability, 'read').Candidate
      ]
    }
  });
}
```

**(Talk Script)**
**BE:** そこで登場するのが、今日のハイライト。`accessibleBy` ヘルパーです。
**BE:** スライドのコードを見てください。Prisma の `findMany` の `where` 句の中に、`accessibleBy(ability, 'read').Candidate` を入れています。
**FE:** これは何をしているんですか？
**BE:** さきほど `defineAbilityFor` で定義した「自部署のIDに含まれるものだけ」という条件がありましたよね？ あれが自動的に Prisma の `WhereInput` オブジェクトに変換されて、ここに注入されるんです。
**FE:** えっ、自動で？
**BE:** 自動です。つまり、プログラマは「フィルタリング条件」を意識して書く必要がないんです。「このユーザーが read できる Candidate」と指定するだけで、SQL レベルでフィルタリングがかかります。
**FE:** それはすごい。つまり、「うっかり `where` 句を書き忘れて、他部署のデータが見えちゃった」という事故が起きない？
**BE:** 起きません。「権限があるデータしか DB から返ってこない」ことが保証されるんです。
**FE:** セキュリティバイデザインですね。
**BE:** しかも、認可ロジックは Ability 定義ファイルに隔離されているので、API のビジネスロジックは純粋に仕様の実装に集中できます。
**FE:** バックエンドの実装がめちゃくちゃクリーンになりますね。

## 5. 実装詳細：Frontend Deep Dive (5分)

### スライド 11: UIでの利用 (Can Component)
**Declarative UI with React**

```tsx
export function CandidateCard({ candidate }) {
  // フックでAbilityを取得
  const ability = useAppAbility();

  return (
    <div className="card">
      <h1>{candidate.name}</h1>
      
      {/* 権限がある時だけボタンを表示 */}
      <Can I="update" this={candidate}>
        <button onClick={handleEdit}>編集</button>
      </Can>

      {/* 権限がない時はdisabled */}
      <button disabled={ability.cannot('delete', candidate)}>
        削除
      </button>
    </div>
  );
}
```

**(Talk Script)**
**FE:** さて、フロントエンド側も見てみましょう。フロントエンドでも全く同じ `Ability` 定義を使います。
**FE:** React では `<Can>` というコンポーネントが提供されていて、これが最高に便利です。
**BE:** `I="update" this={candidate}` って、英語の文章みたいで読みやすいですね。
**FE:** そうなんです。「私は（I）この候補者（this）を更新（update）できますか？」と問いかけるだけで、権限がある場合だけ `children` をレンダリングしてくれます。
**BE:** 直感的ですね。
**FE:** `if` 文を書かなくていいのが本当に楽です。それに、下のボタンの例のように `ability.cannot(...)` を使って `disabled` 属性を制御することもできます。
**BE:** 裏側のロジックはバックエンドと同じ定義ファイルを参照しているから、整合性もバッチリですね。
**FE:** はい。「画面にはボタンがあるのに押すとエラーになる」という悲しいすれ違いはもう起きません。

### スライド 12: フィールドレベル制御
**Field Level Authorization**

```tsx
// APIレスポンスでマスクされたデータ
// { name: "Taro", salary: undefined }

<Can I="read" this={candidate} field="salary">
  <p>年収: {candidate.salary}</p>
</Can>
<Else>
  <p>年収: ***** (閲覧権限がありません)</p>
</Else>
```

**(Talk Script)**
**FE:** さらに、L5 のフィールドレベル制御も可能です。
**BE:** さっき「評価担当者は年収を見れない」って定義しましたね。
**FE:** API からはそもそもデータが返ってこない（`undefined` になる）か、あるいはフロント側でマスク表示にするか。
**FE:** `<Can>` コンポーネントに `field="salary"` プロパティを渡すことで、「このリソースの、特にこのフィールドを読む権限があるか」を判定できます。
**BE:** 細かい！
**FE:** これで「年収: *****」みたいな表示切り替えも、コンポーネントレベルで宣言的に書けます。
**BE:** これも定義ファイル一箇所を変えれば、API のレスポンスフィルタリングと UI の表示制御の両方に反映されるのが強いですね。

## 6. まとめ (5分)

### スライド 13: メリット・デメリット
**Pros & Cons**

- **Good:**
    - 🔒 **堅牢性:** DBレベルで漏洩を防ぐ (accessibleBy)
    - 🤝 **一貫性:** BE/FEでロジックを共有 (Isomorphic)
    - 🚅 **開発速度:** 型補完と宣言的UI
- **Bad:**
    - 🤯 **型パズル:** 定義が複雑になりがち
    - 🐌 **複雑なクエリ:** Prisma変換できない複雑な条件もある

**(Talk Script)**
**BE:** 実際に導入してみて、正直な感想はどうでしたか？
**FE:** 正直に言うと、最初のセットアップ、特に **型定義** はちょっと大変でした。
**BE:** あー、ジェネリクスとかですね。
**FE:** そうです。Prisma の型と CASL の型を合わせるところで、ちょっとした型パズルになりました。でも、一度定義しちゃえば後は楽園でしたね。
**BE:** わかります。
**FE:** 開発者が増えても、「とりあえず `Can` コンポーネントで囲っといて」で通じるし、型補完が効くのでミスも減りました。
**BE:** バックエンドとしては、やはりセキュリティの担保がコードレベルで保証される安心感が絶大でした。
**FE:** 逆に困ったことは？
**BE:** `accessibleBy` は便利なんですが、あまりに複雑な条件、例えば「AテーブルとBテーブルをJoinして、その結果の...」みたいな深いネストがある条件だと、生成されるクエリが複雑になりすぎてインデックスが効かない、みたいな懸念はありました。
**FE:** 銀の弾丸ではないと。
**BE:** そうですね。そういう特殊なケースは、無理に CASL に押し込まず、サービス層で個別にクエリを書くという「逃げ道」も用意しておくのが現実的な運用かなと思います。

### スライド 14: Takeaways
**まとめ**

1.  BtoB SaaSの認可は複雑。早めにアーキテクチャを決めよう。
2.  データモデルと密結合するなら **CASL + Prisma** は最強の選択肢の一つ。
3.  **Isomorphic** な認可で、BE/FE の分断をなくそう。

**(Talk Script)**
**FE:** さて、お時間となりました。まとめです。
**BE:** BtoB SaaS の認可は、最初は簡単でも後から必ず複雑になります。`if` 文で戦うのはやめて、早めにアーキテクチャを整えましょう。
**FE:** その際、TypeScript でバックエンド・フロントエンドを統一しているなら、**CASL + Prisma** という選択肢は非常に強力です。
**BE:** 外部SaaSを使うのも手ですが、データモデルと密結合させたいなら、インプロセスなライブラリの方が開発体験が良い場合も多いです。
**FE:** 「Isomorphic な認可」で、BE と FE の分断をなくし、平和な開発ライフを送りましょう！
**BE:** 権限管理に悩むすべてのエンジニアに幸あれ！
**BE & FE:** ご清聴ありがとうございました！

---

