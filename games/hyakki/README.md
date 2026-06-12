# HYAKKI -百鬼-

指1本のドラッグで白狐を操り、自動攻撃で百鬼夜行を祓いながら夜明け(6分)まで生き残るサバイバーローグライト(Bullet Heaven / Vampire Survivors系)。

- **プレイURL**: https://nrtkumi.github.io/games/hyakki/
- **対応環境**: スマホ(縦画面・タッチ)/ PC(WASD・矢印キー)。依存ライブラリなしの単一HTML。

## 遊び方

1. 画面のどこでもドラッグすると白狐が動く(フローティング仮想スティック)。攻撃は全て自動。
2. 妖を倒すと落ちる**魂(青い火)**を拾ってレベルアップ。3択から武器/お守りを選ぶ。
3. **武器をLv5にして対のお守りを持つと、次のレベルアップで「進化」**が出現し別物の強さになる。
4. **小判**は倒れても持ち帰れる。タイトルの**⛩社**で永続強化(体力/攻撃/速度/吸引/招福)。
5. 夜6:00(リアル6分)まで生き残れば**夜明け=勝利**。さらに「千年夜」(エンドレス、敵が1.5倍ずつ強化)に続行可能。
6. 結果画面からスコアをシェアできる(Web Share API / クリップボード)。

### 武器と進化

| 武器 | 効果 | 対のお守り | 進化 |
|---|---|---|---|
| 霊符 | 最寄りの妖へ札を放つ | 呪玉(攻撃+10%) | 百符乱舞 |
| 狐火 | 周囲を巡る焔 | 迷い提灯(範囲+10%) | 狐火車輪 |
| 雷太鼓 | 天より雷(範囲) | 数珠(間隔-7%) | 万雷 |
| 鎌鼬 | 貫通する風刃が往復 | 韋駄天の草鞋(速度+6%) | 大旋風 |
| 結界 | 触れる妖を祓う輪 | 御神酒(体力+20&全回復) | 大結界 |
| 破魔矢 | 進む向きへ貫く矢 | 勾玉(吸引+25%) | 天羽々矢 |

武器スロット4・お守りスロット4。敵は提灯お化け/からかさ/鬼火/幽霊/濡女/鬼の6種+エリート、ボスは からかさ大将(1:55)→がしゃどくろ(3:35)→酒呑童子(5:00)。

## 作成の背景(市場調査)

2026年6月に実施。サブエージェント4本でバイラル事例/ハイパーカジュアル設計/インディートレンド/ゲームフィールを並行調査し、以下から本企画に絞った:

- **サバイバー系(Bullet Heaven)はジャンルとして確立**。2026年5月にSteamが「Bullet Heaven」を公式タグ化。Megabonk(2025、同接9万)など個人開発ヒットが続出し、配信映え・「画面が埋まる全能感」が拡散の核。サイト内未カバーのジャンルだった。
  - https://en.wikipedia.org/wiki/Vampire_Survivors%E2%80%93like
  - https://www.pcgamer.com/games/roguelike/a-slot-machine-for-gags-disguised-as-a-vampire-survivors-like-is-taking-over-steam-and-im-just-as-addicted-to-pulling-the-lever-as-the-other-90k-people-playing-it/
- **片手バーチャルスティック+自動攻撃はスマホ最適化の完成形**(Survivor.io型)。操作は移動1軸、意思決定はレベルアップ時の3択だけ。セッション3〜6分、リスタート即時、「ラン間に必ず何か残る」設計が鉄則。
  - https://mobilefreetoplay.com/control-mechanics/
  - https://gamegrowthadvisor.com/blog/2026-04-16-hybrid-casual-game-design-strategy-2026/
- **シナジー設計**は「単純×単純=創発」(Balatro)、ランダム3択ドラフト、レリック層の分離が定石。本作は武器×対パッシブ→進化の二層で実装。
  - https://errorandexp.substack.com/p/unpacking-balatros-addicting-game
  - https://zenorogue.medium.com/engine-builders-an-analysis-cd75c4fdd28c
