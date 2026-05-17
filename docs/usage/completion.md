# Tokilog CLI Tab 補完の利用手順

このドキュメントは、Tokilog CLI (`tl`) の Tab 補完を使うための利用者向け手順です。

Tab 補完は任意機能です。Tokilog CLI の初回利用に必須ではありません。

## 補完でできること

Tokilog CLI は `tl completion <shell>` で各 shell 向けの native completion script を標準出力へ出力します。

補完できる候補:

- 静的補完: command / subcommand 名、option 名
- 動的補完: ローカル DB に保存済みの project / tag / detail

動的補完は、`tl` の実行時に参照するローカル SQLite DB が存在する場合に自動的に有効になります。DB が存在しない、または読み取れない場合は空候補として扱われ、補完のエラーが通常コマンド実行に影響することはありません。

補完候補にない project / tag / detail も通常のコマンド入力として引き続き新規入力できます。

## 前提

利用する shell に応じて、次のいずれかのコマンドで completion script を出力できます。

```console
tl completion powershell
tl completion zsh
tl completion bash
```

`tl completion install` のような shell profile 自動編集コマンドは提供しません。出力された script をどの profile に保存・読み込みするかは利用者側で選択してください。

## セットアップの方針

補完セットアップでは、次の構成を推奨します。

1. `tl completion <shell>` の出力は、Tokilog 用の専用ファイルへ保存する
2. shell profile (`.zshrc` / `.bashrc` / PowerShell profile) には、専用ファイルを読み込む小さなブロックだけを追加する

この方式では、補完スクリプト本体を再セットアップする場合も専用ファイルを上書きするだけで済み、shell profile が重複や肥大化する問題を避けられます。

## zsh

### 1. 専用ファイルへ補完スクリプトを保存する

```zsh
mkdir -p ~/.config/tokilog
tl completion zsh > ~/.config/tokilog/completion.zsh
```

### 2. `~/.zshrc` から専用ファイルを読み込む

`~/.zshrc` に次のブロックを追加します。

```zsh
# Tokilog completion
if [ -f "$HOME/.config/tokilog/completion.zsh" ]; then
  source "$HOME/.config/tokilog/completion.zsh"
fi
```

**注意**: このブロックが既に `~/.zshrc` にある場合は、追加せずそのままにしてください。再セットアップ時は、専用ファイル (`~/.config/tokilog/completion.zsh`) を上書きするだけで、`~/.zshrc` への変更は不要です。

### 3. 再読み込みして確認する

```zsh
source ~/.zshrc
```

`source ~/.zshrc` で再読み込みできない場合は、ターミナルを再起動してください。

### 再セットアップする場合

Tokilog を更新した後など、補完スクリプトを再生成したい場合は、専用ファイルを上書きします。

```zsh
tl completion zsh > ~/.config/tokilog/completion.zsh
```

`~/.zshrc` の読み込みブロックはそのままで、ターミナルを再起動すれば新しい補完スクリプトが適用されます。

## bash

### 1. 専用ファイルへ補完スクリプトを保存する

```bash
mkdir -p ~/.config/tokilog
tl completion bash > ~/.config/tokilog/completion.bash
```

### 2. `~/.bashrc` から専用ファイルを読み込む

`~/.bashrc` に次のブロックを追加します。

```bash
# Tokilog completion
if [ -f "$HOME/.config/tokilog/completion.bash" ]; then
  source "$HOME/.config/tokilog/completion.bash"
fi
```

macOS などで login shell に `~/.bash_profile` を使っている場合は、`~/.bashrc` の代わりに `~/.bash_profile` へ追加してください。

**注意**: このブロックが既に `~/.bashrc` (または `~/.bash_profile`) にある場合は、追加せずそのままにしてください。再セットアップ時は、専用ファイル (`~/.config/tokilog/completion.bash`) を上書きするだけで、`~/.bashrc` への変更は不要です。

### 3. 再読み込みして確認する

```bash
source ~/.bashrc
```

`source ~/.bashrc` で再読み込みできない場合は、ターミナルを再起動してください。

### 再セットアップする場合

