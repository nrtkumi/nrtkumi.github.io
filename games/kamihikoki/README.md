# カミヒコーキ (KAMIHIKOKI) — どこまで飛ばせる?

**長押し**でパワーをためて、ゲージが**てっぺん**に来た瞬間に**指を離して**紙ヒコーキを発射。飛んでいる間は**タップで羽ばたき**、**上昇気流(サーマル)**と**ブーストリング**を乗り継いで、ノートの空をどこまでも遠くへ飛ばす——距離を伸ばす一発勝負の飛距離ゲーム。

- **プレイURL**: https://nrtkumi.github.io/games/kamihikoki/
- **対応環境**: スマホ(iOS Safari / Android Chrome)・PCブラウザ。縦画面・タッチ操作。PCはスペースキー(押し続けてためる→離して発射、飛行中は連打で羽ばたき)でも操作可。

## 遊び方

1. 「飛ばす!」をタップ。
2. 左のパワーゲージが上下に振れる。**指で押している間にゲージがてっぺんへ来た瞬間、指を離す**と最大パワーで発射。離すタイミングが早すぎ/遅すぎると弱い発射になる。
3. 飛行中は**タップで羽ばたき**(はばたきスタミナを消費して上昇)。
4. **ブーストリング**をくぐると加速＆スタミナ回復、**上昇気流**に入ると一気に上昇。これらを乗り継ぐと飛距離が伸びる。
5. 着地したら飛距離が確定。記録更新を目指して何度でも。

### 飛距離を伸ばすコツ

| 要素 | 効果 |
|------|------|
| 発射パワー(ゲージのピークで離す) | 初速が決まる。最大パワーで一番遠くへ |
| はばたき(タップ) | スタミナを使って上昇。落下前にもうひと伸び |
| ブーストリング | 前方へ加速＋スタミナ回復 |
| 上昇気流 | 気流の中にいる間ぐんぐん上昇＋スタミナ回復 |
| 着地スキップ | 速い角度で着地するとバウンドして距離を稼ぐ |

記録(最高飛距離)は localStorage に保存される。

## デザインコンセプト

**ノートと方眼紙の空**。水彩の雲、クレヨンタッチの太陽と鳥、パステルの空——子どもが紙ヒコーキを飛ばす昼間の空気感。フォントは手書き風の Yusei Magic とまるい Mochiy Pop One。明るい昼空・緑の地面・ピンクのUIで、夜系・ネオン系の他作品と意図的に対照させた。

## 作成の背景

「長押しでためて離す(charge-and-release)」は、ハイパーカジュアルの代表的メカニクスの一つ。純粋な hold-to-charge の代表は『Stick Hero』(棒を長押しで伸ばし、離して隣の足場へ渡る)や『Sling Kong』(パチンコ式に引いて離し、足場から足場へ跳ぶエンドレスクライム)で、App Store のエディターは Sling Kong を "Flinging a monkey from peg to peg feels great — especially when you narrowly avoid buzzsaws and flame jets" と評する。一方『Slap Kings』『Jetpack Jump』のように**ゲージが左右に振れ、緑ゾーン/ピークで離す**タイプ(タイミング・ロック式)も中毒性が高い("The meter will go back and forth - time it right for maximum power!")。

本作はこの**振れるゲージのピークで離す**チャージに、飛距離ゲーム(『Learn to Fly』系の「どこまで飛ばせる」)の伸ばす快感を合わせた。1ラウンドが短く、飛距離という単一の数字でスコア自慢でき、人に勧めやすい。ハイパーカジュアル設計の通説「最高のタイミングだけが最大スコアを生む/Perfect には大きなフィードバックを」を踏まえ、ピーク発射に強い手応え(WHOOSH・振動・速度線)を与えている。

出典(2026年6月 deep-research 調べ):

- Stick Hero (App Store) — https://apps.apple.com/us/app/stick-hero/id918338898
- Sling Kong (App Store) — https://apps.apple.com/us/app/sling-kong/id989080135
- Slap Kings (App Store) — https://apps.apple.com/us/app/slap-kings/id1500010832
- Popular Hyper-Casual Game Mechanics (Mobidictum) — https://mobidictum.com/popular-hypercasual-game-mechanics/
- Top 10 Game Mechanics for Hyper-Casual Games (mobilefreetoplay.com) — https://mobilefreetoplay.com/top-10-game-mechanics-for-hyper-casual-games/

## 技術構成

- **単一HTMLファイル**。canvas 2D で描画。外部ライブラリ・画像なし(雲・太陽・鳥・紙ヒコーキ・気流・リングはすべてコード生成)。
- フォントのみ Google Fonts(Mochiy Pop One / Yusei Magic)を CDN 読み込み。
- **物理**: 重力＋紙ヒコーキの滑空モデル(前進速度に応じて落下を弱める揚力。ただし揚力は重力未満に制限し、必ずゆっくり沈んで着地する)。前進速度は徐々に減衰し、飛行が自然に終わる。上昇気流・リング・羽ばたきで再加速・上昇できる設計。地面では速い角度の着地でバウンド(スキップ)。
- 効果音は **Web Audio でリアルタイム合成**。チャージ中は音程の上がる持続音、発射WHOOSH、羽ばたき、リングのチャイム、気流、バウンド、新記録ファンファーレ。
- **iOS消音スイッチ対策**: 無音WAVループ＋`resume()` 完了待ち＋`onstatechange` 復帰。
- ダブルタップズーム無効化、入力は Pointer Events(`pointerdown`/`pointerup`/`pointercancel`)。チャージは押下〜離しで完結し `click` 非依存。
- メインループはスタート画面から回り、状態は宣言時に全初期化。記録は localStorage(try/catch)。
- ルート要素は `max-width: min(480px, 64vh)` で縦長カラムに制限し、PCワイド画面でも巨大化しない。

## テスト

1. **構文チェック**: `node --check`。
2. **実ブラウザE2E**(puppeteer-core + ローカルChrome、ローカルHTTPサーバ経由、モバイルビューポート): `pageerror`/console エラーゼロ、canvas 描画ピクセル(色数180超)を確認。
3. **物理の調整スイープ**: チャージ長押し時間を変えて3回ずつ計測し、**発射パワーと飛距離が単調に対応**(power 0.9→約550m、0.3→約160m)し、入力なしでも約1.6秒で必ず着地、羽ばたき/気流で飛距離が伸びることを確認。当初は滑空の揚力が重力を上回り**永久に着地しないバグ**があり、揚力を重力未満に制限して解消した。

### 教訓

- 滑空ゲームは「揚力 < 重力」を保証しないと無限飛行になる。ヘッドレスで着地までの秒数・距離を実測して検証した。
- ゲージが常時振れる仕様では「長押し時間」とパワーが一対一にならない(ピークを過ぎると下がる)。これは仕様(ピークで離すスキル)であり、計測時はパワー値そのものを基準にした。
