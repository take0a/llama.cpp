# llama.cpp

![llama](https://user-images.githubusercontent.com/1991296/230134379-7181e485-c521-4d23-a0d6-f7b3b61ba524.png)

[![License: MIT](https://img.shields.io/badge/license-MIT-blue.svg)](https://opensource.org/licenses/MIT)
[![Release](https://img.shields.io/github/v/release/ggml-org/llama.cpp)](https://github.com/ggml-org/llama.cpp/releases)
[![Server](https://github.com/ggml-org/llama.cpp/actions/workflows/server.yml/badge.svg)](https://github.com/ggml-org/llama.cpp/actions/workflows/server.yml)

[Roadmap](https://github.com/users/ggerganov/projects/7) / [Manifesto](https://github.com/ggml-org/llama.cpp/discussions/205) / [ggml](https://github.com/ggml-org/ggml)

Metaの[LLaMA](https://arxiv.org/abs/2302.13971)モデル（およびその他）の純粋なC/C++での推論

## 最近のAPIの変更

- [Changelog for `libllama` API](https://github.com/ggml-org/llama.cpp/issues/9289)
- [Changelog for `llama-server` REST API](https://github.com/ggml-org/llama.cpp/issues/9291)

## 注目トピック

- 🔥 `llama-server` にマルチモーダルサポートが追加されました: [#12898](https://github.com/ggml-org/llama.cpp/pull/12898) | [ドキュメント](./docs/multimodal.md)
- 新しいバイナリ `llama-mtmd-cli` が導入されました。これは、`llava-cli`、`minicpmv-cli`、`gemma3-cli` ([#13012](https://github.com/ggml-org/llama.cpp/pull/13012))、`qwen2vl-cli` ([#13141](https://github.com/ggml-org/llama.cpp/pull/13141)) の代替となります。`libllava` は非推奨となります。
- FIM 補完用の VS Code 拡張機能: https://github.com/ggml-org/llama.vscode
- `llama-server` におけるユニバーサル [ツール呼び出しサポート](./docs/function-calling.md) https://github.com/ggml-org/llama.cpp/pull/9639
- FIM補完用Vim/Neovimプラグイン: https://github.com/ggml-org/llama.vim
- GGUF-my-LoRAのご紹介 https://github.com/ggml-org/llama.cpp/discussions/10123
- Hugging Face推論エンドポイントがGGUFを標準サポートしました！ https://github.com/ggml-org/llama.cpp/discussions/9669
- Hugging Face GGUFエディタ: [ディスカッション](https://github.com/ggml-org/llama.cpp/discussions/9268) | [ツール](https://huggingface.co/spaces/CISCai/gguf-editor)

----

## クイックスタート

llama.cpp の使い方は簡単です。お使いのマシンにインストールする方法はいくつかあります。

- [brew、nix、または winget](docs/install.md) を使用して `llama.cpp` をインストールする
- Docker で実行する - [Docker ドキュメント](docs/docker.md) をご覧ください。
- [リリースページ](https://github.com/ggml-org/llama.cpp/releases) からビルド済みのバイナリをダウンロードする
- このリポジトリをクローンしてソースからビルドする - [ビルドガイド](docs/build.md) をご覧ください。

インストールが完了したら、使用するモデルが必要になります。詳細については、[モデルの取得と量子化](#obtaining-and-quantizing-models) セクションをご覧ください。

コマンド例:

```sh
# Use a local model file
llama-cli -m my_model.gguf

# Or download and run a model directly from Hugging Face
llama-cli -hf ggml-org/gemma-3-1b-it-GGUF

# Launch OpenAI-compatible API server
llama-server -hf ggml-org/gemma-3-1b-it-GGUF
```

## 説明

`llama.cpp` の主な目的は、最小限のセットアップで、ローカルおよびクラウド上の幅広いハードウェア上で最先端のパフォーマンスを実現し、LLM 推論を実現することです。

- 依存関係のないシンプルなC/C++実装
- Apple Siliconを第一級市民として採用 - ARM NEON、Accelerate、Metalフレームワークで最適化
- x86アーキテクチャ向けAVX、AVX2、AVX512、AMXサポート
- 1.5ビット、2ビット、3ビット、4ビット、5ビット、6ビット、8ビットの整数量子化により、推論速度の向上とメモリ使用量の削減を実現
- NVIDIA GPUでLLMを実行するためのカスタムCUDAカーネル（HIP経由のAMD GPUとMUSA経由のムーアスレッドGPUをサポート）
- VulkanおよびSYCLバックエンドをサポート
- VRAM容量全体を超えるモデルを部分的に高速化するCPU+GPUハイブリッド推論

`llama.cpp`プロジェクトは、[ggml](https://github.com/ggml-org/ggml)ライブラリの新機能開発のための主要なプラットフォームです。

<details>
<summary>Models</summary>

通常、以下の基本モデルの微調整もサポートされています。

新しいモデルのサポートを追加する手順: [HOWTO-add-model.md](docs/development/HOWTO-add-model.md)

#### Text-only

- [X] LLaMA 🦙
- [x] LLaMA 2 🦙🦙
- [x] LLaMA 3 🦙🦙🦙
- [X] [Mistral 7B](https://huggingface.co/mistralai/Mistral-7B-v0.1)
- [x] [Mixtral MoE](https://huggingface.co/models?search=mistral-ai/Mixtral)
- [x] [DBRX](https://huggingface.co/databricks/dbrx-instruct)
- [X] [Falcon](https://huggingface.co/models?search=tiiuae/falcon)
- [X] [Chinese LLaMA / Alpaca](https://github.com/ymcui/Chinese-LLaMA-Alpaca) and [Chinese LLaMA-2 / Alpaca-2](https://github.com/ymcui/Chinese-LLaMA-Alpaca-2)
- [X] [Vigogne (French)](https://github.com/bofenghuang/vigogne)
- [X] [BERT](https://github.com/ggml-org/llama.cpp/pull/5423)
- [X] [Koala](https://bair.berkeley.edu/blog/2023/04/03/koala/)
- [X] [Baichuan 1 & 2](https://huggingface.co/models?search=baichuan-inc/Baichuan) + [derivations](https://huggingface.co/hiyouga/baichuan-7b-sft)
- [X] [Aquila 1 & 2](https://huggingface.co/models?search=BAAI/Aquila)
- [X] [Starcoder models](https://github.com/ggml-org/llama.cpp/pull/3187)
- [X] [Refact](https://huggingface.co/smallcloudai/Refact-1_6B-fim)
- [X] [MPT](https://github.com/ggml-org/llama.cpp/pull/3417)
- [X] [Bloom](https://github.com/ggml-org/llama.cpp/pull/3553)
- [x] [Yi models](https://huggingface.co/models?search=01-ai/Yi)
- [X] [StableLM models](https://huggingface.co/stabilityai)
- [x] [Deepseek models](https://huggingface.co/models?search=deepseek-ai/deepseek)
- [x] [Qwen models](https://huggingface.co/models?search=Qwen/Qwen)
- [x] [PLaMo-13B](https://github.com/ggml-org/llama.cpp/pull/3557)
- [x] [Phi models](https://huggingface.co/models?search=microsoft/phi)
- [x] [PhiMoE](https://github.com/ggml-org/llama.cpp/pull/11003)
- [x] [GPT-2](https://huggingface.co/gpt2)
- [x] [Orion 14B](https://github.com/ggml-org/llama.cpp/pull/5118)
- [x] [InternLM2](https://huggingface.co/models?search=internlm2)
- [x] [CodeShell](https://github.com/WisdomShell/codeshell)
- [x] [Gemma](https://ai.google.dev/gemma)
- [x] [Mamba](https://github.com/state-spaces/mamba)
- [x] [Grok-1](https://huggingface.co/keyfan/grok-1-hf)
- [x] [Xverse](https://huggingface.co/models?search=xverse)
- [x] [Command-R models](https://huggingface.co/models?search=CohereForAI/c4ai-command-r)
- [x] [SEA-LION](https://huggingface.co/models?search=sea-lion)
- [x] [GritLM-7B](https://huggingface.co/GritLM/GritLM-7B) + [GritLM-8x7B](https://huggingface.co/GritLM/GritLM-8x7B)
- [x] [OLMo](https://allenai.org/olmo)
- [x] [OLMo 2](https://allenai.org/olmo)
- [x] [OLMoE](https://huggingface.co/allenai/OLMoE-1B-7B-0924)
- [x] [Granite models](https://huggingface.co/collections/ibm-granite/granite-code-models-6624c5cec322e4c148c8b330)
- [x] [GPT-NeoX](https://github.com/EleutherAI/gpt-neox) + [Pythia](https://github.com/EleutherAI/pythia)
- [x] [Snowflake-Arctic MoE](https://huggingface.co/collections/Snowflake/arctic-66290090abe542894a5ac520)
- [x] [Smaug](https://huggingface.co/models?search=Smaug)
- [x] [Poro 34B](https://huggingface.co/LumiOpen/Poro-34B)
- [x] [Bitnet b1.58 models](https://huggingface.co/1bitLLM)
- [x] [Flan T5](https://huggingface.co/models?search=flan-t5)
- [x] [Open Elm models](https://huggingface.co/collections/apple/openelm-instruct-models-6619ad295d7ae9f868b759ca)
- [x] [ChatGLM3-6b](https://huggingface.co/THUDM/chatglm3-6b) + [ChatGLM4-9b](https://huggingface.co/THUDM/glm-4-9b) + [GLMEdge-1.5b](https://huggingface.co/THUDM/glm-edge-1.5b-chat) + [GLMEdge-4b](https://huggingface.co/THUDM/glm-edge-4b-chat)
- [x] [GLM-4-0414](https://huggingface.co/collections/THUDM/glm-4-0414-67f3cbcb34dd9d252707cb2e)
- [x] [SmolLM](https://huggingface.co/collections/HuggingFaceTB/smollm-6695016cad7167254ce15966)
- [x] [EXAONE-3.0-7.8B-Instruct](https://huggingface.co/LGAI-EXAONE/EXAONE-3.0-7.8B-Instruct)
- [x] [FalconMamba Models](https://huggingface.co/collections/tiiuae/falconmamba-7b-66b9a580324dd1598b0f6d4a)
- [x] [Jais](https://huggingface.co/inceptionai/jais-13b-chat)
- [x] [Bielik-11B-v2.3](https://huggingface.co/collections/speakleash/bielik-11b-v23-66ee813238d9b526a072408a)
- [x] [RWKV-6](https://github.com/BlinkDL/RWKV-LM)
- [x] [QRWKV-6](https://huggingface.co/recursal/QRWKV6-32B-Instruct-Preview-v0.1)
- [x] [GigaChat-20B-A3B](https://huggingface.co/ai-sage/GigaChat-20B-A3B-instruct)
- [X] [Trillion-7B-preview](https://huggingface.co/trillionlabs/Trillion-7B-preview)
- [x] [Ling models](https://huggingface.co/collections/inclusionAI/ling-67c51c85b34a7ea0aba94c32)

#### Multimodal

- [x] [LLaVA 1.5 models](https://huggingface.co/collections/liuhaotian/llava-15-653aac15d994e992e2677a7e), [LLaVA 1.6 models](https://huggingface.co/collections/liuhaotian/llava-16-65b9e40155f60fd046a5ccf2)
- [x] [BakLLaVA](https://huggingface.co/models?search=SkunkworksAI/Bakllava)
- [x] [Obsidian](https://huggingface.co/NousResearch/Obsidian-3B-V0.5)
- [x] [ShareGPT4V](https://huggingface.co/models?search=Lin-Chen/ShareGPT4V)
- [x] [MobileVLM 1.7B/3B models](https://huggingface.co/models?search=mobileVLM)
- [x] [Yi-VL](https://huggingface.co/models?search=Yi-VL)
- [x] [Mini CPM](https://huggingface.co/models?search=MiniCPM)
- [x] [Moondream](https://huggingface.co/vikhyatk/moondream2)
- [x] [Bunny](https://github.com/BAAI-DCAI/Bunny)
- [x] [GLM-EDGE](https://huggingface.co/models?search=glm-edge)
- [x] [Qwen2-VL](https://huggingface.co/collections/Qwen/qwen2-vl-66cee7455501d7126940800d)

</details>

<details>
<summary>Bindings</summary>

- Python: [ddh0/easy-llama](https://github.com/ddh0/easy-llama)
- Python: [abetlen/llama-cpp-python](https://github.com/abetlen/llama-cpp-python)
- Go: [go-skynet/go-llama.cpp](https://github.com/go-skynet/go-llama.cpp)
- Node.js: [withcatai/node-llama-cpp](https://github.com/withcatai/node-llama-cpp)
- JS/TS (llama.cpp server client): [lgrammel/modelfusion](https://modelfusion.dev/integration/model-provider/llamacpp)
- JS/TS (Programmable Prompt Engine CLI): [offline-ai/cli](https://github.com/offline-ai/cli)
- JavaScript/Wasm (works in browser): [tangledgroup/llama-cpp-wasm](https://github.com/tangledgroup/llama-cpp-wasm)
- Typescript/Wasm (nicer API, available on npm): [ngxson/wllama](https://github.com/ngxson/wllama)
- Ruby: [yoshoku/llama_cpp.rb](https://github.com/yoshoku/llama_cpp.rb)
- Rust (more features): [edgenai/llama_cpp-rs](https://github.com/edgenai/llama_cpp-rs)
- Rust (nicer API): [mdrokz/rust-llama.cpp](https://github.com/mdrokz/rust-llama.cpp)
- Rust (more direct bindings): [utilityai/llama-cpp-rs](https://github.com/utilityai/llama-cpp-rs)
- Rust (automated build from crates.io): [ShelbyJenkins/llm_client](https://github.com/ShelbyJenkins/llm_client)
- C#/.NET: [SciSharp/LLamaSharp](https://github.com/SciSharp/LLamaSharp)
- C#/VB.NET (more features - community license): [LM-Kit.NET](https://docs.lm-kit.com/lm-kit-net/index.html)
- Scala 3: [donderom/llm4s](https://github.com/donderom/llm4s)
- Clojure: [phronmophobic/llama.clj](https://github.com/phronmophobic/llama.clj)
- React Native: [mybigday/llama.rn](https://github.com/mybigday/llama.rn)
- Java: [kherud/java-llama.cpp](https://github.com/kherud/java-llama.cpp)
- Zig: [deins/llama.cpp.zig](https://github.com/Deins/llama.cpp.zig)
- Flutter/Dart: [netdur/llama_cpp_dart](https://github.com/netdur/llama_cpp_dart)
- Flutter: [xuegao-tzx/Fllama](https://github.com/xuegao-tzx/Fllama)
- PHP (API bindings and features built on top of llama.cpp): [distantmagic/resonance](https://github.com/distantmagic/resonance) [(more info)](https://github.com/ggml-org/llama.cpp/pull/6326)
- Guile Scheme: [guile_llama_cpp](https://savannah.nongnu.org/projects/guile-llama-cpp)
- Swift [srgtuszy/llama-cpp-swift](https://github.com/srgtuszy/llama-cpp-swift)
- Swift [ShenghaiWang/SwiftLlama](https://github.com/ShenghaiWang/SwiftLlama)
- Delphi [Embarcadero/llama-cpp-delphi](https://github.com/Embarcadero/llama-cpp-delphi)

</details>

<details>
<summary>UIs</summary>

*(to have a project listed here, it should clearly state that it depends on `llama.cpp`)*

- [AI Sublime Text plugin](https://github.com/yaroslavyaroslav/OpenAI-sublime-text) (MIT)
- [cztomsik/ava](https://github.com/cztomsik/ava) (MIT)
- [Dot](https://github.com/alexpinel/Dot) (GPL)
- [eva](https://github.com/ylsdamxssjxxdd/eva) (MIT)
- [iohub/collama](https://github.com/iohub/coLLaMA) (Apache-2.0)
- [janhq/jan](https://github.com/janhq/jan) (AGPL)
- [johnbean393/Sidekick](https://github.com/johnbean393/Sidekick) (MIT)
- [KanTV](https://github.com/zhouwg/kantv?tab=readme-ov-file) (Apache-2.0)
- [KodiBot](https://github.com/firatkiral/kodibot) (GPL)
- [llama.vim](https://github.com/ggml-org/llama.vim) (MIT)
- [LARS](https://github.com/abgulati/LARS) (AGPL)
- [Llama Assistant](https://github.com/vietanhdev/llama-assistant) (GPL)
- [LLMFarm](https://github.com/guinmoon/LLMFarm?tab=readme-ov-file) (MIT)
- [LLMUnity](https://github.com/undreamai/LLMUnity) (MIT)
- [LMStudio](https://lmstudio.ai/) (proprietary)
- [LocalAI](https://github.com/mudler/LocalAI) (MIT)
- [LostRuins/koboldcpp](https://github.com/LostRuins/koboldcpp) (AGPL)
- [MindMac](https://mindmac.app) (proprietary)
- [MindWorkAI/AI-Studio](https://github.com/MindWorkAI/AI-Studio) (FSL-1.1-MIT)
- [Mobile-Artificial-Intelligence/maid](https://github.com/Mobile-Artificial-Intelligence/maid) (MIT)
- [Mozilla-Ocho/llamafile](https://github.com/Mozilla-Ocho/llamafile) (Apache-2.0)
- [nat/openplayground](https://github.com/nat/openplayground) (MIT)
- [nomic-ai/gpt4all](https://github.com/nomic-ai/gpt4all) (MIT)
- [ollama/ollama](https://github.com/ollama/ollama) (MIT)
- [oobabooga/text-generation-webui](https://github.com/oobabooga/text-generation-webui) (AGPL)
- [PocketPal AI](https://github.com/a-ghorbani/pocketpal-ai) (MIT)
- [psugihara/FreeChat](https://github.com/psugihara/FreeChat) (MIT)
- [ptsochantaris/emeltal](https://github.com/ptsochantaris/emeltal) (MIT)
- [pythops/tenere](https://github.com/pythops/tenere) (AGPL)
- [ramalama](https://github.com/containers/ramalama) (MIT)
- [semperai/amica](https://github.com/semperai/amica) (MIT)
- [withcatai/catai](https://github.com/withcatai/catai) (MIT)
- [Autopen](https://github.com/blackhole89/autopen) (GPL)

</details>

<details>
<summary>Tools</summary>

- [akx/ggify](https://github.com/akx/ggify) – download PyTorch models from HuggingFace Hub and convert them to GGML
- [akx/ollama-dl](https://github.com/akx/ollama-dl) – download models from the Ollama library to be used directly with llama.cpp
- [crashr/gppm](https://github.com/crashr/gppm) – launch llama.cpp instances utilizing NVIDIA Tesla P40 or P100 GPUs with reduced idle power consumption
- [gpustack/gguf-parser](https://github.com/gpustack/gguf-parser-go/tree/main/cmd/gguf-parser) - review/check the GGUF file and estimate the memory usage
- [Styled Lines](https://marketplace.unity.com/packages/tools/generative-ai/styled-lines-llama-cpp-model-292902) (proprietary licensed, async wrapper of inference part for game development in Unity3d with pre-built Mobile and Web platform wrappers and a model example)

</details>

<details>
<summary>Infrastructure</summary>

- [Paddler](https://github.com/distantmagic/paddler) - Stateful load balancer custom-tailored for llama.cpp
- [GPUStack](https://github.com/gpustack/gpustack) - Manage GPU clusters for running LLMs
- [llama_cpp_canister](https://github.com/onicai/llama_cpp_canister) - llama.cpp as a smart contract on the Internet Computer, using WebAssembly
- [llama-swap](https://github.com/mostlygeek/llama-swap) - transparent proxy that adds automatic model switching with llama-server
- [Kalavai](https://github.com/kalavai-net/kalavai-client) - Crowdsource end to end LLM deployment at any scale
- [llmaz](https://github.com/InftyAI/llmaz) - ☸️ Easy, advanced inference platform for large language models on Kubernetes.
</details>

<details>
<summary>Games</summary>

- [Lucy's Labyrinth](https://github.com/MorganRO8/Lucys_Labyrinth) - A simple maze game where agents controlled by an AI model will try to trick you.

</details>


## Supported backends

| Backend | Target devices |
| --- | --- |
| [Metal](docs/build.md#metal-build) | Apple Silicon |
| [BLAS](docs/build.md#blas-build) | All |
| [BLIS](docs/backend/BLIS.md) | All |
| [SYCL](docs/backend/SYCL.md) | Intel and Nvidia GPU |
| [MUSA](docs/build.md#musa) | Moore Threads GPU |
| [CUDA](docs/build.md#cuda) | Nvidia GPU |
| [HIP](docs/build.md#hip) | AMD GPU |
| [Vulkan](docs/build.md#vulkan) | GPU |
| [CANN](docs/build.md#cann) | Ascend NPU |
| [OpenCL](docs/backend/OPENCL.md) | Adreno GPU |
| [RPC](https://github.com/ggml-org/llama.cpp/tree/master/tools/rpc) | All |

## モデルの取得と量子化

[Hugging Face](https://huggingface.co) プラットフォームは、`llama.cpp` と互換性のある [多数の LLM](https://huggingface.co/models?library=gguf&sort=trending) をホストしています。

- [Trending](https://huggingface.co/models?library=gguf&sort=trending)
- [LLaMA](https://huggingface.co/models?sort=trending&search=llama+gguf)

GGUF ファイルを手動でダウンロードするか、[Hugging Face](https://huggingface.co/) または [ModelScope](https://modelscope.cn/) などのモデルホスティングサイトから `llama.cpp` 互換モデルを直接使用できます。その場合は、CLI 引数 `-hf <user>/<model>[:quant]` を使用します。例:

```sh
llama-cli -hf ggml-org/gemma-3-1b-it-GGUF
```

デフォルトでは、CLIはHugging Faceからダウンロードしますが、環境変数`MODEL_ENDPOINT`を使用して他のオプションに切り替えることができます。例えば、環境変数`MODEL_ENDPOINT=https://www.modelscope.cn/`を設定することで、ModelScopeやその他のモデル共有コミュニティからモデルチェックポイントをダウンロードするように選択できます。

モデルをダウンロードしたら、CLIツールを使用してローカルで実行します。詳細は以下を参照してください。

`llama.cpp`を使用するには、モデルを[GGUF](https://github.com/ggml-org/ggml/blob/master/docs/gguf.md)ファイル形式で保存する必要があります。他のデータ形式のモデルは、このリポジトリにある`convert_*.py` Pythonスクリプトを使用してGGUFに変換できます。

Hugging Face プラットフォームは、`llama.cpp` を使用してモデルを変換、量子化、ホスティングするためのさまざまなオンラインツールを提供しています。

- [GGUF-my-repo スペース](https://huggingface.co/spaces/ggml-org/gguf-my-repo) を使用して、GGUF 形式に変換し、モデルの重みをより小さなサイズに量子化します。
- [GGUF-my-LoRA スペース](https://huggingface.co/spaces/ggml-org/gguf-my-lora) を使用して、LoRA アダプターを GGUF 形式に変換します (詳細はこちら: https://github.com/ggml-org/llama.cpp/discussions/10123)。
- [GGUF-editor スペース](https://huggingface.co/spaces/CISCai/gguf-editor) を使用して、ブラウザで GGUF メタデータを編集します (詳細はこちら: https://github.com/ggml-org/llama.cpp/discussions/9268)
- [推論エンドポイント](https://ui.endpoints.huggingface.co/) を使用して、クラウドで `llama.cpp` を直接ホストします (詳細: https://github.com/ggml-org/llama.cpp/discussions/9669)

モデルの量子化の詳細については、[こちらのドキュメント](tools/quantize/README.md) をご覧ください

## [`llama-cli`](tools/main)

#### `llama.cpp` のほとんどの機能にアクセスして実験するための CLI ツール。

- <details open>
    <summary>会話モードで実行</summary>

    チャットテンプレートが組み込まれたモデルは、自動的に会話モードを有効にします。有効にならない場合は、`-cnv` を追加し、`--chat-template NAME` で適切なチャットテンプレートを指定することで、手動で有効にすることができます。

    ```bash
    llama-cli -m model.gguf

    # > hi, who are you?
    # Hi there! I'm your helpful assistant! I'm an AI-powered chatbot designed to assist and provide information to users like you. I'm here to help answer your questions, provide guidance, and offer support on a wide range of topics. I'm a friendly and knowledgeable AI, and I'm always happy to help with anything you need. What's on your mind, and how can I assist you today?
    #
    # > what is 1+1?
    # Easy peasy! The answer to 1+1 is... 2!
    ```

    </details>

- <details>
    <summary>カスタムチャットテンプレートを使用して会話モードで実行する</summary>

    ```bash
    # use the "chatml" template (use -h to see the list of supported templates)
    llama-cli -m model.gguf -cnv --chat-template chatml

    # use a custom template
    llama-cli -m model.gguf -cnv --in-prefix 'User: ' --reverse-prompt 'User:'
    ```

    </details>

- <details>
    <summary>シンプルなテキスト補完を実行する</summary>

    会話モードを明示的に無効にするには、`-no-cnv` を使用します。

    ```bash
    llama-cli -m model.gguf -p "I believe the meaning of life is" -n 128 -no-cnv

    # I believe the meaning of life is to find your own truth and to live in accordance with it. For me, this means being true to myself and following my passions, even if they don't align with societal expectations. I think that's what I love about yoga – it's not just a physical practice, but a spiritual one too. It's about connecting with yourself, listening to your inner voice, and honoring your own unique journey.
    ```

    </details>

- <details>
    <summary>カスタム文法で出力を制限する</summary>

    ```bash
    llama-cli -m model.gguf -n 256 --grammar-file grammars/json.gbnf -p 'Request: schedule a call at 8pm; Command:'

    # {"appointmentTime": "8pm", "appointmentDetails": "schedule a a call"}
    ```

    [grammars/](grammars/) フォルダには、いくつかのサンプル文法が含まれています。独自の文法を作成するには、[GBNF ガイド](grammars/README.md) をご覧ください。

    より複雑な JSON 文法を作成するには、https://grammar.intrinsiclabs.ai/ をご覧ください。

    </details>


## [`llama-server`](tools/server)

#### LLM を提供するための軽量の [OpenAI API](https://github.com/openai/openai-openapi) 互換 HTTP サーバー。

- <details open>
    <summary>ポート8080でデフォルト設定のローカルHTTPサーバーを起動します。</summary>

    ```bash
    llama-server -m model.gguf --port 8080

    # Basic web UI can be accessed via browser: http://localhost:8080
    # Chat completion endpoint: http://localhost:8080/v1/chat/completions
    ```

    </details>

- <details>
    <summary>複数ユーザーと並列デコードをサポート</summary>

    ```bash
    # up to 4 concurrent requests, each with 4096 max context
    llama-server -m model.gguf -c 16384 -np 4
    ```

    </details>

- <details>
    <summary>投機的デコードを有効にする</summary>

    ```bash
    # the draft.gguf model should be a small variant of the target model.gguf
    llama-server -m model.gguf -md draft.gguf
    ```

    </details>

- <details>
    <summary>埋め込みモデルを提供する</summary>

    ```bash
    # use the /embedding endpoint
    llama-server -m model.gguf --embedding --pooling cls -ub 8192
    ```

    </details>

- <details>
    <summary>再ランキングモデルを提供する</summary>

    ```bash
    # use the /reranking endpoint
    llama-server -m model.gguf --reranking
    ```

    </details>

- <details>
    <summary>すべての出力を文法で制約する</summary>

    ```bash
    # custom grammar
    llama-server -m model.gguf --grammar-file grammar.gbnf

    # JSON
    llama-server -m model.gguf --grammar-file grammars/json.gbnf
    ```

    </details>


## [`llama-perplexity`](tools/perplexity)

#### 与えられたテキストに対するモデルの困惑度[^1][^2]（およびその他の品質指標）を測定するためのツール。

- <details open>
    <summary>テキストファイルの困惑度を測定する</summary>

    ```bash
    llama-perplexity -m model.gguf -f file.txt

    # [1]15.2701,[2]5.4007,[3]5.3073,[4]6.2965,[5]5.8940,[6]5.6096,[7]5.7942,[8]4.9297, ...
    # Final estimate: PPL = 5.4007 +/- 0.67339
    ```

    </details>

- <details>
    <summary>KLダイバージェンスを測定する</summary>

    ```bash
    # TODO
    ```

    </details>

[^1]: [tools/perplexity/README.md](./tools/perplexity/README.md)
[^2]: [https://huggingface.co/docs/transformers/perplexity](https://huggingface.co/docs/transformers/perplexity)

## [`llama-bench`](tools/llama-bench)

#### さまざまなパラメータに対する推論のパフォーマンスをベンチマークします。

- <details open>
    <summary>デフォルトのベンチマークを実行する</summary>

    ```bash
    llama-bench -m model.gguf

    # Output:
    # | model               |       size |     params | backend    | threads |          test |                  t/s |
    # | ------------------- | ---------: | ---------: | ---------- | ------: | ------------: | -------------------: |
    # | qwen2 1.5B Q4_0     | 885.97 MiB |     1.54 B | Metal,BLAS |      16 |         pp512 |      5765.41 ± 20.55 |
    # | qwen2 1.5B Q4_0     | 885.97 MiB |     1.54 B | Metal,BLAS |      16 |         tg128 |        197.71 ± 0.81 |
    #
    # build: 3e0ba0e60 (4229)
    ```

    </details>

## [`llama-run`](tools/run)

#### `llama.cpp` モデルを実行するための包括的な例。推論に便利です。RamaLama [^3] で使用されます。

- <details>
    <summary>特定のプロンプトでモデルを実行します（デフォルトではOllamaレジストリから取得されます）</summary>

    ```bash
    llama-run granite-code
    ```

    </details>

[^3]: [RamaLama](https://github.com/containers/ramalama)

## [`llama-simple`](examples/simple)

#### `llama.cpp` を使ってアプリを実装するための最小限の例。開発者にとって便利です。

- <details>
    <summary>基本的なテキスト補完</summary>

    ```bash
    llama-simple -m model.gguf

    # Hello my name is Kaitlyn and I am a 16 year old girl. I am a junior in high school and I am currently taking a class called "The Art of
    ```

    </details>


## 貢献

- 貢献者はプルリクエスト（PR）を開くことができます。
- コラボレーターは `llama.cpp` リポジトリのブランチにプッシュし、プルリクエストを `master` ブランチにマージできます。
- コラボレーターは貢献度に基づいて招待されます。
- 問題、プルリクエスト、プロジェクトの管理に関するご協力をいただければ幸いです。
- 最初のコントリビューションに適したタスクについては、[good first issues](https://github.com/ggml-org/llama.cpp/issues?q=is%3Aissue+is%3Aopen+label%3A%22good+first+issue%22) をご覧ください。
- 詳細については、[CONTRIBUTING.md](CONTRIBUTING.md) をご覧ください。
- こちらも必ずお読みください: [Inference at the edge](https://github.com/ggml-org/llama.cpp/discussions/205)
- ご興味のある方のために、少し背景をお伝えします: [Changelog podcast](https://changelog.com/podcast/532)

## その他のドキュメント

- [メイン (cli)](tools/main/README.md)
- [サーバー](tools/server/README.md)
- [GBNF 文法](grammars/README.md)

#### 開発ドキュメント

- [ビルド方法](docs/build.md)
- [Docker での実行](docs/docker.md)
- [Android でのビルド](docs/android.md)
- [パフォーマンスのトラブルシューティング](docs/development/token_generation_performance_tips.md)
- [GGML のヒントとコツ](https://github.com/ggml-org/llama.cpp/wiki/GGML-Tips-&-Tricks)

#### 重要な論文とモデルの背景

モデル生成の品質に関する問題がある場合は、少なくとも以下のリンクと論文を参照して、LLaMAモデルの限界を理解してください。これは、適切なモデルサイズを選択し、LLaMAモデルとChatGPTモデル間の重要な違いと微妙な違いの両方を理解する際に特に重要です。
- LLaMA:
- [LLaMAの紹介：基礎的な650億パラメータの大規模言語モデル](https://ai.facebook.com/blog/large-language-model-llama-meta-ai/)
- [LLaMA：オープンで効率的な基礎言語モデル](https://arxiv.org/abs/2302.13971)
- GPT-3
- [言語モデルは少数ショット学習器です](https://arxiv.org/abs/2005.14165)
- GPT-3.5 / InstructGPT / ChatGPT:
- [指示に従う言語モデルのアライメント](https://openai.com/research/instruction-following)
- [人間による指示に従う言語モデルの学習フィードバック](https://arxiv.org/abs/2203.02155)

## XCFramework
XCFramework は、iOS、visionOS、tvOS、macOS 向けのライブラリのプリコンパイル版です。Swift プロジェクトでは、ソースからライブラリをコンパイルすることなく使用できます。例:
```swift
// swift-tools-version: 5.10
// The swift-tools-version declares the minimum version of Swift required to build this package.

import PackageDescription

let package = Package(
    name: "MyLlamaPackage",
    targets: [
        .executableTarget(
            name: "MyLlamaPackage",
            dependencies: [
                "LlamaFramework"
            ]),
        .binaryTarget(
            name: "LlamaFramework",
            url: "https://github.com/ggml-org/llama.cpp/releases/download/b5046/llama-b5046-xcframework.zip",
            checksum: "c19be78b5f00d8d29a25da41042cb7afa094cbf6280a225abe614b03b20029ab"
        )
    ]
)
```
上記の例では、ライブラリの中間ビルド「b5046」を使用しています。URLとチェックサムを変更することで、別のバージョンを使用するように変更できます。

## 補完
一部の環境ではコマンドライン補完が利用できます。

#### Bash Completion
```bash
$ build/bin/llama-cli --completion-bash > ~/.llama-completion.bash
$ source ~/.llama-completion.bash
```
オプションとして、`.bashrc` または `.bash_profile` にこれを追加して自動的に読み込むこともできます。例:
```console
$ echo "source ~/.llama-completion.bash" >> ~/.bashrc
```

## 依存関係

- [yhirose/cpp-httplib](https://github.com/yhirose/cpp-httplib) - シングルヘッダーHTTPサーバー。`llama-server`で使用。- MITライセンス
- [stb-image](https://github.com/nothings/stb) - シングルヘッダー画像フォーマットデコーダー。マルチモーダルサブシステムで使用。- パブリックドメイン
- [nlohmann/json](https://github.com/nlohmann/json) - シングルヘッダーJSONライブラリ。様々なツールやサンプルで使用。- MITライセンス
- [minja](https://github.com/google/minja) - C++で書かれた最小限のJinjaパーサー。様々なツールやサンプルで使用。- MITライセンス
- [linenoise.cpp](./tools/run/linenoise.cpp/linenoise.cpp) - readlineのような行編集機能を提供するC++ライブラリ。`llama-run`で使用。- BSDライセンス2条項ライセンス
- [curl](https://curl.se/) - 様々なツールやサンプルで使用されるクライアントサイドURL転送ライブラリ - [CURLライセンス](https://curl.se/docs/copyright.html)
- [miniaudio.h](https://github.com/mackron/miniaudio) - マルチモーダルサブシステムで使用されるシングルヘッダーオーディオフォーマットデコーダー - パブリックドメイン
