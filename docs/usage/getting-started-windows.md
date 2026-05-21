# Tokilog CLI Windows Getting Started

このドキュメントは、Windows で Tokilog CLI (`tl`) を使い始めるための一般ユーザー向け手順です。

Tokilog はローカル保存を前提にした軽量な時間記録ツールです。作業の開始、現在のタイマー確認、停止、今日の記録確認を `tl` コマンドで行います。

この手順は Windows 向けです。macOS / Linux 向け Getting Started、MSI / MSIX などのインストーラー、`install.ps1` による自動導入、GUI アプリの導入手順は対象外です。

## 対象環境

- Windows 10 / 11
- x64 環境
- .NET 10 Runtime または .NET 10 SDK
- PowerShell

配布される `tl.exe` は `SelfContained=false` の single-file publish です。実行する PC には **.NET 10 Runtime** が必要です。

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
tokilog-net10.0-win-x64.zip
```

Release に Windows 用 zip がない場合、その Release では Windows 用の配布物がまだ公開されていません。

## 2. zip を展開する

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

## 3. PATH に追加する

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

## 4. 動作確認する

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

## PowerShell 特有の入力ルール

PowerShell では `@` と `#` が shell 固有の構文として扱われます。Tokilog で project (`@project`) や tag (`#tag`) を入力するときは、必ずダブルクォートで囲んでください。

| 文字 | PowerShell での扱い | Tokilog での入力例 |
|------|-------------------|------------------|
| `#` | コメント開始。`#` 以降の入力がすべて無視される | `"#docs"` |
| `@` | splatting 演算子。変数展開が試みられエラーになることがある | `"@tokilog"` |

```powershell
# 正しい: ダブルクォートで囲む
tl start "Getting Started を読む" "@tokilog" "#docs"

# NG: #docs 以降がコメントとして無視され、tag が渡されない
tl start "Getting Started を読む" "@tokilog" #docs

# NG: @tokilog が splatting として解釈され、予期しないエラーになる
tl start "Getting Started を読む" @tokilog "#docs"
```

`"#docs"` や `"@tokilog"` の引用符は PowerShell に対してのもので、Tokilog に渡る値は `#docs` / `@tokilog` と同じです。`#` の問題はエラーが出ず tag が無視されるだけなので特に注意してください。

## データ保存場所

Tokilog CLI は記録データをローカル SQLite DB に保存します。サーバーへの同期は行いません。

Windows のデフォルト DB 保存場所:

```text
%LOCALAPPDATA%\Tokilog\tokilog.db
```

DB ファイルは Tokilog 本体とは別に保存されます。zip の展開先フォルダーを削除しても、記録データは自動では削除されません。

### データディレクトリの上書き（開発・テスト用）

環境変数 `TOKILOG_DATA_DIR` にデータディレクトリの絶対パスを指定すると、データディレクトリを上書きできます。DB ファイル名は `tokilog.db` で固定です。`TOKILOG_DATA_DIR` にはディレクトリパスを指定します。`tokilog.db` まで含めてはいけません。通常利用ではデフォルト保存場所のまま使うことを想定しています。

```powershell
$env:TOKILOG_DATA_DIR = "C:\Users\me\AppData\Local\Tokilog"
tl today
# → DB ファイル: C:\Users\me\AppData\Local\Tokilog\tokilog.db
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

記録データも削除したい場合のみ、DB ファイルを削除します。

```text
%LOCALAPPDATA%\Tokilog\tokilog.db
```

ログファイルも不要であれば、手動で削除します。

```text
%LOCALAPPDATA%\Tokilog\tokilog.log
```

DB と log は Tokilog 本体とは別です。本体を削除しても自動では削除されません。記録データを残したい場合は DB ファイルを削除しないでください。

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
