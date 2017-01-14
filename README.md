#Cesium版歴史的農業環境閲覧システム

![イメージ](/img/6.jpg)


##動作環境

- Cesium 1.19
- Node.js 4.1.1
- npm 2.14.4
- Google Chrome


##0.開発環境の構築
Cesium版歴史的農業環境閲覧システムを使った開発に便利なアプリケーションを紹介します。

###①Gitのインソール
GitHubからソースコードをダウンロードします。[サルでもわかるGit入門](http://www.backlog.jp/git-guide/intro/intro2_1.html) に使用しているOSごとのGitのインストール方法が説明されてます。


###②Node.jsのインストール
###nodebrewのインストール
Node.jsのバージョン管理をするために、nodebrewをインストールします。詳しくは、[こちら](http://qiita.com/sinmetal/items/154e81823f386279b33c)を参照してください。Windowsユーザーの人は[nodeist](http://qiita.com/Kackey/items/b41b11bcf1c0b0d76149)をインストールします。

```bash
$ curl -L git.io/nodebrew | perl - setup
$ export PATH=$HOME/.nodebrew/current/bin:$PATH
$ source ~/.bash_profile
```

###Node.jsのインストール
Node.jsを使うときには安定バージョン（Stable）を利用するといいです。
[https://nodejs.org/en/blog/](https://nodejs.org/en/blog/)

```bash
$ nodebrew ls-remote
$ nodebrew install-binary v4.1.1

### install-binary しただけでは使えません

$ nodebrew ls
	v0.10.38
	v4.1.1
	v5.7.1
	
	current: v0.10.38

$ nodebrew use v4.1.1
	use v4.1.1

$ node -v
	v4.1.1

$ npm -v
	2.14.4
```




##1.Cesiumのローカル環境での立ち上げ
###①GitHubからソースコードのClone

```bash
$ git clone https://github.com/HabsNiaes/HabsCesium.git
$ cd HabsCesium
```

####各ファイルの構成
GitHubからCloneしたファイルの構成は下記の図の通りです。

```bash
cesium-project  
 ├─ node_modules  
 │   └ cesium  
 │     └ Build  
 │       ├ Cesium             <= 圧縮済 本番用  
 │       │ └ Cesium.js  
 │       └ CesiumUnminified   <= 未圧縮 開発用  
 │         └ Cesium.js
 ├─ kml                       <= KMLファイルを格納
 ├─ JavaScript                <= 地理院標高タイルのライブラリ
 ├─ img                       
 ├─ index.html                
 ├─ style.css                 <= 各要素のスタイルを設定
 └─ main.js                   <= Cesiumの各機能を読み込む
```

- `index.html`、`main.js`、`style.css`：開発はこのファイルに記述していきます。
- `node_modules`：Cesiumの元ソースコードが入ってます。
- `kml`：視図、断面図などの図郭外図のKMLデータが入ってます。
- `JavaScript`：[kochizufanさん](https://github.com/kochizufan)の[地理院標高タイルのライブラリ](https://github.com/tilemapjp/Cesium-JapanGSI)が入ってます。


###②ローカルサーバー立ち上げ
[Same-Origin Policy](https://ja.wikipedia.org/wiki/%E5%90%8C%E4%B8%80%E7%94%9F%E6%88%90%E5%85%83%E3%83%9D%E3%83%AA%E3%82%B7%E3%83%BC)のため、Cesiumの開発のためにローカルにサーバーを立ち上げます。

```bash
$ sudo npm i st -g
	st@1.1.0 /usr/local/lib/node_modules/st
	├── graceful-fs@4.1.3
	├── negotiator@0.6.0
	├── mime@1.3.4
	├── fd@0.0.2
	├── async-cache@1.0.0 (lru-cache@2.3.1)
	└── bl@1.0.3 (readable-stream@2.0.5)
$ st -nc  #-ncはキャッシュの無効化
	listening at http://127.0.0.1:1337
```

###③ブラウザでの立ち上げ
ブラウザを立ち上げて、[http://127.0.0.1:1337/index.html](http://127.0.0.1:1337/index.html) をURLに入力すると、Cesiumが立ち上がる。

![/img/0.png](/img/7.png)


##2.各種Cesiumの機能
###①デフォルトのウィジェット設定
デフォルトの状態ではいろいろなウィジェット設定されています。そのため、必要ないウィジェットをオフにします。

```javascript
var viewer = new Cesium.Viewer("cesiumContainer", {
  baseLayerPicker : false,  //デフォルトのレイヤ切り替えウィジェットをオフにする
  timeline : false,         //デフォルトのタイムラインウィジェットをオフにする
  animation : false         //デフォルトのアニメーションウィジェットをオフにする
});
```

###②地理院標高タイルの読み込み

`index.html`に`JapanGSITerrainProvider.js`の読み込みのコードを記述します。

```html
<script src="JavaScript/JapanGSITerrainProvider.js"></script>
```

`JapanGSITerrainProvider.js`の読み込みを設定したうえで、`main.js`に下記のコードを記述します。`heightPower: `で高さの倍率を設定できます。

```javascript
var terrainProvider = new Cesium.JapanGSITerrainProvider({
  heightPower: 2.5 //高さの倍率の設定
});
viewer.terrainProvider = terrainProvider;

```

###②初期の視点設定
関東周辺部に初期の視点になるよう`main.js`に以下を記述します。

```javascript
viewer.camera.setView({
  destination: Cesium.Cartesian3.fromDegrees(139.6721684, 35.7052493, 50000),
  orientation: {
    heading: 0, // 水平方向の回転
    pitch: -Cesium.Math.PI_OVER_TWO, // 垂直方向の回転 上を見上げたり下を見下ろしたり
    roll: 0
  }
});
```

###③KMLの読み込み
KMLファイルを読み込むよう`main.js`に以下を記述します。

```javascript
viewer.dataSources.add(Cesium.KmlDataSource.load('kml/shizu.kmz'),{
         camera: viewer.scene.camera,
         canvas: viewer.scene.canvas
});

```


###④各種レイヤの読み込み
各種レイヤ読み込み方法にはプルダウンメニューとチェックボックスの2つの設定しました。`main.js`に下記の要素を記入することでレイヤの追加、削除ができます。

- `addBaseLayerOption();`：プルダウンメニューでのレイヤ切り替え
- `addAdditionalLayerOption();`：チェックボタンでのレイヤ切り替え


```javascript
var baseLayers = viewModel.baseLayers;
function setupLayers() {
    // Create all the base layers that this example will support.
    // These base layers aren't really special.  It's possible to have multiple of them
    // enabled at once, just like the other layers, but it doesn't make much sense because
    // all of these layers cover the entire globe and are opaque.
    addBaseLayerOption(
            'Bing Maps Aerial',
            undefined); // the current base layer
    // Create the additional layers
    addBaseLayerOption(
      '地理院地図',
            new Cesium.OpenStreetMapImageryProvider({
                url: 'http://cyberjapandata.gsi.go.jp/xyz/std/',
                credit: new Cesium.Credit('地理院タイル', '', 'http://maps.gsi.go.jp/development/ichiran.html')
            }));
    // Create the additional layers
    addBaseLayerOption(
      '電子国土基本図（オルソ画像）',
      new Cesium.OpenStreetMapImageryProvider({
        url: 'http://cyberjapandata.gsi.go.jp/xyz/ort/',
        credit: new Cesium.Credit('地理院タイル', '', 'http://maps.gsi.go.jp/development/ichiran.html')
      }));
    addAdditionalLayerOption(
      '迅速測図',
      new Cesium.OpenStreetMapImageryProvider({
        url: 'http://www.finds.jp/ws/tmc/1.0.0/Kanto_Rapid-900913-L/',
        "ext": "jpg",
        "zmin": 0,
        "zmax": 18,
        credit : 'CC BY 国立研究開発法人農業環境技術研究所 歴史的農業環境閲覧システム',
      }));
}
```
