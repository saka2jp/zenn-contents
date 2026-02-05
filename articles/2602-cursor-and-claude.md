---
title: "CursorヘビーユーザーがClaude Codeを主軸に据えた理由"
emoji: "🐷"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["cursor", "claudecode", "ai", "anthropic", "cli"]
published: false
---

## はじめに

私はこれまで、AI搭載エディタとしてCursorをメインに使ってきました。2025年初頭からBusinessプランで毎日使い続け、約1年間の実務経験があります。

本記事では、そんな私がClaude Codeを開発の主軸に据えるようになった理由と、両ツールの使い分けについて解説します。なお、完全な「乗り換え」ではなく、Claude Codeを中心としつつCursorも併用するスタイルに落ち着いています。

## 1. Cursorの強み

CursorはVS Codeをベースにしており、普段使い慣れている開発体験を維持しながら、高性能なAIアシスタントを統合できる点が魅力です。

特に活用していた場面：

* 既存コードの理解や仕様確認の補助
* バグの原因調査や修正提案
* プロトタイプの最終調整
* 特定箇所のクイック修正

### 1-1. 「エディタ内で完結する」速度感

Cursorは、読んでいるファイル・開いている差分・参照している型など、目の前の文脈を前提に会話が進むため、ちょっとした修正や確認が速いです。「その場で聞いて、その場で反映して、その場で直す」が成立します。

結果として、小さな意思決定の回数が多いフロントエンド開発で特に相性が良いと感じました。

### 1-2. AIがブラウザを操作する「Cursor Browser」

