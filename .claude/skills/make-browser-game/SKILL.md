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
- **ダブルタップズーム無効化** — iOS Safariは`user-scalable=no`を無視するので連打でズームしてしまう(tanetoriで実際に起きた)。3点セットで防ぐ:
  1. 全要素に `touch-action: manipulation`(canvas/ボタンは`none`のまま)
  2. JSで `dblclick`/`gesturestart` をpreventDefault + 350ms以内の連続`touchend`をpreventDefault
  3. touchendのpreventDefaultは合成clickを潰すため、**`click`イベントに依存しない**(Pointer Eventsで拾う)
- 入力はPointer Eventsで統一(マウス兼用)。上記の通り`click`は使わない
- **PCワイド画面対策** — ゲーム内サイズを横幅から比例計算していると全幅表示で巨大化する(tanetoriで実際に起きた)。ルート要素を `max-width: min(520px, 64vh)` 程度の縦長カラムに制限して中央寄せ(`left: 50%; transform: translateX(-50%)` + 暗背景・シャドウ)。キーボード操作(←→/A・D、Space等)も付ける
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

### 必須チェックリスト(OGP・メタタグ)

リリース前ではなく**実装の段階でメタタグを入れておく**(後回しにすると忘れる):

- `<meta name="description">` と OGP一式: `og:title` / `og:description` / `og:image` / `og:type`(website)/ `og:url` + `twitter:card`(summary_large_image)
- `og:image` と `og:url` は**絶対URL**(https://nrtkumi.github.io/...)で書く
- 既存例: `games/tanetori/index.html`、`games/suika-fever/index.html`
- 画像は **1200x630**(またはタネトリの1200x800)、JPEG可。置き場所は `games/<ゲーム名>/assets/ogp.jpg`
- 画像の作り方は2通り。ゲームの見た目が伝わる方を選ぶ:
  1. **ゲームプレイのスクリーンショット**(進行中の画面が一番伝わる) — Playwrightで自動プレイさせて撮る。`deviceScaleFactor: 2` + 600x315のclipでちょうど1200x630になる。盤面が育つまで数十秒プレイさせ、複数タイミングで撮って目視で選ぶ。ポインタを左右にスイープすると操作が散って画になる(実例: スイカフィーバーのOGP制作)
  2. **画像生成** — codex経由のgpt-image-2でタイトルロゴ入りのキービジュアルを生成する(下の「画像アセット」参照)。プレイ画面がOGP映えしないゲームや、文字主体のビジュアルにしたい場合はこちら
- スクショ撮影の注意: 長時間の自動プレイでcanvasの2Dコンテキスト状態がリセットされ、初期化時の `ctx.scale(dpr, dpr)` が消えて描画が崩れることがある(スイカフィーバーで実際に起きた)。**dpr変換は毎フレーム `ctx.setTransform(dpr, 0, 0, dpr, 0, 0)` で再設定**しておく(iOS Safariのコンテキスト喪失対策にもなる)

### 画像アセット(任意) — codex経由の画像生成が使える

グラフィックの基本は絵文字+canvas描画(アセット不要・単一HTML維持)だが、タイトルロゴ・OGP画像・スプライト等が必要な場合は **Codex CLI(`codex`)経由でgpt-image-2の画像生成**が使える:

- 使い方: シェルから `codex` を呼び出し「○○の画像を生成して games/<ゲーム名>/assets/xxx.png に保存して」のように依頼($imagegenスキル)
- ChatGPTサブスクリプション内で動作(API追加課金なし)。日本語テキストの描画にも対応
- 注意: gpt-image-2は**透過背景に非対応**。透過PNGが必要ならgpt-image-1.5(API課金)へのフォールバックが必要
- 生成画像を使う場合もファイル数は最小限にし、READMEの技術構成に生成手段を記録する

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
4. OGPはSNS側にキャッシュされる。シェアしたのに反映されない場合はXのCard Validator等でキャッシュ更新を案内する
