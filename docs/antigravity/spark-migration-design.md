# Claude → Gemini Spark 移植 設計書（Antigravity向け）

作成日：2026-09-24
対象：Antigravity（実装担当エージェント）
依頼主：太郎課長

このファイルは、Antigravity側から返ってきた質問
「具体的なファイル構成や行わせたいタスクの詳細を教えてほしい」
への回答である。上から順に読めば、移植作業に着手できる状態にしてある。

---

## 0. ゴール

Claude上で運用している「太郎課長の発信OS」（スキル群＋ナレッジ）を、
Gemini Spark でも同じ判断基準・同じ文体ルールで動かせるようにする。

- 完全移植ではなく「並行運用」。Claudeは残す。原本はClaude側（このリポジトリ＋Claudeのスキル）
- Sparkに任せたいのは、PCを閉じていても回る定型作業（ネタ集め、鮮度チェック、下書きの叩き台）
- 公開はしない。Sparkの出力は必ず「下書き」で止め、投稿は人間がやる

---

## 1. 前提として確認済みの事実（2026-09-24時点）

| 項目 | 内容 | 出典 |
|---|---|---|
| 日本での提供開始 | 2026-07-16（Ultra）。2026-07-29にGoogle AI Proへ拡大 | Google公式ブログ日本版、Impress Watch ほか |
| 構成要素 | Tasks（単発指示）／Skills（再利用できる手順と文脈）／Schedules（時刻・条件トリガー） | Gemini Apps ヘルプ |
| スキルの取り込み | SKILL.md 単体、または SKILL.md をルート直下に置いた .zip をUIからインポート可能 | Gemini Apps ヘルプ「Create & manage skills」 |
| スキル同士の参照 | 1タスクで複数スキルを使える。スキルから別スキルを参照できる | 同上 |
| 連携先 | Gmail／Calendar／Drive／Docs／Sheets／Slides／Keep ほか。カスタムMCP対応 | Gemini Spark 公式ページ |
| 同時実行上限 | 実行中タスクは最大15件。15件埋まっているとスケジュールは起動しない | Gemini Apps ヘルプ |

### 未確認（Antigravityが最初に実機で確かめること）

1. .zip 内の `references/` サブフォルダを読んでくれるか（SKILL.md 単体しか見ない可能性あり）
2. SKILL.md 1ファイルあたりのサイズ上限、登録できるスキル数の上限
3. frontmatter の `description` の文字数上限と、日本語トリガー語で自動発動するか
4. スキル本文から Google ドライブ上のファイルをパス／リンクで参照させられるか
5. Web検索をスケジュール実行の中で使えるか

ここが分かるまでは、第4章の変換ルールは「references が読めない前提」の安全側で組む。

---

## 2. 移植元の全体構成

原本は2か所に分かれている。

### 2-1. このリポジトリ（公開リポジトリ）

```
tarou/
├── README.md
└── .claude/skills/seo/
    ├── SKILL.md                 # 地図（ルーティング表・鮮度ポリシー・ワークフロー）
    └── references/01〜09-*.md   # 計1,273行
```

### 2-2. Claudeアカウント側のスキル（リポジトリ外。本人がエクスポートして渡す）

| スキル | 役割 | 容量 | references | 移植優先度 |
|---|---|---|---|---|
| taro-strategy | 何者として何を売るか。キャラ・思想の中核 | 4KB | なし | A |
| x-article-writing | X長文・note記事の執筆OS（4ステップ・文体規定・整形） | 12KB | なし | A |
| x-article-check | 公開前の4項目チェック | 11KB | なし | A |
| thumbnail-title-hook-check | サムネ→タイトル→冒頭3行の検証 | 12KB | なし | A |
| x-cta | 固定CTAの唯一の原本 | 6KB | なし | A |
| masking-guard | 実名・取引先・機密数値のマスキング関所 | 4KB | なし | A（必須） |
| taro-materials | フック370選・エピソード目次・視点11種 | 2KB | あり（3本・計559行） | A |
| x-project-files | ナレッジ01〜07の使い分け裁定 | 5KB | なし | A |
| taro-ism | 引用リプ・返信リプの型 | 18KB | なし | B |
| x-quote-others | 他者引用ポストの12型 | 12KB | なし | B |
| conbini-threads | コンビニ・職場あるあるのスレッズ | 11KB | なし | B |
| smb-ai-topdown | 中小企業×AIのSEO記事量産型 | 5KB | なし | B |
| seo-knowledge | SEO・AI検索の関所（一次情報ベース） | 8KB | あり（5本） | B |
| line-broadcast | LINEオプチャ配信文 | 3KB | なし | C |
| x-article-mobile | スマホ貼り付け用の出力整形 | 4KB | なし | C |

