# Tokilog macOS Getting Started

このドキュメントは、macOS で Tokilog CLI (`tl`) または Desktop GUI を手動導入して使い始めるための一般ユーザー向け手順です。

Tokilog はローカル保存を前提にした軽量な時間記録ツールです。CLI では作業開始、停止、今日の記録確認を短いコマンドで実行できます。Desktop GUI は同じローカル DB と Application ロジックを使う alpha build です。

<!-- toc -->

## 目次

- [対象環境](#対象環境)
- [1. GitHub Releases から取得する](#1-github-releases-から取得する)
- [2. CLI の展開先を決める](#2-cli-の展開先を決める)
- [3. CLI に実行権限を付与する](#3-cli-に実行権限を付与する)
- [4. CLI を PATH に追加する](#4-cli-を-path-に追加する)
- [5. CLI の動作確認をする](#5-cli-の動作確認をする)
- [6. 最初に試すコマンド](#6-最初に試すコマンド)
- [Desktop GUI を試す場合](#desktop-gui-を試す場合)
- [DB 保存場所](#db-保存場所)
  - [データディレクトリの上書き（開発・テスト用）](#データディレクトリの上書き開発テスト用)
  - [既存 DB を使い続ける場合（`TOKILOG_DB_PATH` からの移行）](#既存-db-を使い続ける場合tokilog_db_path-からの移行)
- [ログ保存場所](#ログ保存場所)
- [`--debug` を使う](#--debug-を使う)
- [補完を使う場合](#補完を使う場合)
- [アンインストール](#アンインストール)
  - [記録データも削除する（必要な場合のみ）](#記録データも削除する必要な場合のみ)
- [トラブルシュート](#トラブルシュート)

<!-- /toc -->

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

**展開先と `~/Applications/Tokilog` は、上書きではなく作り直します。** `unzip` も `cp -R` も既存のディレクトリへマージするため、新しい版で消えたファイルが古いまま残ります。毎回消してから展開・複写します。**`rm -rf` は指定したディレクトリを中身ごと消すので、下の手順を実行する前に記録の保存先を確認してください。** 既定の保存先（`~/Library/Application Support/Tokilog` 配下）は `~/Applications/Tokilog` の外にあるため、既定のまま使っているなら消えるのは Tokilog 本体だけです。**`TOKILOG_DATA_DIR` または `TOKILOG_LOG_PATH` が `~/Applications/Tokilog` またはその配下（あるいは展開先 `/tmp/tokilog-osx-arm64` の配下）を指している場合は、DB・設定・export state・ログも一緒に消えます。** その場合は、記録を配布先の外へ移すか、設定を配布先の外のパスへ向け直すまで、下の手順を実行しないでください。

```console
mkdir -p ~/Applications \
  && rm -rf /tmp/tokilog-osx-arm64 \
  && unzip tokilog-cli-net10.0-osx-arm64.zip -d /tmp/tokilog-osx-arm64 \
  && test -f /tmp/tokilog-osx-arm64/osx-arm64/tl \
  && rm -rf ~/Applications/Tokilog \
  && cp -R /tmp/tokilog-osx-arm64/osx-arm64/. ~/Applications/Tokilog/
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

Desktop GUI を試す場合は、`tokilog-desktop-net10.0-osx-arm64.zip` を展開し、中に入っている `Tokilog.app` を任意のフォルダーへ移して起動します。

```console
mkdir -p ~/Applications
rm -rf /tmp/tokilog-desktop-osx-arm64
unzip tokilog-desktop-net10.0-osx-arm64.zip -d /tmp/tokilog-desktop-osx-arm64
rm -rf ~/Applications/Tokilog.app
cp -R /tmp/tokilog-desktop-osx-arm64/osx-arm64/Tokilog.app ~/Applications/
```

`chmod +x` は必要ありません。実行権限は zip に記録されています。

**展開先と `Tokilog.app` は、上書きではなく作り直します。** `unzip` も `cp -R` も既存のディレクトリへマージするため、新しい版で消えたファイルが古いまま残ります。残ったファイルはバンドルの中から読み込まれうるので、毎回消してから展開・複写します。**消えるのはアプリ本体だけです** —— 記録した時間や設定は `~/Library/Application Support` 配下（または `TOKILOG_DATA_DIR` の指す先）にあり、`Tokilog.app` の中にはありません。

以前の zip は平置きのファイル群で、`Tokilog.Desktop` を直接起動する形でした。**asset 名は同じままで中身が変わっています。**

macOS Desktop GUI は experimental unsigned build です。Developer ID signing / notarization / stapling / dmg packaging は未対応です。そのため、macOS の Gatekeeper により初回起動がブロックされる可能性があります。

信頼できる配布物であることを確認できる場合のみ、Finder で `Tokilog.app` を Control-click → Open するか、System Settings の Privacy & Security から Open Anyway して起動してください。

将来的に signing / notarization / dmg packaging を検討します。

Desktop GUI も CLI と同じローカル SQLite DB を使います。初回セットアップや schema upgrade が必要な場合は、起動画面に [Set up database] / [Upgrade database] が出ます。ほかの Tokilog（CLI を含む）を閉じてから押すと、DB の写しを DB の隣に保存してから upgrade します。CLI で `tl upgrade` を実行してから Desktop GUI を起動しても構いません。

## DB 保存場所

Tokilog CLI はローカル SQLite DB に記録データを保存します。

macOS の既定 DB 保存場所:

```text
~/Library/Application Support/Tokilog/tokilog.db
```

Tokilog 本体を削除しても、この DB ファイルは自動では削除されません。記録データを残したい場合は、そのままにしてください。

### データディレクトリの上書き（開発・テスト用）

環境変数 `TOKILOG_DATA_DIR` にデータディレクトリの絶対パスを指定するとデータディレクトリを上書きできます。DB ファイル名は `tokilog.db` で固定です。`TOKILOG_DATA_DIR` にはディレクトリパスを指定します。`tokilog.db` まで含めてはいけません。通常利用では既定保存場所のまま使うことを想定しています。

**`TOKILOG_LOG_PATH` も一緒に指定してください。** ログの保存場所は `TOKILOG_DATA_DIR` に従わないため、これを省くと、テスト用の領域を指定したつもりでもエラーログだけが普段の場所に出ます。

```console
export TOKILOG_DATA_DIR="$HOME/Library/Application Support/Tokilog"
export TOKILOG_LOG_PATH="$HOME/Library/Application Support/Tokilog/tokilog.log"
tl today
# → DB ファイル: ~/Library/Application Support/Tokilog/tokilog.db
# → ログファイル: ~/Library/Application Support/Tokilog/tokilog.log
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

ログは **OS 既定の** Tokilog アプリデータディレクトリ配下に保存されます。**この場所は `TOKILOG_DATA_DIR` の影響を受けません** —— `TOKILOG_DATA_DIR` で DB を移しても、ログは下記の場所に出ます。移したい場合は `TOKILOG_LOG_PATH` を指定してください。

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

### 記録データも削除する（必要な場合のみ）

記録データは Tokilog 本体とは別の場所にあり、本体を削除しても自動では削除されません。記録を残したい場合は、この手順を行わないでください。

1. **Tokilog（CLI の `tl` と Desktop）をすべて閉じる。**
2. **記録の保存先（データディレクトリ）を確かめる。** 既定は `~/Library/Application Support/Tokilog` です。`TOKILOG_DATA_DIR` を設定して使っていた場合は、その値のディレクトリです。**既定の場所と `TOKILOG_DATA_DIR` を使い分けていた場合や、`TOKILOG_DATA_DIR` に違う値を指定したことがある場合は、既定の場所と、過去に指定した各 `TOKILOG_DATA_DIR` の場所のそれぞれに記録が残っていることがあります。** それぞれの場所について、この手順 2 以降を行ってください（ディレクトリごと削除してよいかは、場所ごとに手順 3 で決めます）。**あわせて、データディレクトリ自体、その途中のフォルダー、`tokilog.db` のどれかが、ほかの場所を指すリンク（シンボリックリンク。Windows ではジャンクションなど）でないかを確かめ、リンクがあれば実体の場所を控えておいてください**（削除した後では辿れません。リンクを削除しても実体は残ります。下の「この手順で消えない記録」を参照）。**自動エクスポートを使っていた場合は、実際の出力先も控えておいてください**（`config.json` を削除した後では分かりません）。CLI が使えるなら、`tl export auto status` を実行し、`output root` に表示される場所を控えます（`TOKILOG_DATA_DIR` を設定して使っていた場合は、同じ値を指定して実行します。指定しないと、既定の場所の設定を読むので、違う場所か `(unset)` が表示されます）。CLI が無い場合は、`config.json` の `export.auto.path` と `export.auto.sourceId` を控えます。ただし、この 2 つだけでは実際の出力先を確定できないことがあります（`sourceId` が空のときは端末の名前が使われます。`sourceId` が絶対パスや `..` を含む値の場合や、途中にリンクがある場合は、`export.auto.path` の外に書かれることもあります）。**この場合は、控えた場所以外にも記録が残っていることがあります。**
3. **ディレクトリごと削除してよいかを決める。**
   - **ディレクトリごと削除してよいのは、そのディレクトリが Tokilog 専用で、ほかのデータを保存していないことを、ご自身が把握している場合だけです**（既定の場所でも、`TOKILOG_DATA_DIR` で指定した場所でも同じです）。そう言い切れるときは、ディレクトリごと削除します（Finderでゴミ箱へ移すなど）。**ゴミ箱へ移しただけでは記録は残っています。ディレクトリごとでもファイルを個別に消す場合でも、ゴミ箱を空にするまで削除は完了していません**
   - **それ以外の場合（ほかのデータも置いている・置いたか分からない場合）は、ディレクトリごとは削除しません。** 中に見えるファイルだけで専用と判断しないでください（通常の表示では見えないファイルもあります）。下の一覧の**名前と形に正確に当てはまる**ファイルだけを削除します。名前が似ていても形が違うものや、Tokilog が作ったと確かめられないものは削除しないでください
   - `config.json` と `export-state.json` はよくある名前です。**中を開いて Tokilog のものと確かめられた場合だけ**削除してください（Tokilog の `config.json` には `device` や `export` の項目があります）
4. **ログを削除する（不要な場合）。** ログ `tokilog.log` は、`TOKILOG_DATA_DIR` を設定していても、既定の保存先（`~/Library/Application Support/Tokilog`）に置かれます。`TOKILOG_LOG_PATH` を設定していた場合は、その場所です。

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
- **別の環境へ移すための束**: `tl db bundle create --output` で指定した場所に作った束には、名前や拡張子にかかわらず暗号化されていない記録が入っています。標準の拡張子は `.tokilog-backup` ですが、別の名前や拡張子なしで保存した束も残ります
- **手動で書き出したファイル**: `tl export` で書き出したファイル（CSV・TSV・Markdown など）は、書き出した場所に残ります
- **以前の `TOKILOG_DB_PATH` の DB**: `TOKILOG_DB_PATH` から移行するときに DB を新しいデータディレクトリへ**コピー**した場合、以前 `TOKILOG_DB_PATH` で指定していた場所に元の DB が残ります。同じ場所に、SQLite の付随ファイル（元の DB のファイル名の後ろに `-wal`・`-shm`・`-journal` が付いたもの）や、それまでに作られた写し（`.bak`）が残っていることもあります。**保存先と、ご自身のものであることを確かめられたものだけ**を削除してください
- **保存先にリンクがある場合**: データディレクトリ自体、その途中のフォルダー、`tokilog.db` のどれかがほかの場所を指すリンクの場合、実体の DB は別の場所にあります（手順 2 で控えた場所です）。DB の更新などの前の写し（`.bak`）とロック用のファイルも、実体の DB と同じ場所に作られることがあります。**実体の保存先がどこで、ご自身のものであることを確かめた場合だけ**、実体の側も削除してください。**実体のファイル名が `tokilog.db` でない場合は、上の一覧の `tokilog.db` を実体のファイル名に置き換えた名前と形で探します**（例: 実体が `real.db` なら `real.db`・`real.db-wal`・`real.db.before-upgrade-<日時>.bak` など）。実体のディレクトリがほかの用途と共有なら、ディレクトリごとは削除しません。**実体の場所を確かめられない場合は、記録の削除は完了していないものと考えてください**

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
