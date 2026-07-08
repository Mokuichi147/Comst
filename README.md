# camst

カメラ映像をローカルウィンドウ、またはブラウザ（WebRTC）でリアルタイム表示するツールです。
OAK-D LITE / Leap Motion / 一般的な UVC USB カメラに対応し、Raspberry Pi でのヘッドレス運用も想定しています。

## 特長

- **3 種類のカメラソース**
  - `uvc` — 一般的な USB カメラ（OpenCV/V4L2 経由）
  - `oak` — OAK-D LITE（`depthai`。`oak` エクストラで追加インストール）
  - `leap` — Leap Motion のステレオ赤外映像（左右の眼を分離、明るさ補正・ノイズ除去つき）
- **2 つの表示モード**
  - ローカルウィンドウ表示（`cv2.imshow`）
  - WebUI：FastAPI + WebRTC（`aiortc`）でブラウザへ低遅延ストリーミング
- **動体検知の自動録画**（WebUI）：動きを検知したクリップを自動保存し、H.264 へ再エンコードして縮小。サムネイル生成・お気に入り登録に対応
- **映像回転**（0/90/180/270）、Leap 向けの CLAHE 明るさ補正・時間/空間ノイズ除去
- **マルチプラットフォーム**：x86_64 Linux / macOS / Raspberry Pi（armv6l は piwheels のビルド済み wheel を利用）

## 必要環境

- Python 3.12.10 以上
- [uv](https://docs.astral.sh/uv/)

## インストール

```bash
uv sync
```

OAK-D LITE を使う場合のみ `oak` エクストラを追加します（UVC/Leap には不要）。

```bash
uv sync --extra oak
```

## 使い方

### ローカル表示

```bash
# UVC カメラ（デフォルト: --source uvc --device 0）
uv run camst

# デバイスを番号か名前の一部で指定
uv run camst --device 1
uv run camst --source leap --device "Leap"
```

`q` キーでウィンドウを閉じます。

### WebUI（ブラウザ表示）

```bash
uv run camst --webui --host 0.0.0.0 --port 8000
```

ブラウザで `http://<ホスト>:8000` を開きます。

### 動体検知の自動録画

```bash
uv run camst --webui --record --host 0.0.0.0
```

録画は `recordings/` に保存されます（直近 100 件・1 本最大 1 分）。
WebUI の録画一覧からサムネイル確認・再生・お気に入り登録ができます。

### 既存録画の一括再エンコード

保存済みのクリップをまとめて H.264 へ変換し、ファイルサイズを縮小します。

```bash
uv run camst-reencode --recordings-dir recordings --encode-crf 26
```

## 主なオプション

| オプション | 説明 | 既定値 |
| --- | --- | --- |
| `--source` | カメラの種類（`uvc` / `oak` / `leap`） | `uvc` |
| `--device` | デバイス番号または名前の一部 | `0` |
| `--rotate` | 映像の回転角度（0/90/180/270） | `0` |
| `--webui` | ブラウザでストリーム表示 | 無効 |
| `--host` / `--port` | WebUI のバインド先 | `127.0.0.1` / `8000` |
| `--record` | 動体検知で自動録画（WebUI 用） | 無効 |
| `--eye` | Leap: 使う眼（`left` / `right` / `both`） | `left` |
| `--correct` | Leap: CLAHE による明るさ補正 | 無効 |
| `--nlm` | Leap: NLMeans による空間ノイズ除去 | 無効 |

すべてのオプションは `uv run camst --help` / `uv run camst-reencode --help` で確認できます。

## Raspberry Pi (armv6l) について

`av` / `opencv` は armv6l 向けの wheel を [piwheels](https://www.piwheels.org/) から取得します。
プラットフォームごとに取得元を振り分けているため、Pi では piwheels、それ以外では PyPI の wheel が使われます。
