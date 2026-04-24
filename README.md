# lm-deploy

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

``` llama cpp 启动服务
CUDA_VISIBLE_DEVICES=0 \
  ~/llama.cpp/build/bin/llama-server \
  --model unsloth/Qwen3.6-35B-A3B-GGUF/Qwen3.6-35B-A3B-UD-IQ4_NL_XL.gguf \
  --mmproj unsloth/Qwen3.6-35B-A3B-GGUF/mmproj-F16.gguf \
  --alias "unsloth/Qwen3.6-35B-A3B" \
  --temp 0.6 \
  --top-p 0.95 \
  --ctx-size 163840 \
  --top-k 20 \
  --min-p 0.00 \
  --port 8001 \
  --parallel 2 \
  --split-mode none \
  --main-gpu 0
```


``` openclaw 配置
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
