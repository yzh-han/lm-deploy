# lm-deploy

## cuda toolkit and llama.cpp安装

```bash
# cuda toolkit
wget https://developer.download.nvidia.com/compute/cuda/12.4.1/local_installers/cuda_12.4.1_550.54.15_linux.run
sudo sh cuda_12.4.1_550.54.15_linux.run

# llama.cpp
# clone: https://github.com/ggml-org/llama.cpp
cmake -B build -DGGML_CUDA=ON
cmake --build build --config Release
```
或者通过 conda

``` bash
# 安装 Miniconda
wget https://anaconda.com
bash Miniconda3-latest-Linux-x86_64.sh

# 当询问是否运行 conda init 时，输入 yes，这样 Conda 会自动配置环境变量。
source ~/.bashrc # (base)
# 否则 手动:
source ~/miniconda3/bin/activate # 路径可能不同(miniconda3
conda init bash # 执行初始化命令->会写入配置 .bashrc
source ~/.bashrc

# 创建环境
conda create --name <env name>

# 激活环境
conda activate <env name>

# 退出环境
conda deactivate

# 删除环境
conda remove --name <env name> --all

# cuda 安装 toolkit
conda install cuda -c nvidia  # 安装最新版
conda install cuda-toolkit=12.1 -c nvidia # 或者指定特定版本（例如 12.1）

# 验证 cuda toolkit
nvcc -V

# 从 conda-forge 安装 llama cpp
conda install -c conda-forge llama.cpp

# 镜像源
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free/
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main/
```

```bash
# llama cpp 启动服务
# Qwen3.6-27B  max-context  # 65536 98304 131072 163840
CUDA_VISIBLE_DEVICES=0 \
  ~/llama.cpp/build/bin/llama-server \
  --model unsloth/Qwen3.6-27B-GGUF/Qwen3.6-27B-UD-Q4_K_XL.gguf \
  --mmproj unsloth/Qwen3.6-27B-GGUF/mmproj-F16.gguf \
  --alias "unsloth/Qwen3.6-27B" \
  --temp 0.6 \
  --top-p 0.95 \
  --ctx-size 65536 \
  --top-k 20 \
  --min-p 0.00 \
  --host 0.0.0.0 \
  --port 8801 \
  --parallel 2 \
  --split-mode none \
  --main-gpu 0

# Qwen3.6-35B-A3B  max-context  # 131072 163840
CUDA_VISIBLE_DEVICES=0 \
  ~/llama.cpp/build/bin/llama-server \
  --model unsloth/Qwen3.6-35B-A3B-GGUF/Qwen3.6-35B-A3B-UD-IQ4_NL_XL.gguf \
  --mmproj unsloth/Qwen3.6-35B-A3B-GGUF/mmproj-F16.gguf \
  --alias "unsloth/Qwen3.6-35B-A3B" \
  --temp 0.6 \
  --top-p 0.95 \
  --ctx-size 131072 \
  --top-k 20 \
  --min-p 0.00 \
  --host 0.0.0.0 \
  --port 8801 \
  --parallel 1 \
  --split-mode none \
  --main-gpu 0 \
  --n-gpu-layers 99\ 
  --mmproj-offload

# Qwen3-ASR-0.6B
CUDA_VISIBLE_DEVICES=0 \
  ~/llama.cpp/build/bin/llama-server \
  --model mradermacher/Qwen3-ASR-0.6B-i1-GGUF/Qwen3-ASR-0.6B.i1-Q4_K_M.gguf \
  --ctx-size 1024 \
  --host 0.0.0.0 \
  --port 8802 \
  # --threads $(nproc) 自动取 CPU 线程数
  # --n-gpu-layers 0 强制不用gpu

# Qwen3-ASR-1.7B
CUDA_VISIBLE_DEVICES=0 \
  ~/llama.cpp/build/bin/llama-server \
  --model ggml-org/Qwen3-ASR-1.7B-GGUF/Qwen3-ASR-1.7B-Q8_0.gguf \
  --mmproj ggml-org/Qwen3-ASR-1.7B-GGUF/mmproj-Qwen3-ASR-1.7B-Q8_0.gguf \
  --ctx-size 1024 \
  --host 0.0.0.0 \
  --port 8802 \
  --split-mode none \
  --main-gpu 0 \
  --n-gpu-layers 0 \
  --mmproj-offload
  # --mmproj-offload : offload 到 backend 这是 gpu


# Qwen3-ASR-1.7B CPU
CUDA_VISIBLE_DEVICES="" \
GGML_VK_VISIBLE_DEVICES="" \
  ~/llama.cpp/build/bin/llama-server \
  --model ggml-org/Qwen3-ASR-1.7B-GGUF/Qwen3-ASR-1.7B-Q8_0.gguf \
  --mmproj ggml-org/Qwen3-ASR-1.7B-GGUF/mmproj-Qwen3-ASR-1.7B-bf16.gguf \
  --ctx-size 1024 \
  --host 0.0.0.0 \
  --port 8802 \
  --threads 12
  # --threads $(nproc) 自动取 CPU 线程数
  # --n-gpu-layers 0 强制不用gpu
```


```json5
# openclaw 配置
"models": {
    "providers": {
      "llamacpp": {
        "baseUrl": "http://127.0.0.1:8001/v1",
        "apiKey": "no-key",
        "api": "openai-completions",
        "models": [
          {
            "id": "unsloth/Qwen3.6-35B-A3B",
            "name": "Qwen",
            "api": "openai-completions",
            "reasoning": true,
            "contextWindow": 131072,
            "maxTokens": 8192
          }
        ]
      }
    }
  },
```

``` bash
# 下载模型
hf download unsloth/Qwen3.6-35B-A3B-GGUF \
  --local-dir ./unsloth/Qwen3.6-35B-A3B-GGUF \
  --include "Qwen3.6-35B-A3B-UD-IQ4_NL_XL.gguf" # and mmproj*

HF_ENDPOINT=https://hf-mirror.com\
  hf download mradermacher/Qwen3-ASR-0.6B-i1-GGUF \
  --local-dir ./mradermacher/Qwen3-ASR-0.6B-i1-GGUF \
  --include "Qwen3-ASR-0.6B.i1-Q4_K_M.gguf"

HF_ENDPOINT=https://hf-mirror.com\
  hf download ggml-org/Qwen3-ASR-1.7B-GGUF \
  --local-dir ./ggml-org/Qwen3-ASR-1.7B-GGUF \
  --include "Qwen3-ASR-1.7B-Q8_0.gguf" \
  --include "mmproj-Qwen3-ASR-1.7B-bf16.gguf"

HF_ENDPOINT=https://hf-mirror.com\
  hf download unsloth/Qwen3.6-27B-GGUF \
  --local-dir ./unsloth/Qwen3.6-27B-GGUF \
  --include "Qwen3.6-27B-UD-Q4_K_XL.gguf" \
  --include "mmproj-F16.gguf"
```
