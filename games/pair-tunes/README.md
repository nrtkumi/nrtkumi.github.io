# ペアチューン(PAIR TUNES)

カードをめくると曲のワンフレーズが2秒だけ流れる、**耳でそろえる神経衰弱**。
iTunes Search API の30秒プレビューを使い、実際の楽曲で遊べる。

- **プレイURL**: https://nrtkumi.github.io/games/pair-tunes/
- **対応環境**: スマホ(iOS Safari / Android Chrome)優先、PCブラウザ可。縦画面・タッチ操作のみで完結
- **通信**: 楽曲取得・プレビュー再生のためネットワーク接続が必要

## 遊び方

1. モード(かんたん/むずかしい)とジャンル(ヒットチャート / J-POP / アニソン / 洋楽 / 懐メロ 90s-00s)を選ぶ
2. 12枚(6曲×2)のカードをめくると、その曲が**2秒**だけ流れる
3. **同じ曲のペア**を耳で探してそろえる。表のカードはもう一度タップすると聴き直せる(ペナルティなし)
4. 全ペアそろえるまでの**手数とタイム**で記録に挑戦。そろえた曲はジャケットが出て、クリア後「今日の6曲」プレイリストになる

| モード | ペアの音 |
|---|---|
| かんたん | 同じ部分が流れる |
| むずかしい | **同じ曲の違う部分**が流れる(曲として同じかを耳で判断) |

★評価: 8手以内 ★★★ / 12手以内 ★★ / それ以上 ★。ベスト(最少手数)はモード×ジャンルごとに保存。

## 作成の背景(市場調査)

- 神経衰弱(memory matching)は2026年もブラウザカジュアルの定番で、[100+テーマ・難易度展開・デイリーチャレンジ化](https://memorymatching.com/)が進む。[脳トレ・認知ウェルネス文脈での人気も継続](https://lexiloot.app/types-of-brain-games-fun-sharper-skills/)([Analytics Insight](https://www.analyticsinsight.net/top-list/top-10-daily-brain-games-to-boost-focus-and-memory-in-2026))
- 音楽ゲームは [Tiles Hopなどタップ系ハイパーカジュアルがDL数で主流](https://rolling-sky.com/blog/best-mobile-rhythm-games-2026/)、[国内はプロセカ・Cytus II等の本格音ゲーが人気](https://app-liv.jp/games/otoge/0285/)
- 本格リズムゲームは波形解析が必要だが、iTunesプレビューはWebKitの `decodeAudioData` で確実にデコードできるとは限らない(検証でPlaywright WebKitはAACデコード失敗)。解析不要で音楽体験が成立する「定番パズル×実楽曲」の掛け合わせとして、**音だけの神経衰弱**に決定
- サイト内の既存ゲーム(クイズ・破壊・収集・対戦)とジャンルは被らない。むずかしいモードの「同じ曲の違う箇所」判定と、クリア後のプレイリスト化(音楽発見)が独自要素

## 技術構成

- 単一HTML(`index.html`)・依存ライブラリなし。フォントのみGoogle Fonts(Bungee / Zen Kaku Gothic New / Space Mono)
- **楽曲データ**: iTunes Search API。**`media=all&entity=song`** + `kind === "song"` フィルタ(`media=music` はiOS Safariで `musics://` へリダイレクトされ全滅 — phrase-hunterで特定済みの罠)。fetch失敗時はJSONP(8秒タイムアウト)
- **ヒットチャート**: iTunes旧RSS(`jp/rss/topsongs`)。プレビューURL込み・CORS可・iOS可
- **レート制限対策**: 3アーティスト×limit50のバッチ取得+足りた時点で打ち切り、プールはメモリ+localStorageに12時間キャッシュ。「もう一度」はAPIを呼ばない。全失敗時は期限切れキャッシュで起動
- **再生**: 曲ごとに1つの `<audio>` を共有し、カードタップ起点で再生(自動再生制限を回避)。むずかしいモードは `currentTime` で再生開始位置を変える(0秒/14秒)。**スニペット停止は音声イベントに頼らず壁時計(`performance.now`)でrAFループが行う** — 読み込み遅延や再生失敗でもゲーム進行が破綻しない
- 6曲はできるだけ別アーティストから選出(聴き分けの公平性)。カラオケ/Live/Remix除外・曲名正規化で紛らわしいカードを防止
- 効果音はWeb Audio合成。iOS消音スイッチ対策(無音WAVループ+`resume()`待ち+`interrupted`復帰)は hakai と同方式
- メインループ(rAF)はスタート画面の時点から回り、参照する状態は全て宣言時に初期化

## 実施したテスト

1. **構文チェック**: インラインscriptを抽出して `node --check` → PASS
2. **ヘッドレスロジックテスト**: DOM/Audio/fetch/localStorageをスタブし、モード切替→12枚生成(ペアのaudio共有・ハードのオフセット差)→再生→2秒自動停止→不一致(手数加算・伏せ直し)→一致→全ペア→クリア(★・プレイリスト・ベスト保存)→キャッシュリプレイ→チャート経路まで一巡 → PASS
3. **実ブラウザE2E(Chrome + WebKit両方)**: モバイルビューポート+タッチで実APIを使い、不一致1回→全ペア成立→クリア画面→プレイリスト6曲→ベスト保存まで実プレイ。pageerrorゼロ。WebKit(iPhone 14エミュレーション)でも完走 — **iOS固有のAPI問題(media=music等)を踏んでいないことをリリース前に確認**

### 開発で得た教訓

- **スクリーンショット目視は今回も仕事をした**: 固定配置のサウンドボタンがプレイ画面のHUD(TIME表示)に重なる問題は、E2EのアサーションではなくボードのスクリーンショットE2E目視で発見
- リリース前のE2EにWebKitを最初から含めた(phrase-hunterのiOS障害の再発防止)。モバイル向けゲームはChromeだけのE2Eでは不十分
- 音の停止・カードの伏せ直しなどの進行制御は、audioイベント(`ended`等)ではなく壁時計+rAFで行うと、音源の読み込み失敗・遅延があってもゲームが止まらない
