# Tokilog CLI 使い方マニュアル

Tokilog CLI (`tl`) は、ローカル保存を前提にした軽量な時間記録ツールです。

作業を始める、止める、今日の記録を確認する、といった日常操作を短いコマンドで実行できます。記録データはローカル SQLite DB に保存され、時刻の入力と表示は実行環境のローカル時刻で扱います。

インストールや PATH 設定は、先に利用環境ごとの Getting Started を参照してください。

- [macOS Getting Started](./getting-started-macos.md)
- [Windows Getting Started](./getting-started-windows.md)

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
tl start 朝会 "@tokilog" "#meeting" --at 09:00
```

`--at` を指定すると、開始時刻を明示できます。`HH:mm` は今日のローカル時刻として解釈されます。

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

`--at` を指定すると、停止時刻を明示できます。開始時刻より前、未来時刻、日付またぎになる時刻はエラーです。

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

`tl start`、`tl switch`、`tl add`、`tl fill`、`tl now` では、既存エントリの detail / project / tags をテンプレートとして再利用できます。

```console
tl start --template-id a1b2c3d4
tl add --template-within 10:45 --from 13:00 --to 14:00
tl switch 別タスク名 --template-id a1b2c3d4
```

| オプション | 意味 |
|-----------|------|
| `--template-id <id>` | 指定 ID のエントリから作業情報を補完する |
| `--template-within <time>` | 指定時刻を含むエントリから作業情報を補完する |

コマンドラインで明示した detail / `@project` / `#tag` はテンプレートより優先されます。時刻、状態、ID、作成日時はコピーされません。

## 記録を確認する

### 今日の記録

```console
tl today
tl today --dups
tl today --gaps
```

`tl today` は今日のエントリを開始時刻順に表示します。表示には短縮 ID、開始・終了時刻、経過時間、状態、detail、project、tags が含まれます。

`tl today --dups` は重複している完了済みエントリだけを表示します。

`tl today --gaps` は今日の未記録時間帯を表示します。

### 指定日の記録

```console
tl day 2026-05-01
tl day 2026-05-01 --dups
tl day 2026-05-01 --gaps
```

`tl day <YYYY-MM-DD>` は指定日のエントリを表示します。`--gaps` を付けると、指定日の未記録時間帯を表示します。

## 記録を編集・削除する

### ID で編集する

```console
tl edit --id a1b2c3d4 --from 10:00 --to 11:00
tl edit --id a1b2c3d4 --detail "設計レビュー"
tl edit --id a1b2c3d4 設計レビュー "@tokilog" "#review"
```

ID は `tl today` や `tl day` に表示される短縮 ID を使います。短縮 ID が複数候補に一致する場合は、より長い ID を指定して再実行します。

### 時刻で編集する

```console
tl edit --within 10:45 --detail "設計レビュー"
```

`--within` は指定時刻を含むエントリを探します。重複により複数候補がある場合、TTY では候補選択、非 TTY ではエラーになります。

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

```text
復元前バックアップ: /path/to/tokilog.db.before-restore-20260505123456789.bak
DBを復元しました: /path/to/tokilog-backup.db
```

復元は DB 全体の置き換えです。実行中タイマーを含む全データが復元元 DB の内容になります。復元元 DB の形式が現在のアプリで扱えない場合はエラーになり、現在の DB は置き換えられません。

## 対話モード

`tl shell` はまだ利用できません。

実装後は、Tokilog 専用 prompt で `tl` なしのサブコマンドを入力できる想定です。

```console
tl shell
tokilog> start 実装 "@tokilog" "#dev"
tokilog> current
tokilog> stop
tokilog> exit
```

終了コマンドは `exit` または `quit` です。

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

```console
TOKILOG_DATA_DIR=/tmp/tokilog-test tl today
```

通常利用中に `TOKILOG_DATA_DIR` を変えると別のデータディレクトリに切り替わります。

### ログ保存場所

ログは DB と同じ Tokilog アプリデータディレクトリ配下に保存されます。

| OS | 既定ログパス |
|----|--------------|
| Windows | `%LOCALAPPDATA%\Tokilog\tokilog.log` |
| macOS | `~/Library/Application Support/Tokilog/tokilog.log` |
| Linux | `~/.local/share/Tokilog/tokilog.log` |

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

`tl doctor` は実行環境とローカル DB 状態を確認します。DB パス、DB 接続、マイグレーション、ローカルタイムゾーン、OS、.NET runtime を確認したいときに使います。

### `tl upgrade`

```console
tl upgrade
```

Tokilog では、初回セットアップ時や Tokilog を新しいバージョンへ更新した後の migration 適用は `tl upgrade` で明示的に実行します。

`tl doctor` は migration 状態を確認する read-only 診断です。`Migrations: NG - Pending (run tl upgrade)` と表示された場合は `tl upgrade` を実行してください。

## エラーの読み方

Tokilog CLI のエラーは、基本的に次の形式で標準エラー出力に表示されます。

```text
エラー: [E###] メッセージ
```

| 例 | 対処 |
|----|------|
| `エラー: [E002] @project は必須です` | `@project` を追加して再実行する |
| `エラー: [E020] 時刻のフォーマットが不正です` | `09:30` または `2026-05-01T09:30` の形式で指定する |
| `エラー: [E011] 実行中のタイマーがありません` | `tl start` で開始してから `tl stop` する |
| `エラー: [E040] エクスポート対象のエントリがありません` | `--from` / `--to` の範囲、または記録の有無を確認する |
| `エラー: [E083] 指定したIDに複数のエントリが一致します` | より長い ID を `--id` に指定する |
| `エラー: [E901] DB の初期化に失敗しました` | DB 保存先の権限や `TOKILOG_DATA_DIR` の設定を確認し、必要なら `tl doctor` を実行する |
| `エラー: [E910] --output は必須です` | `tl db backup --output <path>` の形式でバックアップ先を指定する |
| `エラー: [E911] バックアップ先ファイルは既に存在します` | 別の出力先を指定するか、上書きする場合だけ `--force` を付ける |
| `エラー: [E914] DB復元には --force が必要です` | 復元元 DB が正しいことを確認してから `--force` を付ける |
| `エラー: [E916] 復元元DBの schema version が不正または非対応です` | 現在の Tokilog で扱える形式のバックアップ DB か確認する |

候補一覧が表示された場合は、表示された ID や候補を使って再実行してください。原因が分からない場合は `--debug` とログファイル、`tl doctor` の結果を確認します。
