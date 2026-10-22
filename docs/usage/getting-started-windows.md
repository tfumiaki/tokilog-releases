# Tokilog Windows Getting Started

このドキュメントは、Windows で Tokilog CLI (`tl`) または Desktop GUI を使い始めるための一般ユーザー向け手順です。

Tokilog はローカル保存を前提にした軽量な時間記録ツールです。CLI では作業の開始、現在のタイマー確認、停止、今日の記録確認を `tl` コマンドで行います。Desktop GUI は同じローカル DB と Application ロジックを使う alpha build です。

この手順は Windows 向けです。macOS / Linux 向け Getting Started、MSI / MSIX などのインストーラー、`install.ps1` による自動導入は対象外です。

<!-- toc -->

## 目次

- [対象環境](#対象環境)
- [1. zip を取得する](#1-zip-を取得する)
- [2. CLI zip を展開する](#2-cli-zip-を展開する)
- [3. CLI を PATH に追加する](#3-cli-を-path-に追加する)
- [4. CLI の動作確認をする](#4-cli-の動作確認をする)
- [5. 最初に試すコマンド](#5-最初に試すコマンド)
- [Desktop GUI を試す場合](#desktop-gui-を試す場合)
- [PowerShell 特有の入力ルール](#powershell-特有の入力ルール)
  - [tl shell で引用符なしに入力する](#tl-shell-で引用符なしに入力する)
- [データ保存場所](#データ保存場所)
  - [データディレクトリの上書き（開発・テスト用）](#データディレクトリの上書き開発テスト用)
  - [既存 DB を使い続ける場合（`TOKILOG_DB_PATH` からの移行）](#既存-db-を使い続ける場合tokilog_db_path-からの移行)
- [ログ保存場所と --debug](#ログ保存場所と---debug)
- [Tab 補完を使う場合](#tab-補完を使う場合)
- [アンインストール](#アンインストール)
  - [記録データも削除する（必要な場合のみ）](#記録データも削除する必要な場合のみ)
- [トラブルシュート](#トラブルシュート)

<!-- /toc -->

## 対象環境

- Windows 10 / 11
- x64 環境
- .NET 10 Runtime または .NET 10 SDK
- PowerShell

配布される CLI (`tl.exe`) と Desktop GUI は `SelfContained=false` の publish です。実行する PC には **.NET 10 Runtime** が必要です。

PowerShell で次のコマンドを実行し、.NET 10 の Runtime または SDK が表示されるか確認してください。

```powershell
dotnet --list-runtimes
```

.NET 10 が表示されない場合は、.NET 10 Runtime をインストールしてから続けてください。

## 1. zip を取得する

GitHub Releases から Windows 用 zip をダウンロードします。

1. [Tokilog Releases](https://github.com/tfumiaki/tokilog-releases/releases) を開く
2. 利用したいバージョンの Release を開く
3. Assets から Windows 用 zip をダウンロードする

現在の Windows 用 asset 名は次を想定しています。

```text
CLI
tokilog-cli-net10.0-win-x64.zip

Desktop GUI
tokilog-desktop-net10.0-win-x64.zip
```

CLI を使う場合は `tokilog-cli-net10.0-win-x64.zip`、Desktop GUI を試す場合は `tokilog-desktop-net10.0-win-x64.zip` を取得してください。Release に Windows 用 zip がない場合、その Release では Windows 用の配布物がまだ公開されていません。

## 2. CLI zip を展開する

zip を任意のフォルダーに展開します。展開先は、あとで PATH に追加しやすく、誤って削除しにくい場所にしてください。

展開先の例:

```text
%LOCALAPPDATA%\Tokilog\bin
C:\Tools\Tokilog
```

たとえば `C:\Tools\Tokilog` に配置する場合は、次のような状態になっていれば問題ありません。

```text
C:\Tools\Tokilog\tl.exe
```

zip の中に `win-x64` フォルダーが入っている場合は、次のどちらかにしてください。

- `win-x64` フォルダーの中身を `C:\Tools\Tokilog` へ移動する
- 後続の PATH に `C:\Tools\Tokilog\win-x64` を追加する

## 3. CLI を PATH に追加する

`tl` をどのフォルダーからでも実行できるように、展開先フォルダーをユーザー環境変数の PATH に追加します。

PowerShell で追加する例:

```powershell
$tokilogHome = "C:\Tools\Tokilog"
$userPath = [Environment]::GetEnvironmentVariable("Path", "User")
[Environment]::SetEnvironmentVariable("Path", "$userPath;$tokilogHome", "User")
```

`%LOCALAPPDATA%\Tokilog\bin` に展開した場合は、`$tokilogHome` を次のように変えてください。

```powershell
$tokilogHome = "$env:LOCALAPPDATA\Tokilog\bin"
```

Windows の設定画面から追加する場合は、ユーザー環境変数の `Path` に Tokilog の展開先フォルダーを追加してください。

PATH を変更した後は、新しい PowerShell を開いて確認します。

```powershell
tl --help
```

`tl` が見つからない場合は、PATH に追加したフォルダーが `tl.exe` のあるフォルダーと一致しているか確認してください。

## 4. CLI の動作確認をする

まず `tl upgrade` を実行して、初回セットアップに必要な DB migration を適用します。続けて `tl doctor` で CLI 実行環境とローカル DB 状態を確認します。

```powershell
tl upgrade
tl doctor
```

Tokilog を新しいバージョンへ更新した後も、最初に `tl upgrade` を実行してから通常コマンドを使ってください。

## 5. 最初に試すコマンド

次の流れで、タイマー開始から今日の記録確認までを試します。

```powershell
tl start "Getting Started を読む" "@tokilog" "#docs"
tl current
tl stop
tl today
```

各コマンドの役割:

| コマンド | 説明 |
|---|---|
| `tl start` | 新しい実行中タイマーを開始する |
| `tl current` | 現在の実行中タイマーを表示する |
| `tl stop` | 実行中タイマーを停止する |
| `tl today` | 今日のエントリ一覧と合計時間を表示する |
| `tl today --gaps` | 今日の最初のエントリ開始から現在時刻までの未記録時間帯を表示する |

PowerShell では `@project` や `#tag` をそのまま書くと別の構文として解釈される場合があります。Windows の PowerShell で試すときは、例のように `"@tokilog"`、`"#docs"` と引用符で囲むと安全です。

## Desktop GUI を試す場合

Desktop GUI を試す場合は、`tokilog-desktop-net10.0-win-x64.zip` を任意のフォルダーに展開し、展開先の `Tokilog.Desktop.exe` を起動します。

zip の中に `win-x64` フォルダーが入っている場合は、その中の `Tokilog.Desktop.exe` を実行してください。

Desktop GUI も CLI と同じローカル SQLite DB を使います。初回セットアップや schema upgrade が必要な場合は、起動画面に [Set up database] / [Upgrade database] が出ます。ほかの Tokilog（CLI を含む）を閉じてから押すと、DB の写しを DB の隣に保存してから upgrade します。CLI で `tl upgrade` を実行してから Desktop GUI を起動しても構いません。

## PowerShell 特有の入力ルール

PowerShell では `@`・`#`・丸括弧が shell 固有の構文として扱われます。Tokilog で project (`@project`) や tag (`#tag`)、丸括弧を含む作業内容を入力するときは、必ずダブルクォートで囲んでください。

| 文字 | PowerShell での扱い | Tokilog での入力例 |
|------|-------------------|------------------|
| `#` | コメント開始。`#` 以降の入力がすべて無視される | `"#docs"` |
| `@` | splatting 演算子。変数展開が試みられエラーになることがある | `"@tokilog"` |
| `(` `)` | PowerShell の構文として扱われ、エラーになることがある | `"会議(定例)"` |

```powershell
# 正しい: ダブルクォートで囲む
tl start "Getting Started を読む" "@tokilog" "#docs"

# NG: #docs 以降がコメントとして無視され、tag が渡されない
tl start "Getting Started を読む" "@tokilog" #docs

# NG: @tokilog が splatting として解釈され、予期しないエラーになる
tl start "Getting Started を読む" @tokilog "#docs"
```

`"#docs"` や `"@tokilog"` の引用符は PowerShell に対してのもので、Tokilog に渡る値は `#docs` / `@tokilog` と同じです。`#` の問題はエラーが出ず tag が無視されるだけなので特に注意してください。

### tl shell で引用符なしに入力する

`tl shell` を起動すると、入力を PowerShell ではなく Tokilog 自身が解釈します。`@`・`#`・丸括弧をそのまま書けます。

```powershell
tl shell
```

```console
tokilog> start Getting Started を読む @tokilog #docs
tokilog> stop
tokilog> exit
```

**`tl shell` の中では Tab 補完は使えません。** 補完を使いたいときは [Tab 補完を使う場合](#tab-補完を使う場合) の手順で設定し、PowerShell から `tl` を直接実行してください（その場合は上の表のとおり引用符が要ります）。

詳しくは [CLI マニュアルの対話モード](cli-manual.md#対話モード) を参照してください。

## データ保存場所

Tokilog CLI は記録データをローカル SQLite DB に保存します。サーバーへの同期は行いません。

Windows のデフォルト DB 保存場所:

```text
%LOCALAPPDATA%\Tokilog\tokilog.db
```

DB ファイルは Tokilog 本体とは別に保存されます。zip の展開先フォルダーを削除しても、記録データは自動では削除されません。

### データディレクトリの上書き（開発・テスト用）

環境変数 `TOKILOG_DATA_DIR` にデータディレクトリの絶対パスを指定すると、データディレクトリを上書きできます。DB ファイル名は `tokilog.db` で固定です。`TOKILOG_DATA_DIR` にはディレクトリパスを指定します。`tokilog.db` まで含めてはいけません。通常利用ではデフォルト保存場所のまま使うことを想定しています。

**`TOKILOG_LOG_PATH` も一緒に指定してください。** ログの保存場所は `TOKILOG_DATA_DIR` に従わないため、これを省くと、テスト用の領域を指定したつもりでもエラーログだけが普段の場所に出ます。

```powershell
$env:TOKILOG_DATA_DIR = "C:\Users\me\AppData\Local\Tokilog"
$env:TOKILOG_LOG_PATH = "C:\Users\me\AppData\Local\Tokilog\tokilog.log"
tl today
# → DB ファイル: C:\Users\me\AppData\Local\Tokilog\tokilog.db
# → ログファイル: C:\Users\me\AppData\Local\Tokilog\tokilog.log
```

`TOKILOG_DATA_PATH` は採用していません（ファイルパスではなくディレクトリであることを明確にするため、`_DIR` を使用しています）。

> **破壊的変更**: `TOKILOG_DB_PATH` は廃止されました。以前 `TOKILOG_DB_PATH` でパスを指定していた場合は `TOKILOG_DATA_DIR` に移行してください。

### 既存 DB を使い続ける場合（`TOKILOG_DB_PATH` からの移行）

以前 `TOKILOG_DB_PATH` で別パスの DB ファイルを指定していた場合は、次の手順で既存 DB を使い続けられます。

1. 新しいデータディレクトリを作成する（例: `C:\Users\me\AppData\Local\Tokilog`）
2. 以前 `TOKILOG_DB_PATH` で指定していた DB ファイルを `tokilog.db` という名前で上記ディレクトリへコピーまたは移動する
3. `TOKILOG_DATA_DIR` にそのディレクトリを設定する

```powershell
$env:TOKILOG_DATA_DIR = "C:\Users\me\AppData\Local\Tokilog"
# → DB ファイル: C:\Users\me\AppData\Local\Tokilog\tokilog.db
```

## ログ保存場所と --debug

原因調査が必要な実行時エラーはログファイルに記録されます。通常の入力ミスや対象なしなど、画面のエラーメッセージだけで対処できるエラーは原則としてログ出力対象外です。

Windows のデフォルトログ保存場所:

```text
%LOCALAPPDATA%\Tokilog\tokilog.log
```

ログファイルの場所を変更したい場合は、環境変数 `TOKILOG_LOG_PATH` に絶対パスを指定します。

```powershell
$env:TOKILOG_LOG_PATH = "C:\tmp\tokilog.log"
tl today
```

詳細な例外情報を画面にも表示したい場合は、`--debug` を指定します。

```powershell
tl --debug today
tl today --debug
```

`--debug` は原因調査用です。コマンドの結果や記録データの扱いは通常実行時と変わりません。

## Tab 補完を使う場合

Tokilog CLI は `tl completion powershell` で PowerShell 向けの native completion script を出力します。

補完は任意機能です。command / option の静的候補に加えて、ローカル DB に存在する project / tag / detail を動的候補として表示できます。

Tokilog は `tl completion install` のような shell profile 自動編集コマンドは提供しません。

補完を使いたい場合は、[CLI Tab 補完の利用手順](./completion.md) を参照してください。補完スクリプトは専用ファイルへ保存し、PowerShell profile からそのファイルを読み込む構成を推奨します。

## アンインストール

Tokilog は zip 展開先に置いた実行ファイルを PATH から呼び出す構成です。専用アンインストーラーはありません。

本体を削除する手順:

1. Tokilog の展開先フォルダーを削除する
2. PATH に追加した Tokilog の展開先フォルダーを削除する
3. 新しい PowerShell を開き、`tl --help` が見つからないことを確認する

### 記録データも削除する（必要な場合のみ）

記録データは Tokilog 本体とは別の場所にあり、本体を削除しても自動では削除されません。記録を残したい場合は、この手順を行わないでください。

1. **Tokilog（CLI の `tl` と Desktop）をすべて閉じる。**
2. **記録の保存先（データディレクトリ）を確かめる。** 既定は `%LOCALAPPDATA%\Tokilog` です。`TOKILOG_DATA_DIR` を設定して使っていた場合は、その値のディレクトリです。**既定の場所と `TOKILOG_DATA_DIR` を使い分けていた場合や、`TOKILOG_DATA_DIR` に違う値を指定したことがある場合は、既定の場所と、過去に指定した各 `TOKILOG_DATA_DIR` の場所のそれぞれに記録が残っていることがあります。** それぞれの場所について、この手順 2 以降を行ってください（ディレクトリごと削除してよいかは、場所ごとに手順 3 で決めます）。**あわせて、データディレクトリ自体、その途中のフォルダー、`tokilog.db` のどれかが、ほかの場所を指すリンク（シンボリックリンク。Windows ではジャンクションなど）でないかを確かめ、リンクがあれば実体の場所を控えておいてください**（削除した後では辿れません。リンクを削除しても実体は残ります。下の「この手順で消えない記録」を参照）。**自動エクスポートを使っていた場合は、実際の出力先も控えておいてください**（`config.json` を削除した後では分かりません）。CLI が使えるなら、`tl export auto status` を実行し、`output root` に表示される場所を控えます（`TOKILOG_DATA_DIR` を設定して使っていた場合は、同じ値を指定して実行します。指定しないと、既定の場所の設定を読むので、違う場所か `(unset)` が表示されます）。CLI が無い場合は、`config.json` の `export.auto.path` と `export.auto.sourceId` を控えます。ただし、この 2 つだけでは実際の出力先を確定できないことがあります（`sourceId` が空のときは端末の名前が使われます。`sourceId` が絶対パスや `..` を含む値の場合や、途中にリンクがある場合は、`export.auto.path` の外に書かれることもあります）。**この場合は、控えた場所以外にも記録が残っていることがあります。**
3. **ディレクトリごと削除してよいかを決める。**
   - **ディレクトリごと削除してよいのは、そのディレクトリが Tokilog 専用で、ほかのデータを保存していないことを、ご自身が把握している場合だけです**（既定の場所でも、`TOKILOG_DATA_DIR` で指定した場所でも同じです）。そう言い切れるときは、ディレクトリごと削除します（エクスプローラーでゴミ箱へ移すなど）。**ゴミ箱へ移しただけでは記録は残っています。ディレクトリごとでもファイルを個別に消す場合でも、ゴミ箱を空にするまで削除は完了していません**
   - **それ以外の場合（ほかのデータも置いている・置いたか分からない場合）は、ディレクトリごとは削除しません。** 中に見えるファイルだけで専用と判断しないでください（通常の表示では見えないファイルもあります）。下の一覧の**名前と形に正確に当てはまる**ファイルだけを削除します。名前が似ていても形が違うものや、Tokilog が作ったと確かめられないものは削除しないでください
   - `config.json` と `export-state.json` はよくある名前です。**中を開いて Tokilog のものと確かめられた場合だけ**削除してください（Tokilog の `config.json` には `device` や `export` の項目があります）
4. **ログを削除する（不要な場合）。** ログ `tokilog.log` は、`TOKILOG_DATA_DIR` を設定していても、既定の保存先（`%LOCALAPPDATA%\Tokilog`）に置かれます。`TOKILOG_LOG_PATH` を設定していた場合は、その場所です。

Tokilog が置くファイル（データディレクトリの中）:

- `tokilog.db`（記録の DB）
- `tokilog.db-wal`・`tokilog.db-shm`・`tokilog.db-journal`（SQLite の付随ファイル）
- `tokilog.db.upgrade.lock`（upgrade のロック用のファイル。**記録は入っていません。個別に削除する場合は、このファイルは残してください** —— Tokilog が動いているあいだに消すと、同時に DB の更新などを始めないようにする仕組みが働かなくなります。ディレクトリごと削除する場合は一緒に消えてかまいません）
- DB の更新・復元・ロックの解除の前に Tokilog が自動で作る写し。**これらにも記録が入っています**
  - `tokilog.db.before-upgrade-<日時>.bak`・`tokilog.db.before-restore-<日時>.bak`・`tokilog.db.before-unlock-<日時>.bak`
  - `<日時>` は 17 桁の数字です（UTC の年月日時分秒とミリ秒）。同じ日時の写しがあったときは、その後ろに `-` と、32 文字の小文字の 16 進数（`0`〜`9` と `a`〜`f`）が付きます（例: `tokilog.db.before-upgrade-<日時>-<32 文字の 16 進数>.bak`）
  - 写しを作っている途中で終了したときは、`tokilog.db.before-upgrade-<日時>.<32 文字の小文字の 16 進数>.partial` の形の作りかけのファイルが残ることがあります（`before-restore`・`before-unlock` も同じ形）
- `config.json`（自動エクスポートの設定と、この端末の識別子）
- `export-state.json`（自動エクスポートの状態）
- `tokilog.log`（既定の保存先を使っている場合のログ）

`tokilog.db` だけを削除して、付随ファイルや写しを残さないでください。記録が残るうえ、付随ファイルが残っていると、次に Tokilog を使い始めたときや新しい DB を置いたときに、古い記録の一部が見えることがあります。

**この手順で消えない記録もあります。** この案内が扱うのは、Tokilog のデータディレクトリの中の既知のファイルと、ご自身が選んだ外部の保存先についての一般的な注意までで、すべての記録を漏れなく消せることを保証するものではありません。記録を残さず消したい場合は、次も確かめてください。

- **自動エクスポートの出力**: 自動エクスポートを使っていた場合、出力先（手順 2 で控えた場所）の年と月のフォルダーの下（`YYYY/MM/`）に日ごとのファイル（`YYYY-MM-DD.<拡張子>`。書き込みの途中で止まった場合は、一時ファイル `YYYY-MM-DD.<拡張子>.tmp` や `YYYY-MM-DD.<拡張子>.<32 文字の小文字の 16 進数>.tmp` も）が残ります。**`export.auto.path`・`export.auto.sourceId`・端末の名前を途中で変えて使っていた場合は、以前の出力先にも記録が残ることがあります**（手順 2 で控えた場所は、今の設定から決まる場所だけです）。**前の版で `export.auto.path` に相対パス（ルートから始まらない形）を設定して使っていた場合、出力先は自動エクスポートを実行したときの作業フォルダーを基準に決まっていました。** 実行した場所ごとに、その下の出力先を確かめてください（この版からは相対パスは使えず、自動エクスポートは書かずに止まり、`tl export auto status` の `output root` は `(invalid)` と表示されます）。出力先の保存先と、そこがご自身のものであることを確かめた場合だけ削除してください
- **自分で作ったバックアップ**: `tl db backup --output` で作った DB は、指定した場所に残ります
- **手動で書き出したファイル**: `tl export` で書き出したファイル（CSV・TSV・Markdown など）は、書き出した場所に残ります
- **以前の `TOKILOG_DB_PATH` の DB**: `TOKILOG_DB_PATH` から移行するときに DB を新しいデータディレクトリへ**コピー**した場合、以前 `TOKILOG_DB_PATH` で指定していた場所に元の DB が残ります。同じ場所に、SQLite の付随ファイル（元の DB のファイル名の後ろに `-wal`・`-shm`・`-journal` が付いたもの）や、それまでに作られた写し（`.bak`）が残っていることもあります。**保存先と、ご自身のものであることを確かめられたものだけ**を削除してください
- **保存先にリンクがある場合**: データディレクトリ自体、その途中のフォルダー、`tokilog.db` のどれかがほかの場所を指すリンクの場合、実体の DB は別の場所にあります（手順 2 で控えた場所です）。DB の更新などの前の写し（`.bak`）とロック用のファイルも、実体の DB と同じ場所に作られることがあります。**実体の保存先がどこで、ご自身のものであることを確かめた場合だけ**、実体の側も削除してください。**実体のファイル名が `tokilog.db` でない場合は、上の一覧の `tokilog.db` を実体のファイル名に置き換えた名前と形で探します**（例: 実体が `real.db` なら `real.db`・`real.db-wal`・`real.db.before-upgrade-<日時>.bak` など）。実体のディレクトリがほかの用途と共有なら、ディレクトリごとは削除しません。**実体の場所を確かめられない場合は、記録の削除は完了していないものと考えてください**

Tab 補完を設定していた場合は、PowerShell profile (`$profile`) から Tokilog 補完の読み込みブロックを削除し、専用ファイルも削除してください。

```powershell
# PowerShell profile から次のブロックを削除
# Tokilog completion
$tokilogCompletion = Join-Path $HOME ".config/tokilog/completion.ps1"
if (Test-Path $tokilogCompletion) {
  . $tokilogCompletion
}

# 専用ファイルも削除
# セッション内で変数が既に定義済みの場合は省略可能です
# 新しいセッションでは次のように定義します
$tokilogCompletion = Join-Path $HOME ".config/tokilog/completion.ps1"
Remove-Item $tokilogCompletion
```

## トラブルシュート

| 症状 | 確認すること |
|---|---|
| `tl` が見つからない | PATH に追加したフォルダーが `tl.exe` のあるフォルダーか確認し、新しい PowerShell を開き直す |
| .NET Runtime のエラーが出る | `dotnet --list-runtimes` で .NET 10 Runtime または SDK が入っているか確認する |
| DB 作成に失敗する | `%LOCALAPPDATA%\Tokilog` を作成できる権限があるか確認する |
| 記録データが見つからない | `TOKILOG_DATA_DIR` を設定して別のデータディレクトリを参照していないか確認する |
| エラーの原因を詳しく見たい | `tl doctor`、`tl --debug today`、ログファイルを確認する |

DB と log の場所:

```text
DB : %LOCALAPPDATA%\Tokilog\tokilog.db
log: %LOCALAPPDATA%\Tokilog\tokilog.log
```
