# Tokilog Releases

Tokilog は、ローカル保存を前提にした軽量な時間記録 CLI ツールです。

この repository は Tokilog の一般ユーザー向け配布専用 repository です。ソースコード、開発中の Issue / PR、内部向け設計ドキュメントは非公開です。

## Alpha Release

Tokilog は現在 alpha 段階です。CLI (`tl`) と Desktop GUI を配布しており、Windows / macOS / Linux (Ubuntu) で手動導入して使うことを想定しています。

現在の配布物:

| Platform | CLI | Desktop GUI |
|---|---|---|
| Windows x64 | `tokilog-cli-net10.0-win-x64.zip` | `tokilog-desktop-net10.0-win-x64.zip` |
| macOS Apple Silicon | `tokilog-cli-net10.0-osx-arm64.zip` | `tokilog-desktop-net10.0-osx-arm64.zip` |
| Linux x64 (Ubuntu) | `tokilog-cli-net10.0-linux-x64.zip` | `tokilog-desktop-net10.0-linux-x64.zip` |

配布物は [GitHub Releases](https://github.com/tfumiaki/tokilog-releases/releases) から取得します。

## Getting Started

- [Windows Getting Started](docs/usage/getting-started-windows.md)
- [macOS Getting Started](docs/usage/getting-started-macos.md)
- [Linux (Ubuntu) Getting Started](docs/usage/getting-started-linux.md)
- [CLI Manual](docs/usage/cli-manual.md)
- [CLI Tab Completion](docs/usage/completion.md)

## Runtime

配布される CLI (`tl` / `tl.exe`) と Desktop GUI は .NET 10 Runtime を必要とします。

```console
dotnet --list-runtimes
```

.NET 10 Runtime または .NET 10 SDK が表示されない場合は、.NET 10 Runtime をインストールしてから Tokilog を実行してください。

## Data Storage

Tokilog は記録データをローカル SQLite DB に保存します。サーバー同期などの機能はありません。

既定の DB 保存場所:

| OS | Path |
|---|---|
| Windows | `%LOCALAPPDATA%\Tokilog\tokilog.db` |
| macOS | `~/Library/Application Support/Tokilog/tokilog.db` |
| Linux | `~/.local/share/Tokilog/tokilog.db`（`$XDG_DATA_HOME` 設定時は `$XDG_DATA_HOME/Tokilog/tokilog.db`） |

Tokilog 本体を削除しても DB ファイルは自動では削除されません。記録データを残したい場合は DB ファイルを削除しないでください。

## Known Limitations

- alpha release です。破壊的変更が入る可能性があります。
- Windows は x64 向け zip のみを想定しています。
- macOS は Apple Silicon (`osx-arm64`) 向け zip のみを想定しています。
- Linux は x64 向け zip のみを想定しています（Ubuntu 22.04 / 24.04 で確認）。
- MSI / MSIX、Homebrew、winget、Scoop、インストーラー、自動アップデートは未提供です。
- macOS の notarization / code signing は未整備です。

## License

Tokilog alpha binaries are provided under the [Tokilog Alpha Evaluation License](LICENSE).

The alpha binaries are provided for personal or internal evaluation use only. Redistribution, resale, modified redistribution, hosted service use, organization-wide rollout, and reverse engineering are not permitted except to the extent required by applicable law.

Third-party components are covered by their own licenses. See [NOTICE](NOTICE).
