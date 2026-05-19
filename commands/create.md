---
description: 入力 Markdown から portfolio.html を生成する。既存生成物がある場合は編集メニューを表示
---

このコマンドは、`/cc-portfoliokit:init` でファイル一式を用意し、`/cc-portfoliokit:hearing` で経歴を入れたあと、それらの Markdown 入力から **1 ページ HTML ポートフォリオ** を生成します。既に `portfolio.html` がある場合は編集メニューを表示します。

> **責任範囲**: HTML 生成と既存ポートフォリオの編集に専念します。初期セットアップ ── フォルダ作成 + プロフィール記入の案内 ── は `/cc-portfoliokit:init`、経歴のヒアリングは `/cc-portfoliokit:hearing` の責務です。
>
> **デザイン (HTML 構造 / CSS / 配色 / 職業別ラベル / Markdown 変換 / XSS 対策) はすべて専用スキル `cc-portfoliokit:portfolio-design` に集約されています。** このコマンドは HTML 生成のタイミングでそのスキルを `Skill` ツール経由で呼び出します。デザインの仕様変更はスキル側 (`skills/portfolio-design/SKILL.md`) で行ってください。

長くなりすぎないよう、詳細フローは 2 つの付録ファイルに切り出しています。

| ファイル | 含む内容 |
|---|---|
| **このファイル** | 導入 / Step 1 状態判定 / ブランチ A (init 誘導) / 共通の制約 |
| [`commands/create-branch-b.md`](./create-branch-b.md) | ブランチ B: 初回 HTML 生成フロー (B-1 Markdown パース 〜 B-5 完了報告) |
| [`commands/create-branch-c.md`](./create-branch-c.md) | ブランチ C: 既存ポートフォリオの編集メニュー (C-1 〜 C-6) |

ブランチ C は内部処理でブランチ B のフローに合流するため、ブランチ C を実行する場合は **B のファイルも合わせて `Read`** してください (詳細は後述)。

最初に下の **短い導入** を表示してから Step 1 に進んでください。

```
【ポートフォリオを生成します!】
現在の状態を確認しています…
```

---

## Step 1: 状態判定

cwd に対して以下 4 つの存在を `Bash` (例: `ls -la`) または `Read` 試行で確認し、状態フラグを取得します。

| フラグ | 対象 | 内容 |
|---|---|---|
| `HAS_CONFIG` | `./config.md` | サイト設定の有無 |
| `HAS_PROFILE` | `./career/profile.md` | プロフィールの有無 |
| `HAS_WORK` | `./career/work-history.md` | 職歴の有無 |
| `HAS_HTML` | `./portfolio.html` | 既存出力の有無 |

`HAS_CONFIG && HAS_PROFILE && HAS_WORK` を `INPUTS_READY` と呼びます。

ブランチ分岐:

| 状況 | ブランチ |
|---|---|
| `INPUTS_READY` 不成立 (= 入力が 1 つでも欠ける) | **A. init 誘導** ── このファイルの下を参照 |
| `INPUTS_READY && !HAS_HTML` | **B. 初回 HTML 生成** ── [`create-branch-b.md`](./create-branch-b.md) を `Read` して実行 |
| `INPUTS_READY && HAS_HTML` | **C. 編集メニュー** ── [`create-branch-c.md`](./create-branch-c.md) を `Read` して実行 (合流先の `create-branch-b.md` も併読) |

---

## ブランチ A: init 誘導

入力 Markdown が揃っていない状態。このコマンドではテンプレートを作らず、`init` コマンドの実行を案内するだけにとどめます。

### A-1. 案内表示

```
❌ ポートフォリオの素材が見つかりません。

足りないファイル:
  - <欠けているパスを列挙>

このコマンドは HTML 生成専用です。最初に次の流れで素材を用意してください:

  1. /cc-portfoliokit:init       — ファイルとフォルダを用意
  2. ./assets/ にアイコン画像を 1 枚入れる
  3. ./config.md と ./career/profile.md を開いてプロフィールを書き込む
  4. /cc-portfoliokit:hearing    — 経歴 (本業/副業/活動) を対話で入れる
  5. もう一度 /cc-portfoliokit:create を実行して HTML を生成
```

→ それ以上は何もせず終了。**勝手にテンプレートを作らないこと**。

---

## ブランチ B / C への進み方

### ブランチ B が選ばれた場合

```
1. `Read` で commands/create-branch-b.md を読み込む
2. 読み込んだ仕様に従って B-1 〜 B-5 を順に実行
3. 完了報告まで済んだら終了
```

### ブランチ C が選ばれた場合

```
1. `Read` で commands/create-branch-c.md を読み込む
2. 同時に `Read` で commands/create-branch-b.md も読み込む
   ── ブランチ C の C-2 / C-4 / C-6 は B のフローに合流するため
3. C-1 のメニューを表示してユーザーの選択を待つ
4. 選ばれた C-2 〜 C-6 のいずれかを実行
5. 最終的には B-1 以降が走って portfolio.html が再生成される
6. 完了報告まで済んだら終了
```

> **Read 順序のポイント**: ブランチ C を処理するときは **B も先に Read** すること。C の選択肢の多くが B のフローに合流するため、B の内容を知らずに C だけ読んでも処理できません。

---

## 共通の制約 (全ブランチ共通)

- **HTML / CSS / 配色 / Markdown 変換 / XSS 対策 のデザイン詳細は `cc-portfoliokit:portfolio-design` スキルが単独で受け持つ**。`create.md` / `create-branch-b.md` / `create-branch-c.md` のどれでも、ここを再実装したり迂回したりしない
- **`portfolio.html` 以外のファイルは、ユーザーが明示的に指示した編集 (ブランチ C-3) のときだけ書き換える**
- **入力に書かれていない情報を勝手に作らない** ── 氏名・経歴を捏造しない
- **テンプレートを勝手に作らない** ── 素材ファイルが無い場合は必ず `init` / `hearing` コマンドへ誘導する
- 空欄のフィールドはスキルに渡さない、または `null` で渡す。スキル側で「その項目ごと HTML から省く」
- 性別 (gender) はスキルに渡さない、HTML 出力にも編集メニューにも含めない
- **入力ファイルは YAML フロントマターを使わない `## 見出し` 構造**。パース時も `Edit` 時もこの前提で扱う。ユーザーが手編集する前提なので、見出し / コメント / 空行を勝手に壊さない
- **対話文は温かいトーン**: CLAUDE.md「文体のトーン」と memory `feedback_warm-tone.md` のルールを守る。`( )` で補足を書くスタイルを画面に出さない
- **「フォルダ」を使う**: ユーザー向け対話文で「ディレクトリ」を出さない
- **並行セッション対策**: `Write` / `Edit` 直前に対象ファイルを `Read` し直す。会話前半の記憶のままで触らない
