# ヤグラ (YAGURA) — 夏祭りの櫓を積み上げろ

流れてくる**提灯タイル**が、下の段にピタッと重なる瞬間にタップして落とす。はみ出した分は切り落とされ、ズレるほど櫓は細くなっていく。**ど真ん中=PERFECT**で太鼓が鳴り、連続するほどコンボが燃え、節目で花火が上がる——何段まで届くかの一発勝負スタッキングアクション。

- **プレイURL**: https://nrtkumi.github.io/games/yagura/
- **対応環境**: スマホ(iOS Safari / Android Chrome)・PCブラウザ。縦画面・タッチ操作。PCはマウスクリック / スペース / Enter でも操作可。

## 遊び方

1. 「積みはじめる」をタップ。
2. 提灯タイルが左右に流れるので、**画面のどこでもタップ**して下の段の真上で落とす。
3. はみ出した部分は切り落とされ、残った幅が次の土台になる。**まったく重ならないと櫓が崩れてゲームオーバー**。
4. ど真ん中でそろえると **PERFECT**。幅が少し戻り、コンボが伸び、スコアが跳ねる。
5. 10段ごとに花火と歓声。どこまで高く積めるか記録に挑戦。

### スコアと演出

| 要素 | 効果 |
|------|------|
| 通常設置 | +1点。コンボはリセット |
| PERFECT(ど真ん中) | +(1+コンボ)点。幅が回復し、太鼓の音程が上がる |
| コンボ | 連続PERFECTでスコア倍率と画面の盛り上がりが増大 |
| 10段ごと | 花火が打ち上がり、歓声が鳴る |

記録(最高段数・スコア)は localStorage に保存される。

## デザインコンセプト

**夏祭りの夜**。藍紺の夜空に提灯の赤・橙・山吹が灯り、太鼓が鳴る世界観。フォントは祭りの幟(のぼり)を思わせる Reggae One と Dela Gothic One。各段は祭りの色帯(紅→山吹→藍→朱→抹茶→藤→桃)を巡回し、登るほど空の色と月が近づく。生成AI的な紫グラデやネオングローは使わず、提灯の暖色＋夜空でまとめている。

## 作成の背景

スタッキング(Stack系)は2016年の Ketchapp『Stack』以来の定番で、2024年以降も『Infinity Stacker』『Spark Stack』などが新たに登場し続けている息の長いメカニクス。コアは「左右に動くブロックを1タップで落とし、PERFECTなら全幅維持・ミスなら幅が削られて徐々に細くなる」という純粋なタイミングゲームで、操作はタップ1つ・1ラウンドが短く・スコア自慢で人に勧めやすい(スマホファースト/縦画面/単一HTML向きの条件を満たす)。

中毒性の核は**「連続PERFECTでコンボ倍率が跳ね、視覚効果がエスカレートする」**点にある(『Spark Stack』: "Perfect alignment triggers combo chains that increase your score and create colorful visual effects."/『Infinity Stacker』: "Combo multiplier - chain perfect drops to send your score sky-high.")。ハイパーカジュアル設計の通説でも「最高のタップだけが最大スコアを生む/Perfect Shotには明確に大きなフィードバックを与える」とされ、本作の太鼓・花火・幅回復・コンボ表示はこれを踏襲した。

出典(2026年6月 deep-research 調べ):

- Infinity Stacker (App Store) — https://apps.apple.com/ar/app/infinity-stacker/id6760549488
- Spark Stack: Combo Tower (App Store) — https://apps.apple.com/gb/app/spark-stack-combo-tower/id6760174596
- Stack レビュー (AndroidGuys) — https://androidguys.com/reviews/stack-a-very-simple-game-done-in-a-beautiful-way-review/
- Top 10 Game Mechanics for Hyper-Casual Games (mobilefreetoplay.com) — https://mobilefreetoplay.com/top-10-game-mechanics-for-hyper-casual-games/

## 技術構成

- **単一HTMLファイル**。canvas 2D で描画。外部ライブラリ・画像なし(疑似3Dの段・提灯の灯り・花火・粒子はすべてコード生成)。
- フォントのみ Google Fonts(Reggae One / Dela Gothic One / Zen Maru Gothic)を CDN 読み込み。
- 効果音は **Web Audio でリアルタイム合成**(太鼓=低周波サイン+ローパスノイズ、鈴、花火、歓声=バンドパスノイズのスウェル)。コンボで太鼓の音程が上がる。
- **iOS消音スイッチ対策**: 初回タップで無音WAVを `<audio>` ループ再生してオーディオセッションを再生カテゴリへ切替、`AudioContext.resume()` の完了待ち、`onstatechange` で `interrupted` から自動復帰。
- iOSのダブルタップズーム無効化(`touch-action`・350ms以内の `touchend` preventDefault・`gesturestart`/`dblclick` 抑止)、入力は Pointer Events 統一で `click` 非依存。
- メインループ(rAF)はスタート画面の時点から回り、ループが触る状態は宣言時に全初期化(初回フレーム例外でループが死ぬのを防止)。
- 記録は localStorage(try/catch フォールバック)。
- `?bot=1` でPERFECT自動ドロップのデバッグ/撮影モード(スクリーンショット・バランス確認用)。

## テスト

1. **構文チェック**: インラインスクリプトを抽出して `node --check`。
2. **実ブラウザE2E**(puppeteer-core + ローカルChrome、ローカルHTTPサーバ経由、モバイルビューポート `hasTouch`): `pageerror`/console エラーがゼロ、canvas に十分な描画ピクセル(色数100超)、タップで段数・スコアが進みゲームオーバーまで遷移することを確認。
3. `?bot=1` のPERFECT自動積みで高い櫓(10段+コンボ・花火)が崩れず描画されることを目視確認(OGP撮影も兼用)。

### 教訓

- file:// だと同梱リソースで canvas が汚染され `getImageData` が SecurityError になるため、E2Eは必ずローカルHTTPサーバ経由で実施。
- スタート画面のボタンは座標固定タップだと外しやすい。E2Eは要素IDクリック＋状態遷移アサーションで検証した。
