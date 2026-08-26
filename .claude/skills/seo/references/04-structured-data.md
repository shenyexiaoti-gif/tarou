# 04. 構造化データ（JSON-LD）

## 目次
1. 原則
2. 実装前の必須確認
3. コアセット（ほぼ全サイトで入れる）
4. 用途別のタイプ
5. JSON-LDテンプレート
6. 検証と監視
7. やってはいけないこと

---

## 1. 原則

- **形式はJSON-LD**（Googleの推奨）。`<script type="application/ld+json">` を `<head>` か `<body>` に配置
- **ページに表示されている内容だけをマークアップする**。非表示情報のマークアップはポリシー違反
- 最も具体的な型を選ぶ（`LocalBusiness` より `Restaurant`）
- 構造化データは**直接の順位上昇要因ではない**。効くのは①リッチリザルトによるCTR ②機械可読性（検索エンジンとAIがページの意味とエンティティ関係を理解する）
- **AI検索時代では②の価値が上がっている**（→ `06-ai-search-geo.md`）

## 2. 実装前の必須確認

**サポートされるリッチリザルトの種類は削減・変更が続いている。実装前に必ずGoogle公式の「structured data / rich results」一覧でその日の対応状況を確認する。**

2026年8月時点で把握している変化:
- **FAQリッチリザルトは2026年5月に廃止**され、リッチリザルトテストやレポートのサポートも外れた（＝FAQPageを入れても検索結果の見た目は変わらない）。ただし機械可読性目的で残す判断はあり得る。**「FAQを付ければリッチリザルトが出る」とは絶対に言わない**
- HowTo系も縮小の流れ
- 商品系（Product / Merchant listing）、Organization、BreadcrumbList、Article、Event、LocalBusiness、Reviewなどは引き続き主要

## 3. コアセット（ほぼ全サイトで入れる）

| 型 | 対象 | 目的 |
|---|---|---|
| `Organization` | サイト全体（トップ or 全ページ） | 運営主体のエンティティ確立。`sameAs` でSNS・公式プロフィールを接続 |
| `WebSite` | トップ | サイト名の明示 |
| `BreadcrumbList` | 全下層 | 階層の明示。検索結果のパンくず表示 |
| `Article` / `BlogPosting` | 記事 | 著者・公開日・更新日の明示 |
| `Person` | 著者ページ | E-E-A-Tの執筆者エンティティ |

## 4. 用途別のタイプ

- EC：`Product` ＋ `Offer` ＋ `AggregateRating` ＋ `Review`（価格・在庫・レビューの表示に必要）
- 実店舗：`LocalBusiness`（住所・営業時間・電話。NAP情報と完全一致させる → `05-offpage-local.md`）
- 求人：`JobPosting`
- イベント：`Event`
- 動画：`VideoObject`
- レシピ：`Recipe`
- ソフトウェア：`SoftwareApplication`
- Q&Aサイト：`QAPage`（ユーザー投稿型のQ&A向け。自社FAQとは別物）

## 5. JSON-LDテンプレート

**記事（Article）＋ 著者 ＋ 発行元**
```json
{
  "@context": "https://schema.org",
  "@type": "BlogPosting",
  "mainEntityOfPage": { "@type": "WebPage", "@id": "https://example.com/blog/seo-guide" },
  "headline": "記事タイトル（110文字以内推奨）",
  "description": "記事の要約",
  "image": ["https://example.com/img/hero-16x9.jpg"],
  "datePublished": "2026-08-26T09:00:00+09:00",
  "dateModified": "2026-08-26T09:00:00+09:00",
  "author": {
    "@type": "Person",
    "name": "著者名",
    "url": "https://example.com/author/taro",
    "jobTitle": "肩書",
    "sameAs": ["https://x.com/xxxx"]
  },
  "publisher": {
    "@type": "Organization",
    "name": "運営者名",
    "logo": { "@type": "ImageObject", "url": "https://example.com/logo.png" }
  }
}
```

