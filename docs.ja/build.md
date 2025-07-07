# llama.cpp をローカルでビルドする

このプロジェクトの主力製品は `llama` ライブラリです。C スタイルのインターフェースは [include/llama.h](../include/llama.h) にあります。

このプロジェクトには、`llama` ライブラリを使用した多くのサンプルプログラムとツールも含まれています。サンプルは、シンプルで最小限のコードスニペットから、OpenAI 互換の HTTP サーバーなどの高度なサブプロジェクトまで多岐にわたります。

**コードを取得するには:**

```bash
git clone https://github.com/ggml-org/llama.cpp
cd llama.cpp
```

次のセクションでは、さまざまなバックエンドとオプションを使用してビルドする方法について説明します。

## CPU Build

`CMake` を使用して llama.cpp をビルドします。

```bash
cmake -B build
cmake --build build --config Release
```

**注**:

- コンパイルを高速化するには、`-j` 引数を追加して複数のジョブを並列実行するか、Ninja などの自動生成ツールを使用してください。例えば、`cmake --build build --config Release -j 8` は 8 つのジョブを並列実行します。
- 繰り返しコンパイルを高速化するには、[ccache](https://ccache.dev/) をインストールしてください。
- デバッグビルドの場合、以下の 2 つのケースがあります。

    1. 単一構成ジェネレーター (例: default = `Unix Makefiles`。`--config` フラグは無視されることに注意してください):

       ```bash
       cmake -B build -DCMAKE_BUILD_TYPE=Debug
       cmake --build build
       ```

    2. マルチ構成ジェネレーター (`-G` パラメータを Visual Studio、XCode などに設定):

       ```bash
       cmake -B build -G "Xcode"
       cmake --build build --config Debug
       ```

    詳細とサポートされているジェネレーターのリストについては、[CMake のドキュメント](https://cmake.org/cmake/help/latest/manual/cmake-generators.7.html) を参照してください。
- 静的ビルドの場合は、`-DBUILD_SHARED_LIBS=OFF` を追加します。
  ```
  cmake -B build -DBUILD_SHARED_LIBS=OFF
  cmake --build build --config Release
  ```

- MSVC または clang をコンパイラとして Windows (x86、x64、arm64) 向けにビルドする場合:
    - Visual Studio 2022 をインストールします (例: [Community Edition](https://visualstudio.microsoft.com/vs/community/))。インストーラーで、少なくとも以下のオプションを選択してください (CMake などの必要な追加ツールも自動的にインストールされます):
    - タブ ワークロード: C++ を使用したデスクトップ開発
    - タブ コンポーネント (検索で素早く選択): C++-_CMake_ ツール (Windows 用)、_Git_ (Windows 用)、C++-_Clang_ コンパイラ (Windows 用)、LLVM ツールセット (clang) 用の MS-Build サポート
    - Git、ビルド、テストには、VS2022 の開発者コマンドプロンプト / PowerShell を必ず使用してください。
    - ARM 版 Windows (arm64、WoA) の場合は、以下のコマンドでビルドします:
    ```bash
    cmake --preset arm64-windows-llvm-release -D GGML_OPENMP=OFF
    cmake --build build-arm64-windows-llvm-release
    ```
    arm64 向けのビルドは、build-arm64-windows-MSVC プリセットを指定した MSVC コンパイラ、または標準の CMake ビルド手順でも実行できます。ただし、MSVC コンパイラは、例えばアクセラレーションされた Q4_0_N_M CPU カーネルで使用されるインライン ARM アセンブリコードをサポートしていないことに注意してください。

    ninja ジェネレータと clang コンパイラをデフォルトとしてビルドする場合：
      -set path:set LIB=C:\Program Files (x86)\Windows Kits\10\Lib\10.0.22621.0\um\x64;C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.41.34120\lib\x64\uwp;C:\Program Files (x86)\Windows Kits\10\Lib\10.0.22621.0\ucrt\x64
      ```bash
      cmake --preset x64-windows-llvm-release
      cmake --build build-x64-windows-llvm-release
      ```
- curlの使用はデフォルトで有効になっていますが、`-DLLAMA_CURL=OFF`で無効にできます。無効にしない場合は、libcurlの開発ライブラリをインストールする必要があります。

## BLAS ビルド

BLAS サポート付きでプログラムをビルドすると、バッチサイズが 32 より大きい場合（デフォルトは 512）にプロンプ​​ト処理のパフォーマンスが若干向上する可能性があります。BLAS を使用しても生成パフォーマンスには影響しません。現在、ビルドして使用できる BLAS 実装が複数あります。

### Accelerate Framework

これはMac PCでのみ利用可能で、デフォルトで有効になっています。通常の手順でビルドできます。

### OpenBLAS

CPUのみを使用したBLASアクセラレーションを提供します。お使いのマシンにOpenBLASがインストールされていることを確認してください。

- Linuxで`CMake`を使用する場合：

    ```bash
    cmake -B build -DGGML_BLAS=ON -DGGML_BLAS_VENDOR=OpenBLAS
    cmake --build build --config Release
    ```

### BLIS

詳細については、[BLIS.md](./backend/BLIS.md) を参照してください。

### Intel oneMKL

oneAPI コンパイラーを使用してビルドすると、avx512 および avx512_vnni をサポートしていない Intel プロセッサーでも avx_vnni 命令セットが利用できるようになります。このビルド構成は **Intel GPU をサポートしていません** のでご注意ください。Intel GPU のサポートについては、[llama.cpp for SYCL](./backend/SYCL.md) を参照してください。

- oneAPI の手動インストールを使用する場合:
  デフォルトでは、`GGML_BLAS_VENDOR` は `Generic` に設定されています。そのため、Intel 環境スクリプトを既にソースコードとして読み込み、cmake で `-DGGML_BLAS=ON` を指定している場合は、Blas の mkl バージョンが自動的に選択されます。それ以外の場合は、oneAPI をインストールし、以下の手順に従ってください。
    ```bash
    source /opt/intel/oneapi/setvars.sh # You can skip this step if  in oneapi-basekit docker image, only required for manual installation
    cmake -B build -DGGML_BLAS=ON -DGGML_BLAS_VENDOR=Intel10_64lp -DCMAKE_C_COMPILER=icx -DCMAKE_CXX_COMPILER=icpx -DGGML_NATIVE=ON
    cmake --build build --config Release
    ```

- oneAPI Dockerイメージの使用：
  環境変数をsourceしてoneAPIを手動でインストールしたくない場合は、Intel Dockerコンテナ[oneAPI-basekit](https://hub.docker.com/r/intel/oneapi-basekit)を使用してコードをビルドすることもできます。その後、上記のコマンドを使用できます。

詳細については、[Intel® CPU上でのLLaMA2の最適化と実行](https://www.intel.com/content/www/us/en/content-details/791610/optimizing-and-running-llama2-on-intel-cpu.html)をご覧ください。

### その他のBLASライブラリ

`GGML_BLAS_VENDOR`オプションを設定することで、その他のBLASライブラリも使用できます。サポートされているベンダーの一覧については、[CMakeドキュメント](https://cmake.org/cmake/help/latest/module/FindBLAS.html#blas-lapack-vendors)をご覧ください。

## Metal ビルド

macOS では、Metal がデフォルトで有効になっています。Metal を使用すると、計算は GPU 上で実行されます。
コンパイル時に Metal ビルドを無効にするには、cmake オプション `-DGGML_METAL=OFF` を使用します。

Metal サポート付きでビルドした場合は、コマンドライン引数 `--n-gpu-layers 0` で GPU 推論を明示的に無効にできます。

## SYCL

SYCLは、様々なハードウェアアクセラレータにおけるプログラミングの生産性を向上させる高水準プログラミングモデルです。

SYCLベースのllama.cppは、**Intel GPU**（Data Center Maxシリーズ、Flexシリーズ、Arcシリーズ、内蔵GPU、iGPU）をサポートするために使用されます。

詳細については、[llama.cpp for SYCL](./backend/SYCL.md)を参照してください。

## CUDA

NVIDIA GPUを使用したGPUアクセラレーションを提供します。[CUDAツールキット](https://developer.nvidia.com/cuda-toolkit)がインストールされていることを確認してください。

#### NVIDIA から直接ダウンロードしてください
公式ダウンロードは、[NVIDIA 開発者サイト](https://developer.nvidia.com/cuda-downloads) から入手できます。

#### Fedora Toolbox コンテナ内でコンパイルおよび実行
Fedora [ツールボックス コンテナ](https://containertoolbx.org/) 内で CUDA ツールキットをセットアップするための [ガイド](./backend/CUDA-FEDORA.md) もご用意しています。

**推奨対象:**
- [Silverblue](https://fedoraproject.org/atomic-desktops/silverblue/) や [Kinoite](https://fedoraproject.org/atomic-desktops/kinoite/) などの [Atomic Desktops for Fedora](https://fedoraproject.org/atomic-desktops/) のユーザーには***必須***です。
  - (これらのシステムではサポートされている CUDA パッケージはありません)
- [サポートされている Nvidia CUDA リリース プラットフォーム](https://developer.nvidia.com/cuda-downloads) 以外のホストを使用しているユーザーには***必須***です。
  - (例えば、ホストOSとして[Fedora 42 Beta](https://fedoramagazine.org/announce-fedora-linux-42-beta/)を使用している場合)
- ***便利*** [Fedora Workstation](https://fedoraproject.org/workstation/)または[Fedora KDE Plasma Desktop](https://fedoraproject.org/spins/kde)をご利用で、ホストシステムをクリーンな状態に保ちたい場合。
- *オプション* ツールボックスパッケージが利用可能です: [Arch Linux](https://archlinux.org/)、[Red Hat Enterprise Linux >= 8.5](https://www.redhat.com/en/technologies/linux-platforms/enterprise-linux)、または[Ubuntu](https://ubuntu.com/download)


### Compilation
```bash
cmake -B build -DGGML_CUDA=ON
cmake --build build --config Release
```

### コンピューティング能力仕様のオーバーライド

`nvcc` が GPU を検出できない場合、次のようなコンパイル警告が表示されることがあります。
 ```text
nvcc warning : Cannot find valid GPU for '-arch=native', default arch is used
```

ネイティブ GPU 検出を無効にするには:

#### 1. NVIDIA デバイスの「コンピューティング能力」を確認します: ["CUDA: お使いの GPU コンピューティング > 能力"](https://developer.nvidia.com/cuda-gpus)。

```text
GeForce RTX 4090      8.9
GeForce RTX 3080 Ti   8.6
GeForce RTX 3070      8.6
```

#### 2. `CMAKE_CUDA_ARCHITECTURES` リストに、各異なる `Compute Capability` を手動でリストします。

```bash
cmake -B build -DGGML_CUDA=ON -DCMAKE_CUDA_ARCHITECTURES="86;89"
```

### ランタイムCUDA環境変数

実行時に[CUDA環境変数](https://docs.nvidia.com/cuda/cuda-c-programming-guide/index.html#env-vars)を設定できます。

```bash
# Use `CUDA_VISIBLE_DEVICES` to hide the first compute device.
CUDA_VISIBLE_DEVICES="-0" ./build/bin/llama-server --model /srv/models/llama.gguf
```

### ユニファイドメモリ

Linuxでは、環境変数「GGML_CUDA_ENABLE_UNIFIED_MEMORY=1」を使用してユニファイドメモリを有効化できます。これにより、GPU VRAMが枯渇した際にクラッシュするのではなく、システムRAMへのスワップが可能になります。Windowsでは、この設定はNVIDIAコントロールパネルの「システムメモリフォールバック」で利用できます。

### パフォーマンスチューニング

以下のコンパイルオプションもパフォーマンス調整に利用できます。

| Option                        | Legal values           | Default | Description                                                                                                                                                                                                                                                                             |
|-------------------------------|------------------------|---------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| GGML_CUDA_FORCE_MMQ | Boolean | false | int8 テンソル コア実装が利用できない場合でも、量子化モデルに対して FP16 cuBLAS ではなくカスタム行列乗算カーネルの使用を強制します (V100、CDNA、RDNA3+ に影響します)。MMQ カーネルは、int8 テンソル コアをサポートする GPU ではデフォルトで有効になっています。MMQ 強制を有効にすると、大きなバッチ サイズの速度は低下しますが、VRAM の消費量は少なくなります。 |
| GGML_CUDA_FORCE_CUBLAS | Boolean | false | 量子化モデルに対して、カスタム行列乗算カーネルではなく FP16 cuBLAS の使用を強制します |
| GGML_CUDA_F16 | Boolean | false |有効にすると、CUDA 逆量子化 + mul mat vec カーネルと q4_1 および q5_1 行列乗算カーネルに半精度浮動小数点演算を使用します。比較的新しい GPU ではパフォーマンスが向上する可能性があります。 |
| GGML_CUDA_PEER_MAX_BATCH_SIZE | Positive integer | 128 | 複数の GPU 間のピア アクセスを有効にする最大バッチ サイズ。ピア アクセスには Linux または NVLink が必要です。NVLink を使用する場合は、より大きなバッチ サイズでピア アクセスを有効にすると効果的です。 |
| GGML_CUDA_FA_ALL_QUANTS | Boolean | false | FlashAttention CUDA カーネルのすべての KV キャッシュ量子化タイプ (組み合わせ) のコンパイル サポート。KV キャッシュ サイズをより細かく制御できますが、コンパイル時間は大幅に長くなります。 |

## MUSA

Moore Threads GPUを使用したGPUアクセラレーションを提供します。[MUSA SDK](https://developer.mthreads.com/musa/musa-sdk)がインストールされていることを確認してください。

#### Moore Threads から直接ダウンロードしてください

公式ダウンロードは、[Moore Threads 開発者サイト](https://developer.mthreads.com/sdk/download/musa) から入手できます。

### Compilation

```bash
cmake -B build -DGGML_MUSA=ON
cmake --build build --config Release
```

#### コンピューティング機能の仕様をオーバーライドする

デフォルトでは、サポートされているすべてのコンピューティング機能が有効になっています。この動作をカスタマイズするには、CMake コマンドで `MUSA_ARCHITECTURES` オプションを指定します。

```bash
cmake -B build -DGGML_MUSA=ON -DMUSA_ARCHITECTURES="21"
cmake --build build --config Release
```

この構成では、コンパイル中にコンピューティング機能 `2.1` (MTT S80) のみが有効になり、コンパイル時間を短縮できます。

#### コンパイルオプション

CUDA で利用可能なコンパイルオプションのほとんどは MUSA でも利用できるはずですが、まだ十分にテストされていません。

- 静的ビルドの場合は、`-DBUILD_SHARED_LIBS=OFF` と `-DCMAKE_POSITION_INDEPENDENT_CODE=ON` を追加します。
  ```
  cmake -B build -DGGML_MUSA=ON \
    -DBUILD_SHARED_LIBS=OFF -DCMAKE_POSITION_INDEPENDENT_CODE=ON
  cmake --build build --config Release
  ```

### 実行時のMUSA環境変数

実行時に[musa環境変数](https://docs.mthreads.com/musa-sdk/musa-sdk-doc-online/programming_guide/Z%E9%99%84%E5%BD%95/)を設定できます。

```bash
# Use `MUSA_VISIBLE_DEVICES` to hide the first compute device.
MUSA_VISIBLE_DEVICES="-0" ./build/bin/llama-server --model /srv/models/llama.gguf
```

### ユニファイドメモリ

Linuxでは、環境変数「GGML_CUDA_ENABLE_UNIFIED_MEMORY=1」を使用してユニファイドメモリを有効化できます。これにより、GPU VRAMが枯渇した際にクラッシュするのではなく、システムRAMへのスワップが可能になります。

## HIP

HIP対応AMD GPUでGPUアクセラレーションを提供します。
ROCmがインストールされていることを確認してください。
Linuxディストリビューションのパッケージマネージャー、または[ROCmクイックスタート（Linux）](https://rocm.docs.amd.com/projects/install-on-linux/en/latest/tutorial/quick-start.html#rocm-install-quick)からダウンロードできます。

- Linuxで`CMake`を使用する場合（gfx1030互換AMD GPUを想定）：
  ```bash
  HIPCXX="$(hipconfig -l)/clang" HIP_PATH="$(hipconfig -R)" \
      cmake -S . -B build -DGGML_HIP=ON -DAMDGPU_TARGETS=gfx1030 -DCMAKE_BUILD_TYPE=Release \
      && cmake --build build --config Release -- -j 16
  ```

  RDNA3+ または CDNA アーキテクチャにおけるフラッシュアテンションのパフォーマンスを向上させるには、`-DGGML_HIP_ROCWMMA_FATTN=ON` オプションを有効にすることで rocWMMA ライブラリを利用できます。ただし、ビルドシステムに rocWMMA ヘッダーがインストールされている必要があります。

  AMD が提供する `rocm` メタパッケージを使用して ROCm SDK をインストールすると、rocWMMA ライブラリはデフォルトで含まれます。メタパッケージを使用していない場合は、システムのパッケージマネージャーに応じて `rocwmma-dev` または `rocwmma-devel` パッケージを使用してライブラリをインストールできます。

  代わりに、公式の[GitHubリポジトリ](https://github.com/ROCm/rocWMMA)からライブラリをクローンし、対応するバージョンタグ（例：`rocm-6.2.4`）をチェックアウトし、CMakeで`-DCMAKE_CXX_FLAGS="-I<path/to/rocwmma>/library/include/"`を設定することで、手動でライブラリをインストールすることもできます。これはAMDによる公式サポートはありませんが、Windowsでも動作します。

  以下のエラーが表示された場合は、ご注意ください:

  ```
  clang: error: cannot find ROCm device library; provide its path via '--rocm-path' or '--rocm-device-lib-path', or pass '-nogpulib' to build without ROCm device library
  ```
  `HIP_PATH` 配下のディレクトリで `oclc_abi_version_400.bc` ファイルがあるディレクトリを探してみてください。次に、コマンドの先頭に以下の行を追加します: `HIP_DEVICE_LIB_PATH=<directory-you-just-found>`、つまり以下のようになります:

  ```bash
  HIPCXX="$(hipconfig -l)/clang" HIP_PATH="$(hipconfig -p)" \
  HIP_DEVICE_LIB_PATH=<directory-you-just-found> \
      cmake -S . -B build -DGGML_HIP=ON -DAMDGPU_TARGETS=gfx1030 -DCMAKE_BUILD_TYPE=Release \
      && cmake --build build -- -j 16
  ```

- Windows 用の `CMake` を使用する (VS 用の x64 ネイティブ ツール コマンド プロンプトを使用し、gfx1100 互換の AMD GPU を想定):

  ```bash
  set PATH=%HIP_PATH%\bin;%PATH%
  cmake -S . -B build -G Ninja -DAMDGPU_TARGETS=gfx1100 -DGGML_HIP=ON -DCMAKE_C_COMPILER=clang -DCMAKE_CXX_COMPILER=clang++ -DCMAKE_BUILD_TYPE=Release
  cmake --build build
  ```

  `AMDGPU_TARGETS` が、コンパイル対象の GPU アーキテクチャに設定されていることを確認してください。上記の例では、Radeon RX 7900XTX/XT/GRE に対応する `gfx1100` を使用しています。ターゲットのリストは [こちら](https://llvm.org/docs/AMDGPUUsage.html#processors) で確認できます。
  `rocminfo | grep gfx | head -1 | awk '{print $2}'` で得られる最も重要なバージョン情報とプロセッサのリストを照合することで、GPU バージョン文字列を検索できます。例: `gfx1035` は `gfx1030` にマッピングされます。


環境変数 [`HIP_VISIBLE_DEVICES`](https://rocm.docs.amd.com/en/latest/understand/gpu_isolation.html#hip-visible-devices) を使用して、使用する GPU を指定できます。
お使いの GPU が公式にサポートされていない場合は、環境変数 [`HSA_OVERRIDE_GFX_VERSION`] を同様の GPU に設定して使用できます。たとえば、RDNA2 では 10.3.0 (例: gfx1030、gfx1031、gfx1035)、RDNA3 では 11.0.0 です。

### 統合メモリ

Linuxでは、環境変数「GGML_CUDA_ENABLE_UNIFIED_MEMORY=1」を設定することで、統合メモリアーキテクチャ（UMA）を使用してCPUと統合GPU間でメインメモリを共有できます。ただし、非統合GPUのパフォーマンスは低下します（ただし、統合GPUでは動作が可能になります）。

## Vulkan

**Windows**

### w64devkit

[`w64devkit`](https://github.com/skeeto/w64devkit/releases) をダウンロードして解凍します。

[`Vulkan SDK`](https://vulkan.lunarg.com/sdk/home#windows) をデフォルト設定でダウンロードしてインストールします。

`w64devkit.exe` を起動し、以下のコマンドを実行して Vulkan の依存関係をコピーします。
```sh
SDK_VERSION=1.3.283.0
cp /VulkanSDK/$SDK_VERSION/Bin/glslc.exe $W64DEVKIT_HOME/bin/
cp /VulkanSDK/$SDK_VERSION/Lib/vulkan-1.lib $W64DEVKIT_HOME/x86_64-w64-mingw32/lib/
cp -r /VulkanSDK/$SDK_VERSION/Include/* $W64DEVKIT_HOME/x86_64-w64-mingw32/include/
cat > $W64DEVKIT_HOME/x86_64-w64-mingw32/lib/pkgconfig/vulkan.pc <<EOF
Name: Vulkan-Loader
Description: Vulkan Loader
Version: $SDK_VERSION
Libs: -lvulkan-1
EOF

```

`llama.cpp` ディレクトリに切り替えて、CMake を使用してビルドします。
```sh
cmake -B build -DGGML_VULKAN=ON
cmake --build build --config Release
```

### Git Bash MINGW64

[`Git-SCM`](https://git-scm.com/downloads/win) をデフォルト設定でダウンロードしてインストールしてください。

[`Visual Studio Community Edition`](https://visualstudio.microsoft.com/) をダウンロードしてインストールし、`C++` を選択してください。

[`CMake`](https://cmake.org/download/) をデフォルト設定でダウンロードしてインストールしてください。

[`Vulkan SDK`](https://vulkan.lunarg.com/sdk/home#windows) をデフォルト設定でダウンロードしてインストールしてください。

`llama.cpp` ディレクトリに移動し、右クリックして「Open Git Bash Here」を選択し、以下のコマンドを実行してください。

```
cmake -B build -DGGML_VULKAN=ON
cmake --build build --config Release
```

これで、`Vulkan`を使用して会話モードでモデルをロードできるようになりました。

```sh
build/bin/Release/llama-cli -m "[PATH TO MODEL]" -ngl 100 -c 16384 -t 10 -n -2 -cnv
```

### MSYS2
[MSYS2](https://www.msys2.org/) をインストールし、UCRT ターミナルで次のコマンドを実行して依存関係をインストールします。
```sh
pacman -S git \
    mingw-w64-ucrt-x86_64-gcc \
    mingw-w64-ucrt-x86_64-cmake \
    mingw-w64-ucrt-x86_64-vulkan-devel \
    mingw-w64-ucrt-x86_64-shaderc
```

`llama.cpp` ディレクトリに切り替えて、CMake を使用してビルドします。
```sh
cmake -B build -DGGML_VULKAN=ON
cmake --build build --config Release
```

**Docker の場合**:

Vulkan SDK をインストールする必要はありません。コンテナ内にインストールされます。

```sh
# Build the image
docker build -t llama-cpp-vulkan --target light -f .devops/vulkan.Dockerfile .

# Then, use it:
docker run -it --rm -v "$(pwd):/app:Z" --device /dev/dri/renderD128:/dev/dri/renderD128 --device /dev/dri/card1:/dev/dri/card1 llama-cpp-vulkan -m "/app/models/YOUR_MODEL_FILE" -p "Building a website can be done in 10 simple steps:" -n 400 -e -ngl 33
```

**Docker なし**:

まず、[Vulkan SDK](https://vulkan.lunarg.com/doc/view/latest/linux/getting_started_ubuntu.html) がインストールされていることを確認する必要があります。

例えば、Ubuntu 22.04 (jammy) では、以下のコマンドを使用します。

```bash
wget -qO - https://packages.lunarg.com/lunarg-signing-key-pub.asc | apt-key add -
wget -qO /etc/apt/sources.list.d/lunarg-vulkan-jammy.list https://packages.lunarg.com/vulkan/lunarg-vulkan-jammy.list
apt update -y
apt-get install -y vulkan-sdk
# To verify the installation, use the command below:
vulkaninfo
```

あるいは、パッケージマネージャーが適切なライブラリを提供している場合もあります。
例えば、Ubuntu 22.04 の場合は、代わりに `libvulkan-dev` をインストールできます。
Fedora 40 の場合は、`vulkan-devel`、`glslc`、`glslang` パッケージをインストールできます。

次に、以下の cmake コマンドを使用して llama.cpp をビルドします。

```bash
cmake -B build -DGGML_VULKAN=1
cmake --build build --config Release
# Test the output binary (with "-ngl 33" to offload all layers to GPU)
./bin/llama-cli -m "PATH_TO_MODEL" -p "Hi you how are you" -n 50 -e -ngl 33 -t 4

# You should see in the output, ggml_vulkan detected your GPU. For example:
# ggml_vulkan: Using Intel(R) Graphics (ADL GT2) | uma: 1 | fp16: 1 | warp size: 32
```

## CANN
これは、Ascend NPUのAIコアを利用したNPUアクセラレーションを提供します。[CANN](https://www.hiascend.com/en/software/cann)は、Ascend NPUをベースにしたAIアプリケーションやサービスを迅速に構築するための階層型APIです。

Ascend NPUの詳細については、[Ascend Community](https://www.hiascend.com/en/)をご覧ください。

CANNツールキットがインストールされていることを確認してください。[CANN Toolkit](https://www.hiascend.com/developer/download/community/result?module=cann)からダウンロードできます。

`llama.cpp`ディレクトリに移動し、CMakeを使用してビルドしてください。
```bash
cmake -B build -DGGML_CANN=on -DCMAKE_BUILD_TYPE=release
cmake --build build --config release
```

次の方法でテストできます:

```bash
./build/bin/llama-cli -m PATH_TO_MODEL -p "Building a website can be done in 10 steps:" -ngl 32
```

次の情報が画面に出力される場合、CANN バックエンドで `llama.cpp` を使用しています:

```bash
llm_load_tensors:       CANN model buffer size = 13313.00 MiB
llama_new_context_with_model:       CANN compute buffer size =  1260.81 MiB
```

モデル/デバイスのサポート、CANN のインストールなどの詳細情報については、[CANN 用 llama.cpp](./backend/CANN.md) を参照してください。

## Arm® KleidiAI™
KleidiAIは、Arm CPU向けに特別に設計された、AIワークロード向けに最適化されたマイクロカーネルのライブラリです。これらのマイクロカーネルはパフォーマンスを向上させ、CPUバックエンドで使用可能にすることができます。

KleidiAIを有効にするには、llama.cppディレクトリに移動し、CMakeを使用してビルドしてください。

```bash
cmake -B build -DGGML_CPU_KLEIDIAI=ON
cmake --build build --config Release
```

KleidiAIが使用されているかどうかを確認するには、以下を実行します。

```bash
./build/bin/llama-cli -m PATH_TO_MODEL -p "What is a car?"
```

KleidiAI が有効になっている場合、出力には次のような行が含まれます:

```
load_tensors: CPU_KLEIDIAI model buffer size =  3474.00 MiB
```

KleidiAI のマイクロカーネルは、dotprod、int8mm、SME などの Arm CPU 機能を使用して最適化されたテンソル演算を実装します。llama.cpp は、実行時の CPU 機能検出に基づいて最も効率的なカーネルを選択します。ただし、SME をサポートするプラットフォームでは、環境変数 `GGML_KLEIDIAI_SME=1` を設定して、SME マイクロカーネルを手動で有効にする必要があります。

ビルドターゲットによっては、他の優先度の高いバックエンドがデフォルトで有効になっている場合があります。CPU バックエンドを確実に使用するには、コンパイル時（例: -DGGML_METAL=OFF）または実行時にコマンドラインオプション `--device none` を使用して、優先度の高いバックエンドを無効にする必要があります。

## OpenCL

これは、最新の Adreno GPU 上で OpenCL を介して GPU アクセラレーションを提供します。
OpenCL バックエンドの詳細については、[OPENCL.md](./backend/OPENCL.md) を参照してください。

### Android

NDKが`$ANDROID_NDK`で利用可能であると仮定します。まず、OpenCLヘッダーとICDローダーライブラリが利用できない場合はインストールします。

```sh
mkdir -p ~/dev/llm
cd ~/dev/llm

git clone https://github.com/KhronosGroup/OpenCL-Headers && \
cd OpenCL-Headers && \
cp -r CL $ANDROID_NDK/toolchains/llvm/prebuilt/linux-x86_64/sysroot/usr/include

cd ~/dev/llm

git clone https://github.com/KhronosGroup/OpenCL-ICD-Loader && \
cd OpenCL-ICD-Loader && \
mkdir build_ndk && cd build_ndk && \
cmake .. -G Ninja -DCMAKE_BUILD_TYPE=Release \
  -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK/build/cmake/android.toolchain.cmake \
  -DOPENCL_ICD_LOADER_HEADERS_DIR=$ANDROID_NDK/toolchains/llvm/prebuilt/linux-x86_64/sysroot/usr/include \
  -DANDROID_ABI=arm64-v8a \
  -DANDROID_PLATFORM=24 \
  -DANDROID_STL=c++_shared && \
ninja && \
cp libOpenCL.so $ANDROID_NDK/toolchains/llvm/prebuilt/linux-x86_64/sysroot/usr/lib/aarch64-linux-android
```

次にOpenCLを有効にしてllama.cppをビルドします。

```sh
cd ~/dev/llm

git clone https://github.com/ggml-org/llama.cpp && \
cd llama.cpp && \
mkdir build-android && cd build-android

cmake .. -G Ninja \
  -DCMAKE_TOOLCHAIN_FILE=$ANDROID_NDK/build/cmake/android.toolchain.cmake \
  -DANDROID_ABI=arm64-v8a \
  -DANDROID_PLATFORM=android-28 \
  -DBUILD_SHARED_LIBS=OFF \
  -DGGML_OPENCL=ON

ninja
```

### Windows Arm64

まず、OpenCLヘッダーとICDローダーライブラリがない場合はインストールします。

```powershell
mkdir -p ~/dev/llm

cd ~/dev/llm
git clone https://github.com/KhronosGroup/OpenCL-Headers && cd OpenCL-Headers
mkdir build && cd build
cmake .. -G Ninja `
  -DBUILD_TESTING=OFF `
  -DOPENCL_HEADERS_BUILD_TESTING=OFF `
  -DOPENCL_HEADERS_BUILD_CXX_TESTS=OFF `
  -DCMAKE_INSTALL_PREFIX="$HOME/dev/llm/opencl"
cmake --build . --target install

cd ~/dev/llm
git clone https://github.com/KhronosGroup/OpenCL-ICD-Loader && cd OpenCL-ICD-Loader
mkdir build && cd build
cmake .. -G Ninja `
  -DCMAKE_BUILD_TYPE=Release `
  -DCMAKE_PREFIX_PATH="$HOME/dev/llm/opencl" `
  -DCMAKE_INSTALL_PREFIX="$HOME/dev/llm/opencl"
cmake --build . --target install
```

次にOpenCLを有効にしてllama.cppをビルドします。

```powershell
cmake .. -G Ninja `
  -DCMAKE_TOOLCHAIN_FILE="$HOME/dev/llm/llama.cpp/cmake/arm64-windows-llvm.cmake" `
  -DCMAKE_BUILD_TYPE=Release `
  -DCMAKE_PREFIX_PATH="$HOME/dev/llm/opencl" `
  -DBUILD_SHARED_LIBS=OFF `
  -DGGML_OPENCL=ON
ninja
```

## Android

Android 上でビルドする方法についてのドキュメントを読むには、[ここをクリック](./android.md)

## IBM Z & LinuxONE

IBM Z および LinuxONE 上でのビルド方法に関するドキュメントを読むには、[ここをクリック](./build-s390x.md)

## GPU アクセラレーション対応バックエンドに関する注意事項

`-ngl 0` オプションを使用している場合でも、計算の一部を高速化するために GPU が使用される場合があります。`--device none` を使用すると、GPU アクセラレーションを完全に無効にできます。

ほとんどの場合、複数のバックエンドを同時にビルドして使用できます。例えば、CMake で `-DGGML_CUDA=ON -DGGML_VULKAN=ON` オプションを使用すると、CUDA と Vulkan の両方をサポートする llama.cpp をビルドできます。実行時には、`--device` オプションを使用して、使用するバックエンドデバイスを指定できます。使用可能なデバイスの一覧を表示するには、`--list-devices` オプションを使用します。

バックエンドは、実行時に動的にロードできる動的ライブラリとしてビルドできます。これにより、同じ llama.cpp バイナリを、異なる GPU を搭載した複数のマシンで使用できます。この機能を有効にするには、ビルド時に `GGML_BACKEND_DL` オプションを使用します。
