# Tokilog Releases

Tokilog は、ローカル保存を前提にした軽量な時間記録 CLI ツールです。

この repository は Tokilog の一般ユーザー向け配布専用 repository です。ソースコード、開発中の Issue / PR、内部向け設計ドキュメントは非公開です。

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

Tokilog alpha binaries are provided under the [Tokilog Alpha Evaluation License](LICENSE).

The alpha binaries are provided for personal or internal evaluation use only. Redistribution, resale, modified redistribution, hosted service use, organization-wide rollout, and reverse engineering are not permitted except to the extent required by applicable law.

Third-party components are covered by their own licenses. See [NOTICE](NOTICE).
