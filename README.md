# tarou — Claude Code スキル置き場

太郎課長の発信・実務を支えるスキルを管理するリポジトリ。

## 収録スキル

| スキル | 内容 |
|---|---|
| [`seo`](.claude/skills/seo/SKILL.md) | SEO実務OS。キーワード戦略／コンテンツ設計とE-E-A-T／テクニカルSEO／構造化データ／被リンク・デジタルPR／AI検索最適化（AI Overviews・GEO・LLMO）／計測とKPI／スパムポリシーと順位下落の復旧 |

## seo スキルの構成

```
.claude/skills/seo/
├── SKILL.md                          # 地図。ルーティング表・大前提・鮮度ポリシー・ワークフロー
└── references/
    ├── 01-strategy-keyword.md        # キーワード戦略・検索意図8分類・トピッククラスタ・SERP分析
    ├── 02-content-eeat.md            # 記事構造・E-E-A-T実装・Information Gain・リライト手順・棚卸し
    ├── 03-technical.md               # クロール／インデックス／レンダリング／内部リンク／Core Web Vitals
    ├── 04-structured-data.md         # JSON-LDテンプレート集と検証・禁止事項
    ├── 05-offpage-local.md           # 被リンク・デジタルPR・サイテーション・ローカル・日本市場
    ├── 06-ai-search-geo.md           # AI検索の動作モデル・引用される5本柱・llms.txt・プラットフォーム別
    ├── 07-measurement.md             # KPI階層・GSC／GA4・AI引用の測定・レポートの型・効果判定
    ├── 08-spam-penalty.md            # 下落の切り分けフロー・手動対策・スパムポリシー・復旧
    └── 09-checklists.md              # 公開前／監査／リライト／緊急対応／移行／月次ルーチン
```

## 使い方

このリポジトリを開いた Claude Code のセッションでは自動的に読み込まれる。
「SEO」「順位が落ちた」「キーワード」「AIに引用されたい」などの相談で発動する。

### 全プロジェクトで使えるようにする（任意）

手元のマシンで以下を実行すると、リポジトリ外でも使えるようになる。

```bash
mkdir -p ~/.claude/skills
cp -r .claude/skills/seo ~/.claude/skills/seo
```

Claude Code を再起動すると `/seo` として認識される。

## 運用上の注意

SEOは仕様変更が速い。`SKILL.md` の「鮮度ポリシー」に該当する項目
（アップデート状況、対応するリッチリザルトの種類、Core Web Vitalsの閾値、
GSC／GA4のレポート項目、AI検索の統計値）は、
**使うたびにWeb検索で最新を確認してから断定する**。
本スキル内の数値は 2026年8月時点のスナップショット。
