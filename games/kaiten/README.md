# カイテン(KAITEN)

流れてくる寿司皿をタップで取って「役」を作り、**チップ×倍率を爆発**させて売上ノルマを食べ切る、回転寿司ローグライト。

- **プレイURL**: https://nrtkumi.github.io/games/kaiten/
- **対応環境**: スマホ(縦画面・タッチ)/ PC(マウス+キーボード 1-6・Space・X・G)
- **デザインコンセプト**: 「寿司湯呑」— 白磁×呉須藍の魚偏漢字パターン×朱の落款。倍率爆発の瞬間だけ金。フォントは筆文字(Yuji Syuku)+明朝(Shippori Mincho)

## 遊び方

1. ベルトを流れる寿司皿をタップして手元に取る(最大5枚)。**取らずに見送るのも自由**だが、1巡に流れる皿は40枚で打ち止め
2. 「いただきます」で手札を食べると **チップ合計 × 役倍率** が売上になる
3. 限られた「食べる回数」(基本4回)で売上ノルマを達成すれば1巡クリア
4. クリア報酬の小判で屋台の「薬味」(パッシブ強化、最大5個)を買い、次の巡へ
5. ノルマは巡ごとに×1.33で増加。3の倍数の巡は「大皿」(ボス: 早回しベルト/銅皿の嵐)。**のれん八巡**でクリア、以降はエンドレス

### 役一覧

| 役 | 条件 | 倍率 |
|---|---|---|
| ペア | 同ネタ2枚 | ×2 |
| 三貫 | 同ネタ3枚 | ×4 |
| 同色フラッシュ | 5枚すべて同色の皿 | ×5 |
| 松竹梅 | 5枚すべて違うネタ | ×3 |
| 四貫 | 同ネタ4枚 | ×7 |
| 定食 | 同ネタ3枚+同ネタ2枚 | ×8 |
| 五貫 | 同ネタ5枚 | ×12 |

チップは皿の色で決まる: 銅5 / 銀10 / 金20 / 黒40(レアほど流れてこない)。

### 薬味(ジョーカー)

わさび(同ネタ役の倍率+3)、ガリ(1巡1回ベルト総入替)、醤油(皿ごとにチップ+4)、あがり(食べる回数+1)、漬けダレ(まぐろ・大トロ2倍)、金箔(金皿・黒皿の出現2倍)、海苔(フラッシュ・松竹梅+3)、大食いの帯(手札6枚)、湯呑(捨てる+2)、出前持ち(開始時に皿1枚)、招き猫(クリアごと小判+2)、玉子職人(たまご・かっぱ3倍)の12種から、巡間の屋台に3種が並ぶ。

## 作成の背景(市場調査の要点)

「中毒性」をコンセプトに、2026年6月時点のリサーチから以下を組み合わせた:

