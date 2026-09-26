# Tokilog Linux Getting Started

このドキュメントは、Linux (Ubuntu) で Tokilog CLI (`tl`) または Desktop GUI を使い始めるための一般ユーザー向け手順です。

Tokilog はローカル保存を前提にした軽量な時間記録ツールです。CLI では作業の開始、現在のタイマー確認、停止、今日の記録確認を `tl` コマンドで行います。Desktop GUI は同じローカル DB と Application ロジックを使う alpha build です。

この手順は Linux (Ubuntu) 向けです。macOS / Windows 向け Getting Started、deb / AppImage / Flatpak などのパッケージ、自動導入スクリプトは対象外です。

<!-- toc -->

## 目次

- [対象環境](#対象環境)
  - [Desktop GUI に必要な OS パッケージ](#desktop-gui-に必要な-os-パッケージ)
- [1. zip を取得する](#1-zip-を取得する)
- [2. CLI zip を展開する](#2-cli-zip-を展開する)
  - [実行権限を付与する](#実行権限を付与する)
- [3. CLI を PATH に追加する](#3-cli-を-path-に追加する)
- [4. CLI の動作確認をする](#4-cli-の動作確認をする)
- [5. 最初に試すコマンド](#5-最初に試すコマンド)
- [Desktop GUI を試す場合](#desktop-gui-を試す場合)
- [データ保存場所](#データ保存場所)
  - [データディレクトリの上書き（開発・テスト用）](#データディレクトリの上書き開発テスト用)
- [ログ保存場所と --debug](#ログ保存場所と---debug)
- [Tab 補完を使う場合](#tab-補完を使う場合)
- [アンインストール](#アンインストール)
- [トラブルシュート](#トラブルシュート)

<!-- /toc -->

## 対象環境

- Ubuntu 22.04 / 24.04（x64）
- .NET 10 Runtime または .NET 10 SDK
- Desktop GUI を使う場合は X11（または XWayland）環境

配布される CLI (`tl`) と Desktop GUI は `SelfContained=false` の publish です。実行する PC には **.NET 10 Runtime** が必要です。

次のコマンドで、.NET 10 の Runtime または SDK が表示されるか確認してください。

```bash
dotnet --list-runtimes
```

.NET 10 が表示されない場合は、Microsoft のパッケージフィード（`apt install dotnet-runtime-10.0`）または公式の `dotnet-install.sh` から .NET 10 Runtime をインストールしてから続けてください。

### Desktop GUI に必要な OS パッケージ

Desktop GUI (Avalonia) を動かすには、X11 とフォント関連のランタイムライブラリが必要です。最小構成の環境では次を導入してください（多くのデスクトップ環境では既に入っています）。

```bash
sudo apt install libx11-6 libice6 libsm6 libfontconfig1
```

CLI のみを使う場合、これらの GUI ライブラリは不要です。

`libicu`（.NET のグローバリゼーション用）は通常 .NET 10 Runtime のインストールと一緒に入ります。最小構成で不足する場合は、Ubuntu のバージョンに合わせて導入してください（24.04: `libicu74` / 22.04: `libicu70`）。

## 1. zip を取得する

GitHub Releases から Linux 用 zip をダウンロードします。

1. [Tokilog Releases](https://github.com/tfumiaki/tokilog-releases/releases) を開く
2. 利用したいバージョンの Release を開く
3. Assets から Linux 用 zip をダウンロードする

現在の Linux 用 asset 名は次を想定しています。

```text
CLI
tokilog-cli-net10.0-linux-x64.zip

Desktop GUI
tokilog-desktop-net10.0-linux-x64.zip
```

CLI を使う場合は `tokilog-cli-net10.0-linux-x64.zip`、Desktop GUI を試す場合は `tokilog-desktop-net10.0-linux-x64.zip` を取得してください。Release に Linux 用 zip がない場合、その Release では Linux 用の配布物がまだ公開されていません。

## 2. CLI zip を展開する

zip を任意のディレクトリに展開します。展開先は、あとで PATH に追加しやすく、誤って削除しにくい場所にしてください。

展開先の例:

```text
~/.local/bin/tokilog
~/tools/tokilog
```

zip を展開すると中に `linux-x64` ディレクトリが入っています。その中身を展開先へ移動します。

**展開先と `~/tools/tokilog` は、上書きではなく作り直します。** `unzip` も `cp -R` も既存のディレクトリへマージするため、新しい版で消えたファイルが古いまま残ります。毎回消してから展開・複写します。**`rm -rf` は指定したディレクトリを中身ごと消すので、下の手順を実行する前に記録の保存先を確認してください。** 既定の保存先（`~/.local/share/Tokilog` 配下、`$XDG_DATA_HOME` を**有効な絶対パスとして**設定している場合はその配下）は `~/tools/tokilog` の外にあるため、既定のまま使っているなら消えるのは Tokilog 本体だけです。**`TOKILOG_DATA_DIR`・`TOKILOG_LOG_PATH`・`XDG_DATA_HOME` のいずれか が `~/tools/tokilog` またはその配下（あるいは展開先 `/tmp/tokilog-linux-x64` の配下）を指している場合は、DB・設定・export state・ログも一緒に消えます。** その場合は、記録を配布先の外へ移すか、設定を配布先の外のパスへ向け直すまで、下の手順を実行しないでください。

```bash
mkdir -p ~/tools \
  && rm -rf /tmp/tokilog-linux-x64 \
  && unzip tokilog-cli-net10.0-linux-x64.zip -d /tmp/tokilog-linux-x64 \
  && test -f /tmp/tokilog-linux-x64/linux-x64/tl \
  && rm -rf ~/tools/tokilog \
  && cp -R /tmp/tokilog-linux-x64/linux-x64/. ~/tools/tokilog/
```

これで `~/tools/tokilog/tl` に CLI が配置されます。

### 実行権限を付与する

配布 zip は実行ビットを保持しないため、展開後に `tl` へ実行権限を付与します。

```bash
chmod +x ~/tools/tokilog/tl
```

## 3. CLI を PATH に追加する

`tl` をどのディレクトリからでも実行できるように、展開先を PATH に追加します。bash の例:

```bash
echo 'export PATH="$HOME/tools/tokilog:$PATH"' >> ~/.bashrc
source ~/.bashrc
```

zsh を使っている場合は `~/.zshrc` に追記してください。追加後、次で確認します。

```bash
tl --help
```

`tl` が見つからない場合は、PATH に追加したディレクトリが `tl` のあるディレクトリと一致しているか確認してください。

## 4. CLI の動作確認をする

まず `tl upgrade` で初回セットアップに必要な DB migration を適用し、続けて `tl doctor` で実行環境とローカル DB 状態を確認します。

```bash
tl upgrade
tl doctor
```

Tokilog を新しいバージョンへ更新した後も、最初に `tl upgrade` を実行してから通常コマンドを使ってください。

## 5. 最初に試すコマンド

```bash
tl start "Getting Started を読む" "@tokilog" "#docs"
tl current
tl stop
tl today
```

| コマンド | 説明 |
|---|---|
| `tl start` | 新しい実行中タイマーを開始する |
| `tl current` | 現在の実行中タイマーを表示する |
| `tl stop` | 実行中タイマーを停止する |
| `tl today` | 今日のエントリ一覧と合計時間を表示する |
| `tl today --gaps` | 今日の最初のエントリ開始から現在時刻までの未記録時間帯を表示する |

bash / zsh では `#` 以降がコメント扱いになるため、`@project` や `#tag` は例のようにクォートで囲むと安全です。detail に空白を含む場合もクォートしてください。

## Desktop GUI を試す場合

`tokilog-desktop-net10.0-linux-x64.zip` を展開し、中の `linux-x64` ディレクトリの中身を配置して `Tokilog.Desktop` を起動します。

**展開先と `~/tools/tokilog-desktop` は、上書きではなく作り直します。** `unzip` も `cp -R` も既存のディレクトリへマージするため、新しい版で消えたファイルが古いまま残ります。毎回消してから展開・複写します。**`rm -rf` は指定したディレクトリを中身ごと消すので、下の手順を実行する前に記録の保存先を確認してください。** 既定の保存先（`~/.local/share/Tokilog` 配下、`$XDG_DATA_HOME` を**有効な絶対パスとして**設定している場合はその配下）は `~/tools/tokilog-desktop` の外にあるため、既定のまま使っているなら消えるのは Tokilog 本体だけです。**`TOKILOG_DATA_DIR`・`TOKILOG_LOG_PATH`・`XDG_DATA_HOME` のいずれか が `~/tools/tokilog-desktop` またはその配下（あるいは展開先 `/tmp/tokilog-desktop-linux-x64` の配下）を指している場合は、DB・設定・export state・ログも一緒に消えます。** その場合は、記録を配布先の外へ移すか、設定を配布先の外のパスへ向け直すまで、下の手順を実行しないでください。

```bash
mkdir -p ~/tools \
  && rm -rf /tmp/tokilog-desktop-linux-x64 \
  && unzip tokilog-desktop-net10.0-linux-x64.zip -d /tmp/tokilog-desktop-linux-x64 \
  && test -f /tmp/tokilog-desktop-linux-x64/linux-x64/Tokilog.Desktop \
  && rm -rf ~/tools/tokilog-desktop \
  && cp -R /tmp/tokilog-desktop-linux-x64/linux-x64/. ~/tools/tokilog-desktop/ \
  && chmod +x ~/tools/tokilog-desktop/Tokilog.Desktop \
  && ~/tools/tokilog-desktop/Tokilog.Desktop
```

GUI の起動には上記「Desktop GUI に必要な OS パッケージ」が必要です。

Desktop GUI も CLI と同じローカル SQLite DB を使います。初回セットアップや schema upgrade が必要な場合は、起動画面に [Set up database] / [Upgrade database] が出ます。ほかの Tokilog（CLI を含む）を閉じてから押すと、DB の写しを DB の隣に保存してから upgrade します。CLI で `tl upgrade` を実行してから Desktop GUI を起動しても構いません。

## データ保存場所

Tokilog はローカル SQLite DB に記録データを保存します。サーバーへの同期は行いません。

Linux のデフォルト DB 保存場所（XDG 準拠）:

```text
~/.local/share/Tokilog/tokilog.db
```

`$XDG_DATA_HOME` を**有効な絶対パスとして**設定している場合は `$XDG_DATA_HOME/Tokilog/tokilog.db` になります（空文字や相対パスのときは使われず `~/.local/share/Tokilog/tokilog.db` になります。実行環境がホームディレクトリなどの基点を解決できない場合、この保存先は保証されないので、`TOKILOG_DATA_DIR` で保存先を明示してください）。DB ファイルは Tokilog 本体とは別に保存され、展開先を削除しても記録データは自動では削除されません。

### データディレクトリの上書き（開発・テスト用）

環境変数 `TOKILOG_DATA_DIR` にデータディレクトリの絶対パスを指定すると、データディレクトリを上書きできます。DB ファイル名は `tokilog.db` で固定です。`TOKILOG_DATA_DIR` にはディレクトリパスを指定し、`tokilog.db` まで含めてはいけません。

**`TOKILOG_LOG_PATH` も一緒に指定してください。** ログの保存場所は `TOKILOG_DATA_DIR` に従わないため、これを省くと、テスト用の領域を指定したつもりでもエラーログだけが普段の場所に出ます。

```bash
# これは「明示指定」の例です。書いたパスがそのまま使われ、$XDG_DATA_HOME は参照されません。
export TOKILOG_DATA_DIR="$HOME/.local/share/Tokilog"
export TOKILOG_LOG_PATH="$HOME/.local/share/Tokilog/tokilog.log"
tl today
# → DB ファイル: ~/.local/share/Tokilog/tokilog.db
# → ログファイル: ~/.local/share/Tokilog/tokilog.log
```

## ログ保存場所と --debug

原因調査が必要な実行時エラーはログファイルに記録されます。Linux のデフォルトログ保存場所（XDG 準拠）:

```text
~/.local/share/Tokilog/tokilog.log
```

`$XDG_DATA_HOME` を**有効な絶対パスとして**設定している場合は `$XDG_DATA_HOME/Tokilog/tokilog.log` になります（空文字や相対パスのときは使われず `~/.local/share/Tokilog/tokilog.log` になります。実行環境がホームディレクトリなどの基点を解決できない場合、この保存先は保証されないので、`TOKILOG_LOG_PATH` でログの保存先を明示してください）。

ログファイルの場所を変更したい場合は、環境変数 `TOKILOG_LOG_PATH` に絶対パスを指定します。詳細な例外情報を画面にも表示したい場合は `--debug` を指定します。

```bash
export TOKILOG_LOG_PATH="/tmp/tokilog.log"
tl --debug today
```

## Tab 補完を使う場合

Tokilog CLI は `tl completion bash` / `tl completion zsh` で shell 向けの補完スクリプトを出力します。補完は任意機能で、command / option の静的候補に加えて、ローカル DB の project / tag / detail を動的候補として表示できます。

Tokilog は shell profile を自動編集するコマンドは提供しません。詳細は [CLI Tab 補完の利用手順](./completion.md) を参照してください。

## アンインストール

Tokilog は展開先に置いた実行ファイルを PATH から呼び出す構成です。専用アンインストーラーはありません。

1. Tokilog の展開先ディレクトリを削除する
2. `~/.bashrc`（または `~/.zshrc`）に追加した PATH 行を削除する
3. 新しいシェルを開き、`tl --help` が見つからないことを確認する

記録データも削除したい場合のみ、DB / log ファイルを削除します。

```text
~/.local/share/Tokilog/tokilog.db
~/.local/share/Tokilog/tokilog.log
```

`$XDG_DATA_HOME` を**有効な絶対パスとして**設定している場合は `$XDG_DATA_HOME/Tokilog/` 配下です（空文字や相対パスのときは使われず `~/.local/share/Tokilog/` 配下になります。実行環境がホームディレクトリなどの基点を解決できない場合、この保存先は保証されません）。`TOKILOG_DATA_DIR` / `TOKILOG_LOG_PATH` を設定して使っていた場合は、そちらのパスを見てください。

DB と log は Tokilog 本体とは別です。本体を削除しても自動では削除されません。

## トラブルシュート

| 症状 | 確認すること |
|---|---|
| `tl` が見つからない | PATH に追加したディレクトリが `tl` のあるディレクトリか確認し、新しいシェルを開き直す |
| `Permission denied` で起動できない | `chmod +x` で実行権限を付与したか確認する |
| .NET Runtime のエラーが出る | `dotnet --list-runtimes` で .NET 10 Runtime または SDK が入っているか確認する |
| Desktop GUI が起動しない | 「Desktop GUI に必要な OS パッケージ」を導入し、X11 / XWayland 環境か確認する |
| 記録データが見つからない | `TOKILOG_DATA_DIR` で別のデータディレクトリを参照していないか確認する |
| エラーの原因を詳しく見たい | `tl doctor`、`tl --debug today`、ログファイルを確認する |
