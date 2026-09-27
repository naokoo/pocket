<!--
  Document Type : README.md
  Project       : SOW·33 K.O! — PO-33 Style Pocket Operator Emulator
  Last Updated  : 2026-09-25
-->

# SOW·33 K.O!

Teenage Engineering PO-33 K.O! にインスパイアされた、**単一 HTML ファイルの 16 ボイス・サンプラー / シーケンサー**。
Web Audio API のみで実装。ビルド・依存・サーバー不要。

## Quick Start

1. `pocket-operator.html` をブラウザで開く（iPhone Safari / Chrome / Firefox / Edge）
2. LCD に `TAP PLAY` と出たら **PLAY** をタップ → デモパターン（ドラム＋メロディ）が流れる
3. **SYNTH** を押すとメロディの音色が切り替わる

画面右下の `?` で操作ヘルプを表示。

## 操作

| 操作 | 内容 |
|---|---|
| **パッド 1–16** | タップで即発音。1–8 メロディ（Cマイナーペンタ 2オクターブ）、9–16 ドラム |
| **PLAY** | 再生 / 停止 |
| **SYNTH** | メロディ音色を巡回 `SIMPLE → FM → WAVE → ADD`。押すと試聴音が鳴る |
| **WRITE** | 点灯中: 停止時はパッドで「選択音」のステップ ON/OFF。再生中はリアルタイム録音（現在ステップにクオンタイズ） |
| **SOUND** + パッド | 編集対象の音を選択（ノブ A/B の対象になる） |
| **PATTERN** + パッド | パターン選択。続けてタップするとチェーン（最大 16） |
| **BPM** | 点灯中にノブ A で 60–240 |
| **FX** | 点灯中にパッド 1=FILTER / 2=DELAY / 3=DRIVE を選び、ノブ A/B で操作 |
| **JAM** | ランダムパターンを生成して再生 |
| **MIC** | 点灯中にパッドをタップ → 約 1.5 秒マイク録音してそのスロットに割当 |
| **SAVE / LOAD** | BPM・パターン・チェーン・ボイスパラメータ・SYNTH 選択・音階を保存 / 復元 |
| **LOAD 長押し（0.6 秒）** | プリセット読込（下記） |
| **VOL − / +** | マスター音量 |

### ノブ A / B

| モード | A | B |
|---|---|---|
| 通常 | 選択音のピッチ（±12 半音） | 選択音のディケイ（0.25–2.0） |
| BPM | BPM | — |
| FX FILTER | カットオフ | レゾナンス |
| FX DELAY | ディレイタイム | ウェット量（＋フィードバック連動） |
| FX DRIVE | 歪み量 | 出力レベル |

モード切替時にノブ位置は現在値へ同期するので、値が飛びません。

## プリセット

`LOAD` を長押しすると読み込まれます。読込後 `PLAY`。

| 名前 | 内容 |
|---|---|
| ZOMBIE | Em → C → G → D の 4 小節ループ、84 BPM、ロックビート。メロディパッドを E ナチュラルマイナー（E3–G4）に差し替え、ベース＝ルート 8 分＋三和音を 1・3 拍に重ねる。音色 WAVE、ディケイ 1.6。4 小節目にフィル |

プリセットは `PRESETS` 配列に追加できます。各パターンは `{ ボイス番号: [ステップ...] }`（0 始まり）、`scale` を持たせるとメロディパッドの音階を差し替えます。

## 音源

- **メロディ (1–8)**
  - `SIMPLE`: square + ローパスエンベロープのプラック
  - `FM`: 2 オペレータ FM ベル
  - `WAVE`: デチューン saw ×2 + triangle
  - `ADD`: 加算合成オルガン（5 倍音）
- **ドラム (9–16)**: KICK / SNARE / CLAP / CH HAT / OP HAT / TOM L / TOM H / COWBELL（全て合成、サンプル不使用）
- **マイクサンプル**: 録音したスロットは合成音の代わりにサンプルを再生。ピッチ・ディケイのノブも効く

信号経路: `voices → bus → drive(WaveShaper) → lowpass → master`（＋ delay send）

## iPhone / iOS について

- AudioContext は読込時に生成し、初回タップで resume します（自動再生制限に対応）
- 消音スイッチ対策として `navigator.audioSession.type = "playback"` を設定します（Safari 16.4+ の Audio Session API）。API が無い環境では無音 `<audio>` の再生でフォールバック
- LCD 左上に診断表示: `RUN` = AudioContext 動作中、`SUSP` = resume 待ち、`INTR` = iOS の割込み、`NO AC` = 生成失敗。`+P` = Audio Session playback 適用、`+M` = `<audio>` フォールバック適用
- **Claude アプリ内のアーティファクトプレビューは iOS 27 で無音になります**（`RUN+P` 表示でも鳴らない）。ホストアプリ側の WKWebView オーディオセッションの問題で HTML からは対処不可。Safari / Edge で開くと iOS 27 でも鳴ります（Edge で動作確認済み）
- 「ファイル」App から `.html` を開くと Quick Look のプレビューになり Safari で開けません。GitHub Pages 等でホストして URL を開くか、ホーム画面に追加してください

## 制約

| 項目 | 内容 |
|---|---|
| claude.ai アーティファクトプレビュー | `localStorage` とマイクがブロックされる。SAVE は同一セッション限りのメモリ保存（LCD に `SAVED*`）、MIC は `MIC ERR`。ファイルをダウンロードして直接開けば両方動く |
| サンプルの保存 | 録音サンプルは SAVE 対象外（リロードで消える） |
| ポリフォニー制御 | なし。全ボイスが同一バスに入る |

## カスタマイズ

すべて `<script>` 内の先頭付近。編集してリロードするだけ。

```js
// 音階（8 音、Hz）
const SCALE_FREQS = [261.63, 311.13, 349.23, 392.00, 466.16, 523.25, 622.25, 698.46];

// メロディ音色: SYNTHS 配列に { name, play(f, t, g, dk) } を追加すれば SYNTH ボタンで巡回対象になる
//   f: 周波数, t: 発音時刻, g: 出力 GainNode（bus に接続済み）, dk: ディケイ係数
const SYNTHS = [ { name: "SIMPLE", play(f, t, g, dk){ ... } }, ... ];

// ドラム音色: playDrum() の case 8–15
// エフェクト初期値: FX_DEFAULTS
// LCD スプライト: BOXER_IDLE / BOXER_PUNCH（14 行 × 12 列、'#' がドット）
```

## 検証

- Node + `node-web-audio-api` のオフラインレンダリングで、16 パッド＋ FM/WAVE/ADD の全 19 ボイスが実グラフ経由で可聴レベル（RMS > 0.005）で発音することを確認
- ZOMBIE プリセットを読み込み 64 ステップ（4 小節）をスケジューリングして実レンダリング。4 小節すべて可聴、ピーク 0.88 でクリップなし、4 小節後にパターン 1 へ戻ることを確認
- AudioContext が生成できない環境でも、ロード・描画ループ・全ボタン操作が例外なく動作することを確認

## ファイル構成

```
pocket-operator.html   単一ファイル（約 36KB、依存なし）
README.md
.gitignore
LICENSE                MIT
```

## 動作環境

Web Audio API / Canvas 2D / Pointer Events / ES2020 対応ブラウザ。iOS Safari 14.5+、Chrome、Firefox、Edge の最近のバージョン。

## License

MIT
