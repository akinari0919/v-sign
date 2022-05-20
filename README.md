[![Image from Gyazo](https://i.gyazo.com/31ff6d41b383bf24f830cc70c761a632.png)](https://gyazo.com/31ff6d41b383bf24f830cc70c761a632)

### 2022/5/6リリース
https://www.peace-sign-competition.com/

ピースサインで開いた指の角度で競い合ってみるサービスです✌️

<br>

### __🔻実装済みの機能__
- 全ユーザー（ログイン不要）
  - カメラへ✌️サインをかざすだけで指の角度を測定
  - MAX値の画像をリアルタイムで更新
  - 画像データは保存・公開でき、角度に応じてランキング表示される
  - 保存時点のラインキングと角度結果をTwitterでつぶやける

<br>

# ピースサインの角度取得について✌️
本サービスの根幹部分ですが、MediaPipeという手の骨格ランドマークを識別する機械学習ライブラリを使用しました。
[![Image from Gyazo](https://i.gyazo.com/0e6e149b6209cbe6116372201880c256.png)](https://gyazo.com/0e6e149b6209cbe6116372201880c256)
以下は簡単な実装手順です。

|①ランドマークの取得|②角度算出式導入|③Vの表示|
|---|---|---|
|<img src="https://i.gyazo.com/0a694d4b2ed3fd8dcc46c9fb51905477.gif" width="500">|<img src="https://i.gyazo.com/7ebd7163525a44a2a23aae4b3ff911f4.png" width="400">|<img src="https://i.gyazo.com/54678b3e07099607e772ff7b8657a959.gif" width="500">|
|公式や開発記事を元に導入はスムーズでした。|∠ABC角度を求める計算式を使用。<br>B点については上記ランドマークの⑤と⑨の中間点を別途算出してます。<br>（実装コードは後ほど紹介）|Canvasを用いてランドマーク位置に沿って描写させました。<br>リアルタイム表示が良きです...!|

<br>

上記の時点で完成と思いきや所々に欠陥が散見されたためV表示の条件を整備しました。
以下は見られた欠陥の一例です。

|✌️サインのみに対応していない|左手に対応していない|裏手に対応していない|
|---|---|---|
|<img src="https://i.gyazo.com/4b787856e7d700f0fe83e76893073e78.gif" width="400">|<img src="https://i.gyazo.com/bbce9f1238ba828d511bb4b8687d10bf.gif" width="400">|<img src="https://i.gyazo.com/b69607ec8b72d7629498c70527d059d8.gif" width="400">|

<br>

条件を増やしすぎると今度はV表示の難易度が上がってしまうため程々設定で落ち着きました。

<br>

## 🔻工夫ポイント3つ🔻
### ①自動撮影でMAX角度を表示
ピースサインしながら撮影ボタンを押すのは器用さが必要で、セルフタイマーにして撮影するのも撮影の瞬間にV表示がされていなければ角度が結果表示されない等の懸念がありました。考え抜いた末、角度がMAXになったら撮影するようにしてリアムタイムで更新されるようにしました。
```js
export default {
  data: function() {
    return {
      item: {
        angle: 0 // <= MAX角度が格納される
      },
      check: {
        angle: 0 // <= リアルタイムで角度が格納される
      }
    }
  },

  methods: {
    // 省略（撮影やランドマーク取得etc.）

    findHands(results) {
      // 省略（V表示や角度算出etc.）

      // MAXデータをリアルタイムデータを上回った場合（バグ発生のため10°以上の条件追加）
      if (this.check.angle > this.item.angle && this.check.angle > 10) {
        // 省略（画像データや角度データを更新）
      }
    }
  }
}
```

<br>

### ②ページ遷移させず操作感向上
シンプルなTOPページ完結型のサービスにし、補足が必要な箇所はモーダル表示を駆使しました。

<br>

### ③ページネーションで表示データ取得を制限
ランキング登録が増えれば増えるほど、TOPページ表示と同時に大量の画像データの取得がされてパフォーマンスを落としてしまうため、Vuetifyのページネーションを活用し最大１０件までのデータ取得で済むようにしました。

<br>

## 🔻苦労ポイント3つ🔻
### ①∠ABC角度算出
はい、これで算出できます（どーん）
[![Image from Gyazo](https://i.gyazo.com/e824b3828378c63bff5261f535115d41.png)](https://gyazo.com/e824b3828378c63bff5261f535115d41)
幸い数学は嫌いではないので良かったですが、コードに落とし込む方法を模索しました。
こちらは泥臭いですが何とか実装できました。
```js
const ba0 = indexX - mcpX // A点(X軸) - B点(X軸)
const ba1 = indexY - mcpY // A点(Y軸) - B点(Y軸)
const bc0 = middleX - mcpX // C点(X軸) - B点(X軸)
const bc1 = middleY - mcpY // C点(Y軸) - B点(Y軸)
const babc = ba0 * bc0 + ba1 * bc1
const ban = (ba0 * ba0) + (ba1 * ba1)
const bcn = (bc0 * bc0) + (bc1 * bc1)
const radian = Math.acos(babc / (Math.sqrt(ban * bcn)))
const angle = radian * 180 / Math.PI
```

<br>

### ②Vue.js移行
公式ではJavaScriptまでの実装コードのみの公開だったため、別の開発記事を参考に実装しました。

<br>

### ③スマホ対応
こちらも公式で公開している実装コードでは独自のWEBカメラ実装となり、どうやらスマホに対応していないことが分かりました。
ごにょごにょと`navigator.mediaDevices.getUserMedia`を取り入れることで何とか対応させることができました。

<br>

## 🔹使用技術🔹
### __🔻バックエンド__
- Ruby(3.1.0) ※APIモード
- Ruby on Rails(7.0.2.3)
### __🔻Gem__
- rspec-rails
- factory_bot_rails
- rubocop
- pry-byebug
- awesome_print

### __🔻フロント__
- Vue.js(2.6.14)
- Vuetify
### __🔻ライブラリ__
- MediaPipe
### __🔻テスト__
- Rspec
### __🔻インフラ__
- Docker(20.10.11)
- PostgreSQL
- Heroku

<br>

## 🔹テーブル設計・ER図🔹
※ 2022/5/6時点ではSignsテーブルのみ使用
[![Image from Gyazo](https://i.gyazo.com/865ca9483d8eaca593e17da27b20758b.png)](https://gyazo.com/865ca9483d8eaca593e17da27b20758b)

### 🔻Signsテーブル
投稿データを格納するテーブル<br>
- ゲストユーザーでも投稿者名を残せるように`name`カラムを追加
- `type`カラムのenumを用いて✌️or🖖を判別
- `angle`カラムに取得した角度を格納
- `user_id`カラムはnull許容

### 🔻Usersテーブル
ログインユーザー用テーブル

### 🔻Commentsテーブル
投稿データにログインユーザーがコメントする為のテーブル


<br>

## 🔹画面遷移図(初期構想)🔹
[![Image from Gyazo](https://i.gyazo.com/6dd90e07ef3a10151b1c2dd11162dd62.png)](https://gyazo.com/6dd90e07ef3a10151b1c2dd11162dd62)
※別途管理画面で投稿&ユーザー管理画面を作成
