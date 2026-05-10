# docs/usage

Tokilog CLI の利用者向けセットアップ手順や操作ガイドを格納します。

## どこから読めばよいか

初めて Tokilog CLI を使う場合は、利用環境の Getting Started を最初に読んでください。インストール・PATH 設定・最初のコマンドまでひととおり通せます。

1. 利用環境に合わせた Getting Started を読み、`tl` を実行できる状態にする
   - macOS: [getting-started-macos.md](./getting-started-macos.md)
   - Windows: [getting-started-windows.md](./getting-started-windows.md)
2. 日常操作のコマンドは [cli-manual.md](./cli-manual.md) を参照する
3. （任意）Tab 補完を使う場合は [completion.md](./completion.md) をセットアップする

## ドキュメント一覧

| ファイル | 説明 |
|---------|------|
| [getting-started-macos.md](./getting-started-macos.md) | macOS 向け Tokilog CLI Getting Started |
| [getting-started-windows.md](./getting-started-windows.md) | Windows 向け Tokilog CLI Getting Started |
| [cli-manual.md](./cli-manual.md) | Tokilog CLI の日常操作マニュアル |
| [completion.md](./completion.md) | Tokilog CLI Tab 補完の利用手順（任意） |

## 不具合報告・要望

不具合の報告や機能要望は GitHub Issues から受け付けています。

- [Tokilog Releases Issues](https://github.com/tfumiaki/tokilog-releases/issues)

`tl doctor` の結果や、`--debug` 付きで実行したときのエラー内容を添えていただけると調査に役立ちます。詳細は [cli-manual.md](./cli-manual.md) の「DB / ログ / 診断」「エラーの読み方」を参照してください。
