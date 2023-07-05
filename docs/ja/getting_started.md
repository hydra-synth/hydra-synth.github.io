## 使い方

https://hydra.ojack.xyz を開いて、

* CTRL-Enter: 現在の行を実行
* CTRL-Shift-Enter: スクリーン上のすべてのコードを実行
* ALT-Enter: （空行にはさまれた）ブロックを実行
* CTRL-Shift-H: コードを非表示・表示
* CTRL-Shift-F: [Prettier](https://prettier.io/) によりフォーマット
* CTRL-Shift-S: スクリーンショットを取って画像ファイルとしてダウンロード
* CTRL-Shift-G: twitter の [@hydra_patterns](https://twitter.com/hydra_patterns) に保存（現在は動作していません）

コードはブラウザ上のエディタからでもコンソールからでも実行できます。

使い始める前に、 [@hydra_patterns](https://twitter.com/hydra_patterns) からコミュニティ・メンバーのシェアしたパターンをご覧いただければイメージがつかめると思います。

#### 基本の関数
オシレーターを周波数、シンク、 RGB オフセットのパラメータでレンダー：
```javascript
osc(20, 0.1, 0.8).out()
```

オシレーターを 0.8 ラジアンだけ回転：
```javascript
osc(20, 0.1, 0.8).rotate(0.8).out()
```
上の関数の出力をピクセル化：
```javascript
osc(20, 0.1, 0.8).rotate(0.8).pixelate(20, 30).out()
```
ウェブカメラの画像を表示：
```javascript
s0.initCam() // ソース・バッファ s0 をウェブカメラに初期化
src(s0).out() // ソース・バッファ s0 をレンダー
```
カメラが複数接続されている場合は、インデックスを用いてカメラを指定できます：
```javascript
s0.initCam(1) // ソース・バッファ s0 をウェブカメラ(1)に初期化
```
ウェブカメラ・カレイドスコープ（万華鏡）：
```javascript
s0.initCam() // ソース・バッファ s0 をウェブカメラに初期化
src(s0).kaleid(4).out() // ウェブカメラをカレイドとして表示
```

複数のソースをかけ合わせることもできます：
```javascript
osc(10)
  .rotate(0.5)
  .diff(osc(200))
  .out()
```

デフォルトの環境では4つの出力バッファがあり、それぞれに別のビジュアルをレンダ―することができます。出力は変数 `o0`, `o1`, `o2`, `o3` によりアクセスできます。

バッファ `o1` にレンダ―するには：
```javascript
osc().out(o1)
render(o1) // o1 のコンテンツを表示
```
`out()` の引数として出力バッファが指定されていない場合、 `o0` に結果がレンダーされます。
全てのレンダー・バッファを表示するには：
```javascript
render()
```

出力バッファはそれからミキシングやコンポジションにより画面に表示するグラフィックを生成することができます。
```javascript
s0.initCam() // ソース・バッファ s0 をウェブカメラに初期化
src(s0).out(o0) // o0 のソースをウェブカメラのバッファに指定
osc(10, 0.2, 0.8).diff(o0).out(o1) // グラデーションを o0 の内容とコンポジションにして、出力バッファ o1 につなぐ
render(o1) // o1 を画面に描画
```

コンポジション関数は `blend()`, `diff()`, `mult()`, `add()` 等で、四則演算により入力されたテクスチャとベース・テクスチャの色をかけ合わせることができます。フォトショップのブレンドモード（描画モード）のイメージです。

`modulate(texture, amount)` （`texture`: 入力テクスチャ、`amount`：度合) は入力テクスチャの赤、緑チャンネルに応じてベース・テクスチャの x, y 座標を変化させます。詳しくはこちら（英語）： https://lumen-app.com/guide/modulation/
```javascript
osc(21, 0).modulate(o1).out(o0)
osc(40).rotate(1.57).out(o1)
```

動画をソースとして用いる：
```javascript
s0.initVideo("https://media.giphy.com/media/AS9LIFttYzkc0/giphy.mp4")
src(s0).out()
```


画像をソースとして用いる：
```javascript
s0.initImage("https://upload.wikimedia.org/wikipedia/commons/2/25/Hydra-Foto.jpg")
src(s0).out()
```

#### 関数を引数として使う
Hydra の関数の引数は、固定した値ではなく関数として定義することもできます。例えば：
```javascript
osc(function(){return 100 * Math.sin(time * 0.1)}).out()
```
この例ではオシレータの周波数を時間の関数として定義しています（`time` はグローバル変数で、ページがロードされてから経過した秒数を表します）。 ES6 文法を使えばさらに短く書けます：
```javascript
osc(() => (100 * Math.sin(time * 0.1))).out()
```

## デスクトップのキャプチャ
ダイアログを表示してスクリーンやウインドウを入力テクスチャに指定することができます：
```javascript
s0.initScreen()
src(s0).out()
```

## Connecting to remote streams
Any hydra instance can use other instances/windows containing hydra as input sources, as long as they are connected to the internet and not blocked by a firewall. Hydra uses webrtc (real time webstreaming) under the hood to share video streams between open windows. The included module rtc-patch-bay manages connections between connected windows, and can also be used as a standalone module to convert any website into a source within hydra. (See standalone camera source below for example.)

To begin, open hydra simultaneously in two separate windows.
In one of the windows, set a name for the given patch-bay source:
```javascript
pb.setName("myGraphics")
```
The title of the window should change to the name entered in setName().

From the other window, initiate "myGraphics" as a source stream.
```javascript
s0.initStream("myGraphics")
```
render to screen:
```javascript
s0.initStream("myGraphics")
src(s0).out()
```
The connections sometimes take a few seconds to be established; open the browser console to see progress.
To list available sources, type the following in the console:
```javascript
pb.list()
```

## Using p5.js with hydra

```javascript
// Initialize a new p5 instance It is only necessary to call this once
p5 = new P5() // {width: window.innerWidth, height:window.innerHeight, mode: 'P2D'}

// draw a rectangle at point 300, 100
p5.rect(300, 100, 100, 100)

// Note that P5 runs in instance mode, so all functions need to start with the variable where P5 was initialized (in this case p5)
// reference for P5: https://P5js.org/reference/
// explanation of instance mode: https://github.com/processing/P5.js/wiki/Global-and-instance-mode

// When live coding, the "setup()" function of P5.js has basically no use; anything that you would have called in setup you can just call outside of any function.

p5.clear()

for(var i = 0; i < 100; i++){
  p5.fill(i*10, i%30, 255)
  p5.rect(i*20, 200, 10,200)
}

// To live code animations, you can redefine the draw function of P5 as follows:
// (a rectangle that follows the mouse)
p5.draw = () => {
  p5.fill(p5.mouseX/5, p5.mouseY/5, 255, 100)
  p5.rect(p5.mouseX, p5.mouseY, 30, 150)
}

// To use P5 as an input to hydra, simply use the canvas as a source:
s0.init({src: p5.canvas})

// Then render the canvas
src(s0).repeat().out()
```

## Loading external scripts
The `await loadScript()` function lets you load other packaged javascript libraries within the hydra editor. Any javascript code can run in the hydra editor.

Here is an example using Three.js from the web editor:
```javascript
await loadScript("https://threejs.org/build/three.js")

scene = new THREE.Scene()
camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000)

renderer = new THREE.WebGLRenderer()
renderer.setSize(width, height)
geometry = new THREE.BoxGeometry()
material = new THREE.MeshBasicMaterial({color: 0x00ff00})
cube = new THREE.Mesh(geometry, material);
scene.add(cube)
camera.position.z = 1.5

// 'update' is a reserved function that will be run every time the main hydra rendering context is updated
update = () => {
  cube.rotation.x += 0.01;
  cube.rotation.y += 0.01;
  renderer.render( scene, camera );
}

s0.init({ src: renderer.domElement })

src(s0).repeat().out()
```

And here is an example loading the Tone.js library:
```javascript
await loadScript("https://unpkg.com/tone")

synth = new Tone.Synth().toDestination();
synth.triggerAttackRelease("C4", "8n");
```
## Running locally
To run locally, you must have nodejs, yarn and npm installed. Install node and npm from: https://nodejs.org/en/

Install yarn from the command line
```
npm install --global yarn
```
open terminal and enter directory
```
cd hydra
```
install dependencies:
```
yarn install
```
run server
```
yarn serve
```
go to https://localhost:8000 in the browser


## To develop
Edit [frontend/public/index.html](frontend/public/index.html) to load 'bundle.js' rather than 'bundle.min.js'

Run development server
```
yarn dev
```



## Audio Responsiveness 
FFT functionality is available via an audio object accessed via "a". The editor uses https://github.com/meyda/meyda for audio analysis.
To show the fft bins,
```
a.show()
```
Set number of fft bins:
```
a.setBins(6)
```
Access the value of the leftmost (lowest frequency) bin:
```
a.fft[0]
```
Use the value to control a variable:
```
osc(10, 0, () => (a.fft[0]*4))
  .out()
```
It is possible to calibrate the responsiveness by changing the minimum and maximum value detected. (Represented by blur lines over the fft). To set minimum value detected:
```
a.setCutoff(4)
```

Setting the scale changes the range that is detected.
```
a.setScale(2)
```
The fft[<index>] will return a value between 0 and 1, where 0 represents the cutoff and 1 corresponds to the maximum.

You can set smoothing between audio level readings (values between 0 and 1). 0 corresponds to no smoothing (more jumpy, faster reaction time), while 1 means that the value will never change.
```
a.setSmooth(0.8)
```
To hide the audio waveform:
```
a.hide()
```
## MIDI (experimental)

MIDI controllers can work with Hydra via WebMIDI an example workflow is at [/docs/midi.md](https://github.com/ojack/hydra/blob/master/docs/midi.md) .

## API

There is an updated list of functions at [/docs/funcs.md](https://github.com/ojack/hydra/blob/master/docs/funcs.md).

As well as in the [source code for hydra-synth](https://github.com/ojack/hydra-synth/blob/master/src/glsl/glsl-functions.js).

#### CHANGELOG 
See [CHANGELOG.md](CHANGELOG.md) for recent changes.



 #### Libraries and tools used:
 * [Regl: functional webgl](http://regl.party/)
 * glitch.io: hosting for sandbox signalling server
 * codemirror: browser-based text editor
 * simple-peer

 ## Inspiration:
 * Space-Time Dynamics in Video Feedback (1984). [video](https://www.youtube.com/watch?v=B4Kn3djJMCE) and [paper](http://csc.ucdavis.edu/~cmg/papers/Crutchfield.PhysicaD1984.pdf) by Jim Crutchfield about using analog video feedback to model complex systems.
 * [Satellite Arts Project (1977) - Kit Galloway and Sherrie Rabinowitz](http://www.ecafe.com/getty/SA/)
 * [Sandin Image Processor](http://www.audiovisualizers.com/toolshak/vidsynth/sandin/sandin.htm)
 * [kynd - reactive buffers experiment](https://kynd.github.io/reactive_buffers_experiment/)

 #### Related projects:
 * [Lumen app (osx application)](https://lumen-app.com/)
 * [Vsynth (package for MaxMSP)](https://cycling74.com/forums/vsynth-package)
 * [VEDA (VJ system within atom)](https://veda.gl/)
 * [The Force](https://videodromm.com/The_Force/)
