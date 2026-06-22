# こい — 朱鯉の遡上

一本指で朱鯉を操り、激流・岩・渦巻きをかわしながら遡上する無限ウィーブ回避ゲーム。

**デザインコンセプト**: 水墨画(sumi-e) — 和紙地に墨筆の岩と流れ、朱一色の鯉が焦点。

## プレイURL

<https://nrtkumi.github.io/games/koi/>

## 対応環境

- iOS Safari (iPhone / iPad)
- Android Chrome
- PC ブラウザ (Chrome / Firefox / Safari) — キーボード操作あり

## 遊び方・ルール

| 操作 | 内容 |
|------|------|
| 画面をドラッグ | 鯉を左右に誘導（指はどこに置いてもOK） |
| ←→ / A・D キー | PC でのステアリング |

鯉は自動で上流へ泳ぐ。岩や渦巻きに当たるとゲームオーバー。

### スコアの仕組み

| 要素 | 得点 |
|------|------|
| 遡上距離 | 毎フレーム自動加算 |
| 花びら（桜）回収 | +15 × コンボ倍率 |
| 水面輝き回収 | +30 × コンボ倍率 |
| 岩のニアミス (<40px) | +10 × コンボ数 + コンボ +1 |

### コンボと気の流れ

ニアミスを連続すると **コンボ倍率** が上がり、**気の流れゲージ** が蓄積する。  
ゲージが満タンになると **激流モード** 発動 — 速度1.6倍・スコア2倍・6秒間持続。  
激流は高コンボとの掛け合わせで爆発的にスコアが伸びる。

| コンボ数 | 効果 |
|---------|------|
| 1〜2 | スコア倍率 ×1〜2 |
| 3以上 | コンボ効果音が高くなる |
| 気ゲージ満 | 激流モード突入 |

スローモーションの一瞬（ニアミスブリップ）と金色パーティクルが近い判定を知らせる。

### ベストスコア

最高スコア・最長距離・最大コンボは localStorage に保存され、タイトルに表示される。

## 作成の背景

ニアミス判定と瞬間的なスローモーションは「損失回避 × 報酬予測」を同時に刺激する。  
ハイパーカジュアルゲームの研究では、**ニアミス緊張 → コンボ報酬 → 即時リスタート** のサイクルが持続的なループエンゲージメントを生むと指摘されている。

> "Hyper-casual games create dopamine loops through near-miss mechanics and instant replay."  
> — [StudioKrew Blog (2025)](https://studiokrew.com/blog/how-hyper-casual-games-are-driving-growth-in-mobile-game-industry-2025/)

> [MAF Top Mobile Games 2025](https://maf.ad/en/blog/top-mobile-games-2025/) でも、スコアアタック型の即時フィードバックがリテンションの主因として挙げられている。

操作はドラッグ（指一本）のみ。スマホ親指の可動域を考慮し、タップ位置を画面下部に限定せず  
**画面どこでも操作できる**設計にした（親指ゾーン設計の参考: [Smashing Magazine — One-Hand Usage](https://www.smashingmagazine.com/2020/02/design-mobile-apps-one-hand-usage/)）。

## 技術構成

- **単一HTML** (inline CSS + JS, CDN不使用)
- **Canvas 2D** — `ctx.setTransform(DPR,0,0,DPR,0,0)` を毎フレーム再適用 (iOS コンテキスト喪失対策)
- **Web Audio API** — 外部音源なし。ニアミス・ピックアップ・ヒット・激流をリアルタイム合成
- **iOS 音声解除**: 無音WAV (`<audio loop>`) + AudioContext.resume() をポインタ初回タップ時に実行
- **Pointer Events 統一** — タッチ/マウス共用。`click` 依存なし (ダブルタップズーム対策)
- **dt ベース物理** — `requestAnimationFrame` の差分 dt で速度計算。フレームレートに依存しない
- **localStorage** — ベストスコア/距離/コンボを try/catch で永続化
- フォント: Yuji Syuku (タイトル) + Zen Maru Gothic (UI) via Google Fonts

## 実施したテストと教訓

### 1. 構文チェック
`node --check` でインライン JS を検証 → **PASS**

### 2. ヘッドレスロジックテスト (Node.js)
DOM・Canvas・AudioContext をスタブし Function コンストラクタでゲームを評価:
- 初期状態 → initGame → playing 遷移
- 左右ステアリングで koi.x が移動すること
- 距離・スコアがフレームごとに増加すること
- ニアミス (gap 6px) でコンボ+1・flowGauge 増加
- 岩衝突 → dead 遷移・bestScore/bestDist 保存
- リスタートでスコアリセット
- ピックアップ回収でスコア増加
- dt 大のフレームで移動量が大きいこと (DT ベース確認)

結果: **16/16 PASS**

### 3. 実ブラウザ E2E (puppeteer-core + Chrome)
モバイルビューポート (`hasTouch:true`, `deviceScaleFactor:2`), `python3 -m http.server` 経由:
- Canvas 描画ピクセル確認: **YES**
- pageerror/console エラー (favicon 除く): **NONE**
- 2.5秒ポインタドラッグ操作でニアミス・コンボ発生
- 非背景ピクセル数: **24913** (expect > 200)

結果: **PASS**

### 4. OGP
- Puppeteer で 480x1000 モバイルビューポート、3秒間プレイ後スクショ
- `clip {x:0, y:563, w:480, h:315}` → DPR2 → 960x630 PNG → `sips -z 630 1200` → **1200x630 JPEG**
- `file assets/ogp.jpg` → JPEG 確認済み

### 教訓

- ニアミス判定は「ヒットボックス半径の合計」を gap ゼロより大きくしないと衝突判定と被るため、テストでは厳密な距離計算が必要
- `eval` でのヘッドレステストは `let` がスコープ外になるため `Function` コンストラクタで wrap が必要
- Puppeteer の `pointer` API は旧版では存在しないため `page.evaluate` で `PointerEvent` を直接 dispatch するほうが安定
- OGP のクロップ Y 座標はゲーム内の鯉位置 (y=72%×H) から逆算して合わせる必要がある