- **チップ×倍率の「数字爆発」**: Balatroの中毒ループ(役×ジョーカーで倍率が compounding する快感)は2026年現在も市場最強クラス。Slay the Spire 2が初週300万本、SteamのDeckbuilders Festに約2,800作が並ぶなど、ローグライトデッキビルダーは定番ジャンル化([Choost Games](https://choostgames.com/blog/best-roguelite-deckbuilders-2026/)、[Switchblade Gaming](https://www.switchbladegaming.com/strategy-games/games-like-balatro/))。Scrabble型などBalatroの「役」を別ドメインに移植する流派が活発([PC Gamer](https://www.pcgamer.com/games/roguelike/balatro-meets-scrabble-in-this-new-roguelike-from-the-creators-of-timesplitters-and-im-just-thrilled-to-finally-find-a-word-game-that-lets-me-score-with-swear-words/))
- **変動報酬+消える希少性**: 「次に何の皿が来るか分からない」変動比率報酬と、「取らないと流れて消える」FOMOの組み合わせ([Compulsion loop — Wikipedia](https://en.wikipedia.org/wiki/Compulsion_loop))。回転寿司×時間プレッシャーの先行例として寿司打([sushida.net](https://sushida.net/))、物理ボドゲの「回転ずしポーカー」([ボドゲーマ](https://bodoge.hoobby.net/games/kaiten-zushi-poker/reviews/23042))はあるが、**リアルタイムのベルトドラフト×役×倍率爆発のブラウザゲームは見当たらない**
- **1〜3分セッション+メタ進行**: ハイブリッドカジュアルの定石(D7リテンションがハイパーカジュアルの2.5倍)([Antier](https://www.antiersolutions.com/blogs/hybrid-casual-games-vs-hypercasual-whats-driving-higher-retention-ltv-and-revenue-in-2026/)、[Game Growth Advisor](https://gamegrowthadvisor.com/blog/2026-04-16-hybrid-casual-game-design-strategy-2026/))。1巡=1〜2分、即リスタート
- **5秒で理解できる操作**: 2026年のバイラルブラウザゲームは「複雑さより直感」([primepulselogic](https://primepulselogic.com/browser-games-2026-viral-trend/)、[ArionisGames](https://arionisgames.com/blog/top-casual-games/))

企画段階で **codex(GPT)にデザインレビュー**を依頼し、以下を反映:

- 「高速で流れる皿を反射神経で取る」案を、**低速ベルト(タップ猶予十分)+皿の総数制限**による戦略的push-your-luckへ転換
- ネタ別チップを廃止して皿色=チップに統一(理解コスト削減)
- 役倍率の平準化(五貫×25→×12)、ノルマ成長×1.7→×1.33
- ベルト=上半分/手札・ボタン=下半分の操作分離、捨てるは「選択→ボタン」方式

バランスは固定シードの**1万ランシミュレーション**で調整(無強化ボットの1巡目突破率99.9%・8巡目=薬味5個で89.7%。実プレイは取り逃しがある分これより下がる)。

## 技術構成

- 単一HTML(canvas 2D、外部ライブラリなし)。フォントのみGoogle Fonts
- ネタ画像8種とOGPは **codex CLI経由のgpt-image-2** で生成(「和モダンなフラットイラスト調・純白背景・真上から」の前置きで統一)→ 256px JPEGに軽量化(計140KB)。読み込み失敗時は漢字スタンプ描画にフォールバック
- 効果音はWeb Audioでリアルタイム合成(呂音階のカウントアップ、倍率スラム等)。iOS消音スイッチ対策(無音WAVループ+resume待ち+interrupted復帰)実装済み
- ベスト記録はlocalStorage(try/catch)
- ダブルタップズーム3点対策、Pointer Events統一(clickなし)、`ctx.setTransform`毎フレーム再設定、PCワイド画面は縦カラム制限

## 実施したテスト

1. `node --check` による構文チェック
2. DOMスタブ+Nodeでのヘッドレスロジックテスト30件(役判定×薬味の全組み合わせ、ノルマ・ボス配置、開始→食事→クリア→購入→次巡→敗北の状態遷移、ベスト保存)
3. 固定シード1万ラン×5条件のバランスシミュレーション
4. puppeteer-core+ローカルChrome(モバイルビューポート・タッチ)の実ブラウザE2E: タイトル→開始→皿タップ取得→食事演出→クリア→ショップ購入→2巡目→放置、pageerror/consoleエラーゼロ、canvas描画ピクセル確認、スクリーンショット目視
5. codexによるコードレビュー(バグ・iOS固有問題・パフォーマンス)

## 開発で得た教訓

- 薬味の効果を「説明文」と「ロジック」の2箇所に書くと片方だけ直して食い違う(わさび+3への変更でロジック側を直し忘れ、テストが検出)。効果値は定数1箇所にまとめるべきだった
- リアルタイム要素のあるドラフトは、ボットシミュレーション(完全情報)が実プレイより1〜2割甘く出る前提でバランスを読む
