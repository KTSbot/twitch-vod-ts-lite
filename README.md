# Twitch VOD TS Lite

Twitch VODの音声から文字起こしとタイムスタンプ候補を作る、macOS向けの簡易CLIです。RAWは見どころを探すための索引です。完成版TSは、文字起こし全文を確認してから手動で作成します。

## Macでの導入

検証環境はmacOSのPython 3.14.7です。Homebrewが使えるMacのターミナルで、次のコマンドを実行します。インストールされる3.14系のパッチバージョンは時期によって変わるため、`python3.14 --version` で確認してください。

```sh
brew install python@3.14 yt-dlp ffmpeg ollama
python3.14 --version
cd ~/twitch-vod-ts-lite
python3.14 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
```

フォルダを別の場所に保存した場合は、`cd` の行をその場所に合わせてください。Ollamaが起動していなければ、別のターミナルで次のコマンドを実行し、そのまま開いておきます。

```sh
ollama serve
```

元のターミナルでRAW用モデルを準備します。文字起こし用のmediumモデルは、キャッシュがない場合に初回実行時に取得されます。モデル準備には通信とディスク容量が必要です。

```sh
ollama pull qwen3.5:9b
```

## 実行

`<VOD_ID>` をVODページのURLにある数字へ置き換えてください。クエリや再生位置を表す部分は取り除きます。認証なしで再生でき、利用する権限のあるVODを使ってください。

```sh
cd ~/twitch-vod-ts-lite
source .venv/bin/activate
python twitch_vod_ts.py 'https://www.twitch.tv/videos/<VOD_ID>'
```

`~/twitch-vod-ts-lite-runs/` の下に実行ごとのフォルダができます。ターミナルに表示される保存先を開くと、取得した `audio.mp3`、発話本文と開始・終了時刻を含む `transcript.json`、簡易候補の `raw_timestamps.txt` を確認できます。ASRが返した単語時刻もJSONに保存します。時刻は取得音声の先頭を基準にします。作業ファイルは各ジョブに残ります。

## 動作確認

- **実VODの工程別検証はPASSです。** Python 3.14.7で、yt-dlpによる音声取得、faster-whisper mediumによる全文文字起こしと `transcript.json` の保存、Ollamaの `qwen3.5:9b` による候補生成と `raw_timestamps.txt` の保存を確認しました。音声取得後に見つかったPyAVの互換性問題を `av==17.1.0` の固定で修正し、取得済み音声を使って文字起こし以降を検証しました。
- **修正後の通常CLI一括E2EはNOT RUNです。** 音声取得からRAW保存までの通し実行は未実施です。

## ChatGPTで完成版TSを作る

1. 共有する権限のある `transcript.json` と、作成できた場合は `raw_timestamps.txt` を手動で渡します。
2. 「文字起こし全文を確認し、見どころを時刻順にまとめてください。RAWは索引として使い、候補外にも目を通してください。時刻はtranscriptから確認し、曖昧な箇所は明示してください」と依頼します。
3. 完成案の時刻、要約、引用を元音声や文字起こしで確認します。発話や単語時刻には認識誤りがあり得ます。

## 制約と失敗時の対応

- 音声専用形式を取得できないVOD、認証が必要なVOD、取得に失敗したVODでは停止します。URL、公開状態、yt-dlpとFFmpegの導入、ディスクの空き容量を確認してください。
- 文字起こしはCPUで音声をまとめて処理します。長時間VODでは時間とメモリを消費します。中断・再実行時は別のジョブになります。
- Ollamaが使えない場合や応答を読み取れない場合は、保存済みの `transcript.json` で手動作業を続けられます。終了コード2は文字起こし完了後のRAW失敗です。
- 簡易版のため、本番と同等の精度・機能を保証しません。開発環境はmacOSです。他の環境での動作は保証しません。

## ライセンスと保守

このリポジトリの自作部分は [MIT License](LICENSE) で提供します。利用・改変・再配布が可能です。ソフトウェアの複製物または実質的な部分には、著作権表示と許諾表示を保持してください。第三者の素材、外部ライブラリ、モデルにはそれぞれの利用条件が適用されます。

現状のまま提供し、動作を保証しません。質問対応、保守、不具合修正は原則行いません。GitHub Issuesは無効にしています。利用者による自由な改良を歓迎します。
