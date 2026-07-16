---
title: "AI/ML機能の実装パターン"
free: true
---

# AI/ML機能の実装パターン

## AI/ML機能の全体マップ

```
ATS における AI/ML 機能
├── 1. レジュメパーシング（情報抽出）
├── 2. 候補者スコアリング（マッチング）
├── 3. 候補者推薦（レコメンデーション）
├── 4. 求人票最適化（JD生成・改善）
├── 5. 面接質問生成
└── 6. バイアス検出・DE&I分析
```

---

## 1. レジュメパーシング

### 概要

レジュメPDF/DOCXから構造化データを抽出するプロセス（出典: [CrawlJobs - How ATS Work](https://www.crawljobs.com/blog/how-applicant-tracking-systems-work)）。

```
入力: 非構造化テキスト（PDF/DOCX/HTML）
  ↓
OCR（PDFスキャンの場合）
  ↓
テキスト正規化
  ↓
NLP処理
  ↓
出力: 構造化JSON
{
  "name": "Taro Yamada",
  "email": "taro@example.com",
  "experience": [
    {"company": "ABC Inc", "title": "SWE", "start": "2020-04", "end": "2023-03"}
  ],
  "skills": ["Python", "AWS", "PostgreSQL"],
  "education": [{"degree": "B.S.", "field": "CS", "school": "東京大学"}]
}
```

### 実装技術スタック

| フェーズ | 技術 |
|---------|------|
| PDF解析 | PDFMiner, PyMuPDF, Apache Tika |
| テキスト抽出 | OCR (Tesseract, AWS Textract) |
| セクション分割 | ルールベース or 分類モデル |
| NER（固有表現認識） | spaCy, Hugging Face NER |
| スキル抽出 | スキル辞書 + NERの組み合わせ |
| 機械学習モデル | BERT, RoBERTa（ファインチューニング） |
| LLM活用（最新） | GPT-4, Gemini による汎用パーシング |

### パーシング精度の課題

- **多様なフォーマット**: テンプレートが統一されていない
- **日本語対応**: MeCab/SudachiによるJapanese NLP処理が必要
- **スキル表記のゆらぎ**: "Python" / "Python 3" / "Py" の正規化
- **日付形式**: "2020/4〜" / "Apr 2020" / "4月2020年" の統一

---

## 2. 候補者スコアリング

### 加重スコアリングモデル

従来型の基本スコアリングモデル（出典: [CrawlJobs](https://www.crawljobs.com/blog/how-applicant-tracking-systems-work)）：

```
総合スコア =
  スキルマッチ    × 40%
  + 経験年数     × 20%
  + 職歴類似性   × 20%
  + 学歴         × 10%
  + 勤務地一致   × 10%
```

### TF-IDF ベースのキーワードマッチ

```python
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics.pairwise import cosine_similarity

def score_resume(job_description: str, resume_text: str) -> float:
    vectorizer = TfidfVectorizer(stop_words='english')
    tfidf_matrix = vectorizer.fit_transform([job_description, resume_text])
    score = cosine_similarity(tfidf_matrix[0:1], tfidf_matrix[1:2])[0][0]
    return float(score)
```

### セマンティック類似度（Embedding ベース）

Resume2Vecなどの研究では、BERTやRoBERTa等のTransformerモデルで生成したEmbeddingを使い、コサイン類似度でレジュメとJDをマッチングする（出典: [Resume2Vec - MDPI Electronics](https://www.mdpi.com/2079-9292/14/4/794)）。

```python
from sentence_transformers import SentenceTransformer
import numpy as np

model = SentenceTransformer('all-MiniLM-L6-v2')

def semantic_score(job_desc: str, resume: str) -> float:
    embeddings = model.encode([job_desc, resume])
    similarity = np.dot(embeddings[0], embeddings[1]) / (
        np.linalg.norm(embeddings[0]) * np.linalg.norm(embeddings[1])
    )
    return float(similarity)
```

### 分類モデル（Fit/Close-Fit/Not-Fit）

AIスクリーニングシステムでは候補者を3分類に仕分けする（出典: [Medium - AI for Resume Screening](https://medium.com/@gracyjaiswal2003/ai-for-resume-screening-building-an-ats-friendly-resume-ranking-system-using-nlp-ml-streamlit-6887af6690e0)）。

```
Fit（適合）
  → 全必須スキル充足 + 経験年数充足

Close Fit（準適合）
  → 必須スキル70%以上 OR 関連経験あり

Not Fit（不適合）
  → 必須スキル不足 OR 最低経験年数未達
```

---

## 3. 候補者推薦システム

### 協調フィルタリング

過去の採用実績データを使い、採用につながりやすい候補者プロファイルを学習する。

```
学習データ:
  (候補者特徴ベクトル, 採用結果) のペア

推薦ロジック:
  過去に採用された候補者の特徴に近い未評価の候補者を優先表示
```

### Content-Based Filtering

JDの要件ベクトルと候補者プロファイルベクトルのコサイン類似度に基づく推薦。

### 実装アーキテクチャ

```
オフライン処理:
  候補者DB → Embedding生成（バッチ処理）→ Vector DB格納
  (Pinecone / pgvector / Weaviate)

オンライン処理:
  Job作成/更新
    → JD Embedding生成
    → Vector DB近傍検索（ANN: Approximate Nearest Neighbor）
    → Top-K候補者を採用担当者にサジェスト
```

---

## 4. 求人票（JD）生成・最適化

生成AIを活用した機能。各ATSが2023〜2024年から実装を進めている。

```
入力:
  - 職種名
  - 部門・チーム情報
  - 必須スキル
  - 給与レンジ

出力（LLM生成）:
  - 職務概要
  - 責任範囲
  - 必要スキル
  - 待遇・福利厚生

品質チェック:
  - バイアスワード検出（"ninja"/"rockstar" → より中立的な表現へ）
  - 性別中立化
  - 読みやすさスコア（Flesch-Kincaid）
```

---

## 5. バイアス検出・DE&Iコンプライアンス

### EEOC（雇用機会均等）データの設計

EEOC関連データ（人種・性別・障害等）は応募書類とは完全に分離して保存する設計が必要。

```sql
-- EEOCデータは別テーブル（アクセス権限も分離）
CREATE TABLE eeoc_data (
  id UUID PRIMARY KEY,
  application_id UUID REFERENCES applications(id),
  gender VARCHAR,
  race VARCHAR,
  veteran_status VARCHAR,
  disability_status VARCHAR,
  -- NOTE: 採用選考に使用禁止。統計分析専用
  CONSTRAINT not_used_in_screening CHECK (1=1) -- アプリ層で制御
);
```

### AI採用バイアスのリスク

2018年のAmazonのAI採用ツール廃止事例（女性候補者に低スコアを付けるバイアスが判明）を受け、モデルの公平性（Fairness）評価が重要視されるようになった（推定: 一般に知られた事例）。

対策:
- 保護属性（性別・人種等）をモデルの特徴量から除外
- 公平性指標（Equal Opportunity / Demographic Parity）の定期評価
- ヒューマンレビュー（Human in the Loop）の義務化
