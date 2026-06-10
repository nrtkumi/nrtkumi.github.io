# イントロハンター(INTRO HUNTER)

1秒だけ流れるイントロを聴いて、4択から曲名を当てる音楽クイズゲーム。
iTunes Search API の30秒プレビューを使い、実際の楽曲で遊べる。

- **プレイURL**: https://nrtkumi.github.io/games/intro-hunter/
- **対応環境**: スマホ(iOS Safari / Android Chrome)優先、PCブラウザ可。縦画面・タッチ操作のみで完結
- **通信**: 楽曲取得・プレビュー再生のためネットワーク接続が必要

## 遊び方

1. ジャンルを選ぶ(ヒットチャート / J-POP / アニソン / 洋楽 / 懐メロ 90s-00s)
2. ▶ ボタンでイントロを **1秒** だけ聴く(何度でも聴き直し可)
3. 4択から曲名を選ぶ。わからなければ「もっと聴く」で再生時間を延長できるが得点が減る
4. ライフは3つ。不正解で1減り、0でゲームオーバー。ジャンルごとのベストスコアを保存

### 得点

| 聴いた長さ | 1秒 | 3秒 | 7秒 | 15秒 |
|---|---|---|---|---|
| 基礎点 | 300 | 200 | 120 | 60 |

連続正解でコンボ倍率(+0.1/コンボ、最大 ×2.0)が掛かる。

## 作成の背景(市場調査)

- 「聴いて当てる、外すと再生時間がアンロックされる」形式は [Heardle](https://techcrunch.com/2022/07/12/spotify-acquired-heardle-wordle-music-game/)(2022年にSpotifyが買収)がバイラル化させたメカニクスで、[Songless](https://songless.pro/) など後継ゲームが2026年現在も活発([trendhunter](https://www.trendhunter.com/trends/heardle))
- 国内では Apple Music の楽曲を使う「イントロ王」や1万曲超の「[うたドン!](https://www.utadon.com/)」などイントロクイズアプリが人気で、ジャンル別・年代別出題が定番([アプリブ](https://app-liv.jp/games/puzzles/1601/)、[Smartlog](https://smartlog.jp/182789))
- モバイル音楽ゲーム市場は成長中で、タップ中心の簡単操作がカジュアル層の主流([Verified Market Reports](https://www.verifiedmarketreports.com/product/mobile-music-rhythm-games-market/))

これらを踏まえ、「段階的アンロック(Heardle)× ジャンル別イントロクイズ(イントロ王/うたドン)× コンボスコアアタック」を1画面・タッチのみで遊べる形にした。サイト内の既存ゲーム(破壊アクション・タイミング収集・2人対戦)とジャンルは被らない。

## 技術構成

- 単一HTML(`index.html`)・依存ライブラリなし。フォントのみGoogle Fonts(Bungee / Zen Kaku Gothic New / Space Mono)
- **楽曲データ**: iTunes Search API(`itunes.apple.com/search`、`attribute=artistTerm` でジャンル別アーティストプールから取得)。fetch失敗時はJSONP(8秒タイムアウト付き)にフォールバック(KAITEN `itunes-music-player.html` と同方式)
- **ヒットチャート**: Apple RSS(`rss.marketingtools.apple.com` most-played 100。旧 `rss.applemarketingtools.com` は301移転済み)→ Lookup APIでプレビューURL補完。RSSがCORS不可の環境ではJ-POPプールへ自動フォールバック
- **レート制限対策**: iTunes Search APIは短時間の連続呼び出しで403や空200を返す。(1) 取得は3アーティスト×limit 50のバッチ単位とし、プールが足りた時点で打ち切り (2) 取得済みプールはメモリ+localStorageに12時間キャッシュし、「もう一度」やリトライではAPIを呼ばない (3) 全取得失敗時は期限切れキャッシュでもあれば使用 (4) 個々のアーティスト取得失敗はcatchして残りでプールを構築
- **出題プール**: previewUrl必須・カラオケ/Live/Remix等を除外・正規化した曲名+アーティストで重複排除。選択肢は同プールから曲名が被らない3曲を混ぜる
- **プレビュー再生**: 単一の `<audio>` 要素を使い回し、再生開始は必ずタップ起点(自動再生制限を回避)。ステージ上限秒数はrAFループ内で `currentTime` を監視して自動停止
- **効果音**: Web Audioでリアルタイム合成。iOS消音スイッチ対策(無音WAVループ+`AudioContext.resume()`待ち+`interrupted`復帰)は hakai と同方式
- プレビュー音源はクロスオリジンのためWeb Audio Analyserが使えず、レコード盤ビジュアライザの波形は演出(擬似EQ)
- ベストスコアは localStorage(`intro-hunter-v1`、try/catchフォールバック)
- メインループ(rAF)はスタート画面の時点から回り、参照する状態は全て宣言時に初期化

## 実施したテスト

1. **構文チェック**: インラインscriptを抽出して `node --check` → PASS
2. **ヘッドレスロジックテスト**: DOM/Audio/fetch/localStorageをスタブし、ジャンル選択→出題→未試聴ガード→再生→ステージ上限自動停止→もっと聴く→正解(スコア加算・コンボ・倍率)→不正解→ゲームオーバー→ベスト保存→リスタート→チャート経路まで一巡 → PASS(890点・3コンボ確認)
3. **実ブラウザE2E**: puppeteer-core + ローカルChrome、モバイルビューポート(390×844・`hasTouch`)+タップ操作で実APIを叩いて実プレイ。pageerrorゼロ(consoleのAPI失敗ログはフォールバック処理済みのため結果ベースで判定)、canvas描画ピクセル13万+、実プレビュー音源の再生→1秒自動停止、正解発表→次の問題への遷移、ヒットチャート出題を確認 → PASS。実際にAPIが403を返している最中の実行でもプール構築〜プレイまで成功
4. **レート制限シナリオE2E**: リクエストインターセプトで iTunes API/RSS を全断し、(a) キャッシュなし→ハングせずエラー画面表示 (b) 期限切れlocalStorageキャッシュあり→そのまま出題開始、を確認 → PASS

### 開発で得た教訓

- **スクリーンショットの目視確認が表示バグを拾った**: 初回正解なのにコンボ倍率が「×1.1」と表示されるバグ(加点計算はコンボ加算前、表示用の倍率計算が加算後に行われていた)は、ロジックテストでもE2Eのアサーションでも検出できず、リザルト画面のスクリーンショット目視で発見した
- **iTunes Search APIのレート制限は実害がある**: リリース直後に実機で「曲を取得できませんでした」が発生。検証したところ短時間に数十回の呼び出しで403/空200が返り始めた(共有IPの携帯回線では初回から起こり得る)。呼び出し回数の最小化+プールキャッシュ+期限切れキャッシュフォールバックで解消。レスポンスは「空ボディの200」のこともあるため `res.ok` チェックだけでは不十分で、空ボディ検出とJSONPタイムアウトも必要
- RSSチャートフィードはオリジンによってCORS不可(localhostでは失敗)。「失敗したらSearch APIのアーティストプールに自動フォールバック」を最初から入れておくと、E2Eも本番も壊れない。また旧ドメイン `rss.applemarketingtools.com` は301移転しており、fetchのリダイレクト+CORSで詰まるため新ドメインを直接叩く
- E2Eのconsoleエラー判定は、メッセージ文字列ではなく `m.location().url` でフィルタしないとfavicon 404とRSSのCORSエラーを区別できない
