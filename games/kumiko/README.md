# 組子

**デザインコンセプト**: 組子細工(kumiko woodwork)の木片を和紙の盤面に積み上げるパズル — ヒノキの温かみある木目・細格子紋様・藍と弁柄赤のアクセント。

## 概要

8x8の和紙盤面に木製ポリオミノ(組子の断片)をドラッグして置き、列または段を全て埋めると消えてスコアになる。連続して消すと「技(コンボ)」が上がり得点倍率が伸びる。置ける場所がなくなれば終了。

## プレイURL

[https://nrtkumi.github.io/games/kumiko/](https://nrtkumi.github.io/games/kumiko/)

## 対応環境

- iOS Safari / Android Chrome (タッチ操作)
- デスクトップブラウザ (マウス操作)
- 単一HTMLファイル、外部アセット不使用

## 遊び方・ルール

1. 画面下のトレイに3つの木片が表示される
2. 木片をドラッグして盤面の空いている場所に置く(ゴーストプレビューで配置可否を確認)
3. 3つ全て置くとトレイが補充される
4. **列(横)または段(縦)が全て埋まると消えてスコアになる**
5. どの木片も置ける場所がなくなるとゲームオーバー

### スコアリング表

| 内容 | 得点 |
|------|------|
| 木片を置く | ×セル数 × 5 pt |
| おじゃま木片(十字)を置く | +15 pt ボーナス |
| 1列消し | 50 pt × 技倍率 |
| 2列同時消し | 80 pt × 技倍率 |
| 3列以上同時消し | 110 pt以上 × 技倍率 |

### コンボ(技)

| 状況 | 技倍率 |
|------|--------|
| 消しなし | ×1 |
| 連続消し1回 | ×1 |
| 連続消し2回 | ×2 |
| 連続消しN回 | ×N |

- 消しが続く限り技が上がる。1度でも消せない配置をすると技が1下がる
- 最大技・最大連続消しはゲームオーバー画面で表示
- スコアとベストスコアはブラウザのlocalStorageに自動保存

### おじゃま木片

低確率で出現する十字型の特殊ピース。5セル占有するが追加15ptと音声ボーナス付き。

## 作成の背景

2025年後半のハイブリッドカジュアル市場において、**決定論的ブロック配置パズル**は最も多くダウンロードされたジャンルの一つとなった。Block Blast・1010!系のメカニクスはタイマーなし・理不尽なRNG排除・短いセッション長から「待ち時間キラー」として定番化している。

- 参考: [AppMagic Q3 2025 Hybrid Casual Report](https://appmagic.rocks/blog/q3hybrid2025)
- 参考: [Gamigion Top 10 Hybrid Casual Q3 2025](https://www.gamigion.com/top-10-hybridcasual-games-in-q3-2025/)
- 参考: [MAF Top Mobile Games 2025](https://maf.ad/en/blog/top-mobile-games-2025/)

このゲームでは**組子細工**の幾何学的な美しさとパズルの「あと1手」の緊張感を組み合わせた。コンボシステムと多列同時消しボーナスによる指数的なスコア成長が繰り返しプレイを促す。

UIはスマホの**親指ゾーン**設計に従い、よく触るトレイを画面下部(親指の届く位置)に配置した。参考: [Smashing Magazine — Designing for One-Hand Usage](https://www.smashingmagazine.com/2020/02/design-mobile-apps-one-hand-usage/)

## 技術構成

- **単一HTMLファイル** (外部ファイルなし)
- Google Fonts: Shippori Mincho + Zen Maru Gothic (CDN)
- Canvas 2D: 格子・木目・組子細工パターンをすべてJSで描画
- Web Audio API: 効果音をリアルタイム合成(カチッ音・鐘・太鼓)
- iOS音声対策: 無音WAVをArrayBufferで生成しaudioセッションをunlock
- ダブルタップズーム防止: touchend 350ms判定 + dblclick/gesturestart preventDefault
- PointerEvents統一: touchとmouseを一本化、click非依存
- DPR対応: 毎フレーム `ctx.setTransform(DPR,0,0,DPR,0,0)` 再適用
- localStorage: try/catchでスコア永続化
- PC対応: `width: min(100vw, calc(100dvh*0.56), 420px)` でポートレートカラムに固定

## 実施したテストと教訓

### 1. 構文チェック (node --check)
インラインscriptをPython正規表現で抽出 → `node --check` → **PASS**

### 2. ヘッドレスロジックテスト
DOMをスタブしたNode.js環境で26項目テスト実施:
- グリッド初期化・境界チェック・セル配置・ライン消し・コンボ計算・ゲームオーバー検出・localStorage永続化・pickPiece 100回試行 → **26/26 PASS**

### 3. 実ブラウザE2E (Puppeteer + Chrome)
モバイルビューポート (390×844, deviceScaleFactor:2, hasTouch:true) にて:
- ページエラー: **0件**
- コンソールエラー: 1件 (OGP画像未生成時のfavicon同等の404、修正後は0件)
- canvas描画確認 (getImageData): **PASS**
- タイトル画面表示: **PASS**
- スタート操作: **PASS**
- トレイ全3ピース描画: **PASS** (3/3)
- ドラッグでのピース配置: スコアが0→15に増加 → **PASS**

### 4. OGP
Puppeteer (600×315 clip × dpr:2 = 1200×630) でミッドゲームのスクリーンショット → `sips` でJPEG変換
- ファイル: `assets/ogp.jpg`
- サイズ: **1200×630 px** (JPEG, 72dpi) → **PASS**

**教訓**: headlessテストでは「ドラッグが実際に機能するか」が確認できないため、実ブラウザE2Eは必須。またrAFループが触る全変数を宣言時に初期化しないと初回フレームでループが死ぬ。