- **ハイブリッドカジュアル化**: ラン内ビルド+ラン間恒久強化の二重ループが2025-26年の標準。Ball x Pit(2025)も「既知メカニクス+ローグライト+拠点強化」でAAAを上回るヒット。
  - https://howtomarketagame.com/2025/12/01/ball-x-pit-my-game-of-the-year-2025/
  - https://www.antiersolutions.com/blogs/hybrid-casual-games-vs-hypercasual-whats-driving-higher-retention-ltv-and-revenue-in-2026/
- **juice(ゲームフィール)は5層同時**(アニメ/パーティクル/画面効果/音/数字)。ピッチ上昇連鎖・ヒットストップ・スケールパンチ等をBalatro分析から採用。
  - https://blakecrosley.com/guides/design/balatro
  - https://abagames.github.io/joys-of-small-game-development-en/make_game_juicy.html
- **バイラル設計**: ネタバレなし自慢シェア(Wordle型の結果テキスト)、配信者リアクション装置(見た瞬間ルールが分かる)、「あと1回」ループ。
  - https://webflow.com/blog/wordle-design
  - https://howtomarketagame.com/2025/05/12/benchmark-itch-io-traffic/

テーマは百鬼夜行×白狐。ダーク背景+ネオン和風(2025-26のインディーUIトレンド「ダーク+発光アクセント、色に役割を固定」)で統一。

## 技術構成

- 依存ライブラリなしの単一HTML(canvas 2D)。フォントのみGoogle Fonts(Bungee / Zen Kaku Gothic New / Space Mono)。
- **描画**: 敵スプライトはグロー付きで起動時にオフスクリーンcanvasへプリレンダリング(被弾フラッシュ用の白版も生成)し、毎フレームdrawImage。dpr変換は毎フレーム `ctx.setTransform` で再設定。200体+パーティクル400でヘッドレスChrome 120fps。
- **衝突**: 敵は72pxセルの空間ハッシュで分離・接触判定。貫通弾はSetで多段ヒット防止、持続武器(狐火/結界)は敵ごとのヒットクールダウン。
- **入力**: Pointer Events統一(clickイベント不使用)。フローティング相対ジョイスティック+キーボード。ダブルタップズームは touch-action / dblclick・gesturestart 阻止 / 350ms連続touchend阻止の3点で無効化。
- **音**: Web Audioでリアルタイム合成(SE+テンポが夜の進行で上がるBGMスケジューラ)。iOS消音スイッチ対策に無音WAV `<audio>` ループ+`AudioContext.resume()`のPromise待ち+`interrupted`からの自動復帰。
- **永続化**: localStorage(try/catch)。小判・社の強化・ベスト記録・ミュート設定。
- **OGP**: ゲームプレイ実スクショ(puppeteerで敵を大量湧きさせて撮影)をHTML合成で1200x630に。
- メインループはタイトル画面から常時稼働(初回フレーム例外対策で全状態を宣言時初期化)。

## 実施したテスト

1. `node --check` による構文チェック
2. **ヘッドレスロジックテスト**(DOMスタブ+vm、23項目): タイトル→出陣→接触ダメージ→スポーン→レベルアップ→ボス→夜明け勝利→エンドレス続行→死亡→記録保存→リトライ→社の購入、全PASS
3. **実ブラウザE2E**(puppeteer-core+ローカルChrome、モバイルビューポート+タッチ、ローカルHTTP経由、18項目): pageerror/consoleエラーゼロ、canvasピクセル描画、タッチドラッグでの移動、レベルアップカードのタップ選択、ポーズ/再開、夜明け→リザルト→千年夜、保存。全PASS
4. 敵193体時のFPS計測(120fps)とスクリーンショット目視確認

## 開発で得た教訓

- 静止プレイヤーは約60秒で死ぬ=サバイバー系として正しい難度だが、ヘッドレステストでは「無入力で長時間回す」前提が崩れる。無敵化してから進行系を検証し、ダメージ系は敵を直置きして個別に検証するのが安定。
- E2Eのタッチドラッグを全方向スイープにすると変位が相殺されて「動いていない」ように見える。片方向ドラッグで検証する。
- 縦長カラムレイアウトのゲームはワイドビューポートでクリップしてもOGP映えしない。実スクショ+HTML合成が手軽で見栄えも良い。
