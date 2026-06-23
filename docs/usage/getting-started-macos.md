# Tokilog macOS Getting Started

このドキュメントは、macOS で Tokilog CLI (`tl`) または Desktop GUI を手動導入して使い始めるための一般ユーザー向け手順です。

Tokilog はローカル保存を前提にした軽量な時間記録ツールです。CLI では作業開始、停止、今日の記録確認を短いコマンドで実行できます。Desktop GUI は同じローカル DB と Application ロジックを使う alpha build です。

## 対象環境

- macOS
- zsh
- Apple Silicon Mac (`osx-arm64`)
- .NET 10 Runtime

現在配布している macOS 向け成果物は **`osx-arm64` のみ**です。Intel Mac / `osx-x64` 向けの配布手順は現時点では未整備です。

配布される CLI (`tl`) と Desktop GUI は `SelfContained=false` の publish です。実行環境には **.NET 10 Runtime** が必要です。

```console
dotnet --list-runtimes
```

`.NET 10` の Runtime または SDK が表示されない場合は、.NET 10 Runtime をインストールしてください。

## 1. GitHub Releases から取得する

[Tokilog Releases](https://github.com/tfumiaki/tokilog-releases/releases) から macOS Apple Silicon 用 zip をダウンロードします。

```text
CLI
tokilog-cli-net10.0-osx-arm64.zip

Desktop GUI
tokilog-desktop-net10.0-osx-arm64.zip
```

CLI を使う場合は `tokilog-cli-net10.0-osx-arm64.zip`、Desktop GUI を試す場合は `tokilog-desktop-net10.0-osx-arm64.zip` を取得してください。

現在の publish 条件:

| 配布物 | Target Framework | Runtime Identifier | Single file | Self contained |
|--------|------------------|--------------------|-------------|----------------|
| CLI | `net10.0` | `osx-arm64` | `PublishSingleFile=true` | `false` |
| Desktop GUI | `net10.0` | `osx-arm64` | `false` | `false` |

## 2. CLI の展開先を決める

展開先は任意です。次のような場所を想定しています。

| 展開先 | 向いている使い方 |
|--------|------------------|
| `~/Applications/Tokilog` | Tokilog 用のフォルダーとしてまとめて置く |
| `~/.local/bin` | CLI ツールをまとめて PATH に置く |
| `~/bin` | 個人用コマンド置き場を使っている場合 |

`~/Applications/Tokilog` に展開する例:

```console
mkdir -p ~/Applications/Tokilog
unzip tokilog-cli-net10.0-osx-arm64.zip -d /tmp/tokilog-osx-arm64
cp -R /tmp/tokilog-osx-arm64/osx-arm64/. ~/Applications/Tokilog/
```

`~/.local/bin` に `tl` を置く例:

```console
mkdir -p ~/.local/bin
unzip tokilog-cli-net10.0-osx-arm64.zip -d /tmp/tokilog-osx-arm64
cp /tmp/tokilog-osx-arm64/osx-arm64/tl ~/.local/bin/
```

## 3. CLI に実行権限を付与する

展開した `tl` に実行権限がない場合は、`chmod +x` を実行します。

```console
chmod +x ~/Applications/Tokilog/tl
```

`~/.local/bin` に置いた場合:

```console
chmod +x ~/.local/bin/tl
```

## 4. CLI を PATH に追加する

zsh では、`~/.zshrc` に Tokilog の展開先を追加します。

`~/Applications/Tokilog` に展開した場合:

```console
echo 'export PATH="$HOME/Applications/Tokilog:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

`~/.local/bin` に置いた場合:

```console
echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc
```

設定後、新しいターミナルを開くか `source ~/.zshrc` で再読み込みしてから確認します。

```console
tl --help
```

## 5. CLI の動作確認をする

まず `tl upgrade` を実行して、初回セットアップに必要な DB migration を適用します。その後、`tl doctor` で実行環境とローカル DB 状態を確認します。

```console
tl upgrade
tl doctor
```

Tokilog を新しいバージョンへ更新した後も、最初に `tl upgrade` を実行してから通常コマンドを使ってください。

## 6. 最初に試すコマンド

```console
tl start "getting started" "@tokilog" "#docs"
tl current
tl stop
tl today
```

`tl start` で作業を開始し、`tl current` で実行中タイマーを確認します。作業が終わったら `tl stop` で停止し、`tl today` で今日の記録を確認します。未記録時間帯を確認したい場合は `tl today --gaps` を使います。

## Desktop GUI を試す場合

Desktop GUI を試す場合は、`tokilog-desktop-net10.0-osx-arm64.zip` を任意のフォルダーに展開し、展開先の `Tokilog.Desktop` を起動します。

```console
mkdir -p ~/Applications/TokilogDesktop
unzip tokilog-desktop-net10.0-osx-arm64.zip -d /tmp/tokilog-desktop-osx-arm64
cp -R /tmp/tokilog-desktop-osx-arm64/osx-arm64/. ~/Applications/TokilogDesktop/
chmod +x ~/Applications/TokilogDesktop/Tokilog.Desktop
```

macOS Desktop GUI は experimental unsigned build です。Developer ID signing / notarization / stapling / dmg packaging は未対応です。そのため、macOS の Gatekeeper により初回起動がブロックされる可能性があります。

信頼できる配布物であることを確認できる場合のみ、Finder で `Tokilog.Desktop` を Control-click → Open するか、System Settings の Privacy & Security から Open Anyway して起動してください。

将来的に signing / notarization / dmg packaging を検討します。

Desktop GUI も CLI と同じローカル SQLite DB を使います。初回セットアップや schema upgrade が必要な場合は、CLI で `tl upgrade` を実行してから Desktop GUI を起動してください。

## DB 保存場所

Tokilog CLI はローカル SQLite DB に記録データを保存します。

macOS の既定 DB 保存場所:

```text
~/Library/Application Support/Tokilog/tokilog.db
```

Tokilog 本体を削除しても、この DB ファイルは自動では削除されません。記録データを残したい場合は、そのままにしてください。

### データディレクトリの上書き（開発・テスト用）

環境変数 `TOKILOG_DATA_DIR` にデータディレクトリの絶対パスを指定するとデータディレクトリを上書きできます。DB ファイル名は `tokilog.db` で固定です。`TOKILOG_DATA_DIR` にはディレクトリパスを指定します。`tokilog.db` まで含めてはいけません。通常利用では既定保存場所のまま使うことを想定しています。

```console
export TOKILOG_DATA_DIR="$HOME/Library/Application Support/Tokilog"
tl today
# → DB ファイル: ~/Library/Application Support/Tokilog/tokilog.db
```

`TOKILOG_DATA_PATH` は採用していません（ファイルパスではなくディレクトリであることを明確にするため、`_DIR` を使用しています）。

> **破壊的変更**: `TOKILOG_DB_PATH` は廃止されました。以前 `TOKILOG_DB_PATH` でパスを指定していた場合は `TOKILOG_DATA_DIR` に移行してください。

### 既存 DB を使い続ける場合（`TOKILOG_DB_PATH` からの移行）

以前 `TOKILOG_DB_PATH` で別パスの DB ファイルを指定していた場合は、次の手順で既存 DB を使い続けられます。

1. 新しいデータディレクトリを作成する（例: `~/Library/Application Support/Tokilog`）
2. 以前 `TOKILOG_DB_PATH` で指定していた DB ファイルを `tokilog.db` という名前で上記ディレクトリへコピーまたは移動する
3. `TOKILOG_DATA_DIR` にそのディレクトリを設定する

```console
export TOKILOG_DATA_DIR="$HOME/Library/Application Support/Tokilog"
# → DB ファイル: ~/Library/Application Support/Tokilog/tokilog.db
```

## ログ保存場所

ログは DB と同じ Tokilog アプリデータディレクトリ配下に保存されます。

macOS の既定ログ保存場所:

```text
~/Library/Application Support/Tokilog/tokilog.log
```

ログファイルパスは `TOKILOG_LOG_PATH` で上書きできます。指定する場合は絶対パスを使います。

```console
TOKILOG_LOG_PATH=/tmp/tokilog.log tl today
```

## `--debug` を使う

原因調査が必要な場合は、`--debug` を付けると通常のエラー表示に加えて詳細な例外情報が標準エラー出力に表示されます。

```console
tl --debug today
tl today --debug
```

`--debug` は調査用です。記録データやコマンドの結果を変更するものではありません。

## 補完を使う場合

Tokilog CLI は `tl completion zsh` で zsh 向けの native completion script を出力します。

補完は任意機能です。command / option の静的候補に加えて、ローカル DB に存在する project / tag / detail を動的候補として表示できます。

セットアップ手順は [CLI Tab 補完の利用手順](./completion.md) を参照してください。補完スクリプトは専用ファイルへ保存し、`~/.zshrc` からそのファイルを読み込む構成を推奨します。Tokilog は `tl completion install` のような shell profile 自動編集コマンドは提供しません。

## アンインストール

Tokilog 本体を削除する場合は、展開先フォルダーまたは `tl` 実行ファイルを削除します。

`~/Applications/Tokilog` に展開した場合:

```console
rm -rf ~/Applications/Tokilog
```

`~/.local/bin` に `tl` を置いた場合:

```console
rm ~/.local/bin/tl
```

PATH に Tokilog の展開先を追加していた場合は、`~/.zshrc` から該当する行を削除してください。

```text
export PATH="$HOME/Applications/Tokilog:$PATH"
export PATH="$HOME/.local/bin:$PATH"
```

記録データも削除したい場合のみ、DB ファイルを削除します。ログも不要であれば削除してください。

```console
rm "$HOME/Library/Application Support/Tokilog/tokilog.db"
rm "$HOME/Library/Application Support/Tokilog/tokilog.log"
```

Tokilog 用のアプリデータディレクトリごと削除する場合:

```console
rm -rf "$HOME/Library/Application Support/Tokilog"
```

Tab 補完を設定していた場合は、`~/.zshrc` から Tokilog 補完の読み込みブロックを削除し、専用ファイルも削除してください。

```zsh
# ~/.zshrc から次のブロックを削除
# Tokilog completion
if [ -f "$HOME/.config/tokilog/completion.zsh" ]; then
  source "$HOME/.config/tokilog/completion.zsh"
fi

# 専用ファイルも削除
rm ~/.config/tokilog/completion.zsh
```

## トラブルシュート

`tl` が見つからない場合は、PATH に追加したフォルダーが正しいか確認し、新しいターミナルで再実行してください。

```console
which tl
echo $PATH
```

.NET Runtime のエラーが出る場合は、.NET 10 Runtime がインストールされているか確認してください。

```console
dotnet --list-runtimes
```

実行権限がない場合は、`tl` に実行権限を付与してください。

```console
chmod +x ~/Applications/Tokilog/tl
```

macOS Desktop GUI が Gatekeeper でブロックされた場合は、信頼できる配布物であることを確認できる場合のみ、Finder で Control-click → Open するか、System Settings の Privacy & Security から Open Anyway して起動してください。

DB やログの状態を確認したい場合は、次のコマンドを使います。

```console
tl doctor
tl --debug today
```

DB 作成に失敗する場合は、`~/Library/Application Support/Tokilog` を作成できる権限があるか確認してください。
