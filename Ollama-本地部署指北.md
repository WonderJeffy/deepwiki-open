# 使用 Ollama 本地部署 DeepWiki 指北

我的环境:

- MacOS 26.0
- Python 3.11.10
- Node v22.14.0
- Ollama 0.13.0 

## 第 1 步：下载所需模型 (Download Required Models)

在终端中运行：

```bash
ollama pull llama3:8b 
ollama pull nomic-embed-text
ollama pull qwen3:30b-a3b
```

解释：
- `llama3:8b`：用于 DeepWiki 自身运行的本地大语言模型（LLM）。
- `nomic-embed-text`：DeepWiki 用于为代码创建向量嵌入（embeddings）。
- `qwen3:30b-a3b`：本地语言大模型（LLM），用于生成文档（你也可以替换成其他模型，见“高级”部分）。

## 第 2 步：配置 DeepWiki (Set Up DeepWiki)

克隆仓库并进入目录：

```bash
git clone https://github.com/AsyncFuncAI/deepwiki-open.git
cd deepwiki-open
```

在项目根目录创建 `.env`（使用本地 Ollama 时无需云 API Key）：

```
# No need for API keys when using Ollama locally
PORT=8001
OLLAMA_HOST=http://localhost:11434
#Required placeholders (even if unused):
GOOGLE_API_KEY=your_google_api_key
OPENAI_API_KEY=your_openai_api_key
```

切换为 Ollama 本地 Embedder（会覆盖当前 embedder 配置）：

```
cp api/config/embedder.ollama.json.bak api/config/embedder.json
# overwrite api/config/embedder.json? (y/n [n]) y
```

启动后端：

```bash
cd api
python -m pip install poetry==2.0.1 && poetry install
python -m api.main
```

启动前端：

```bash
npm install
npm run dev
```

## 第 4 步：使用 DeepWiki (Use DeepWiki with Ollama)

1. 打开浏览器访问 http://localhost:3000
2. 输入 GitHub / GitLab / Bitbucket 仓库地址
3. 勾选 “Use Local Ollama Model”
4. 点击 “Generate Wiki”

## 故障排除 (Troubleshooting)

### "Python 版本问题"

- 系统自带的 Python 版本可能过低，建议使用 `pyenv` 安装并切换到 Python 3.11.10 以上版本

### “Cannot connect to Ollama server”

- 确认 Ollama 在后台运行：`ollama list`
- 确认 Ollama 监听默认端口 `11434`
- 尝试重启 Ollama

### 生成较慢（Slow generation）

- 本地模型通常慢于云端；可试更小的仓库或更强的机器
- `qwen3:1.7b` 在速度/质量上较均衡，更大模型更慢但质量或更好

### 内存不足（Out of memory）

- 选择更小的模型（如 `phi3:mini`）
- 关闭其他占用内存较高的应用

## 高级：使用不同模型 (Advanced: Using Different Models)

你可以修改 `api/config/generator.json` 指定不同的本地 LLM（下面代码块保留英文配置原样）：

```python
"generator_ollama": {
    "model_client": OllamaClient,
    "model_kwargs": {
        "model": "qwen3:1.7b",  # Change this to another model
        "options": {
            "temperature": 0.7,
            "top_p": 0.8,
        }
    },
},
```

同样，可以更换 embedding 模型：

```python
"embedder_ollama": {
    "model_client": OllamaClient,
    "model_kwargs": {
        "model": "nomic-embed-text"  # Change this to another embedding model
    },
},
```

可用模型列表参见 Ollama Library：https://ollama.com/library 或运行 `ollama list`。

## 性能考虑 (Performance Considerations)

### 硬件建议（Hardware Requirements）

- CPU：建议 4+ 核
- RAM：8GB 起步，推荐 16GB+
- 存储：建议预留 10GB+（用于模型）
- GPU：可选，但可显著加速

### 模型选择参考（Model Selection Guide）

| 模型 (Model) | 体积 (Size) | 速度 (Speed) | 质量 (Quality) | 适用场景 (Use Case)               |
| ------------ | ----------- | ------------ | -------------- | --------------------------------- |
| phi3:mini    | 1.3GB       | Fast         | Good           | 小型项目、快速测试                |
| qwen3:1.7b   | 3.8GB       | Medium       | Better         | 默认，速度与质量平衡              |
| llama3:8b    | 8GB         | Slow         | Best           | 复杂项目、需要更详细的分析        |

## 限制 (Limitations)

当使用 Ollama 搭配 DeepWiki：

1. 无网络访问（No Internet Access）：模型完全离线，无法访问外部信息
2. 上下文窗口较小（Limited Context Window）：通常小于最新云端模型
3. 整体能力弱于最新云端模型（Less Powerful）：质量可能不及 SOTA 云模型

## 结语 (Conclusion)

使用 Ollama 搭配 DeepWiki，可以构建一个完全本地、注重隐私的代码文档方案。虽然速度和质量可能不及云端，但对大多数项目已足够实用，且零 API 成本。
