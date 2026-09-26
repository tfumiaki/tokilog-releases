# Tokilog CLI 使い方マニュアル

Tokilog CLI (`tl`) は、ローカル保存を前提にした軽量な時間記録ツールです。

作業を始める、止める、今日の記録を確認する、といった日常操作を短いコマンドで実行できます。記録データはローカル SQLite DB に保存され、時刻の入力と表示は実行環境のローカル時刻で扱います。

インストールや PATH 設定は、先に利用環境ごとの Getting Started を参照してください。

- [macOS Getting Started](./getting-started-macos.md)
- [Windows Getting Started](./getting-started-windows.md)

<!-- toc -->

## 目次

- [最初に覚える流れ](#最初に覚える流れ)
- [作業情報の入力ルール](#作業情報の入力ルール)
- [タイマー操作](#タイマー操作)
  - [作業を開始する](#作業を開始する)
  - [実行中タイマーを確認する](#実行中タイマーを確認する)
  - [作業を停止する](#作業を停止する)
  - [作業を切り替える](#作業を切り替える)
- [手動で記録する](#手動で記録する)
  - [時間帯を指定して追加する](#時間帯を指定して追加する)
  - [直近の空き時間を埋める](#直近の空き時間を埋める)
  - [直近の終了時刻からタイマーを開始する](#直近の終了時刻からタイマーを開始する)
- [既存記録の作業情報を再利用する](#既存記録の作業情報を再利用する)
- [記録を確認する](#記録を確認する)
  - [今日の記録](#今日の記録)
  - [指定日の記録](#指定日の記録)
  - [過去の記録を検索する](#過去の記録を検索する)
- [記録を編集・削除する](#記録を編集削除する)
  - [ID で編集する](#id-で編集する)
  - [時刻で編集する](#時刻で編集する)
  - [タグを削除・再設定する](#タグを削除再設定する)
  - [削除する](#削除する)
- [project / tag を確認・整理する](#project--tag-を確認整理する)
  - [一覧を確認する](#一覧を確認する)
  - [名前を変更する](#名前を変更する)
  - [統合する](#統合する)
- [export](#export)
  - [自動エクスポート（`tl export auto`）](#自動エクスポートtl-export-auto)
- [config](#config)
- [DB バックアップ / 復元](#db-バックアップ--復元)
  - [現在の DB をバックアップする](#現在の-db-をバックアップする)
  - [バックアップから復元する](#バックアップから復元する)
- [対話モード](#対話モード)
- [Tab 補完](#tab-補完)
  - [セットアップ概要](#セットアップ概要)
- [DB / ログ / 診断](#db--ログ--診断)
  - [DB 保存場所](#db-保存場所)
  - [ログ保存場所](#ログ保存場所)
  - [`--debug`](#--debug)
  - [`tl doctor`](#tl-doctor)
  - [`tl upgrade`](#tl-upgrade)
- [エラーの読み方](#エラーの読み方)

<!-- /toc -->

## 最初に覚える流れ

```console
tl start 実装 "@tokilog" "#dev"
tl current
tl stop
tl today
```

| コマンド | 使う場面 |
|---------|----------|
| `tl start` | 作業を開始する |
| `tl current` | 実行中タイマーを確認する |
| `tl stop` | 実行中タイマーを停止する |
| `tl today` | 今日の記録を確認する |

実行中タイマーは同時に 1 件だけです。別の作業へ切り替える場合は `tl switch` を使うと、現在のタイマーを止めて新しいタイマーを開始できます。

## 作業情報の入力ルール

Tokilog の記録には、作業内容、project、tag を入力します。

```console
tl start API 実装 "@tokilog" "#backend" "#dev"
```

| 入力 | 意味 | 例 |
|------|------|----|
| `detail` | 作業内容。`@project`、`#tag`、オプション以外の文字列を空白で結合する | `API 実装` |
| `@project` | project。必須。未登録なら初回使用時に自動作成される | `@tokilog` |
| `#tag` | tag。0 個以上指定できる。未登録なら初回使用時に自動作成される | `#backend` |

detail は複数単語に分けて入力できます。

```console
tl start 仕様 レビュー "@tokilog" "#review"
```

上の detail は `仕様 レビュー` として扱われます。

shell によっては `@project` や `#tag` が特別な構文またはコメントとして扱われることがあります。迷った場合は `@project` と `#tag` を例のように引用符で囲んでください。空白や shell が特別扱いする文字を 1 token として渡したい場合も引用符を使います。

project 名は大文字小文字を区別します。tag 名は小文字に正規化されます。

## タイマー操作

### 作業を開始する

```console
tl start 実装 "@tokilog" "#dev"
tl start 朝会 "@tokilog" "#meeting" --from 09:00
```

`--from` を指定すると、開始時刻を明示できます。`HH:mm` / `HHmm` は今日のローカル時刻として解釈されます。`--at` も後方互換のために引き続き使用できます。

### 実行中タイマーを確認する

```console
tl current
```

実行中の記録があれば、開始時刻、経過時間、detail、project、tags が表示されます。

### 作業を停止する

```console
tl stop
tl stop --at 12:00
```

`--at` を指定すると、停止時刻を明示できます。開始時刻より前、未来時刻はエラーです。
停止結果が日を跨ぐ場合は、日単位の複数 completed entries に分割されます。

```text
タイマーを停止しました:
[a1b2c3d4] 23:50 - 24:00 (0h10m) 作業メモ
[e5f6a7b8] 00:00 - 00:10 (0h10m) 作業メモ
```

### 作業を切り替える

```console
tl switch レビュー "@tokilog" "#review"
tl switch レビュー "@tokilog" "#review" --at 13:00
```

`tl switch` は現在の実行中タイマーを停止し、同じ時刻から新しいタイマーを開始します。

## 手動で記録する

### 時間帯を指定して追加する

```console
tl add 設計 "@tokilog" "#design" --from 10:00 --to 11:30
tl add 調査 "@tokilog" --from 2026-05-01T14:00 --to 2026-05-01T15:00
```

`--from` と `--to` は必須です。完了済みエントリ同士の重複は許容されます。

### 直近の空き時間を埋める

```console
tl fill メール確認 "@admin" "#misc"
```

`tl fill` は、直近の完了済みエントリの終了時刻から現在時刻までを埋める記録を作成します。

指定時刻を含む gap を埋めたい場合は `--within` を使います。通常モードでは実行中タイマーがあるとエラーになりますが、`--within` 指定時は過去の gap を指定して埋められます。

```console
tl fill レビュー "@tokilog" "#review" --within 10:45
```

### 直近の終了時刻からタイマーを開始する

```console
tl now 実装 "@tokilog" "#dev"
```

`tl now` は、今日の直近完了済みエントリの終了時刻を開始時刻として、実行中タイマーを開始します。

## 既存記録の作業情報を再利用する

`tl start`、`tl switch`、`tl add`、`tl fill`、`tl now`、`tl edit` では、既存エントリの detail / project / tags をテンプレートとして再利用できます。

```console
tl start --template-id a1b2c3d4
tl add --template-within 10:45 --from 13:00 --to 14:00
tl switch 別タスク名 --template-id a1b2c3d4
tl edit --id a1b2c3d4 --template-id e5f6a7b8
```

| オプション | 意味 |
|-----------|------|
| `--template-id <id>` | 指定 ID のエントリから作業情報を補完する |
| `--template-within <time>` | 指定時刻を含むエントリから作業情報を補完する |

コマンドラインで明示した detail / `@project` / `#tag` はテンプレートより優先されます。`tl edit --reset-tags` も tags の明示指定として扱われます。時刻、状態、ID、作成日時はコピーされません。

## 記録を確認する

### 今日の記録

```console
tl today
tl today --dups
tl today --gaps
tl today --summary
```

`tl today` は今日のエントリを開始時刻順に表示します。表示には短縮 ID、開始・終了時刻、経過時間、状態、detail、project、tags が含まれます。

`tl today --dups` は重複している完了済みエントリだけを表示します。

`tl today --gaps` は今日の未記録時間帯を表示します。

`tl today --summary` は今日の完了済みエントリを project / tag / detail 単位で集計します。実行中タイマーは集計対象外です。

### 指定日の記録

```console
tl day 2026-05-01
tl day 2026-05-01 --dups
tl day 2026-05-01 --gaps
tl day 2026-05-01 --summary
```

`tl day <YYYY-MM-DD>` は指定日のエントリを表示します。`--gaps` を付けると、指定日の未記録時間帯を表示します。`--summary` を付けると、指定日の完了済みエントリを project / tag / detail 単位で集計します。

### 過去の記録を検索する

```console
tl search 会議
tl search 会議 "@ProjectX" --from 2026-07-01 --to 2026-08-31
tl search "#urgent" "#review" --limit 100
```

`tl search` は、日付をまたいで記録を探します。条件には `tl start` / `tl add` と同じ書き方を使います。

| 条件 | 意味 |
|------|------|
| detail の語 | detail の**部分一致**。ASCII の英字（A〜Z）だけは大文字・小文字を区別しません（`é` と `É` のような ASCII 以外の文字は区別します） |
| `@project` | project 名の完全一致。1 つだけ指定できます |
| `#tag` | tag 名の一致（tag は小文字で保存されるので、大文字・小文字は区別しません）。複数指定すると、**すべての tag が付いている**記録だけが残ります |
| `--from <date>` / `--to <date>` | 対象期間（**開始日の**ローカル日付で判定し、両端を含みます）。`tl start` などと違い、時刻ではなく**日付**を指定します |
| `--limit <n>` | 表示する最大件数。既定は 50、指定できるのは 1〜500 です |

- 条件を 1 つも指定しないとエラーになります（`--limit` だけでは条件になりません）
- 新しい記録が上に並びます。各行に日付が付きます
- 完了済みの記録と実行中の記録の両方が対象です。削除した記録は対象外です
- 件数が `--limit` を超えた場合は、先頭の件数だけを表示し「さらに結果あり」と表示します
- 該当が無いときは `該当するエントリはありません` と表示します（エラーにはなりません）

```text
$ tl search 会議 "@ProjectX" --from 2026-07-01 --to 2026-08-31
[a1b2c3d4] 2026-08-03 09:00 - 10:30 (1h30m) [completed] 定例会議 @ProjectX #urgent
[b2c3d4e5] 2026-07-28 13:00 - 14:00 (1h00m) [completed] 会議準備 @ProjectX

2 件
```

## 記録を編集・削除する

### ID で編集する

```console
tl edit --id a1b2c3d4 --from 10:00 --to 11:00
tl edit --id a1b2c3d4 --detail "設計レビュー"
tl edit --id a1b2c3d4 設計レビュー "@tokilog" "#review"
tl edit --id a1b2c3d4 --template-id e5f6a7b8
```

ID は `tl today` や `tl day` に表示される短縮 ID を使います。短縮 ID が複数候補に一致する場合は、より長い ID を指定して再実行します。

### 時刻で編集する

```console
tl edit --within 10:45 --detail "設計レビュー"
```

`--within` は指定時刻を含むエントリを探します。重複により複数候補がある場合、TTY では候補選択、非 TTY ではエラーになります。

### タグを削除・再設定する

```console
tl edit --id a1b2c3d4 --reset-tags
tl edit --id a1b2c3d4 --reset-tags dev review
tl edit --id a1b2c3d4 --reset-tags "#dev" "#review"
```

`--reset-tags` は tags を空にします。値を続けた場合は、その tag 一覧に置き換えます。`#tag` と `--reset-tags` は同時に指定できません。

### 削除する

```console
tl delete --id a1b2c3d4
tl delete --within 10:45
tl delete --id a1b2c3d4 --yes
```

TTY では確認プロンプトが表示されます。非 TTY で削除する場合は `--yes` が必要です。`--force` は `--yes` の別名として使えます。

## project / tag を確認・整理する

### 一覧を確認する

```console
tl projects
tl tags
```

保存済みの project / tag を名前順に表示します。

### 名前を変更する

```console
tl project rename tokilog Tokilog
tl tag rename reviews review
```

rename は変更後の名前がまだ存在しない場合に使います。TTY では確認プロンプトが表示されます。非 TTY では `--force` が必要です。

```console
tl project rename tokilog Tokilog --force
```

### 統合する

```console
tl project merge toki-log Tokilog
tl tag merge reviews review
```

merge は統合先がすでに存在する場合に使います。統合元の project / tag は削除され、関連エントリは統合先へ付け替えられます。

## export

```console
tl export csv
tl export csv --from 2026-05-01 --to 2026-05-31
tl export tsv
tl export tsv --from 2026-05-01 --to 2026-05-31
tl export markdown
tl export markdown --from 2026-05-01 --to 2026-05-31
tl export auto sync
tl export auto sync --from 2026-05-19 --to 2026-05-21
tl export auto status
```

| コマンド | 出力 |
|---------|------|
| `tl export csv` | CSV を標準出力へ出力する |
| `tl export tsv` | TSV を標準出力へ出力する |
| `tl export markdown` | Markdown を標準出力へ出力する |
| `tl export auto sync` | `export.auto.*` 設定に基づいて日次ファイルを指定範囲で生成する |
| `tl export auto status` | 自動エクスポートの設定と最終実行状態を表示する |

`tl export` の後に `csv` / `tsv` / `markdown` を指定します。`--from` / `--to` を省略した場合は今日が対象です。ファイルへ保存したい場合は shell のリダイレクトを使います。

```console
tl export csv --from 2026-05-01 --to 2026-05-31 > tokilog.csv
tl export markdown --from 2026-05-01 --to 2026-05-31 > tokilog.md
```

`tl export xlsx` はまだ利用できません。

### 自動エクスポート（`tl export auto`）

`tl export auto sync` は `export.auto.*` 設定を読み取り、指定範囲の日次ファイルを `<export.auto.path>/<sourceId>/daily/YYYY/MM/YYYY-MM-DD.<ext>` へ書き出します。

- `--from` / `--to` を省略した場合は今日が対象です。
- `export.auto.format` が未設定の場合は `markdown` を使用します。
- `export.auto.sourceId` が未設定の場合はホスト名（小文字）を使用します。
- `export.auto.path` が未設定の場合はエラーになります。
- `export.auto.enabled` が `false` でも、明示的な `tl export auto sync` は実行できます。
- ファイルは一時ファイルへ書き出した後にリネームする方式で、途中書き込み失敗による破損ファイルの残留を避けます。
- 実行結果には、今回書き出したファイルパス一覧（エントリ 0 件でスキップした日は除く）が表示されます。

`export.auto.enabled = true` の場合、次の書き込み系コマンドが成功した直後にも、影響日の日次ファイルを自動更新します。

- `tl start`
- `tl switch`
- `tl stop`
- `tl add`
- `tl fill`
- `tl now`
- `tl edit`
- `tl delete`

自動更新は元コマンドの DB 更新成功後に実行されます。自動更新が失敗しても元コマンドは成功のまま（終了コード `0`）で、標準エラーの警告と `export-state.json`（`tl export auto status`）で失敗内容を確認できます。`export.auto.path` が未設定の場合も同様に、元コマンドは失敗扱いにしません。

```console
# 設定を確認する
tl export auto status

# 今日の日次ファイルを生成する
tl export auto sync

# 指定範囲を再生成する
tl export auto sync --from 2026-05-01 --to 2026-05-31
```

実行結果の最終状態（最終成功時刻・最終エラー・最後に sync した範囲）は `export-state.json` に記録されます。

## config

```console
tl config get
tl config get export.auto.path
tl config set export.auto.enabled true
tl config set export.auto.path "/Users/me/Sync/tokilog-outbox"
tl config set export.auto.format markdown
tl config set export.auto.sourceId office-pc
```

| キー | 既定値 | 説明 |
|------|--------|------|
| `export.auto.enabled` | `false` | 自動エクスポートの有効/無効 |
| `export.auto.path` | `(unset)` | 同期先として使う出力先ディレクトリ |
| `export.auto.format` | `(unset)` | 出力形式。`markdown` / `csv` / `tsv` |
| `export.auto.sourceId` | `(unset)` | 出力元端末を識別する ID |

- `tl config get` は既知の設定を `key = value` 形式で表示します。
- `tl config get <key>` は指定キーの値だけを表示します。
- `tl config set` は既知キーのみ受け付けます。
- `export.auto.enabled` は `true` / `false` のみ指定できます。
- `export.auto.format` は `markdown` / `csv` / `tsv` のみ指定できます。
- `export.auto.path` はディレクトリパス文字列を保存します。この時点では存在確認を必須にしません。

## DB バックアップ / 復元

### 現在の DB をバックアップする

```console
tl db backup --output ./tokilog-backup.db
tl db backup --output ./tokilog-backup.db --force
```

`tl db backup` は、現在使用中の SQLite DB ファイルを指定したパスへ丸ごとコピーします。CSV / TSV export と違い、Tokilog の復元用 DB ファイルとして保存する操作です。

バックアップ先のディレクトリは事前に存在している必要があります。出力先ファイルがすでに存在する場合は、`--force` を付けたときだけ上書きします。

成功すると、バックアップ先の絶対パスが表示されます。

```text
DBをバックアップしました: /path/to/tokilog-backup.db
```

### バックアップから復元する

```console
tl db restore ./tokilog-backup.db --force
```

`tl db restore` は、指定した SQLite DB ファイルで現在の DB を丸ごと置き換えます。誤操作を避けるため、復元には `--force` が必須です。

復元前には、現在の DB が同じディレクトリへ自動バックアップされます。成功すると、復元前バックアップのパスと復元元 DB のパスが表示されます。

復元元が現在の DB と同じパス、または大文字小文字だけが違うパスのときは、何もせずに `E920` で止まります（復元前バックアップも作りません）。Linux などで、名前の大小だけが違う**別のファイル**から復元したいときは、復元元のファイル名を変えてから実行してください。

復元前バックアップを作成できなかったときは、`E921` で止まり、現在の DB は置き換えません。macOS / Linux では復元前バックアップの作成にハードリンクを使うため、ハードリンクを作れない保存先では、この `E921` で止まることがあります。

```text
復元前バックアップ: /path/to/tokilog.db.before-restore-20260505123456789.bak
DBを復元しました: /path/to/tokilog-backup.db
```

復元は DB 全体の置き換えです。実行中タイマーを含む全データが復元元 DB の内容になります。復元元 DB の形式が現在のアプリで扱えない場合はエラーになり、現在の DB は置き換えられません。

## 対話モード

`tl shell` を起動すると、Tokilog 専用の prompt で `tl` を付けずにサブコマンドを入力できます。

```console
tl shell
tokilog> start 実装 @tokilog #dev
tokilog> current
tokilog> stop
tokilog> exit
```

- 終了コマンドは `exit` または `quit` です。入力の誤り（解釈・検証のエラー）やコマンドの実行エラーでは、shell は終了しません。**例外は `upgrade` の `E909`** で、このときは次の入力を実行せずに shell が終了します（[`tl upgrade`](#tl-upgrade) の表を参照）
- **入力は Tokilog 自身が解釈します。** bash / zsh / PowerShell では `#tag` がコメントとして捨てられ、PowerShell では `@` や丸括弧も構文として扱われますが、`tl shell` の中ではそのどちらも起きません。`@project` や `#tag` を引用符なしで入力できます
- **`tl shell` の中では Tab 補完は使えません。** 補完が必要なときは、[Tab 補完](#tab-補完) を設定し、通常の shell から `tl` を直接実行してください

## Tab 補完

Tokilog CLI は `tl completion <shell>` で native completion script を出力します。

補完できる候補は主に次のとおりです。

| 種類 | 内容 |
|------|------|
| 静的補完 | コマンド名、サブコマンド名、オプション名 |
| 動的補完 | ローカル DB の既存 project / tag / detail 候補 |

未登録の project / tag や、まだ使ったことのない detail は、通常の入力としては新規に使えます。
DB が存在しない場合や補完用の DB 参照に失敗した場合、動的候補は空として扱われます。

### セットアップ概要

補完スクリプトは専用ファイルへ保存し、shell profile からそのファイルを読み込む構成を推奨します。

zsh の例:

```console
mkdir -p ~/.config/tokilog
tl completion zsh > ~/.config/tokilog/completion.zsh
```

`~/.zshrc` に次を追加:

```zsh
# Tokilog completion
if [ -f "$HOME/.config/tokilog/completion.zsh" ]; then
  source "$HOME/.config/tokilog/completion.zsh"
fi
```

bash の例:

```console
mkdir -p ~/.config/tokilog
tl completion bash > ~/.config/tokilog/completion.bash
```

`~/.bashrc` に次を追加:

```bash
# Tokilog completion
if [ -f "$HOME/.config/tokilog/completion.bash" ]; then
  source "$HOME/.config/tokilog/completion.bash"
fi
```

PowerShell の例:

```powershell
$tokilogConfigDir = Join-Path $HOME ".config/tokilog"
if (!(Test-Path $tokilogConfigDir)) { New-Item -ItemType Directory -Path $tokilogConfigDir -Force | Out-Null }
tl completion powershell > (Join-Path $tokilogConfigDir "completion.ps1")
```

PowerShell profile (`$profile`) に次を追加:

```powershell
# Tokilog completion
$tokilogCompletion = Join-Path $HOME ".config/tokilog/completion.ps1"
if (Test-Path $tokilogCompletion) {
  . $tokilogCompletion
}
```

Tokilog は shell profile の自動編集を行いません。`tl completion install` は提供していません。

詳細な手順と再セットアップ方法は [CLI Tab 補完の利用手順](./completion.md) を参照してください。

`--id`、`--within`、`--from`、`--to`、`--at` などの値候補や、過去エントリ由来の `detail + project + tags` をまとめて挿入する補完は提供していません。project / tag / detail は、それぞれ既存 DB から候補を表示します。

## DB / ログ / 診断

### DB 保存場所

Tokilog CLI はローカル SQLite DB に記録データを保存します。

| OS | 既定 DB パス |
|----|--------------|
| Windows | `%LOCALAPPDATA%\Tokilog\tokilog.db` |
| macOS | `~/Library/Application Support/Tokilog/tokilog.db` |

開発・テスト用途では、環境変数 `TOKILOG_DATA_DIR` にデータディレクトリの絶対パスを指定するとデータディレクトリを上書きできます。DB ファイル名は `tokilog.db` で固定です。`TOKILOG_DATA_DIR` にはディレクトリパスを指定します。`tokilog.db` まで含めてはいけません。

**`TOKILOG_LOG_PATH` も一緒に指定してください。** `TOKILOG_DATA_DIR` だけを向けても**ログは付いてきません**。エラーが起きたときの `tokilog.log` は、**`TOKILOG_DATA_DIR` とは無関係に決まる OS 既定のアプリケーションデータ配下**（Windows は `%LOCALAPPDATA%\Tokilog`、macOS は `~/Library/Application Support/Tokilog`、Linux は `$XDG_DATA_HOME/Tokilog`、`XDG_DATA_HOME` が未設定または空白なら `~/.local/share/Tokilog`）に書かれます。**普段 `TOKILOG_DATA_DIR` を設定して使っている場合でも、そちらには出ません。****どちらも同じ専用ディレクトリの下**（普段のデータディレクトリではない場所）を、絶対パスで指すようにしてください。

```console
TOKILOG_DATA_DIR=/tmp/tokilog-test TOKILOG_LOG_PATH=/tmp/tokilog-test/tokilog.log tl today
```

通常利用中に `TOKILOG_DATA_DIR` を変えると別のデータディレクトリに切り替わります。

### ログ保存場所

ログは **OS 既定の** Tokilog アプリデータディレクトリ配下に保存されます。**この場所は `TOKILOG_DATA_DIR` の影響を受けません** —— `TOKILOG_DATA_DIR` で DB を移しても、ログは下表の場所に出ます。移したい場合は `TOKILOG_LOG_PATH` を指定してください。

| OS | 既定ログパス |
|----|--------------|
| Windows | `%LOCALAPPDATA%\Tokilog\tokilog.log` |
| macOS | `~/Library/Application Support/Tokilog/tokilog.log` |
| Linux | `~/.local/share/Tokilog/tokilog.log`（`$XDG_DATA_HOME` を**有効な絶対パスとして**設定している場合は `$XDG_DATA_HOME/Tokilog/tokilog.log`）|

環境変数 `TOKILOG_LOG_PATH` に絶対パスを指定すると、ログファイルパスを上書きできます。

```console
TOKILOG_LOG_PATH=/tmp/tokilog.log tl today
```

### `--debug`

原因調査が必要な場合は `--debug` を付けます。

```console
tl --debug today
tl today --debug
```

`--debug` は通常のエラー表示に加えて詳細な例外情報を標準エラー出力へ表示します。記録データやコマンド結果は変更しません。

### `tl doctor`

```console
tl doctor
```

`tl doctor` は実行環境とローカル DB 状態を確認します。DB パス、DB 接続、マイグレーション、マイグレーションのロック行、ローカルタイムゾーン、OS、.NET runtime を確認したいときに使います。

`Migration lock` の行は、upgrade が使うロック行が DB に残っているかどうかを表示します。

| 表示 | 意味 |
|---|---|
| `None` | ロック行はありません |
| `Present (since <時刻>; cannot tell whether an upgrade is running)` | ロック行があります。**実行中の upgrade のものか、途中で終わった upgrade の残りかは判定できません**。時刻が古くても安全だとは限りません。これだけでは終了コードは `1` になりません |
| `NG - Unknown (failed to determine migration lock state)` | ロック行の状態を読めませんでした。終了コードは `1` です |

### `tl upgrade`

```console
tl upgrade
```

Tokilog では、初回セットアップ時や Tokilog を新しいバージョンへ更新した後の migration 適用は `tl upgrade` で明示的に実行します。

`tl doctor` は migration 状態を確認する read-only 診断です。`Migrations: NG - Pending (run tl upgrade)` と表示された場合は `tl upgrade` を実行してください。

`tl upgrade` は、実行しているあいだ **upgrade ロック**を持ちます。DB と同じフォルダの `tokilog.db.upgrade.lock` を OS の機能でロックし、この仕組みを持つ Tokilog どうしで、upgrade と次の `tl db unlock-upgrade` が同時に走らないようにします。このファイルは空のまま残り、削除しません（消さないでください）。ほかのコマンドの読み書きは止めません。

upgrade ロックが確かめるのは「**この仕組みを持つ Tokilog が、同じ PC で upgrade またはロックの解除を実行していない**」ことだけです。**この仕組みを持たない古い版の Tokilog・ほかの PC・ネットワーク上のフォルダ**で動いている upgrade は見えません。

`tl upgrade` は次のときに、待たずに止まります（終了コードは `1`）。

| エラー | 状態 | 対処 |
|---|---|---|
| `E905` | ほかの Tokilog が upgrade またはロックの解除を実行している | その処理が終わってから、もう一度実行する |
| `E906` | この保存先では upgrade ロックを確かめられない（ロックに対応しないフォルダなど） | この版では、この保存先では upgrade できません |
| `E908` | マイグレーションのロック行が DB に残っている | 下の「途中で終わった upgrade のロックを解除する」を読む |
| `E909` | upgrade のあとで、DB への古い接続を片付けられなかった（upgrade が成功していても出ます） | `tl` を起動し直す。`tl shell` の中で出た場合、shell はそこで終了しています |
| `E902` | マイグレーションのロック行を読めない | `tl doctor` で状態を確認する |

`E908` は、このコマンドが migration を適用していないことを表します。**ロック行を自動で削除することはしません** —— 残ったロック行と、実行中の upgrade のロック行は、DB の中身からは見分けられないためです。

### 途中で終わった upgrade のロックを解除する（`tl db unlock-upgrade`）

```console
tl db unlock-upgrade --force --confirm-no-upgrade-running
```

upgrade の途中でプロセスが強制終了すると、マイグレーションのロック行が DB に残り、次の `tl upgrade` が `E908` で止まります。そのロック行を取り除くコマンドです。

**実行する前に、ほかの Tokilog（CLI・Desktop・ほかの PC で動いているもの・古い版を含む）をすべて閉じ、upgrade が実行中でないことをご自分で確かめてください。** `--confirm-no-upgrade-running` はその確認をしたことの明示です。`--force` と `--confirm-no-upgrade-running` の両方が無ければ、何もせずに `E917` で終わります。

**実行中の upgrade のロック行を取り除くと、2 つの upgrade が同時に走り、DB が途中の状態で残るおそれがあります。** このコマンドは、ほかの Tokilog（古い版・ほかの PC を含む）が upgrade を実行していないことを証明できません。

このコマンドは次の順で動きます。

1. upgrade ロックを取ります（ほかの Tokilog が持っていれば `E905`、この保存先で確かめられなければ `E906`）
2. ロック行を読みます。無ければ「解除するロックはありません」と表示して終わります（DB への書込みも写しの作成もしません）
3. 警告とロック行の取得時刻を表示します
4. DB の写しを DB と同じフォルダに作ります（名前は `tokilog.db.before-unlock-<UTC の時刻>.bak`。既にある写しは上書きしません）。写しを作れなければ `E918` で終わり、ロック行は削除しません
5. ロック行を削除します。削除できなければ `E919` で終わります（自動で再試行しません。写しは残ります）
6. 写しのパスを表示します

```text
解除前の写し: /path/to/tokilog.db.before-unlock-20260924123456789.bak
解除しました。続けて tl upgrade を実行してください
```

**upgrade は自動では実行しません。** 続けて `tl upgrade` を実行してください。

## エラーの読み方

Tokilog CLI のエラーは、基本的に次の形式で標準エラー出力に表示されます。

```text
エラー: [E###] メッセージ
```

| 例 | 対処 |
|----|------|
| `エラー: [E002] @project は必須です` | `@project` を追加して再実行する |
| `エラー: [E020] 時刻のフォーマットが不正です` | `09:30` / `0930` または `2026-05-01T09:30` の形式で指定する |
| `エラー: [E011] 実行中のタイマーがありません` | `tl start` で開始してから `tl stop` する |
| `エラー: [E040] エクスポート対象のエントリがありません` | `--from` / `--to` の範囲、または記録の有無を確認する |
| `エラー: [E083] 指定したIDに複数のエントリが一致します` | より長い ID を `--id` に指定する |
| `エラー: [E901] DB の初期化に失敗しました` | DB 保存先の権限や `TOKILOG_DATA_DIR` の設定を確認し、必要なら `tl doctor` を実行する |
| `エラー: [E910] --output は必須です` | `tl db backup --output <path>` の形式でバックアップ先を指定する |
| `エラー: [E911] バックアップ先ファイルは既に存在します` | 別の出力先を指定するか、上書きする場合だけ `--force` を付ける |
| `エラー: [E914] DB復元には --force が必要です` | 復元元 DB が正しいことを確認してから `--force` を付ける |
| `エラー: [E916] 復元元DBの schema version が不正または非対応です` | 現在の Tokilog で扱える形式のバックアップ DB か確認する |
| `エラー: [E920] 復元元が現在の DB と同じパスか、大文字小文字だけが違うパスのため、何も置き換えていません` | 復元元のパスを確かめる。名前の大小だけが違う別のファイルなら、ファイル名を変えてから実行する |
| `エラー: [E921] 復元前バックアップを作成できず、復元していません（現在の DB は置き換えていません）` | DB と同じフォルダの空き容量と書込み権限を確かめる。詳細ログに元の失敗が残る |
| `エラー: [E905] ほかの Tokilog が upgrade またはロックの解除を実行中です` | その処理が終わってから、もう一度実行する |
| `エラー: [E906] この場所では upgrade ロックを確かめられないため、処理しません` | この版では、この保存先では upgrade もロックの解除もできません |
| `エラー: [E908] マイグレーションのロック行が残っているため、migration を適用していません…` | 「途中で終わった upgrade のロックを解除する」を読み、条件を満たすときだけ `tl db unlock-upgrade` を使う |
| `エラー: [E917] ロックの解除には --force と --confirm-no-upgrade-running の両方が必要です` | 条件を確かめてから、両方を付けて実行する |
| `エラー: [E918] 写しを作れなかったため、ロック行を削除しませんでした` | DB と同じフォルダの空き容量と書込み権限を確認する |
| `エラー: [E919] ロック行を削除できませんでした。このコマンドはロック行を削除していません` | ほかの Tokilog を閉じてから、もう一度実行する |

候補一覧が表示された場合は、表示された ID や候補を使って再実行してください。原因が分からない場合は `--debug` とログファイル、`tl doctor` の結果を確認します。
