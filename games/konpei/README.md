# こんぺい — 昭和の駄菓子屋バブルシューター

デザインコンセプト: 昭和の駄菓子屋カウンター — クラフト紙背景、木枠、値札タグ風HUD、淡い水飴色のこんぺいとうスター。

## プレイURL

https://nrtkumi.github.io/games/konpei/

## 対応環境

- スマートフォン（iOS Safari / Android Chrome）: タッチ操作
- PC（Chrome / Safari / Firefox）: マウスドラッグ操作 + キーボード（←/→で角度、Spaceで発射）

## 遊び方・ルール

画面上部に吊り下がったこんぺいとうキャンディのクラスターをめがけて、下のランチャーからキャンディを撃ち込む。

**操作**
- 画面をドラッグして発射角度を決め、指を離すと発射
- 破線の照準ガイドが壁への反射も表示する
- 壁に当てて反射させることでコンボを狙える

**スコア・チェーン表**

| 出来事 | 得点 |
|---|---|
| 3つ以上同色ポップ | 個数 × 10 点 |
| ぶら下がり落下 | 個数 × 20 点（ボーナス） |

| チェーン数 | 大まかな追加得点 |
|---|---|
| 1〜5個落下 | +20〜100 |
| 6〜10個落下 | +120〜200 |
| 11個以上 | +220〜 |

**ゲームオーバー**: こんぺいとうが画面下の赤いラインを超えたら終了。  
**難易度ランプ**: 20ショットごとに色数が増加（3色→5色）。18ショットごとに新しい列が天井から追加される（上限10ショットごとに短縮）。

## 作成の背景

バブルシューター（エイム→マッチ→ポップ）はアーケード起源の古典メカニクスだが、「クラスターが崩れてぶら下がりが落下する」連鎖ボーナスが強烈な可変報酬になり、2025年のハイパーカジュアル市場でも再評価されている。

- ハイパーカジュアルゲームの成長要因分析: [Studio Krew — How Hyper-Casual Games Are Driving Growth (2025)](https://studiokrew.com/blog/how-hyper-casual-games-are-driving-growth-in-mobile-game-industry-2025/)
- 2025年モバイルゲームトップトレンド: [MAF.ad — Top Mobile Games 2025](https://maf.ad/en/blog/top-mobile-games-2025/)
- ランチャーをサム（親指）ゾーンに置く設計根拠: [Smashing Magazine — Design for One-Hand Mobile Usage](https://www.smashingmagazine.com/2020/02/design-mobile-apps-one-hand-usage/)

チェーン落下の「いつ来るかわからない大報酬」は可変強化スケジュールそのもので、古典的な依存性フックとして実証済み。こんぺいとう（金平糖）という昭和の駄菓子屋アイコンに落とし込むことで、日本的な懐かしさと視覚的独自性を付与した。

## 技術構成

- 単一 HTML ファイル（CSS + JS インライン）
- Canvas 2D API によるこんぺいとう描画（放射状スパイク＋ラジアルグラデーション＋シュガーシーン）
- 六角形オフセットグリッド（奇数行に COL_W/2 のずれ）
- BFS によるフラッドフィル（同色クラスター検出）と連結性チェック（ぶら下がり検出）
- Web Audio API によるリアルタイム SFX（発射音・ポップ音・落下音・ゲームオーバー音）
- iOS Audio 対策: 無音 WAV を `<audio>` ループ再生してオーディオセッションを再生カテゴリに切り替え
- localStorage によるベストスコア・ベストチェーン永続化
- Pointer Events のみ（click 非依存）、ダブルタップズーム防止 3 点セット
- PC ワイド画面対応: `min(100vw, calc(100dvh * 0.56), 480px)` の縦長カラムに制限
- DPR: 毎フレーム `ctx.setTransform(DPR, 0, 0, DPR, 0, 0)` で再設定（コンテキスト喪失対策）

外部ライブラリ: Google Fonts (Reggae One / Zen Maru Gothic)

## 実施したテストと教訓

1. **構文チェック**: `node --check` — PASS
2. **ヘッドレスロジックテスト**: DOM/Canvas/AudioContext をスタブし Node.js で 11 項目検証
   - initGame / randomColor 境界 / setCell-getCell / floodFill 3セル / findDisconnected / 3マッチポップスコア / 落下チェーン / addNewRow シフト / ゲームオーバー判定 / ベストスコア保存 — 全 PASS
3. **実ブラウザ E2E（最重要）**: puppeteer-core + ローカル Chrome (`python3 -m http.server`) でモバイルビューポート (hasTouch:true, dSF:2) にて検証
   - ページエラー: なし / コンソールエラー: なし（favicon 404 のみ、仕様上無視）/ Canvas 描画ピクセル: あり / 10 ショット後スコア加算: 確認 — PASS
4. **OGP**: puppeteer で 30 ショット後のカラフルなクラスター状態を撮影、`sips` で 1200×630 JPEG 化 — PASS

**教訓**: AudioContext の `currentTime` を返す `o.frequency.setValueAtTime(f, t + 0.08)` のような算術式は、再帰プロキシで AC をスタブすると `t + 0.08` が `[object Object] + 0.08` になり TypeError になる。ロジックテストでは sfx 関数を呼ばない環境を作るか、適切な数値を返すスタブを用意すること。
