# 小判 — 江戸の両替商

**デザインコンセプト**: 漆黒の勘定台に金貨を積む江戸の両替商 — 金の楕円小判、朱赤の危険ライン、筆文字風の額面表示。

## プレイURL

[https://nrtkumi.github.io/games/koban/](https://nrtkumi.github.io/games/koban/)

## 対応環境

- iOS Safari (iPhone / iPad)
- Android Chrome
- PC ブラウザ (Chrome / Firefox / Safari)

## 遊び方・ルール

列をタップ（またはドラッグ）して小判を落とす。同じ額面の小判が**縦隣(直下)**・**横隣(左右同行)**に接触すると自動マージして上の額面に昇格する。マージが連鎖(chain)すると倍率ボーナスが乗る。列が上まで埋まるとゲームオーバー。

### 額面ラダー

| 段階 | 額面 | スコア値 |
|------|------|---------|
| 1 | 一文 | 1 |
| 2 | 五文 | 5 |
| 3 | 百文 | 100 |
| 4 | 一分 | 500 |
| 5 | 一両 | 1,000 |
| 6 | 小判 | 5,000 |
| 7 | 大判 | 20,000 |
| 8 | 千両箱 | 100,000 |

### スコア計算

- マージ1回 = 昇格後の額面値
- 連鎖 n 回目: ×(1 + n×2) 倍
- ベストスコアと最高位はlocalStorageに永続保存

### マージ判定ルール

落とした小判が列の一番上に積まれた後、以下の隣接セルを検索:
1. 同じ列の直下セル (row-1)
2. 左列の同高セル
3. 右列の同高セル

いずれかに同額面があればマージし、元の列の上端に昇格コインを配置。その後、新コインで再度マージ検索(cascade)。

## 作成の背景

2025〜2026年のモバイルゲームシーンでは、マージ系・2048系ループが依然として中毒性の高いバイラルカジュアルメカニクスとして君臨している。「同じものを近づけて合体」という直感的な操作と、連鎖(chain)によるドーパミン予測誤差が強力な「あと一手」ループを生む。

- AppMagic Q3 2025 レポートでは Hybrid-Casual ジャンルにおいてマージ系メカニクスが最多採用カテゴリと報告: https://appmagic.rocks/blog/q3hybrid2025
- MAF 2025年上半期ランキングでも Drop & Merge 系タイトルが上位に連続ランクイン: https://maf.ad/en/blog/top-mobile-games-2025/
- スマホ片手持ちの「サムゾーン」研究(Smashing Magazine)により、列タップ型の縦積みUIが最も親指で届きやすいと確認: https://www.smashingmagazine.com/2020/02/design-mobile-apps-one-hand-usage/

江戸時代の貨幣制度(一文→千両箱)というユニークな額面ラダーで、世界観と数値スケールを同時に表現している。

## 技術構成

- **単一HTMLファイル** (CSS・JS全インライン)
- **Canvas 2D API** — 楕円小判をグラデーション+縁取りで描画、DPR毎フレーム `setTransform` 再適用
- **Web Audio API** — 落下音・マージ音・連鎖音・ゲームオーバー音をリアルタイム合成(外部ファイルなし)
- **iOS音声対策** — 無音WAVループ(`<audio playsinline>`)でAudioSessionをPlaybackカテゴリに切り替え
- **ダブルタップズーム防止** — `touchend` 350ms間隔検出+`dblclick`/`gesturestart` preventDefault
- **Pointer Events統一** — `click`不使用、タッチ・マウス共通
- **PCワイド対応** — `min(100vw, calc(100dvh*0.6), 480px)` 縦カラム中央寄せ
- **永続化** — localStorage (try/catch フォールバック)
- **フォント** — Kaisei Decol (タイトル・HUD) + Zen Maru Gothic (ラベル) via Google Fonts

## 実施したテストと教訓

1. **構文チェック**: `node --check` → PASS
2. **ヘッドレスロジックテスト**: DOMスタブのNodeで実行
   - 縦方向マージ(下セルと合体)→五文生成 PASS
   - カスケード合体(連鎖マージ)→百文生成、スコア加算 PASS
   - 横方向マージ(左右同行) PASS
   - 列オーバーフロー検出 PASS
   - localStorage保存・読み込み PASS
3. **実ブラウザE2E(puppeteer-core + Chrome)**: モバイルビューポート(hasTouch:true, dSF:2)
   - pageerror ゼロ、consoleエラー(ogp.jpg以外)ゼロ PASS
   - Canvasピクセル描画確認 PASS
   - 16回タップ操作 → スコア上昇・最高位「五文」確認 PASS
4. **OGP**: 1200x630 JPEG、`sips` で変換、`file` コマンドでJPEG確認 PASS

**教訓**: `file://` プロトコルではcanvas getImageDataがSecurityErrorになるため、必ずpython3 HTTPサーバ経由でテスト。rAFループはタイトル画面から起動し、全状態を宣言時初期化してinitへの依存を排除。
