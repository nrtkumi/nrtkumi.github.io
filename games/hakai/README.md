# HAKAI — 壊して、強くなれ。

ブロック崩し × ローグライトの1人用ブラウザゲーム。

- **プレイURL**: https://nrtkumi.github.io/games/hakai/
- **対応環境**: スマホ(iOS / Android)・PCブラウザ。インストール不要、単一HTMLファイル
- **旧URL** `/hakai-breaker.html` は本ディレクトリへリダイレクトされます

## 遊び方

1. **ドラッグで狙いを定めて、離して発射**。ボールは壁とブロックで跳ね返る
2. ブロックには体力(数字)があり、ボールが当たるたびに削れて、0で破壊
3. シアンの **+1** ジェムを拾うと次のターンからボールが1個増える
4. 1ターンごとにブロックが1段降りてくる。**下の赤いラインに届いたらゲームオーバー**
5. **5ウェーブごとに強化を3択から1つ選択**。重ね取りでどんどん強くなる
6. 長期戦は右上の **×1/×2/×3** ボタンと自動加速でテンポアップ

### 強化一覧

| 強化 | 効果 |
|---|---|
| 🟡 マルチボール | ボールが2個増える |
| 💥 パワーアップ | ボールの威力 +1 |
| ✨ 会心の一撃 | 15%の確率で3倍ダメージ(重複可) |
| 🧨 爆裂弾 | 破壊時、周囲8マスに爆発ダメージ(連鎖あり) |
| 🪡 貫通 | 20%の確率でブロックを貫通(重複可) |
| 🔨 粉砕 | 毎ウェーブ開始時にブロック1個を自動破壊 |

ベストスコアは端末の localStorage に保存されます。

## 作成の背景(市場調査)

2026年6月に実施した市場調査をもとに、ジャンルとメカニクスを決定した。

- **ローグライト×シンプル操作が現在の勝ち筋**: 2025年はBalatroに続き、ブロック崩し×ローグライトの「Ball X Pit」がヒット。「1ランごとに強化を選ぶ」ループがインディーゲームの主流に
  - [10 Indie Game Trends from 2025 (No Small Games)](https://nosmallgames.com/2026/01/10-indie-game-trends-from-2025/)
  - [The Best New Roguelikes and Roguelites (Rogueliker)](https://rogueliker.com/best-new-roguelikes/)
- **ハイパーカジュアルは「5秒で理解できる操作」**: Block Blast(3億DL)、Helix Jump(1億DL)など、タップ/スワイプだけの直感操作が強い
  - [世界的にヒットしたハイパーカジュアルゲーム9選 (source code)](https://s-code.co.jp/hypercasualgame-hit/)
  - [The top most viral and addictive casual games (ArionisGames)](https://arionisgames.com/blog/top-casual-games/)
- **2026年は「ハイブリッドカジュアル」へ**: 簡単な操作+成長システムの組み合わせがトレンド
  - [Casual Games Market in 2026 (Udonis)](https://www.blog.udonis.co/mobile-marketing/mobile-games/casual-games)
  - [Hybrid Casual Games 2026 (Game Growth Advisor)](https://gamegrowthadvisor.com/blog/2026-04-16-hybrid-casual-game-design-strategy-2026/)

この3点を踏まえ、**「引っ張って跳ね返すブロック崩し(BBTAN系)×ローグライト強化選択」**に決定。サイト内の既存ゲーム(スイカゲーム=マージ系)とジャンルが被らないことも条件にした。

## 技術構成

- 依存ライブラリなしの単一HTMLファイル(Canvas 2D + Web Audio API)
- スマホファースト: タッチ操作(Pointer Events + `touch-action: none`)、セーフエリア対応(`viewport-fit=cover` + `env(safe-area-inset-*)`)、タップハイライト除去
- **iOS消音スイッチ対策**: Web Audioの音はマナーモードで消えるため、初回タップ時に無音WAVを `<audio>` 要素でループ再生してオーディオセッションを再生カテゴリへ切り替える(unmute.js方式)。`AudioContext.resume()` のPromise完了を待ってからアンロック判定
- 効果音はすべてWeb Audioでリアルタイム合成(外部音源ファイルなし)
- ベストスコアは localStorage(`hakai_best`)

## テスト

リリース前に以下を実施:

1. `node --check` によるスクリプト構文チェック
2. DOMスタブを使ったNode上のヘッドレスロジックテスト(発射→衝突→ウェーブ進行→強化選択→ゲームオーバー→記録保存)
3. puppeteer-core + 実Chrome(モバイルビューポート・タッチイベント)でのE2Eテスト: スタート→ドラッグ発射を7ショット連続実行し、描画ピクセル・スコア進行・`pageerror` ゼロを確認

### 教訓

初回リリース時、メインループがスタート前から回るのに状態配列を `init()` 内でしか初期化していなかったため、初回フレームの例外で描画ループが停止する不具合があった(ヘッドレステストはスタート後しかフレームを回しておらず検出漏れ)。**実ブラウザでのE2Eテストをリリース前に必ず行うこと。**
