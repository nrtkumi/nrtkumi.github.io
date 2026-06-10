---
name: make-browser-game
description: 市場調査から1人用ブラウザゲームの企画・実装・実機テスト・ドキュメント作成・公開までの一連フロー。新しいゲームを作る/追加するとき、ゲームのアイデア出しから公開まで頼まれたときに使う。
---

# ブラウザゲーム制作フロー

このサイト(nrtkumi.github.io)に1人用ブラウザゲームを追加するときの手順。
過去の制作(games/hakai/)で確立した流れと教訓をまとめている。

## 1. 市場調査

WebSearchで以下を調べ、結果と出典URLを記録する(後でREADMEに載せる):

- 直近のヒットゲーム・バイラルゲームのメカニクス(国内外)
- ハイパーカジュアル/カジュアルゲームのトレンド
- インディーゲームのトレンド(ローグライト要素など)

検索例: 「人気 ブラウザゲーム トレンド <年>」「viral indie games trends <year>」「hyper casual game mechanics <year>」

## 2. 企画

以下の条件で1案に絞る:

- **1文で説明できるコアメカニクス**(5秒で理解できる操作)
- **サイト内の既存ゲームとジャンルが被らない**(games/ 配下と index を確認)
- スマホファースト(タッチ操作のみで完結、縦画面)
- トレンド要素を1つ以上入れる(例: ローグライト強化、マージ、成長システム)
- 単一HTMLファイルで実装できる規模

## 3. 実装

`games/<ゲーム名>/index.html` に依存ライブラリなしの単一HTMLで実装する。

### 必須チェックリスト(スマホ対応)

- `<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover, user-scalable=no">`
- セーフエリア: `env(safe-area-inset-*)` でホームバー回避
- `touch-action: none`(canvas)、`-webkit-tap-highlight-color: transparent`、`user-select: none`、`overscroll-behavior: none`
- 入力はPointer Eventsで統一(マウス兼用)
- フォントは既存ページと統一: Bungee + Zen Kaku Gothic New + Space Mono(Google Fonts)

### 必須チェックリスト(iOS音声) — 過去のバグの再発防止

Web Audioの音はiOSの消音スイッチ(マナーモード)で無音になる。必ず以下を入れる:

1. 初回タップ時に**無音WAVを `<audio>` 要素でループ再生**(オーディオセッションを再生カテゴリへ切り替える。無音WAVはJSでArrayBuffer生成→Blob URL)
2. `AudioContext.resume()` の**Promise完了を待ってから**アンロック判定
3. `AC.onstatechange` で `interrupted` からの自動復帰
4. 実装例は `games/hakai/index.html` の audio セクションを参照

### 必須チェックリスト(設計)

- **メインループ(rAF)はスタート画面の時点から回る。ループが触る状態は全て宣言時に初期化する**(init()任せにすると初回フレームの例外でループが死ぬ — hakaiで実際に起きたバグ)
- 効果音はWeb Audioでリアルタイム合成(外部音源不使用)
- スコア等の永続化は localStorage(try/catchでフォールバック)
- 長いセッション向けに速度ボタンや自動加速を検討

## 4. テスト(リリース前に必ず全部やる)

1. **構文チェック**: インラインscriptを抽出して `node --check`
2. **ヘッドレスロジックテスト**: DOMをスタブしてNodeでゲームループを回し、開始→操作→進行→ゲームオーバー→記録保存を通す
3. **実ブラウザE2E(最重要)**: puppeteer-core + ローカルChromeで、モバイルビューポート(`hasTouch: true`)+タッチイベントで実際にプレイする
   - `pageerror` / consoleエラーがゼロであること
   - canvasの描画ピクセルがあること(getImageDataで確認)
   - 数ターン操作してスコア・進行が動くこと
   - スクリーンショットを撮って見た目を目視確認
   - 過去、ヘッドレステストだけでは「スタート前のフレームで例外→ループ停止」を見逃した。**実ブラウザE2Eを省略しない**
   - 雛形: `/Applications/Google Chrome.app/Contents/MacOS/Google Chrome` を executablePath に、一時ディレクトリに `npm install puppeteer-core`

## 5. ドキュメント

`games/<ゲーム名>/README.md` を作成:

- 概要・プレイURL・対応環境
- 遊び方・ルール(強化があれば一覧表)
- 作成の背景(市場調査の要点と出典リンク)
- 技術構成(単一HTML、iOS音声対策など)
- 実施したテスト内容と、開発で得た教訓

## 6. 公開

1. `git add` → コミット(本文に変更概要、Co-Authored-By付き)→ `git push`(mainがそのままGitHub Pages)
2. ファイルを移動した場合は旧URLに meta refresh + `location.replace` のリダイレクトHTMLを残す
3. ユーザーにプレイURL(https://nrtkumi.github.io/games/<ゲーム名>/)を案内。反映に1〜2分かかる旨を添える
