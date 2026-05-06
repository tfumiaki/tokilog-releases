# Tokilog Releases

Tokilog は、ローカル保存を前提にした軽量な時間記録 CLI ツールです。

この repository は Tokilog の一般ユーザー向け配布専用 repository です。ソースコード、開発中の Issue / PR、内部向け設計ドキュメントはここには置きません。

## Alpha Release

Tokilog は現在 alpha 段階です。CLI (`tl`) を先行配布しており、Windows / macOS で手動導入して使うことを想定しています。

現在の配布物:

| Platform | Asset |
|---|---|
| Windows x64 | `tokilog-net10.0-win-x64.zip` |
| macOS Apple Silicon | `tokilog-net10.0-osx-arm64.zip` |

配布物は [GitHub Releases](https://github.com/tfumiaki/tokilog-releases/releases) から取得します。

## Getting Started

- [Windows Getting Started](docs/usage/getting-started-windows.md)
- [macOS Getting Started](docs/usage/getting-started-macos.md)

## Runtime

配布される `tl` / `tl.exe` は .NET 10 Runtime を必要とします。

```console
dotnet --list-runtimes
```

.NET 10 Runtime または .NET 10 SDK が表示されない場合は、.NET 10 Runtime をインストールしてから Tokilog を実行してください。

## Data Storage

Tokilog は記録データをローカル SQLite DB に保存します。サーバーへの同期は行いません。

既定の DB 保存場所:

| OS | Path |
|---|---|
| Windows | `%LOCALAPPDATA%\Tokilog\tokilog.db` |
| macOS | `~/Library/Application Support/Tokilog/tokilog.db` |

Tokilog 本体を削除しても DB ファイルは自動では削除されません。記録データを残したい場合は DB ファイルを削除しないでください。

## Known Limitations

- alpha release です。破壊的変更が入る可能性があります。
- Windows は x64 向け zip のみを想定しています。
- macOS は Apple Silicon (`osx-arm64`) 向け zip のみを想定しています。
- MSI / MSIX、Homebrew、winget、Scoop、インストーラー、自動アップデートは未提供です。
- macOS の notarization / code signing は未整備です。

## License

LICENSE / NOTICE は公開 release asset を配布する前に追加します。現時点では、この repository は alpha release の配布導線を準備するための初期状態です。
