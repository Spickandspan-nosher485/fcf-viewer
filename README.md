# FCF Viewer

FCF Viewer is a minimal FeliCa utility that reads the FCF (FeliCa Common Format) area of a card and prints the result to the console.  
The CLI logic lives in `fcf_viewer/cli.py` and is also reused by the Tkinter GUI for a live view of the same data.

## What it reads
The reader performs a raw FeliCa polling against system code `0xFE00` and decodes only the areas that `cli.py` touches:

| Section | Source | Details |
| --- | --- | --- |
| Card identifiers | Polling response | IDm / PMm (hex, upper case) |
| Basic information | Service `0x1A8B`, blocks 0–3 | 利用者区分, 個人ID, 再発行回数, 性別, 氏名, 学校識別番号, 発行日, 有効期限, 発行事業者 |
| FCF-UN | Service `0x7A0B`, block 0 | Decoded as ASCII. Printed only when the block is accessible. |

If the card does not expose the FCF services, the tool prints the same error strings defined in `cli.py`.

## Requirements
- Python 3.14+
- Poetry 1.8+
- A FeliCa reader supported by [nfcpy](https://nfcpy.readthedocs.io) (e.g. Sony RC-S380)
- System drivers that allow python to access the USB device

## Installation

```bash
poetry install
```

## CLI usage
1. Connect your FeliCa reader.
2. Run the entry point while the card is touching the reader.

```bash
poetry run fcf-viewer
```

The CLI prints three sections in Japanese:
- `カード識別`: IDm / PMm
- `基本情報`: the nine fields listed above
- `FCF-UN`: value from the dedicated block, or an error if the card does not support it

All decoding is handled inside `FcfTagReporter` (see `fcf_viewer/cli.py`).

## GUI usage

```bash
poetry run fcf-viewer-gui
```

The GUI continuously polls the reader, mirrors the CLI decoding, and updates the on-screen values (IDm, PMm, 基本情報, FCF-UN).  
Status messages are shown for unsupported cards, read errors, or reader initialization failures.

## Troubleshooting
- `このカードは FCF に対応していません`: the service blocks in `cli.py` are unavailable on the card. Try a different card.
- `NFC リーダーを初期化できません`: the USB device is busy or the required driver/libusb stack is missing.
- Non-FeliCa tags are ignored; keep the supported card on the reader until the read completes.

## Development
- Format code with `poetry run black fcf_viewer`
- The CLI and GUI both rely on `nfcpy`, so desktop testing requires a compatible reader. Use unit tests or dependency injection for offline development when possible.

## License

[MIT](https://opensource.org/licenses/MIT)

Copyright (c) 2025 KIRISHIKI Yudai