移植しないもの：docx／pptx／xlsx／pdf／skill-creator／code-review／commit-message／explain-code／import-memory／docs（Claude環境専用、またはGoogle側に同等機能がある）。
morning は Spark の標準機能（朝のブリーフ）と重複するので移植しない。

### 2-3. Claudeプロジェクトのナレッジ（本人がエクスポートして渡す）

スキル本文から名前で参照されているファイル。これが無いとスキルが空回りする。

| ファイル | 参照元スキル |
|---|---|
| X記事作成プログラムプロンプト 統合版（v20260703） | x-article-writing／x-article-check／taro-strategy／taro-materials |
| 挫折エピソード全文集 | x-article-writing／taro-strategy／taro-materials |
| 01_思想・人生・哲学.md | x-project-files |
| 02_職務経歴.md | x-project-files |
| 03_過去のX記事.md | x-project-files |
| 04_参考記事.md（毎回差し替え） | x-project-files |
| 05_X記事分析レポート.md | x-project-files |
| 06_挫折エピソード全文集.md／07_アカウント分析.md（任意） | x-project-files |

### 2-4. 個人設定（Claudeの「プロフィール > 個人設定」）

トンマナ・一人称・禁止事項の原本。Sparkでは全スキル共通の土台スキル `taro-base` として登録する（第4章）。

---

## 3. Googleドライブの配置

```
マイドライブ/
└── taro-os/
    ├── 00_README.md                  # この設計書のコピー
    ├── skills/                       # Sparkにインポートする .zip を置く
    │   ├── taro-base.zip
    │   ├── taro-strategy.zip
    │   └── …
    ├── knowledge/                    # 2-3 のナレッジ。ファイル名は変えない
    │   ├── 01_思想・人生・哲学.md
    │   ├── …
    │   ├── 統合版_v20260703.md
    │   └── 挫折エピソード全文集.md
    ├── inbox/                        # Sparkの出力（下書き・ネタ）が溜まる場所
    │   ├── drafts/
    │   └── neta/
    └── logs/                         # スケジュール実行の記録（Sheets）
```

ルール：
- ファイル名はClaude側と完全一致させる。スキルがファイル名で参照しているため
- `knowledge/` はSparkに書き込ませない（読み取り専用の扱い）。書いてよいのは `inbox/` と `logs/` だけ
- 共有設定は「自分のみ」。リンク共有をオンにしない

---

## 4. SKILL.md の変換ルール

Claude形式のSKILL.mdはほぼそのまま使える。変えるのは以下だけ。

### 4-1. frontmatter
- `name`：半角小文字＋ハイフンのまま（Claude側と同名にする）
- `description`：そのまま。上限に引っかかったら、発動トリガー語の列挙を優先して残し、説明文を削る

### 4-2. 本文の置換

| Claude側の記述 | Spark側での置き換え |
|---|---|
| 「プロジェクトのナレッジ『○○』を参照」 | 「Googleドライブ `taro-os/knowledge/○○` を読む」 |
| `references/xx.md` を読む | 1章の未確認事項1が可なら zip に同梱。不可なら SKILL.md 末尾に全文を連結する |
| 「Web検索で確認」「web_search」 | Sparkの検索機能で確認。使えない実行環境なら「未確認」と明記して止まる |
| アーティファクト／HTMLで出力 | Googleドキュメントで `inbox/drafts/` に保存 |
| 「ユーザーに確認する」 | スケジュール実行中は確認できないので、`inbox/` のドキュメント冒頭に「要確認：」として書き残して終了 |
| 他スキル名（`x-article-writing` など） | 同名のSparkスキルを参照。未移植のスキル名なら、その旨を注記 |

### 4-3. 土台スキル `taro-base` を新設する

中身は 2-4 の個人設定を転記するだけ。Claudeの個人設定にあたる場所がSparkに無い前提で、全タスクにこのスキルを必ず付ける。
含める項目：情報の扱い（日本語圏での実装日を確認）、トンマナ、一人称、太郎課長の人物設定、禁止事項すべて。
特に「太字記法（**）を本文に出さない」「根拠のない数字を作らない」「実名・社名を出さない」の3つは、出力前の最終チェックとして末尾に再掲する。

### 4-4. 変換してはいけないもの
- 文体サンプル・フック・CTAの文面は一字も変えない
- 禁止事項を要約・短縮しない
- 数値（文字数、行数、チェック項目数）を丸めない

---

## 5. Sparkに任せるタスクとスケジュール（案）

どれも「下書きを置くまで」で止める。投稿・送信・公開はさせない。

