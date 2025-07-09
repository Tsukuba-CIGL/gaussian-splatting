# 3DGSインストールにおける手順と注意点
## 1. 環境
・Windows11  
・Conda  
・C++ Compiler for PyTorch extensions (Visual Studio 2019 for Windowsだとうまくいく)  
・CUDA SDK 11 for PyTorch extensions, install after Visual Studio (Cuda11.8推奨)  
・C++ Compiler and CUDA SDK must be compatible  
・CMake（4.1.0を使用）  
・7zip(only Windows)  

## 2. Optimizerのインストールとビルド
### リポジトリのクローン
サブモジュールまで含むようにクローンするため，末尾に--recursiveをつけるのを忘れないように
```shell
# SSH
git clone git@github.com:graphdeco-inria/gaussian-splatting.git --recursive
```
or
```shell
# HTTPS
git clone https://github.com/graphdeco-inria/gaussian-splatting --recursive
```

### セットアップ
1. Conda  
仮想環境を構築するためにcondaをインストールする．以降はcondaを用いた環境構築方法について説明する．

2. Visual Studio 2019のインストール  
下記のページからVisual Studio 2019をインストール．
https://learn.microsoft.com/ja-jp/visualstudio/releases/2019/history
**※Visual Studio 2022だと後の環境構築でエラーが出ることが多い 2019を使うべき**
インストールできたらVisual Studio Installerを立ち上げ，「C++によるデスクトップ開発」を有効にする．
オプションはいらないはず．
ここまで正常にできたらanaconda promptを立ち上げclコマンドが機能するか確認する．
そのままではclを認識しないことがあるので，その場合は
"C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise\VC\Auxiliary\Build\vcvars64.bat"
のバッチファイルを実行し，環境変数を設定する．カレントファイルは何処でも可．
```shell
# clコマンドの確認
(base) C:\Users\_s2110838\Research\3DGS\gaussian-splatting\SIBR_viewers\install\bin>cl
'cl' は、内部コマンドまたは外部コマンド、
操作可能なプログラムまたはバッチ ファイルとして認識されていません。

(base) C:\Users\_s2110838\Research\3DGS\gaussian-splatting\SIBR_viewers\install\bin>"C:\Program Files (x86)\Microsoft Visual Studio\2019\Enterprise\VC\Auxiliary\Build\vcvars64.bat"
**********************************************************************
** Visual Studio 2019 Developer Command Prompt v16.11.48
** Copyright (c) 2021 Microsoft Corporation
**********************************************************************
[vcvarsall.bat] Environment initialized for: 'x64'

(base) C:\Users\_s2110838\Research\3DGS\gaussian-splatting\SIBR_viewers\install\bin>cl
Microsoft(R) C/C++ Optimizing Compiler Version 19.29.30159 for x64
Copyright (C) Microsoft Corporation.  All rights reserved.

使い方: cl [ オプション... ] ファイル名... [ /link リンク オプション... ]
```

4. Cudaのインストール  
inriaの環境に合わせるため**Cuda11.8**をインストール．パスが通っているかどうか・nvccコマンドが使えるか確認する．カレントファイルは何処でも可．
```shell
(base) C:\Users\_s2110838\Research\3DGS\gaussian-splatting\SIBR_viewers\install\bin>nvcc --version
nvcc: NVIDIA (R) Cuda compiler driver
Copyright (c) 2005-2022 NVIDIA Corporation
Built on Wed_Sep_21_10:41:10_Pacific_Daylight_Time_2022
Cuda compilation tools, release 11.8, V11.8.89
Build cuda_11.8.r11.8/compiler.31833905_0
```

5. environment.yamlの編集  
後の仮想環境構築中のエラーを防ぐため，クローンしてきたgaussian-splattingのフォルダ内にあるeivironment.ymlの中身を下記の通り変更する．
```shell
name: gaussian_splatting
channels:
  - pytorch
  - nvidia
  - conda-forge
  - defaults
dependencies:
  - numpy=1.26.4 
  - cudatoolkit=11.8
  - plyfile
  - python=3.11.7
  - pip=22.3.1
  - pytorch=2.2.2
  - pytorch-cuda=11.8
  - torchaudio=2.2.2
  - torchvision=0.17.2
  - tqdm
  - pip:
    - submodules/diff-gaussian-rasterization
    - submodules/simple-knn
    - submodules/fused-ssim
    - opencv-python
    - joblib
```
主な変更点としてはchannels中にnvidiaを追加，dependancies中にnumpy=1.26.4を追加，cudatoolkit=11.6→11.8に変更それに合わせた依存環境のバージョンも変更．

6. ローカルのセットアップ
次のコマンドをanaconda promprtで実行する．カレントディレクトリはgaussian-splatting
```shell
cd gaussian-splatting
SET DISTUTILS_USE_SDK=1 # Windows only
conda env create --file environment.yml
```

以上でoprimizerを実行するローカルの環境が構築される．


## ビューワのビルド
次を実行する．カレントディレクトリはgaussian-splatting内のSIBR_viewers
```shell
cd SIBR_viewers
cmake -Bbuild .
cmake --build build --target install --config RelWithDebInfo
```
Cmakeのバージョンによりエラーが出ることがある．その場合は下記のようなパスにあるCMakeList.txtをメモ帳で編集する．
"C:\Users\_s2110838\Research\3DGS\gaussian-splatting\SIBR_viewers\extlibs\xatlas\xatlas\CMakeLists.txt"

```shell
cmake_minimum_required(VERSION 3.1)
```
を
```shell
cmake_minimum_required(VERSION 3.5...3.27)
```
に変更した．(自分で使った環境はCMake4.1.0であった)

また，cooda_toolsetが見つからないというエラーが出ることがある．これについては次を参考に
https://github.com/NVlabs/tiny-cuda-nn/issues/164 
C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v11.7\extras\visual_studio_integration\MSBuildExtensions から4つのファイルをすべてコピーし、C:\Program Files (x86)\Microsoft Visual　Studio\2019\Enterprise\MSBuild\Microsoft\VC\v160\BuildCustomizations に貼り付ける．パスは適宜読み替える．

さらに，cudart64_12.dllが見つからないというエラーが出る場合がある．これについては次を参考に
https://github.com/graphdeco-inria/gaussian-splatting/issues/1136
https://www.dllme.com/dll/files/cudart64_12
上記から必要なファイルをダウンロードし，
```shell
./<SIBR install dir>/bin/CUDART64_12.DLL
```
に置く．  
これでビルドが通るはず．

# 参考記事
3D Gaussian Splatting (3DGS) の使い方 生成方法 – Windows環境構築  
https://lilea.net/lab/how-to-setup-3d-gaussian-splatting/  
3D Gaussian Splattingをやってみる(Windows11での環境構築)  
https://note.com/thinkandcraft/n/n62f89885a550  


   
