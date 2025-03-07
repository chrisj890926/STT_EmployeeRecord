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
  - [Demo](#demo)
    - [transcription](#transcription-text)
    - [LLM](#deepseek-r1-8b)
    - [LLM](#phi4latest-1)
# Install

```bash
pip install openai-whisper pandas ollama ffmpeg-python
```

[FFmpeg](https://www.gyan.dev/ffmpeg/builds/)
- choose **ffmpeg-git-full.7z**
- **Win+R** → 輸入 **sysdm.cpl**
- 進入「**進階**」→「**環境變數**」
- 在 **系統變數** 找到 **Path**
- 點選「**編輯**」，新增 **C:\ffmpeg\bin**

### mac

```bash
brew install ffmpeg
```

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

# Demo

The large-v2 model has the best transcription result after testing few models.

## transcription text
```bash
我是員工編號EMP123今天是2025年2月11日早上10點我今天主要負責的專案是AI研發計畫今天完成了模型訓練與部署測試但遇到的問題是GPU記憶體不足明天計畫優化LLM推理速度嘗試用量化技術減少記憶體的使用
```
## phi4:latest

The **phi4:latest** has the relatively great performance：

根據提供的員工日誌內容，我們可以解析出以下信息：
- **專案名稱**：AI研發計畫
- **員工 ID**：EMP123
- **時間戳記**：2025年2月11日早上10點
- **總結**：
  - 主要負責的專案是AI研發計畫。
  - 完成了模型訓練與部署測試。
  - 遇到的問題是GPU記憶體不足。
  - 明天的計畫包括優化LLM推理速度，並嘗試用量化技術來減少記憶體使用。

以下是將這些信息以 JSON 格式輸出：

```json
{
  "project_name": "AI研發計畫",
  "employee_id": "EMP123",
  "timestamp": "2025-02-11 10:00",
  "summary": {
    "responsibilities": "主要負責的專案是AI研發計畫。",
    "accomplishments": "完成了模型訓練與部署測試。",
    "challenges": "遇到的問題是GPU記憶體不足。",
    "next_steps": "明天計畫優化LLM推理速度，嘗試用量化技術減少記憶體的使用。"
  }
}
```

## DeepSeek-r1 8b

<think>
嗯，好的，我需要帮助用户将他们的员工日志信息转化为纯JSON格式，并添加一个摘要。首先，我要确认日志中的内容：

“我是員工編號EMP123今天是2025年2月11日早上10點我今天主要負責的專案是AI研發計畫今天完成了模型訓練與部署測試但遇到的問題是GPU記憶體不足明天計畫優化LLM推理速度嘗試用量化技術減少記憶     
體的使用”

从日志中，我可以提取出以下信息：

- 员工ID：EMP123
- 日期和时间：2025年2月11日早上10點
- 专案名称：AI 研发 计划

在用户之前的查询中，我已经将这些信息转换为JSON格式。现在，用户希望添加一个摘要（summary），因此我需要根据日志内容撰写一个简短的总结。

总结应该包括主要事件、遇到的问题以及下一步计划。具体来说：

1. 员工完成了模型训练和部署测量。
2. 遇到了GPU内存不足的问题。
3. 明天计划优化LLM推理速度，尝试使用量化技术减少内存使用。

基于这些信息，我可以写一个摘要，简要说明当天的工作内容、遇到的挑战以及明天的计划。

接下来，我需要将这些整合成一个JSON对象，并添加summary字段。确保整个结构符合用户要求的纯JSON格式，不使用任何Markdown或其他格式。

```json
{
  "employee_id": "EMP123",
  "project_name": "AI 研发 计划",
  "timestamp": "2025年2月11日早上10點",
  "summary": "完成模型训练和部署测量，遇到GPU内存不足的问题，明天计划优化LLM推理速度并尝试使用量化技术减少内存使用。"
}
```

这样，我就将员工日志信息整理成了一个包含employee_id、project_name、timestamp以及summary的JSON对象。摘要部分简要总结了当天的工作内容和下一步计划，符合用户的要求。

在处理这个问题时，我需要确保不遗漏任何重要信息，同时保持数据的准确性和格式的正确性。此外，还要注意JSON的语法是否正确，避免使用非标准的字符或格式。

总结来说，这个过程包括：

1. 理解用户的需求：提取员工ID、专案名称和时间戳，并添加一个摘要。
2. 分析日志内容，提取所需信息。
3. 撰写摘要，涵盖主要事件和解决问题的计划。
4. 整合所有信息，形成正确的JSON格式。

通过这些步骤，我能够准确地满足用户的要求，并提供一个结构清晰、内容完整的响应。
</think>

```json
{
  "employee_id": "EMP123",
  "project_name": "AI 研发 计划",
  "timestamp": "2025年2月11日早上10點",
  "summary": "完成模型训练和部署测量，遇到GPU内存不足的问题，明天计划优化LLM推理速度并尝试使用量化技术减少内存使用。"
}
```