補完スクリプトを再生成したい場合は、専用ファイルを上書きします。

```bash
tl completion bash > ~/.config/tokilog/completion.bash
```

`~/.bashrc` (または `~/.bash_profile`) の読み込みブロックはそのままで、ターミナルを再起動すれば新しい補完スクリプトが適用されます。

## PowerShell

### 1. 専用ファイルへ補完スクリプトを保存する

```powershell
$tokilogConfigDir = Join-Path $HOME ".config/tokilog"
if (!(Test-Path $tokilogConfigDir)) { New-Item -ItemType Directory -Path $tokilogConfigDir -Force | Out-Null }
tl completion powershell > (Join-Path $tokilogConfigDir "completion.ps1")
```

### 2. PowerShell profile から専用ファイルを読み込む

PowerShell profile のパスは `$profile` で確認できます。profile ファイルがない場合は作成してから、次のブロックを追加します。

```powershell
if (!(Test-Path $profile)) { New-Item -ItemType File -Path $profile -Force | Out-Null }
```

profile ファイルをエディターで開き、次のブロックを追加します。

```powershell
# Tokilog completion
$tokilogCompletion = Join-Path $HOME ".config/tokilog/completion.ps1"
if (Test-Path $tokilogCompletion) {
  . $tokilogCompletion
}
```

**注意**: このブロックが既に PowerShell profile にある場合は、追加せずそのままにしてください。再セットアップ時は、専用ファイル (`$HOME/.config/tokilog/completion.ps1`) を上書きするだけで、PowerShell profile への変更は不要です。

### 3. 再読み込みして確認する

```powershell
. $profile
```

`. $profile` で再読み込みできない場合は、新しい PowerShell を開いてください。

### 再セットアップする場合

補完スクリプトを再生成したい場合は、専用ファイルを上書きします。

```powershell
$tokilogConfigDir = Join-Path $HOME ".config/tokilog"
tl completion powershell > (Join-Path $tokilogConfigDir "completion.ps1")
```

PowerShell profile の読み込みブロックはそのままで、新しい PowerShell を開けば新しい補完スクリプトが適用されます。

## 補完候補が出ない場合

補完候補が出ない場合は、次の点を確認してください。

- `tl` が PATH から実行できるか
- 専用ファイル (`~/.config/tokilog/completion.zsh` など) が存在するか
- shell profile に専用ファイルを読み込むブロックが追加されているか
- shell を再起動したか、profile を再読み込みしたか
- 利用している shell と、専用ファイルの種類が一致しているか (zsh なら `.zsh`、bash なら `.bash`、PowerShell なら `.ps1`)

## アンインストール / 設定解除

Tab 補完を使わない場合は、次の手順で設定を解除します。

### zsh

1. `~/.zshrc` から Tokilog 補完の読み込みブロックを削除する

```zsh
# Tokilog completion
if [ -f "$HOME/.config/tokilog/completion.zsh" ]; then
  source "$HOME/.config/tokilog/completion.zsh"
fi
```

2. 専用ファイルを削除する

```zsh
rm ~/.config/tokilog/completion.zsh
```

### bash

1. `~/.bashrc` (または `~/.bash_profile`) から Tokilog 補完の読み込みブロックを削除する

```bash
# Tokilog completion
if [ -f "$HOME/.config/tokilog/completion.bash" ]; then
  source "$HOME/.config/tokilog/completion.bash"
fi
```

2. 専用ファイルを削除する

```bash
rm ~/.config/tokilog/completion.bash
```

### PowerShell

1. PowerShell profile (`$profile`) から Tokilog 補完の読み込みブロックを削除する

```powershell
# Tokilog completion
$tokilogCompletion = Join-Path $HOME ".config/tokilog/completion.ps1"
if (Test-Path $tokilogCompletion) {
  . $tokilogCompletion
}
```

2. 専用ファイルを削除する

```powershell
# セッション内で変数が既に定義済みの場合は省略可能です
# 新しいセッションでは次のように定義します
$tokilogCompletion = Join-Path $HOME ".config/tokilog/completion.ps1"
Remove-Item $tokilogCompletion
```
