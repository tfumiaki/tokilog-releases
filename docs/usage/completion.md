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

## zsh

zsh で補完を使う場合は、`tl completion zsh` の出力を zsh の起動時に読み込まれる場所へ追加します。

```zsh
tl completion zsh >> ~/.zshrc
source ~/.zshrc
```

`source ~/.zshrc` で再読み込みできない場合は、ターミナルを再起動してください。

## bash

bash で補完を使う場合は、`tl completion bash` の出力を bash の起動時に読み込まれる profile に追加します。

```bash
tl completion bash >> ~/.bashrc
source ~/.bashrc
```

macOS などで login shell に `~/.bash_profile` を使っている場合は、`~/.bashrc` の代わりに `~/.bash_profile` へ追加してください。

`source ~/.bashrc` で再読み込みできない場合は、ターミナルを再起動してください。

## PowerShell

PowerShell profile のパスは `$profile` で確認できます。profile ファイルがない場合は作成してから、`tl completion powershell` の出力を追加します。

```powershell
if (!(Test-Path $profile)) { New-Item -ItemType File -Path $profile -Force | Out-Null }
tl completion powershell >> $profile
. $profile
```

`. $profile` で再読み込みできない場合は、新しい PowerShell を開いてください。

## 補完候補が出ない場合

補完候補が出ない場合は、次の点を確認してください。

- `tl` が PATH から実行できるか
- shell profile に `tl completion <shell>` の出力が追加されているか
- shell を再起動したか、profile を再読み込みしたか
- 利用している shell と、出力した completion script の種類が一致しているか

## アンインストール / 設定解除

Tab 補完を使わない場合は、shell profile に追加した `tl completion <shell>` の出力部分を削除します。
