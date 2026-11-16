# 配布ファイル一覧（win32 プロジェクト）

この `files/` 以下は、win32 プロジェクト（ユーザーモードQEMU＋Wine＋独自スーパーバイザ q32sup による、制約サンドボックスVM上での32ビットWindows互換環境）の自作部分を、単体でダウンロードできる形にまとめたものです。全て MIT License（`LICENSE`）で提供します。

## 収録ファイル

### q32sup — 監督プロセス本体
- `q32sup/q32sup.c` — 監督プロセスの本体ソース。ptrace/seccomp で子プロセスを監督し、ELF32 の execve のみを傍受して QEMU 側へ橋渡しします。memfd 固定による TOCTOU 対策、内容検証つきディスクモード、子の終了ステータスの正確な伝播を実装。既定のパスは環境変数 `Q32_QEMU` / `Q32_SYSROOT` / `Q32_WRAPDIR` / `Q32_CPU` で上書きできます（ソース内のフォールバック既定値は原開発環境のパスです）。
- `q32sup/noexec32.c` — ネイティブ実行が塞がれた環境の切り分け用に使う小さな補助ソース。

### scripts — セットアップと起動
- `scripts/setup.sh` — 環境一式のセットアップ（冪等）。
- `scripts/verify.sh` — セットアップ検証（段階別チェック。不明なオプションは usage を表示して exit 2）。
- `scripts/env.sh` — 各スクリプト共通の環境変数定義。
- `scripts/start-x.sh` — X サーバ（Xvfb）の起動。
- `scripts/start-pa.sh` — 仮想オーディオ（PulseAudio、TCP 47130、シンク名 "game"）の起動。
- `scripts/boot32.sh` / `scripts/run32.sh` / `scripts/ws32.sh` / `scripts/launch.sh` — 32ビットスタックの起動補助スクリプト群。
- `scripts/fetch-online.sh` — 第三者コンポーネント（QEMU / Wine 等）を出典から再取得する手順スクリプト。本配布には第三者バイナリは含みません（出典とライセンスは `docs/THIRD-PARTY.md`）。
- `scripts/games/launch-game.sh` — **無改造の** 32ビット Windows ゲームを汎用に起動するランチャ。ゲーム側のファイルや設定は変更しません。新規プレフィックスの自動ブート（Wine Mono のモーダル回避のため `mscoree=d` を wineboot より前に設定）、日本語ロケールの生成、音声ルート選択（ALSA null / Pulse / ALSA→Pulse）までを担います。
- `scripts/games/setup-game-sysroot.sh` — ゲーム実行に必要な音響＋OpenGL の依存閉包を sysroot に追加し、EGL プローブで検証します。
- `scripts/games/setup-game-fonts.sh` — ゲーム用フォント環境のセットアップ。
- `scripts/games/asound-null.conf` — 無音の仮想 ALSA デバイス設定（音声を実際にストリームさせつつ捨てる経路）。

### tools — 計測・診断ユーティリティ
- `tools/delivery_ratio.py` — 音声の供給率（delivery ratio）を float32 キャプチャから算出します。
- `tools/eglprobe.py` — EGL の可用性を実際に初期化して検証するプローブ（存在確認だけでなく初期化成否まで見る）。
- `tools/set-audio-driver.py` — Wine プレフィックスの音声ドライバ設定をレジストリへ書き込み、再読込で検証します（書けなかった場合は大きな音で失敗を返す）。
- `tools/make-subset-fonts.py` — 日本語サブセットフォント（10,250 グリフ／書体）の生成器。文字化け（グリフ欠落）対策。
- `tools/fpsprobe.sh` — 画面キャプチャから実効フレームレートを推定するプローブ（分母は実キャプチャ時間）。
- `tools/cpubench.py` / `tools/dlprobe.py` — CPU と動的リンクの小さな切り分け用プローブ。

### tests — q32sup テストバッテリーのソース
- `tests/cpuid.c` / `tests/cpuid.s` — CPUID 挙動の確認用。
- `tests/exectest.s` / `tests/h.s` — execve 傍受と最小バイナリの確認用アセンブリ。
- `tests/d3d9test.c` — Direct3D9 の描画経路を切り分けるテスト（ウィンドウ作成→デバイス生成→クリア→Present→ピクセル読み戻し）。
- `tests/run-d3d9test.sh` — 上記テストの実行ランナー（コンパイル済み `d3d9test.exe` は本配布に含まず、フル環境でソースからビルドされます）。

### docs
- `docs/THIRD-PARTY.md` — 第三者コンポーネントの出典とライセンス一覧。
- `docs/RESTORE-TEST.md` — 「VM が消滅しても GitHub から復元できる」ことをクリーンな clone で検証した記録（2026-11-12）。

### その他
- `Makefile` — 原ツリーの便宜ターゲット（setup / verify / q32sup ビルド）。原ツリーでは q32sup のソースは `src/` に置かれています。本配布では `q32sup/` に配置しています。
- `LICENSE` — MIT License。

## 含まれないもの
- QEMU / Wine / 32ビット sysroot 等の第三者バイナリ一式（出典は `docs/THIRD-PARTY.md`、再取得は `scripts/fetch-online.sh`）。
- Wine プレフィックスのスナップショット、ゲーム本体・体験版等の著作物データ、録画・録音・スクリーンショット類。
- 計測ライン専用の運用スクリプト（特定環境のパスに強く依存するもの）。

## 配布にあたっての変更点
- `scripts/games/launch-game.sh`、`scripts/games/setup-game-sysroot.sh`、`tests/run-d3d9test.sh` の3ファイルは、検証記録のコメントに含まれていた特定の市販ゲーム体験版のタイトル名を、一般的な表現（「無改造の32ビット Windows ゲーム体験版」等）に置き換えました。動作・機能に変更はありません。
- それ以外のファイルは原ツリーのままです。
