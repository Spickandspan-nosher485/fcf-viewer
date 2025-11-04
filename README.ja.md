# FCF Viewer

FCF Viewer は、カードの FCF (FeliCa Common Format) 領域を読み出してコンソールに表示する最小構成の FeliCa ユーティリティです。  
CLI のロジックは `fcf_viewer/cli.py` にまとまっており、同じ処理を Tkinter 製 GUI でも流用してリアルタイムの表示に使っています。

## 読み取る内容
システムコード `0xFE00` で FeliCa のポーリングを行い、`cli.py` が扱っている領域のみをデコードします。

| セクション | 取得元 | 詳細 |
| --- | --- | --- |
| カード識別子 | ポーリング応答 | IDm / PMm（16 進・大文字） |
| 基本情報 | サービス `0x1A8B`、ブロック 0〜3 | 利用者区分、個人ID、再発行回数、性別、氏名、学校識別番号、発行日、有効期限、発行事業者 |
| FCF-UN | サービス `0x7A0B`、ブロック 0 | ASCII で復号し、ブロックにアクセスできた場合のみ表示 |

カードが FCF サービスを公開していない場合は、`cli.py` で定義されているのと同じエラーメッセージを表示します。

## 必要環境
- Python 3.14 以上
- Poetry 1.8 以上
- [nfcpy](https://nfcpy.readthedocs.io) が対応している FeliCa リーダー（例: Sony RC-S380）
- USB デバイスへアクセスできる OS / ドライバー設定

## インストール

```bash
poetry install
```

## CLI の使い方
1. FeliCa リーダーを接続します。
2. カードをかざした状態で以下のエントリーポイントを実行します。

```bash
poetry run fcf-viewer
```

CLI では次の 3 セクションを日本語で出力します。
- `カード識別`: IDm / PMm
- `基本情報`: 上記 9 フィールド
- `FCF-UN`: 対象ブロックの値。未対応カードの場合はエラー表示

復号処理はすべて `FcfTagReporter`（`fcf_viewer/cli.py`）内で行われます。

## GUI の使い方

```bash
poetry run fcf-viewer-gui
```

GUI はリーダーを継続的にポーリングし、CLI と同じデコード結果を画面上の IDm / PMm / 基本情報 / FCF-UN に反映します。  
未対応カードや読み取りエラー、リーダー初期化失敗などの状態はステータスメッセージで確認できます。

## トラブルシューティング
- `このカードは FCF に対応していません`: `cli.py` が参照するサービスブロックにアクセスできません。別のカードを試してください。
- `NFC リーダーを初期化できません`: USB デバイスが使用中、もしくは libusb などのドライバーが不足しています。
- FeliCa 以外のタグは無視されます。読み取りが終わるまで対応カードをリーダーに載せたままにしてください。

## 開発
- コード整形: `poetry run black fcf_viewer`
- CLI / GUI ともに `nfcpy` に依存するため、デスクトップでの動作確認には対応リーダーが必要です。オフライン検証時はユニットテストや依存の差し替えを活用してください。

## ライセンス

[MIT](https://opensource.org/licenses/MIT)

Copyright (c) 2025 KIRISHIKI Yudai
