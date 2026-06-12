# 修行 (SHUGYO)

体で挑む十二の瞬間修行。WarioWare系の高速マイクロゲーム集で、**お題ごとに別のブラウザAPIで遊ぶ**のが特徴。スマホを傾ける・振る・静止させる、マイクに息を吹きかける、画面をこする・連打する・複数の指で同時に押さえる——失敗3回で破門。テンポはクリアするほど速くなる。

- **プレイURL**: https://nrtkumi.github.io/games/shugyo/
- **対応環境**: スマホ(iOS Safari / Android Chrome)推奨・縦画面。PCはマウス+キーボード(Space=タップ、矢印=スワイプ/傾き)でも遊べる
- センサー・マイクはタイトル画面のトグルでOFFにでき、**非対応環境や許可拒否時は自動でタッチ版の修行に置き換わる**

## 遊び方

1. タイトルで「修行開始」をタップ(iOSは傾きセンサーとマイクの許可ダイアログが出る)
2. 朱色の円に表示されるお題(「つけ!」「斬れ!」…)を見て、制限時間内に体で応える
3. 成功で⛩+1、失敗で❤️-1。❤️が尽きると破門。クリア数に応じて称号(見習い→木人→下忍→…→仙人→神域)

## 十二の修行と使用API

| 修行 | お題 | 操作 | 使用API |
|---|---|---|---|
| 餅つき | つけ! | 連打 | Pointer Events |
| 居合 | 斬れ! | 合図までタップを我慢 | Pointer Events |
| 弓 | 射れ! | 長押し→赤帯で放す | Pointer Events |
| 風斬り | 払え! | 矢印の向きへスワイプ | Pointer Events |
| 鏡磨き | 磨け! | こすって曇りを除去 | Pointer Events(move軌跡) |
| 封印 | 封じろ! | 札を**複数の指で同時に**押さえる | マルチタッチ(maxTouchPoints) |
| 蛍 | 捕えろ! | 動く蛍をタップ | Pointer Events |
| 玉転がし | 転がせ! | **端末を傾けて**玉を穴へ | DeviceOrientation |
| 水盤 | 保て! | **端末を水平に**保つ | DeviceOrientation |
| 祓い | 振れ! | **端末をシェイク** | DeviceMotion |
| 座禅 | 動くな! | 端末を静止+画面に触れない | DeviceMotion |
| 吹き消し | 吹き消せ! | **マイクに息を吹く** | getUserMedia + AnalyserNode |

センサー系4種とマイクは、非対応・許可なしの場合それぞれ「なぞる」「押さえる」「往復スワイプ」「触れるな」「上スワイプで扇ぐ」のタッチ版に自動フォールバックする。

このほか裏方として **Vibration API**(成否のハプティクス、Android のみ)、**Screen Wake Lock API**(プレイ中の画面消灯防止)、**Fullscreen API**(⛶ボタン)、**Web Audio API**(全効果音をリアルタイム合成)、**localStorage**(最高記録)を使用。

## 作成の背景(市場調査)

2026年6月の調査より:

- ブラウザゲーム市場はハイパーカジュアル・パズルが圧倒的で、「数秒で理解できる直感的な操作」「インストール不要の即時性」が勝ち筋とされる([Outlook Respawn](https://respawn.outlookindia.com/gaming/gaming-news/hypercasual-and-puzzle-titles-dominate-99-of-web-browser-gaming)、[nerdbot](https://nerdbot.com/2026/04/24/why-casual-browser-games-are-making-a-comeback/)、[udonis](https://www.blog.udonis.co/mobile-marketing/mobile-games/casual-games))
- WarioWare系マイクロゲーム集はインディーで存在感が続くトレンド(MINDWAVEなど。[PC Gamer](https://www.pcgamer.com/games/this-indie-warioware-like-released-a-teaser-just-to-show-off-its-game-over-music-and-after-hearing-it-i-can-see-why/)、[itch.io warioware タグ](https://itch.io/games/tag-warioware))
- ブラウザのセンサー系API(DeviceOrientation/Generic Sensor)はモバイルの90%以上で利用可能になり、モーション操作ゲームへの活用が紹介されている([MDN Sensor APIs](https://developer.mozilla.org/en-US/docs/Web/API/Sensor_APIs)、[Chrome Developers](https://developer.chrome.com/docs/capabilities/web-apis/generic-sensor))

既存17作と被らない「マイクロゲーム集」枠で、依頼テーマ「ブラウザの様々なAPIを駆使」をゲームメカニクスそのものに落とし込んだ。

## 技術構成

- 依存ライブラリなしの単一HTML(canvas 2D)。フォントはサイト共通(Bungee + Zen Kaku Gothic New + Space Mono)
- 各修行は `{init, update, draw, onDown/onMove/onUp/onKeyDir}` を持つオブジェクトとして定義し、状態機械(title→ready→splash→play→result→over)が回す
- iOS音声対策: 初回タップで無音WAVを`<audio>`ループ再生(消音スイッチ回避)+ `AudioContext.resume()` のPromise完了待ち + `onstatechange` 自動復帰
- iOSのセンサー許可: `DeviceOrientationEvent/DeviceMotionEvent.requestPermission()` をスタートボタンのジェスチャ内で同期的に呼ぶ
- マイク入力は `getUserMedia` → `AnalyserNode` のRMSから環境ノイズ基準を引いた値で判定(エコーキャンセル・ノイズ抑制はOFF)
- 傾き系は修行開始時の姿勢をゼロ点としてキャリブレーション(持ち方の個人差を吸収)
- dpr変換は毎フレーム `ctx.setTransform` で再設定(長時間プレイでのコンテキストリセット対策)
- メインループはタイトル時点から回り、フレーム例外はループを殺さず記録+その修行を失敗扱いにして続行
- OGP画像はCodex CLI経由のgpt-image-2で生成(墨絵+朱色の道場キービジュアル、1536x1024→1200x630へクロップ)

## 実施したテスト

1. `node --check` による構文チェック
2. ヘッドレスロジックテスト(DOMスタブでループを回し、開始→全修行ランダム操作→ゲームオーバー→リトライ→センサー/キーボードイベント注入を確認)
3. puppeteer-core + ローカルChrome(モバイルビューポート・タッチ・ローカルHTTPサーバ経由)の実ブラウザE2E:
   - モンキーテストでpageerror/consoleエラーゼロ・ループ生存を確認
   - **12種の修行すべてを正攻法で攻略するストラテジーテスト**(14修行クリア)で各修行の勝利可能性を検証
   - canvas描画ピクセル・localStorage保存・スクリーンショット目視確認

### 開発で得た教訓

- フレーム例外をtry/catchで握りつぶすだけだと「playのまま無限に固まる」事故になる。**初回例外のログ+その修行を強制失敗**にしてゲームを前に進める設計が安全
- 「ループが生きているか」をcanvasピクセル差分で判定すると、静止画面(タイトル等)で偽陽性になる。**フレームカウンタをデバッグフックで公開**して判定するのが確実
- ランダム操作のE2Eではマイクロゲームは勝てずスコア検証ができない。ゲーム内部状態を読み取り専用フック(`window.__shugyo`)で公開し、**修行ごとの攻略ストラテジーを書く**と「全ゲームが勝利可能」まで自動検証できる
