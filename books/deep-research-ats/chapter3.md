---
title: "歴史的背景と年表"
free: true
---

# 歴史的背景と年表

## ATSが生まれた背景

### 1990年代の課題

インターネット普及前の採用は「履歴書の郵送 → 電話スクリーニング → 面接」という完全アナログプロセスだった。インターネットの登場により応募者数が爆発的に増加したことで、従来の手作業管理が限界を迎えた。

### 台頭を後押しした技術・市場要因

| 要因 | 内容 |
|------|------|
| インターネット求人の普及 | Monster.com（1994年）、HotJobs等の登場で応募量が急増 |
| リレーショナルDBの成熟 | 大量候補者データの構造化管理が可能に |
| クラウドコンピューティング | SaaSモデルによる初期コスト削減 |
| スマートフォン普及 | モバイル応募・採用担当者のモバイル操作ニーズ |
| AI/ML技術の民主化 | レジュメパーシング・スコアリングの自動化 |

## 年表

| 年 | 出来事 |
|----|------|
| **1994** | Monster.comの前身「The Monster Board」開設。オンライン求人の黎明期 |
| **1990年代後半** | 初期のATS製品（Resumix等）がエンタープライズ向けに登場 |
| **1999** | Oracle TaleoのベースとなるTaleo創業 |
| **2000年代前半** | SaaS型ATSへの移行開始。PeopleSoft等のオンプレ製品が主流 |
| **2007** | Greenhouse Softwareの前身となる取り組みが始まる時期 |
| **2009** | Lever創業（サンフランシスコ） |
| **2010** | Greenhouse Software設立 |
| **2012** | Oracle、Taleo社を19億ドルで買収（出典: 推定/一般情報） |
| **2012** | SAP、SuccessFactorsを34億ドルで買収 |
| **2016頃** | AI/MLベースのレジュメパーシング・候補者スクリーニングが台頭 |
| **2018** | GDPR施行。EU候補者データ管理要件が厳格化 |
| **2019** | LinkedIn、Recruiter System Connect（RSC）を拡張。ATS連携深化 |
| **2020** | COVID-19によるリモート採用急拡大。ATSのビデオ面接連携が標準化 |
| **2022** | Greenhouse Harvest API v3リリース |
| **2023〜** | ChatGPT等の生成AIを活用した求人票作成・候補者評価AI機能が各社で展開 |
| **2025年9月** | SAPがSmartRecruitersを買収。エンタープライズATSの再編が加速 |
| **2026年8月** | Greenhouse Harvest API v1/v2の廃止予定（出典: [Greenhouse Support](https://support.greenhouse.io/hc/en-us/articles/10568627186203-Greenhouse-API-overview)） |

## Lever固有の歴史的経緯

LeverはCandidateエンティティを2020年に廃止し、**Opportunity**（候補者×職種の組み合わせ）を主要エンティティに一本化した。この設計は「採用CRM」思想を反映しており、1人の候補者が複数のポジションに紐づく構造を自然に表現できる（出典: [Lever Developer Documentation](https://hire.lever.co/developer/documentation)）。