2025年に登場した **[Cursor Browser](https://docs.cursor.com/agent/browser)** は特筆すべき進化でした。AIエージェントがWebブラウザを直接操作する機能で、スクリーンショット・コンソールログ・ネットワーク通信まで参照しながらテストやデバッグを進められます。

主な機能：

* **アプリの動作確認**：フォーム入力、画面遷移、レスポンシブ確認、エラーメッセージ検証
* **デバッグ**：コンソールログやネットワークトラフィックを参照して原因切り分け
* **アクセシビリティ監査**：コントラスト、セマンティックHTML、ARIA、キーボード操作のチェック
* **デザイン→コード**：画面やデザインを解析して、HTML/CSSの実装に落とし込む

特に強力なのが「見た目を触ってコードに反映」できるDesign sidebarです。ページ要素のレイアウト・サイズ・色・角丸・影などをGUI的に調整でき、調整後に「適用」することでコード変更に変換されます。

また、Cookie・localStorage・IndexedDBなどがワークスペース単位で保持されるため、ログイン状態を保ったままテストしやすい設計になっています。

### 1-3. Cursorの課題

一方で、以下のような課題も感じていました：

* **コンテキストの肥大化**：長いセッションで会話が続くと、不要な情報で文脈が圧迫され、応答品質が低下することがある
* **並列処理の限界**：複数の調査や作業を同時に進めたい場面で、1つの会話で逐次処理するしかない
* **役割の固定化**：汎用的なAIアシスタントとしては優秀だが、タスクごとに専門化した振る舞いを定義しにくい

---

## 2. Claude Codeに主軸を移した理由

Cursorに不満があったわけではありません。むしろ完成度は高いです。それでもClaude Codeに惹かれた理由は、**Skills**と**Subagents**という2つの仕組みにあります。

Skillsは「AIに専門知識やツールを教え込む」機能、Subagentsは「AIを役割分担されたチームとして動かす」機能です。この2つを組み合わせることで、単発の回答者ではなく「自分専用の熟練エンジニアチーム」を構築できる点に、大きな可能性を感じました。

### 2-1. Skillsとは何か

[Skills](https://docs.anthropic.com/en/docs/claude-code/skills)は、Claudeに新しい専門スキルを教え込むための仕組みです。具体的には「指示書（インストラクション）」「実行スクリプト」「参考リソース」をひとつのフォルダにまとめたパッケージで、特定のタスクに関するノウハウやツールをClaudeに与えます。

まるでAIに業務マニュアルやツール一式を渡すようなイメージで、Claudeをあなた専用の熟練エンジニア同僚のように振る舞わせることができます。

**段階的開示（Progressive Disclosure）による効率化**

Skillsの仕組みで重要なのは「段階的開示」です。Claudeは起動時に全Skillsの「名前」と「説明」だけをシステムプロンプトに読み込みます（各スキル数十〜百トークン程度）。そしてユーザーからの指示内容を見て、該当しそうなスキルを判断すると初めてそのスキルの詳細（SKILL.mdの内容）を読み込みます。

このように「必要な情報を必要な時にだけ読む」ため、多数のスキルを登録しても普段の対話には負荷がかからず、しかし必要な場面では豊富な知識やツールを即座に利用できます。

**Skillsで得られる主なメリット**

* **繰り返し説明が不要**：従来は毎回プロンプトで教えていた書式・手順も、一度Skillとして登録すれば以後のセッションで繰り返し説明する必要がなくなります
* **暗黙知の形式知化**：チーム内のノウハウや独自ルールをスキルとして定義し、メンバー間で共有できます
* **コード実行との連携**：スキル内にPythonなどのスクリプトを含めておけば、Claudeがそれを実行して得た結果を回答に反映できます

**ソフトウェア開発における主なユースケース**

| ユースケース | 説明 |
|------------|------|
| **コードリファクタリング** | ファイルが一定行数を超えた時に自動でリファクタ提案を行う |
| **テスト生成・QA自動化** | ペアワイズ法でテストケースを生成、Playwrightと連携したUIテスト実行 |
| **ドキュメント生成** | 社内の用語集や記法ガイドラインを組み込み、常に標準に沿ったドキュメントを生成 |
| **コードレビュー・規約チェック** | コーディング規約やスタイルガイドをスキル化し、規約違反を自動で指摘 |
| **プロジェクト管理** | LinearやJiraの運用手順をスキル化し、「バグをログして」で必要項目を埋めたチケットを作成 |

### 2-2. カスタムSkillの作成

開発者は自分のニーズに合わせて新たなSkillを定義できます。基本的な作成手順は以下のとおりです。

1. **スキル用ディレクトリを作成**：`~/.claude/skills/my-skill/`のようにフォルダを用意
2. **SKILL.mdファイルを作成**：YAMLフロントマターでメタデータを記述し、続いてMarkdown本文に指示を記述
3. **必要に応じて追加ファイルを配置**：`scripts/`（実行コード）、`references/`（参照資料）など

以下は「プロジェクトメモリ」用Skill定義の例です：

```markdown
---
name: project-memory
description: Track project decisions and learnings. Use when asked to "remember this", "log a decision", "set up project memory", or "track our decisions".
---
You help maintain a project memory system that tracks important decisions, learnings, and context.

## Memory File Location
Store all memories in `.claude/memory/` directory.

## Memory Types
1. **decisions.md** - Architecture and design decisions
2. **learnings.md** - Bug fixes and lessons learned
3. **context.md** - Project-specific context and conventions

## When Adding Memories
- Use consistent date format: YYYY-MM-DD
- Include context about why, not just what
- Link to relevant files or PRs when applicable
```

このファイルを配置すると、`description`にマッチするタスク（「この決定を記録して」「プロジェクトメモリをセットアップして」等）で自動的にスキルが読み込まれます。

**プロンプト設計のポイント**

* **説明文（description）の重要性**：ユーザーが発しそうな具体的フレーズを3〜5種類は盛り込むと、多少言い回しが変わってもClaudeがスキルを関連付けやすくなります
* **既知情報を繰り返さない**：モデルがすでに知っている一般知識は冗長に書かず、プロジェクト固有の指針や手順にフォーカスします
* **長大な指示は分割**：膨大になる場合は複数スキルに分けるか、セクションに分けて必要に応じて読む仕組みにします

### 2-3. Subagentsとは何か

[Subagents](https://docs.anthropic.com/en/docs/claude-code/sub-agents)は、特定のタスクに特化した「AIアシスタント」を複数持たせる設計思想に基づいています。単一の汎用エージェントにすべてを任せるのではなく、各Subagentに専門分野を持たせることで、より的確な支援と効率化を図ります。

この設計により得られる主なメリット：

**文脈の分離と維持**
各Subagentは独立したコンテキストウィンドウ（対話履歴）を持ち、メインの開発対話とは別個に動作します。大量のログや検索結果を扱っても、不要な情報でメインの会話コンテキストが圧迫されるのを防げます。必要な要約や結果だけをメイン会話に返すことで、長いセッションでも応答品質の劣化を抑え、トークン消費も節約できます。

**タスク固有の最適化**
Subagentごとにシステムプロンプト（人格・方針）を設定でき、専門領域に特化した指示を与えられます。コードレビュー専用のチェックリストや、デバッグ専用の原因究明手順を持たせることで、汎用モデルより精度の高い助言が可能になります。

**ツールや権限の限定**
各Subagentには使用可能なツールやファイル操作権限を個別に設定できます。読み取り専用のエージェントや、危険な操作をしないよう制限付きのエージェントを用いることで、安全性・信頼性を高めます。

**並行処理と分業**
複数のSubagentを並行起動することで、複数の処理を同時に実行できます。大規模なコードベースの横断的調査や、複数モジュールの同時検証で威力を発揮します。

### 2-4. 組み込みSubagentの活用

Claude Codeにはあらかじめいくつかの組み込みSubagentが用意されており、状況に応じて自動的に呼び出されます。

| Subagent | 役割 | 特徴 |
|----------|------|------|
| **Explore** | コードベースの検索や構造把握 | 軽量モデル（Haiku）使用、読み取り専用 |
| **Plan** | プランモード時のリサーチ | 読み取り専用、コード変更前の分析に特化 |
| **General-Purpose** | 複雑なタスクの処理 | 読み取り・編集両方可能、多段階の推論に対応 |

例えば、「このプロジェクトの認証フローを調べて」と指示すると、ExploreエージェントがHaikuモデルで高速に検索・分析を行い、結果だけをメイン会話にフィードバックします。

### 2-5. カスタムSubagentの作成

開発者は自分のニーズに合わせて新たなSubagentを定義できます。SubagentはMarkdownファイルで定義され、YAMLフロントマターで基本設定を記述し、続いて振る舞いを指示するシステムメッセージを記述します。

以下は公式ドキュメントに掲載されている「コードレビュア」用Subagent定義の例です：

```markdown
---
name: code-reviewer
description: Expert code review specialist. Proactively reviews code for quality, security, and maintainability. Use immediately after writing or modifying code.
tools: Read, Grep, Glob, Bash
model: inherit
---
You are a senior code reviewer ensuring high standards of code quality and security.

When invoked:
1. Run git diff to see recent changes
2. Focus on modified files
3. Begin review immediately

Review checklist:
- Code is clear and readable
- Functions and variables are well-named
- No duplicated code
- Proper error handling
- No exposed secrets or API keys
- Input validation implemented
- Good test coverage
- Performance considerations addressed

Provide feedback organized by priority:
- Critical issues (must fix)
- Warnings (should fix)
- Suggestions (consider improving)
```

このファイルを`.claude/agents/code-reviewer.md`に配置すると、`description`にマッチするタスク（コードレビュー、変更箇所のチェック等）で自動的に起動されるようになります。

### 2-6. 実践例：9つのSubagentによる並列コードレビュー

ある開発者は、9種類のSubagentを並行起動してコードレビューを行う仕組みを構築しています。各Agentにそれぞれ異なるレビュー視点を担当させています：

1. **テスト実行チェッカー**：変更箇所に関連するテストを走らせ、すべて通るか確認
2. **リンタ／静的解析**：lintersや型チェックを実行し、スタイル違反や型エラーを検出
3. **一般コードレビュー**：非自明な改善点を最大5つまで提案、影響度と修正コストで優先度付け
4. **セキュリティレビュー**：SQLインジェクションや認可漏れ、秘密情報の露出がないか精査
5. **品質・スタイル**：複雑度や重複、プロジェクトのコーディング規約から外れた部分をチェック
6. **テスト網羅性**：テストカバレッジやテスト戦略の妥当性、フレークのリスクを評価
7. **パフォーマンス**：N+1クエリやボトルネックになりそうな処理、メモリリークを検出
8. **依存関係とデプロイ安全性**：新規ライブラリ導入の影響や移行手順、設定ミスを確認
9. **簡素化提案**：設計が過剰に複雑でないか、変更が単一責務に収まっているかを検証

各Subagentが並行してレビューを行い、その結果をClaude（メイン）が集約・要約します。最終的に「マージして問題ない」「注意が必要」「修正が必要」といった総合判定まで自動で提示されます。

### 2-7. 実践例：7体のSubagentによるドキュメント生成パイプライン

Rick Hightower氏は、7体のSubagentから成るドキュメント生成パイプラインを構築しています：

1. **オーケストレータ**：他のエージェントを順序だてて起動し、全体の進行・エラーハンドリングを管理
2. **Mermaid抽出エージェント**：Markdown文書からMermaid記法のダイアグラムを検出して抽出
3. **画像生成エージェント**：Mermaid CLIを用いてダイアグラムをPNG画像化
4. **Markdown再構成エージェント**：Mermaidコードブロックを画像リンクに差し替え
5. **Pandoc変換エージェント**：MarkdownをPDFやWordに変換
6. **UTF-8サニタイズエージェント**：文字エンコーディングエラー発生時のみ起動
7. **ファイル整理エージェント**：生成物を所定のディレクトリに分類配置

実際にこのパイプラインを動かした結果、4つのMarkdownから8つの図を抽出・画像化し、5つのPDFと4つのWord文書を自動生成、UTF-8エラー27箇所をすべて修正、**初回でエラー無く完遂、所要時間約15分、人手による介入ゼロ**という成果が得られています。

### 2-8. SkillsとSubagentsの使い分け

SkillsとSubagentsは補完的な関係にあり、用途に応じて使い分けることで効果を最大化できます。

| 観点 | Skills | Subagents |
|------|--------|-----------|
| **主な目的** | 知識・手順・ツールの拡張 | タスクの並列処理・分業 |
| **動作タイミング** | 関連キーワードで自動発動 | 複雑なタスクで自動起動 |
| **コンテキスト** | メインの対話に情報を追加 | 独立したコンテキストで実行 |
| **適したタスク** | 繰り返し発生する定型作業、規約チェック | 大規模調査、複数ファイルの同時処理 |

**Skillsが向いている場面**

* プロジェクト固有のコーディング規約やスタイルガイドの適用
* 毎回同じフォーマットで出力が必要なドキュメント生成
* チーム共通の手順書やテンプレートの適用
* 特定ツール（GitHub、Linear等）との連携手順の定義

**Subagentsが向いている場面**

* 複数ファイルにまたがる大規模な調査・分析
* 並列実行したいコードレビューやテスト
* メインの対話コンテキストを汚したくない重い処理
* 複数の専門家視点が必要なタスク

**組み合わせの例**

実践的には両者を組み合わせて使うことも効果的です。例えば、コードレビュー用のSkillでチームの規約を定義しておき、それを9つのSubagentに読み込ませて並列レビューを実行する、といった構成が考えられます。

### 2-9. 結果として「AIの使い分け」から「Claude中心」へ

最初は「日常はCursor、重い設計はClaude」という使い分けをしていました。しかし、SkillsとSubagentsの発想（AIに専門知識を与え、役割分担して働かせる）に慣れてくると、作業の中心が自然とClaude Codeに寄っていきました。

Cursorの強みは依然として「エディタ内での速さ・手触りの良さ」です。ただ、私が最終的に欲しかったのは、単発の補完ではなく「企画や設計も含めて、AIと一緒に前に進む開発体験」でした。SkillsとSubagentsによる拡張性の高さが、そこに最も近い道具だと感じています。

---

## 3. Claude Codeを快適に使うための環境構築

Claude CodeはCLIベースのツールです。そのため、ターミナル環境の整備が快適な開発体験の鍵になります。

### 3-1. 高速ターミナル：Ghostty

[Ghostty](https://ghostty.org/)は、Zig言語で書かれた高速・軽量な次世代ターミナルです。

```bash
# macOSでのインストール
brew install ghostty
```

Ghosttyの設定例（`~/.config/ghostty/config`）：

```ini
font-family = "JetBrains Mono"
font-size = 14
theme = catppuccin-mocha
window-padding-x = 10
window-padding-y = 10
```

### 3-2. セッション管理：tmux

[tmux](https://github.com/tmux/tmux)は、ターミナルセッションの維持やレイアウト管理に使用します。

```bash
# インストール
brew install tmux
```

基本的な設定例（`~/.tmux.conf`）：

```bash
# プレフィックスキーをCtrl+aに変更
set -g prefix C-a
unbind C-b
bind C-a send-prefix

# ペイン分割のキーバインド
bind | split-window -h
bind - split-window -v

# マウス操作を有効化
set -g mouse on

# 256色対応
set -g default-terminal "screen-256color"
```

Claude Codeを使う際の典型的なレイアウト：

```
+------------------+------------------+
|                  |                  |
|   Claude Code    |    lazygit       |
|                  |                  |
+------------------+------------------+
|                                     |
|          ファイル確認用              |
|                                     |
+-------------------------------------+
```

### 3-3. 並列作業：Git Worktree

[Git Worktree](https://git-scm.com/docs/git-worktree)は、1つのリポジトリで複数のブランチを同時に作業ディレクトリとして展開できる機能です。

```bash
# worktreeの作成
git worktree add ../my-project-feature feature-branch

# worktreeの一覧
git worktree list

# worktreeの削除
git worktree remove ../my-project-feature
```

Claude Codeは基本的に1セッション=1プロジェクトなので、git worktreeで分岐させた作業ディレクトリごとに別のセッションを立ち上げることで、「一人ペアプロ」的な開発スタイルが取れます。

例えば、以下のような使い分けが可能です：

```
~/projects/my-app/           # mainブランチ（安定版確認用）
~/projects/my-app-feature/   # featureブランチ（新機能開発用）
~/projects/my-app-bugfix/    # bugfixブランチ（バグ修正用）
```

それぞれのディレクトリでClaude Codeセッションを起動し、並行して作業を進められます。

---

## 4. CursorとClaude Codeの使い分け

現時点での私のスタイルは「Claude Code中心、Cursor併用」です。

### Cursorを使う場面

* **軽微な修正**：数行レベルの変更、タイポ修正
* **UI実装のライブプレビュー**：Cursor Browserでの動作確認
* **コード補完重視の作業**：IDEとしての補完・ナビゲーション機能が必要な場面

### Claude Codeを使う場面

* **構成設計・初期設計**：アーキテクチャの検討、ディレクトリ構成の決定
* **深いバグ調査・原因分析**：複数ファイルにまたがる調査
* **複数ファイルにまたがるリファクタ**：大規模な変更
* **複数タスクを並行処理したいとき**：Subagentによるセッション分離

---

## 5. Claude Codeの課題

公平を期すため、Claude Codeの課題も挙げておきます：

* **CLI操作の学習コスト**：ターミナルに不慣れな開発者にはハードルがある
* **IDEとの統合がない**：コード補完、シンタックスハイライト、定義ジャンプなどはエディタ側で別途必要
* **ブラウザ操作機能がない**：Cursor Browserのような統合されたブラウザ操作機能は現時点でない
* **コスト予測のしにくさ**：API直接利用のため、使用量に応じた従量課金となり、月額固定のCursorより予測しにくい

これらの理由から、完全な移行ではなく「併用」という結論に至っています。

---

## 6. Anthropicの思想への共感

Claudeを開発するAnthropic社は、[Constitutional AI（憲法的AI）](https://www.anthropic.com/constitution)に基づいてAIを設計しています。これは、AIの訓練時に守るべき原則（憲法）を与え、その原則に照らして出力を評価・改善していく手法です。

また、同社が[Public Benefit Corporation](https://www.anthropic.com/news/the-long-term-benefit-trust)という形態をとり、商業的利益より社会的意義を優先する姿勢にも共感しています。

---

## 7. まとめ

私は現在、Claude Codeを主軸に据えた開発環境で作業しています。ただしCursorも日常的に活用しており、用途に応じた使い分けによって、以前よりも柔軟な開発ができています。

特にSkillsとSubagentsの仕組みは、AIを「賢い回答者」から「専門知識を持った役割分担チーム」に変える発想の転換です。Skillsでプロジェクト固有のノウハウを教え込み、Subagentsで並列処理や分業を実現することで、大規模な開発や複雑なタスクで威力を発揮します。

ツール単体の比較にとどまらず、「自分の開発スタイルをAIとどう統合していくか」を考えるフェーズに入ってきたと実感しています。

## 参考資料

* [Extend Claude with skills - Claude Code Docs](https://docs.anthropic.com/en/docs/claude-code/skills)
* [Equipping agents for the real world with Agent Skills - Anthropic](https://www.anthropic.com/news/agent-skills)
* [Create custom subagents - Claude Code Docs](https://docs.anthropic.com/en/docs/claude-code/sub-agents)
* [How we built our multi-agent research system - Anthropic](https://www.anthropic.com/engineering/multi-agent-research-system)
* [Claude Code Sub-Agents: Build a Documentation Pipeline in Minutes - Rick Hightower](https://pub.spillwave.com/claude-code-sub-agents-build-a-documentation-pipeline-in-minutes-not-weeks-c0f8f943d1d5)
* [9 Parallel AI Agents That Review My Code - hamy.xyz](https://hamy.xyz/blog/2026-02_code-reviews-claude-subagents)
* [Cursor Browser Documentation](https://docs.cursor.com/agent/browser)
