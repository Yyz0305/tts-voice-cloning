# TTS Voice Cloning

基于 [GPT-SoVITS](https://github.com/RVC-Boss/GPT-SoVITS) 构建的零样本实时语音克隆 Web 应用。通过浏览器麦克风录制 3-10 秒参考音频，即可合成任意中文/英文/日文/粤语/韩语文本。


## 特性

- **零样本克隆** — 无需训练，3 秒音频即可复刻音色
- **麦克风直录** — 浏览器内一键录制参考音频
- **多语言支持** — 中文、英文、日文、粤语、韩语
- **GPU 加速** — 支持 CUDA 推理，RTX 4060 约 3 秒合成
- **即开即用** — 双击 bat 脚本启动 WebUI


## 安装

### 环境要求

- Windows 10+ / Linux
- Python 3.10-3.11
- NVIDIA GPU + CUDA 12.x（推荐，CPU 也可运行但较慢）
- 8GB+ 显存（推荐），最低 4GB

### 1. 克隆项目

```bash
git clone https://github.com/yyzyyz0527-eng/tts-voice-cloning.git
cd tts-voice-cloning
```

### 2. 创建虚拟环境

**Windows:**
```powershell
python -m venv venv
.\venv\Scripts\python.exe -m pip install --upgrade pip
```

**Linux/macOS:**
```bash
python3 -m venv venv
source venv/bin/activate
pip install --upgrade pip
```

### 3. 安装依赖与 PyTorch

**Windows CUDA 12.6:**
```powershell
.\venv\Scripts\python.exe -m pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple --trusted-host pypi.tuna.tsinghua.edu.cn
.\venv\Scripts\python.exe -m pip install torch torchaudio -f https://mirror.sjtu.edu.cn/pytorch-wheels/torch_stable.html
```

**CPU 模式:**
```bash
pip install -r requirements.txt
pip install torch torchaudio --index-url https://download.pytorch.org/whl/cpu
```

### 4. 下载预训练模型

```powershell
# 创建模型目录
New-Item -ItemType Directory -Force -Path "GPT_SoVITS\pretrained_models"

# 下载预训练模型（约 4.3GB，推荐用镜像站）
curl.exe -L -o pretrained_models.zip "https://hf-mirror.com/XXXXRT/GPT-SoVITS-Pretrained/resolve/main/pretrained_models.zip"
Expand-Archive pretrained_models.zip -DestinationPath GPT_SoVITS -Force
Remove-Item pretrained_models.zip

# 下载 G2PW 模型
curl.exe -L -o G2PWModel.zip "https://hf-mirror.com/XXXXRT/GPT-SoVITS-Pretrained/resolve/main/G2PWModel.zip"
Expand-Archive G2PWModel.zip -DestinationPath "GPT_SoVITS\text" -Force
Remove-Item G2PWModel.zip
```

> 镜像说明：国内用户建议用 `https://hf-mirror.com` 或 `https://www.modelscope.cn` 下载模型。

### 5. 下载 v2ProPlus 底模（可选）

```powershell
# 创建目录
New-Item -ItemType Directory -Force -Path "GPT_SoVITS\pretrained_models\v2Pro"

# 下载模型文件并放入 GPT_SoVITS\pretrained_models\v2Pro\s2Gv2ProPlus.pth
```

> 如果不需要 v2ProPlus，可在 `voice_clone_app/settings.py` 中切换到 v3 或其他模型版本。


## 使用

### 启动

**Windows（双击）:**
```
run_realtime_voice_clone.bat
```

**命令行:**
```powershell
.\venv\Scripts\python.exe -m voice_clone_app
```

### 操作步骤

1. 浏览器打开 `http://127.0.0.1:7860`
2. 允许浏览器麦克风权限
3. 点击麦克风按钮录制 **3-10 秒**参考音频
4. 在「合成文本」输入要生成的文字
5. 选择目标语言
6. 点击「开始合成」
7. 播放/下载生成的音频


## 目录结构

```
voice_clone_app/           # 核心应用封装
  __main__.py              # Gradio 启动入口
  settings.py              # 端口、模型路径等配置
  languages.py             # 语言选项与归一化
  audio.py                 # 音频转码、校验
  service.py               # 模型加载与推理服务
  ui.py                    # WebUI 页面定义

mic_tts.py                 # 兼容入口（同 voice_clone_app）
quick_tts.py               # 命令行零样本 TTS 脚本
run_realtime_voice_clone.bat   # Windows 双击启动
run_realtime_voice_clone.ps1   # Windows PowerShell 启动

GPT_SoVITS/                # GPT-SoVITS 框架（上游依赖）
```

## 配置

通过环境变量覆盖默认配置：

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `VOICE_CLONE_PORT` | 7860 | 服务端口 |
| `VOICE_CLONE_GPT_PATH` | s1v3.ckpt | GPT 模型路径 |
| `VOICE_CLONE_SOVITS_PATH` | s2Gv2ProPlus.pth | SoVITS 模型路径 |
| `VOICE_CLONE_MIN_SECONDS` | 3 | 最小录音秒数 |
| `VOICE_CLONE_MAX_SECONDS` | 10 | 最大录音秒数 |
| `VOICE_CLONE_TOP_K` | 20 | GPT Top-K 采样 |
| `VOICE_CLONE_TOP_P` | 0.6 | GPT Top-P 采样 |
| `VOICE_CLONE_TEMPERATURE` | 0.6 | GPT 温度 |
| `VOICE_CLONE_SPEED` | 1.0 | 语速倍率 (0.6-1.65) |

示例：
```powershell
$env:VOICE_CLONE_PORT = "7861"
$env:VOICE_CLONE_DEFAULT_TEXT = "欢迎使用语音克隆"
.\venv\Scripts\python.exe -m voice_clone_app
```

## 命令行 TTS

提供独立命令行脚本用于批量或脚本化合成：

```powershell
.\venv\Scripts\python.exe quick_tts.py "参考音频.wav" "音频里说的文字" "要合成的文字" 中文 中文 output.wav
```

## 常见问题

**端口被占用**：改为其他端口 `$env:VOICE_CLONE_PORT = "7861"`

**缺少模块**：确认已安装 `requirements.txt` 全部依赖

**CUDA 不可用**：PyTorch 可能被覆盖为 CPU 版，重装 `pip install torch torchaudio -f https://mirror.sjtu.edu.cn/pytorch-wheels/torch_stable.html`

**麦克风无权限**：浏览器地址栏左侧点击站点设置 → 允许麦克风


## 致谢

- [GPT-SoVITS](https://github.com/RVC-Boss/GPT-SoVITS) - 底层 TTS 框架
- [Gradio](https://gradio.app) - WebUI 框架

## License

MIT