| # | 名前 | 種別 | トリガー | 使うスキル | 出力先 |
|---|---|---|---|---|---|
| T1 | 週次ネタ出し | Schedule | 毎週月曜 7:00 | taro-base, taro-strategy, taro-materials | `inbox/neta/YYYY-MM-DD.gdoc`（切り口5本＋使えるフック候補） |
| T2 | X記事の構成案 | Task（手動） | 都度 | taro-base, x-project-files, x-article-writing, thumbnail-title-hook-check | `inbox/drafts/` |
| T3 | 公開前チェック | Task（手動） | 都度 | taro-base, x-article-check, masking-guard, x-cta | 指摘をドキュメントのコメントで返す |
| T4 | SEO・AI検索の鮮度チェック | Schedule | 毎月1日 7:00 | seo-knowledge | `logs/seo-freshness` シートに「項目／変化の有無／出典URL／日本での実装日」を1行ずつ追記 |
| T5 | AIツールのアップデート監視 | Schedule | 平日 7:30 | taro-base | `inbox/neta/`。日本語圏での実装日が発表済みのものだけ採用。未発表は「保留」欄へ |

T1〜T5は案である。どれを有効にするかは本人が決める。まずT2とT3（手動）で文体の再現度を確かめ、合格してからT1・T4・T5のスケジュールを入れる。

---

## 6. 移植できない・しないもの

- ローカルのターミナル操作、git操作、Claude Codeのフック：Sparkはクラウド実行なので対象外。リポジトリの更新はClaude側で続ける
- Claudeのアーティファクト、Claude Docs：Googleドキュメントで代替
- Claudeプロジェクト単位の切り替え：Sparkではタスクごとに付けるスキルとドライブのフォルダで代替

---

## 7. Antigravityの作業手順

1. 1章の未確認事項1〜5を実機で確かめ、結果をこのファイルの末尾「検証ログ」に追記する
2. 本人から受け取った 2-2 のスキル一式と 2-3 のナレッジを、3章の構成でドライブに置く
3. 2-4 から `taro-base` の SKILL.md を作る
4. 優先度Aのスキルを4章のルールで変換し、zip化して `skills/` に置き、Sparkにインポートする
5. 8章の受け入れテストを実行する
6. 合格したら優先度B、Cの順に同じ手順を繰り返す
7. 最後に5章のスケジュールを、本人が選んだものだけ登録する

変換で判断に迷った箇所は、勝手に書き換えず「変換メモ」として列挙し、本人に返す。

---

## 8. 受け入れテスト

同じ入力をClaudeとSparkの両方に投げ、以下を比べる。1つでも落ちたら不合格。

| # | 入力 | 合格条件 |
|---|---|---|
| 1 | 「管理職×AIでX記事の構成案を作って」 | 一人称が「俺」。ですます調でない。本文に `**` が無い。疑問形で終わる本ポストが無い |
| 2 | 1の出力に x-article-check をかける | 項目0（痛み変換）と4項目すべてに判定が付く |
| 3 | 実名・社名・金額入りの体験談を渡して「記事にして」 | masking-guard が発動し「A社」等に伏せて出る |
| 4 | 「最後の締めどうする」 | x-cta の固定CTAが一字違わず出る。旧CTAが出ない |
| 5 | 「最新のコアアップデートは」 | 出典URLと日付が付く。確認できない場合は断定しない |
| 6 | ナレッジ04を消した状態で記事依頼 | 無い前提で書かず、確認を求める（スケジュール時は「要確認：」を残す） |

---

## 9. 原本の管理ルール

- 原本はClaude側。Spark側を直接直したら、同じ修正をClaude側にも入れる（逆も同じ）
- 変換後のSKILL.mdの1行目のコメントに、元にしたClaude側スキルの更新日を書く
- 統合版やCTAが更新されたら、影響するスキルを2-2・2-3の表から逆引きして全部差し替える

---

## 出典

- Google公式ブログ日本版「Gemini Spark：24時間365日対応する自分だけのAIエージェントが日本のProユーザーにも拡大」 https://blog.google/intl/ja-jp/company-news/technology/gemini-spark-comes-to-japan/
- Impress Watch「Gemini Spark、日本でスタート」 https://www.watch.impress.co.jp/docs/news/2125584.html
- Gemini Apps ヘルプ「Use Gemini Spark to manage your tasks & workflows」 https://support.google.com/gemini/answer/17094507
- Gemini Apps ヘルプ「Create & manage skills」 https://support.google.com/gemini/answer/17094296
- Gemini Spark 公式 https://gemini.google/overview/agent/spark/

---

## 検証ログ（Antigravityが追記）

| 日付 | 項目 | 結果 |
|---|---|---|
| | | |
