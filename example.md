# Example

Note: 
硬件使用的是一张24G 的 4090 部署, GPU 上部署了Qwen3.6-35B-A3B, 
剩下的显存放不下 ASR-1.7B了, 因此ASR-1.7B 放在了CPU上, 因此可能会慢,
可以尝试部署到自己的电脑上， 大概需要 显存 3352MB

## Using OpenAI SDK

### Qwen3.6-35B-A3B
```python
from openai import OpenAI
import json
openai_client = OpenAI(
    # base_url = "http://127.0.0.1:8001/v1",
    # api_key = "sk-no-key-required",
    base_url = "http://js1.blockelite.cn:19395/v1",
    api_key = "EMPTY",
)
completion = openai_client.chat.completions.create(
    model = "unsloth/Qwen3.6-35B-A3B",
    messages = [{"role": "user", "content": "你好你是谁？"},],
)
print(completion.choices[0].message.content)
```

### 语音 Qwen3-ASR-1.7B-Q8_0.gguf + mmproj-Qwen3-ASR-1.7B-Q8_0.gguf

```python
import base64
import requests
from openai import OpenAI

# Initialize client
client = OpenAI(
    # base_url="http://localhost:8002/v1",
    base_url = "http://js1.blockelite.cn:19396/v1",
    api_key="EMPTY"
)

url = "https://qianwen-res.oss-cn-beijing.aliyuncs.com/Qwen3-ASR-Repo/asr_en.wav"
response = requests.get(url)
response.raise_for_status()
wav_data = response.content
encoded_string = base64.b64encode(wav_data).decode('utf-8')

# Create multimodal chat completion request
response = client.chat.completions.create(
    model="Qwen/Qwen3-ASR-1.7B",
    messages=[
        {
            "role": "user",
            "content": [
                {
                    "type": "input_audio",
                    "input_audio": {
                        "data": encoded_string,
                        "format": "wav"
                    }
                }
            ]
        },
    ]
)

print(response.choices[0].message.content)
```

## Opneclaw.json

```json5
# openclaw 配置
"models": {
    "providers": {
      "llamacpp": {
        "baseUrl": "http://127.0.0.1:19396/v1",
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

