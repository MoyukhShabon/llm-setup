# Local LLM Deployment Guide

Runing a LLM on **gpu02** (`gpu02.sbms.hku.hk`):

Some models have been already downloaded using either Ollama or SGLang.

- refer to nct2ctml/config.py for a list of predownloaded models.
- refer to 'llm_comparison' spreadsheet on google drive (tab: all_llms_library) for the details of llms that were downloaded and tested on GPU02 server.
- The spreadsheet also mentions which models were tried and tested on which platform (SGLang/ Ollama), and how much memory would they approximately consume.

Based on the benchmarking results, we mostly use either of the following models:
- neuralmagic/DeepSeek-R1-Distill-Qwen-32B-quantized.w4a16
- gemma4:31b

## Running a LLM via SGLang:

Default port used: 30000.

**Conda environment**:

For the models that use SGLang, we can use the existing conda env: "sglang-env" on GPU02 server.
This environment runs SGLang version (insert-version) , along with required dependencies.

**1. Model: Qwen3.5-35B-A3B-GPTQ-Int4,**
   
 Conda env: sglang-env
 - The model needs 2 GPUs, so check which 2 GPUs are free and pass those ids to the 'CUDA_VISIBLE_DEVICES' env variable.
 This command uses GPU 0 and GPU 1 as an example.

```bash
CUDA_VISIBLE_DEVICES=0,1 python -m sglang.launch_server \
  --model-path Qwen/Qwen3.5-35B-A3B-GPTQ-Int4 \
  --port 30000 \
  --tp-size 2 \
  --mem-fraction-static 0.8 \
  --context-length 262144 \
  --reasoning-parser qwen3
```

For convenience, the script "run_qwen3.5_sglang.sh" can be executed directly.
```bash
./run_qwen3.5_sglang.sh
```

**2. Model: DeepSeek-R1-Distill-Qwen-32B-quantized.w4a16,**
 Conda env: sglang-env
This model needs 1 GPU and the command below expects the id of the GPU that is available.

```bash
CUDA_VISIBLE_DEVICES=2 python -m sglang.launch_server \
  --model neuralmagic/DeepSeek-R1-Distill-Qwen-32B-quantized.w4a16 \
  --context-length 131072 \
  --trust-remote-code \
  --host 0.0.0.0
```

Note: For convenience, the script "run_deepseek_sglang.sh" can be executed directly.

```bash
./run_deepseek_sglang.sh
```
For running a new model via SGLang, refer to the instructions on hugging face page or check out [SGLang's cookbook](https://lmsysorg.mintlify.app/cookbook/intro)
For adjusting server parameters for different models, refer to [Server Arguments](https://lmsysorg.mintlify.app/docs/advanced_features/server_arguments)

## Running a LLM via Ollama

Ollama runs as a service on GPU02, so there is no need to start/launch any command.
Default port used: 11434.

Refer to [Ollama commands](https://docs.ollama.com/cli), for a list of commands provided by ollama.

The pre-downloaded models on GPU02 can be looked up via:
```bash
curl http://localhost:11434/api/tags
```
The model(s) currently loaded in memory can be checked with:
```bash
ollama ps
```
The following command can be run on server side to load a model(gemma4:31b for example) in memory:
```bash
ollama run gemma4:31b
```

When a client sends a request to a model that is not pre downloaded, Ollama does not pull missing models and would return HTTP 404 `model not found`.

Smoke test for the pre-downloaded Gemma model:

```bash
curl http://localhost:11434/api/chat \
  -H "Content-Type: application/json" \
  -d '{"model": "gemma4:31b", "messages": [{"role": "user", "content": "Hello"}], "stream": false}'
```

Only if you need a model that is not pre-downloaded, run:

```bash
ollama pull <model>
```


**Client-side configuration**:

Currently nct2ctml and matchminer-patient are the 2 clients that needs these LLM models running.
Refer to the individual README files for any client related configuration for accessing these models on server side.
