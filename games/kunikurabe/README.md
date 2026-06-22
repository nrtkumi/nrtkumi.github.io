# くにくらべ — 万国 数くらべ 勝ち残りクイズ

2つの国を見て、**人口・面積・GDP がより多いのはどっち？** を当て続ける、勝ち残り式のハイ&ロー(Higher/Lower)クイズ。数値は **世界銀行(World Bank Open Data)の実データ**、国旗は **flagcdn** の本物。連勝が続く限りエンドレス、1回でも外したら終了 — 「もう1回」が止まらない片手クイズです。

- **プレイ:** https://nrtkumi.github.io/games/kunikurabe/
- **対応:** スマートフォン(タッチ)推奨 / PC(マウス・↑↓キー)も可。縦画面・1人用・単一HTML。

---

## 遊び方

1. タイトルでモード(**人口 / 面積 / GDP**)を選ぶ。
2. 画面**上の国**を基準に、**下の国の数値が「多い ▲」か「少ない ▼」か**を下のボタンでタップ。
3. 正解すると下の国が新チャンピオンになり、次の挑戦国が登場。**連勝が続く限り無限**に続きます。
4. 外したら終了。**最高連勝**はモードごとに記録されます(同点はどちらの回答でも正解扱い)。
5. 5連勝ごとに「○連勝！」の演出。数値は世界銀行の最新値をカウントアップで公開します。

### モード

| モード | 指標 | World Bank 指標コード |
|---|---|---|
| 👥 人口くらべ | 総人口 | `SP.POP.TOTL` |
| 🗺️ 面積くらべ | 国土面積(km²) | `AG.LND.TOTL.K2` |
| 💰 GDPくらべ | 名目GDP(現在US$) | `NY.GDP.MKTP.CD` |

各指標は `mrnev=1`(最新の非欠損値)で取得。約195の国・地域を収録(集計地域・人口7万人未満の微小国は除外)。

---

## 作成の背景

「ブラウザから使える無料の公開API」を主役にしたゲームとして企画しました。

- **メカニクス選定:** 2つの選択肢の大小を当て続ける **Higher/Lower(勝ち残り)** は、外すまで終われない単純明快なループで強い中毒性を持ちます。心理学的には、勝敗が予測不能なほど **報酬予測誤差(ドーパミン)** が大きく、「あと1問」で **near-miss(惜しい)** が継続意欲を高めることが知られています(Schultz 1997 ほか / [可変報酬の心理](https://neurolaunch.com/variable-reward-psychology/)、[near-miss の神経科学](https://pmc.ncbi.nlm.nih.gov/articles/PMC3077261/))。ハイパーカジュアルの定石どおり、操作は1〜2種・即時フィードバックに絞っています([CHI'22: hyper-casual loop](https://dl.acm.org/doi/fullHtml/10.1145/3491101.3519837))。
- **データ源:** 信頼できる実数値が無料・キー不要・CORS対応で得られる **[World Bank Open Data API](https://data.worldbank.org/)** を採用。国旗は CORS 対応の画像CDN **[flagcdn.com](https://flagcdn.com/)** から取得(canvas汚染なし)。
- **片手・縦画面設計:** 重要操作(多い/少ない)を親指の届く画面下部に配置([thumb zone](https://www.smashingmagazine.com/2020/02/design-mobile-apps-one-hand-usage/))。

> 既存ゲームと API・メカニクスが被らないことを確認(itsunoe=シカゴ美術館、doappu=iNaturalist、chart-samurai=Frankfurter、pedia-battle=Wikimedia など)。Higher/Lower 形式・World Bank API はサイト初。

## 技術構成

- **単一HTML**(インラインCSS/JS、外部スクリプト/ビルド不要)。
- **データ取得:** 起動時に内蔵スナップショット(World Bank由来・約195カ国を埋め込み)で**即プレイ可能**にしつつ、裏で World Bank API の最新3指標を `fetch` して取得成功すれば次ランから差し替え。**通信失敗時はスナップショットで継続**(無言で壊れない)。
- **国旗:** `https://flagcdn.com/w320/{iso2}.png`。読み込み失敗時は ISO コード表示にフォールバック。
- **国名(日本語):** ビルド時に [umpirsky/country-list] の ja データを World Bank の国一覧へ突き合わせて埋め込み。
- **デザインコンセプト:** 「古い万国地図(アトラス)」— 褪せた紙・経緯線・真鍮色・明朝体(Zen Old Mincho)+数値は Space Mono。AI的な紫グラデ/グラスモーフィズムは不使用。
- **iOS対応:** Web Audio はマナーモードで無音化するため、初回タップで無音WAVループ + `AudioContext.resume()` 待ち + `onstatechange` 復帰。効果音は Web Audio 合成。ダブルタップズーム無効化3点セット、Pointer Events 統一(`click`非依存)、セーフエリア対応、PCワイド対策(縦カラム中央寄せ)。
- **永続化:** localStorage にモード別の最高連勝(try/catch)。

## 実施したテストと教訓

1. **構文チェック:** インラインJSを抽出し `node --check` → OK。
2. **ヘッドレスロジックテスト:** 埋め込みデータの整合(195カ国・iso2一意・pop/area/gdp正値・日本語名)、ハイ&ローの正誤判定ルール(同点は両方正解)、桁区切りフォーマットを検証 → 15/15 PASS。
3. **実ブラウザE2E(puppeteer-core + Chrome, モバイルVP `hasTouch`):** 起動→モード選択→「多い」連打で5ラウンド進行。**勝敗が実データと一致(mismatch 0)**、連勝カウント増加、**国旗の実画像ロード(naturalWidth=320)**、**World Bank ライブ取得成功(「✓ 最新データを取得」表示)** を確認。`pageerror`/コンソールエラーは favicon 以外ゼロ。スクリーンショット目視OK。

**教訓:** REST Countries v3.1 は2026年時点で**廃止(deprecated)**され利用不可だった → 代替に World Bank API を採用。外部データ依存ゲームは **内蔵スナップショットを必ず同梱**し、ライブ取得は「成功したら上書き」に留めると、API障害・オフラインでも遊べて起動も速い。flagcdn は CORS 対応なので canvas を汚さずスクショ(getImageData)も通る。