**組織（Organization）**
```json
{
  "@context": "https://schema.org",
  "@type": "Organization",
  "name": "株式会社サンプル",
  "url": "https://example.com/",
  "logo": "https://example.com/logo.png",
  "description": "事業内容",
  "sameAs": [
    "https://x.com/example",
    "https://www.youtube.com/@example",
    "https://ja.wikipedia.org/wiki/xxxx"
  ],
  "contactPoint": {
    "@type": "ContactPoint",
    "telephone": "+81-3-0000-0000",
    "contactType": "customer support",
    "areaServed": "JP",
    "availableLanguage": ["Japanese"]
  }
}
```

**パンくず（BreadcrumbList）**
```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    { "@type": "ListItem", "position": 1, "name": "ホーム", "item": "https://example.com/" },
    { "@type": "ListItem", "position": 2, "name": "SEO", "item": "https://example.com/seo/" },
    { "@type": "ListItem", "position": 3, "name": "テクニカルSEO" }
  ]
}
```
（最後の要素に `item` を付けないのが現在のパターン。仕様は変わり得るので検証ツールで確認）

**商品（Product）**
```json
{
  "@context": "https://schema.org",
  "@type": "Product",
  "name": "商品名",
  "image": ["https://example.com/p/1.jpg"],
  "description": "商品説明",
  "sku": "SKU-001",
  "brand": { "@type": "Brand", "name": "ブランド名" },
  "offers": {
    "@type": "Offer",
    "url": "https://example.com/p/1",
    "priceCurrency": "JPY",
    "price": "12800",
    "availability": "https://schema.org/InStock",
    "itemCondition": "https://schema.org/NewCondition"
  },
  "aggregateRating": {
    "@type": "AggregateRating",
    "ratingValue": "4.5",
    "reviewCount": "128"
  }
}
```
※ `aggregateRating` は**実在するレビューがページに表示されている場合のみ**。自作自演の評価は重大な違反。

**実店舗（LocalBusiness）**
```json
{
  "@context": "https://schema.org",
  "@type": "LocalBusiness",
  "name": "店舗名",
  "image": "https://example.com/shop.jpg",
  "address": {
    "@type": "PostalAddress",
    "streetAddress": "○○1-2-3",
    "addressLocality": "渋谷区",
    "addressRegion": "東京都",
    "postalCode": "150-0001",
    "addressCountry": "JP"
  },
  "telephone": "+81-3-0000-0000",
  "openingHoursSpecification": [{
    "@type": "OpeningHoursSpecification",
    "dayOfWeek": ["Monday","Tuesday","Wednesday","Thursday","Friday"],
    "opens": "09:00", "closes": "18:00"
  }],
  "url": "https://example.com/"
}
```

**エンティティを繋ぐ（推奨）**：`@id` を使って Organization / Person / WebSite / Article を相互参照すると、検索エンジンとAIがエンティティ関係を追いやすくなる。
```json
"author": { "@id": "https://example.com/author/taro#person" }
```

## 6. 検証と監視

1. Googleの**リッチリザルトテスト**（対応タイプの検出）
2. **Schema Markup Validator**（schema.org準拠の構文チェック。Google非対応タイプもここで検証）
3. **GSCの拡張レポート**でエラー・警告を継続監視
4. テンプレート実装後は**必ず数ページ実データで確認**（動的生成は値の欠損・型崩れが起きやすい）

## 7. やってはいけないこと

- ページに無い情報のマークアップ（非表示のFAQ、存在しないレビュー）
- 自作・依頼で作った評価のマークアップ
- 無関係な型の乱用（全ページに `Product` を付けるなど）
- 型と実体の不一致（記事ページに `LocalBusiness`）
- 構造化データだけ更新して本文を放置（`dateModified` の詐称は信頼を落とす）

構造化データのスパムは**手動対策（リッチリザルトのスパム）の対象**になる（→ `08-spam-penalty.md`）。
