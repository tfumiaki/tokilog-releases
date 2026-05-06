# Tokilog CLI macOS Getting Started

このドキュメントは、macOS で Tokilog CLI (`tl`) を手動導入して使い始めるための一般ユーザー向け手順です。

Tokilog はローカル保存を前提にした軽量な時間記録ツールです。作業開始、停止、今日の記録確認を CLI から短いコマンドで実行できます。

## 対象環境

- macOS
- zsh
- Apple Silicon Mac (`osx-arm64`)
- .NET 10 Runtime または .NET 10 SDK

alpha で配布している macOS 向け成果物は **`osx-arm64` のみ**です。Intel Mac / `osx-x64` 向けの配布手順は現時点では未整備です。

配布される `tl` は `SelfContained=false` の single-file publish です。実行環境には **.NET 10 Runtime** が必要です。

```console
dotnet --list-runtimes
```

`.NET 10` の Runtime または SDK が表示されない場合は、.NET 10 Runtime をインストールしてください。

## 1. GitHub Releases から取得する

[Tokilog Releases](https://github.com/tfumiaki/tokilog-releases/releases) から macOS Apple Silicon 用 zip をダウンロードします。

```text
tokilog-net10.0-osx-arm64.zip
```

現在の publish 条件:

| 項目 | 値 |
|------|----|
| Target Framework | `net10.0` |
| Runtime Identifier | `osx-arm64` |
| Single file | `PublishSingleFile=true` |
| Self contained | `false` |

## 2. 展開先を決める

展開先は任意です。次のような場所を想定しています。

| 展開先 | 向いている使い方 |
|--------|------------------|
| `~/Applications/Tokilog` | Tokilog 用のフォルダーとしてまとめて置く |
| `~/.local/bin` | CLI ツールをまとめて PATH に置く |
| `~/bin` | 個人用コマンド置き場を使っている場合 |

`~/Applications/Tokilog` に展開する例:

```console
mkdir -p ~/Applications/Tokilog
unzip tokilog-net10.0-osx-arm64.zip -d /tmp/tokilog-osx-arm64
cp -R /tmp/tokilog-osx-arm64/osx-arm64/. ~/Applications/Tokilog/
```

`~/.local/bin` に `tl` を置く例:

```console
mkdir -p ~/.local/bin
unzip tokilog-net10.0-osx-arm64.zip -d /tmp/tokilog-osx-arm64
cp /tmp/tokilog-osx-arm64/osx-arm64/tl ~/.local/bin/
```

## 3. 実行権限を付与する

展開した `tl` に実行権限がない場合は、`chmod +x` を実行します。

```console
chmod +x ~/Applications/Tokilog/tl
```

`~/.local/bin` に置いた場合:

```console
chmod +x ~/.local/bin/tl
```

## 4. PATH に追加する

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

## 5. 動作確認する

Tokilog CLI の診断コマンドで、実行環境とローカル DB 状態を確認します。

```console
tl doctor
```

初回起動時に DB ファイルが存在しない場合は自動作成し、未適用の EF Core マイグレーションも自動適用します。

## 6. 最初に試すコマンド

```console
tl start "getting started" "@tokilog" "#docs"
tl current
tl stop
tl today
```

`tl start` で作業を開始し、`tl current` で実行中タイマーを確認します。作業が終わったら `tl stop` で停止し、`tl today` で今日の記録を確認します。未記録時間帯を確認したい場合は `tl today --gaps` を使います。

## DB 保存場所

Tokilog CLI はローカル SQLite DB に記録データを保存します。

macOS の既定 DB 保存場所:

```text
~/Library/Application Support/Tokilog/tokilog.db
```

Tokilog 本体を削除しても、この DB ファイルは自動では削除されません。記録データを残したい場合は、そのままにしてください。

開発・テスト用途では環境変数 `TOKILOG_DB_PATH` で DB パスを上書きできます。通常利用では既定保存場所のまま使うことを想定しています。本番利用中に上書きする場合は、別 DB に切り替わる点に注意してください。

```console
TOKILOG_DB_PATH=/tmp/tokilog-test.db tl today
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

Tokilog CLI は `System.CommandLine` の `[suggest]` directive と `dotnet-suggest` を使って Tab 補完を提供します。zsh で補完を使う場合は `dotnet-suggest` のセットアップが必要です。

最小セットアップ例:

```console
dotnet tool install -g dotnet-suggest
dotnet-suggest register --command-path ~/Applications/Tokilog/tl
dotnet-suggest script Zsh >> ~/.zshrc
source ~/.zshrc
```

`~/.local/bin/tl` を使っている場合は、`--command-path` に実際の `tl` のパスを指定してください。

Tokilog は alpha では `tl completion zsh` や `tl completion install` のような補完セットアップコマンドを提供しません。

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

`dotnet-suggest` は Tokilog 以外でも使う可能性があります。不要な場合のみアンインストールしてください。

```console
dotnet tool uninstall --global dotnet-suggest
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

macOS の Gatekeeper でブロックされた場合は、展開先の quarantine 属性を削除してから再実行してください。

```console
xattr -dr com.apple.quarantine ~/Applications/Tokilog
```

DB やログの状態を確認したい場合は、次のコマンドを使います。

```console
tl doctor
tl --debug today
```

DB 作成に失敗する場合は、`~/Library/Application Support/Tokilog` を作成できる権限があるか確認してください。
