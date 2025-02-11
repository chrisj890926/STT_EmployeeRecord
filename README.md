## Table of Contents

  - [STT](#STT)
    - [WhisperSmall](#whisperSmall)
    - [whispermedium](#whispermedium)
    - [large-v2](#large-v2)
  - [Ollama API](#ollama)
    - [mistral-latest](#mistral)
    - [deepseek-r1 8b](#deepseek)
    - [phi4:latest](#phi4latest)
    - [OutputCSV](#output-csv)
  - [OllamaChat](#ollama-chat)
    - [ChooseModel](#search-the-list-from-ollama-that-the-model-you-can-use)
    - [Input](#input-message-like-below)
# Install

```bash
pip install openai-whisper pandas ollama ffmpeg-python
```
[text](https://www.gyan.dev/ffmpeg/builds/)
- choose **ffmpeg-git-full.7z**
- **Win+R** → 輸入 **sysdm.cpl**
- 進入「**進階**」→「**環境變數**」
- 在 **系統變數** 找到 **Path**
- 點選「**編輯**」，新增 **C:\ffmpeg\bin**

# STT
## WhisperSmall

```python
import whisper

# 加載 Whisper 並讓模型運行在 GPU
model = whisper.load_model("small").to("cuda")

# 轉錄音檔
file_path = r"C:\Users\User\Dropbox\May4s\employee_log.m4a"
transcription = model.transcribe(file_path)

# 輸出轉錄結果
print(transcription["text"])
```
## whispermedium
```python
model = whisper.load_model("medium").to("cuda")
```
## large-v2
```python
model = whisper.load_model("large-v2").to("cuda")
```

---

# Ollama
## mistral
```python
import requests
import json
import re
# 連接到 Ollama 伺服器
OLLAMA_URL = "http://localhost:11434/api/generate"

def query_ollama(prompt):
    payload = {
        "model": "mistral:latest",
        "prompt": prompt,
        "stream": False  # 設為 False 讓回應一次返回
    }
    response = requests.post(OLLAMA_URL, json=payload)
    return response.json()["response"]

# 產生 Prompt
prompt = f"""
請根據以下員工日誌內容，解析出專案名稱、員工 ID、時間戳記，並輸出成 JSON 格式：
---
{transcription["text"]}
---
請以 JSON 格式輸出，包含：
- "employee_id"
- "timestamp"
- "project"
- "summary"
"""

# 呼叫 Ollama
output_text = query_ollama(prompt)
print("LLM 輸出結果：", output_text)


# 解析 JSON，去除 LLM 可能產生的多餘文字
match = re.search(r"\{.*\}|\[.*\]", output_text, re.DOTALL)
if match:
    json_text = match.group(0)  # 提取純 JSON
    try:
        parsed_output = json.loads(json_text)
        print("✅ 解析成功：", parsed_output)
    except json.JSONDecodeError as e:
        print("❌ JSON 解析錯誤：", e)
else:
    print("❌ 解析失敗，請檢查 LLM 輸出")
```
### output csv
```python
import pandas as pd

# 轉成 DataFrame
df = pd.DataFrame([parsed_output])

# 顯示表格
print(df)

# 存成 CSV
df.to_csv("employee_log.csv", index=False)
```

## deepseek

choose the model to deepseek

```python
"model": "deepseek-r1:8b"
```

## phi4:latest

take the same way like above.

```python
"model": "phi4:latest"
```

# Ollama Chat

## Search the List from Ollama that the model you can use.

```bash
ollama list
ollama run deepseek-r1:8b
```

## Input Message like below

```bash
根據以下員工日誌內容，解析出專案名稱、員工 ID、時間戳記、總結，並輸出成 JSON 格式：我是員工編號EMP123今天是2025年2月11日早上10點我今天主要負責的專案是AI研發計畫今天完成了模型訓練與部署測試但遇到的問題是GPU記憶體不足明天計畫優化LLM推理速度嘗試用量化技術減少記憶體的使用
```
