# MediaPipeのビルド

## インストール

基本的には以下のページを参考に行う

<https://ai.google.dev/edge/mediapipe/framework/getting_started/install?hl=ja>

今回はWindowsへのインストールを行った。

mediapipeのバージョンはmediapipe 0.10.14を使用した。

0.10.21が最新だが、最新バージョンはシンボリックリンク生成が上手く行かないので敢えて下げている。

コンパイラはVisual Studio 2022 Communityを使用。元々入っていた奴を使用した。

### インストール時の注意点

- masterブランチではなくtag付きのバージョンを落とす
- MSYS2を入れた後、環境変数への追加はシステム環境変数に行う
  - wslを使用している人は`bash`の実行時にwslのbashが使われてしまうため
- bazelのバージョンを6.3.0に上げる(`.bazelversion`から変更)
  - bazelがVisualStudio C++コンパイラを認識してくれない問題が存在したため

### エラーと対処

#### C7555エラー

tensorflowのコミットでC++20以上のコードが書かれていたことが原因で発生。

具体的には、`{}`初期化しを使用した初期化のコードが使われていた。
今回は`.bazelrc`内のC++バージョンを`C++17`から`C++20`にすることで対処

#### error C2475: 'Hoge': 再定義。'constinit' 指定子が一致しません

C++20で`constinit`指定子が追加されたことによるエラー

どうやら`ABSL_CONST_INIT`というマクロが、`constinit`が存在する場合使用するマクロだったらしくC++20へ上げたことによってコードに配置されてしまった模様。

```c++:gpu_service.h
// ABSL_CONST_INIT extern const GraphService<GpuResources> kGpuService;
  extern const GraphService<GpuResources> kGpuService;
```

そしてconstinitは一度だけ初期化することが求められるので、chatgpt君によると以下のようにヘッダーとソースファイルで分割する際に宣言側でconstinitを使うと再定義になってしまうらしい。
なので以下のように宣言側ではなくソース側で`constinit`を使えばいいらしい。

```c++
// ヘッダファイル mediapipe.h
namespace mediapipe {
    extern const int kGpuService;  // 宣言
}

// ソースファイル mediapipe.cpp
namespace mediapipe {
    constinit int kGpuService = 1;  // 定義
}
```

複数ファイルで修正が必要。
Windowsの場合はC++17ベースで書かれているので、コメントアウトで良い場面も恐らく多い。

```c++:legacy_calculator_support.h
/* mediapipe/framework/legacy_calculator_support.h */

#ifndef __APPLE__
    // ABSL_CONST_INIT
#endif                                // !__APPLE__
    static thread_local C* current_;  // NOLINT
```

#### external/org_tensorflow/tensorflow/lite/kernels/stablehlo_reduce_window.cc(361): error C4430: 型指定子がありません - int と仮定しました。メモ: C++ は int を既定値としてサポートしていません

先人がいらっしゃった。 メソッドの前のコメントが原因っぽい..?

コメントとメソッドに空行追加で対処

<https://zenn.dev/link/comments/58d9b0af5ca280>

#### Calculator::Open() for node "handlandmarktrackingcpu__handlandmarkcpu__handlandmarkmodelloader__LocalFileContentsCalculator" failed: ; Can't find file: mediapipe/modules/hand_landmark/hand_landmark_full.tflite

githubで名前掛けて検索したら、普通にdocsに書いてあった。普通にダウンロードする。docs読んで無くてゴメン。

<https://github.com/google-ai-edge/mediapipe/blob/82d02ac03a0fbf6199ed0d234aaa2c191cdcd2b1/docs/solutions/models.md?plain=1#L72>

### 実行周りで躓いたこと

カメラの映像が映らない。

使ってないカメラ・映像デバイスを無効化 or デバイスを外す等で無理やり対応。恐らくカメラ0を使っている?

今回は荒業で対応したが、普通はindex指定できるようにすると思うので探せばオプションとかで指定できるのではと推測。
