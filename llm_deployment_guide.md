# Local LLM Deployment Guide

Runing a LLM on **gpu02** (`gpu02.sbms.hku.hk`):

Based on the benchmarking results, we mostly use either of the following models:
- neuralmagic/DeepSeek-R1-Distill-Qwen-32B-quantized.w4a16
- gemma4:31b

The following documentation describes how to run `neuralmagic/DeepSeek-R1-Distill-Qwen-32B-quantized.w4a16` vis SGLang and `gemma4:31b` via Ollama.

## Running a LLM via SGLang:

Default port used: 30000.

#### Conda environment:

For the models that use SGLang, we can use the existing conda env: `sglang-env` on GPU02 server, which contains SGLang version '0.5.6.post2'.

**To recreate the current `sglang-env` environment, run the following commands:**

```bash
conda create -n <sglang-env-name> python=3.12.13 pip -y
conda activate <sglang-env-name>
pip install uv
uv pip install 'git+https://github.com/sgl-project/sglang.git@f41832795ee73bdc6b1b6e00eff3cd676dc1223f#subdirectory=python&egg=sglang[all]'
```

Verify the installation:
```bash
python -c "import sglang; print(sglang.__version__)"
```


**1. Model: neuralmagic/DeepSeek-R1-Distill-Qwen-32B-quantized.w4a16**

This model needs 1 GPU and the command below sets the environment variable `CUDA_VISIBLE_DEVICES` to the Id of the GPU that is available.

```bash
CUDA_VISIBLE_DEVICES=2 python -m sglang.launch_server \
  --model neuralmagic/DeepSeek-R1-Distill-Qwen-32B-quantized.w4a16 \
  --context-length 131072 \
  --trust-remote-code \
  --host 0.0.0.0
```

**2. Model: Qwen3.5-35B-A3B-GPTQ-Int4,**
   
This model needs 2 GPUs, so check which 2 GPUs are free and pass those ids to the 'CUDA_VISIBLE_DEVICES' env variable.
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

To run other models via SGLang, refer to the instructions on hugging face page or check out [SGLang's cookbook](https://lmsysorg.mintlify.app/cookbook/intro).

Newer models might need a newer version of SGLang, follow the installation steps here: https://lmsysorg.mintlify.app/docs/get-started/install.

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

Smoke test for an API request to Gemma model:

```bash
curl http://localhost:11434/api/chat \
  -H "Content-Type: application/json" \
  -d '{"model": "gemma4:31b", "messages": [{"role": "user", "content": "Hello"}], "stream": false}'
```

Only if you need a model that is not pre-downloaded, run:

```bash
ollama pull <model>
```

**Notes**:
- Refer to nct2ctml/config.py for a list of predownloaded models.
- Refer to 'llm_comparison' spreadsheet on google drive (tab: all_llms_library) for the details of llms that were downloaded and tested on GPU02 server.
- The spreadsheet also mentions which models were tried and tested on which platform (SGLang/ Ollama), and how much memory would they approximately consume.



**Client-side configuration**:

Currently nct2ctml and matchminer-patient are the 2 clients that needs these LLM models running.
Refer to the individual README files for any client related configuration for accessing these models on server side.
