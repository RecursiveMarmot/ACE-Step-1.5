# ACE-Step 1.5 启动指南

本文档包含针对 RTX 4060 Laptop（8GB 显存）优化的 ACE-Step 1.5 启动命令与参数说明。

## 推荐启动命令

在 PowerShell / 终端中运行以下命令：

```powershell
$env:ACESTEP_SAVE_MEMORY="1"
uv run acestep --lm_model_path acestep-5Hz-lm-0.6B --backend pt --quantization none --offload_to_cpu true --offload_dit_to_cpu false --batch_size 1 --language zh
```

## 参数说明

| 参数 | 值 | 说明 |
| :--- | :--- | :--- |
| `$env:ACESTEP_SAVE_MEMORY` | `"1"` | 开启省显存模式，减少不必要的中间张量缓存 |
| `--lm_model_path` | `acestep-5Hz-lm-0.6B` | 使用针对 6-8GB 显存优化的 0.6B 轻量语言模型 |
| `--backend` | `pt` | 使用原生 PyTorch 后端，避免 Windows 下 vLLM 的 CUDA Graph 冲突 |
| `--quantization` | `none` | 禁用 INT8 量化，避免 torchao 算子在 Windows 环境下崩溃 |
| `--offload_to_cpu` | `true` | LM 语言模型生成完 Token 后释放显存，把显存留给渲染引擎 |
| `--offload_dit_to_cpu` | `false` | 将 4.5GB 的 DiT 主模型常驻显存，避免频繁跨总线搬运导致崩溃并大幅提升速度 |
| `--batch_size` | `1` | 锁定单次生成 1 首歌曲，防止 8GB 显存溢出 |
| `--language` | `zh` | 将 Web 界面语言设置为中文 |

## 使用步骤

1. 打开终端，进入项目目录：
   ```powershell
   cd D:\learning\ace\ACE-Step-1.5
   ```
2. （可选）提前下载模型文件（见下方“模型文件下载”）。
3. 执行上述启动命令。
4. 终端显示 `Running on local URL: http://127.0.0.1:7860` 后，在浏览器中打开该地址。
5. 输入音乐描述，点击**创建样本**后点击**生成音乐**即可。

## 模型文件下载

ACE-Step 1.5 在首次运行时会**自动检测并下载**缺少的模型文件（默认保存至 `./checkpoints` 目录）。如果不希望在启动时等待自动下载，或由于网络原因需要手动/提前下载，可使用以下方法。

### 方法一：使用内置 CLI 工具下载（推荐）

ACE-Step 提供了便捷的 `acestep-download` 命令行工具：

1. **国内加速下载（推荐使用 ModelScope 源）：**
   ```powershell
   # 下载主模型包（包含 DiT Turbo、VAE、Embedding 等）
   uv run acestep-download --download-source modelscope

   # 下载 8GB 显存推荐的 0.6B 轻量语言模型
   uv run acestep-download --model acestep-5Hz-lm-0.6B --download-source modelscope
   ```

2. **从 Hugging Face 源下载：**
   ```powershell
   # 下载主模型
   uv run acestep-download --download-source huggingface

   # 下载 0.6B LM 模型
   uv run acestep-download --model acestep-5Hz-lm-0.6B --download-source huggingface
   ```

3. **常用 CLI 下载指令：**
   ```powershell
   uv run acestep-download --list          # 查看所有可选模型列表
   uv run acestep-download --all           # 一键下载所有可用的模型（占用空间较大）
   ```

### 方法二：使用 Python / 命令行手动下载

如果你已安装 `modelscope` 或 `huggingface-cli`，可以直接通过 CLI 方式独立下载：

- **ModelScope 下载命令（国内极速）：**
  ```powershell
  # 主模型
  modelscope download --model ACE-Step/Ace-Step1.5 --local_dir ./checkpoints

  # 0.6B 语言模型
  modelscope download --model ACE-Step/acestep-5Hz-lm-0.6B --local_dir ./checkpoints/acestep-5Hz-lm-0.6B
  ```

- **Hugging Face CLI 下载命令：**
  ```powershell
  # 主模型
  huggingface-cli download ACE-Step/Ace-Step1.5 --local-dir ./checkpoints

  # 0.6B 语言模型
  huggingface-cli download ACE-Step/acestep-5Hz-lm-0.6B --local-dir ./checkpoints/acestep-5Hz-lm-0.6B
  ```

### 核心模型与推荐说明

| 模型名称 | 路径/标识 | 说明 | 建议使用场景 |
| :--- | :--- | :--- | :--- |
| **Ace-Step1.5 主模型** | `ACE-Step/Ace-Step1.5` | 核心仓库，含 `vae`, `acestep-v15-turbo`, `Qwen3-Embedding` | **必选**（所有模式均需此模型） |
| **acestep-5Hz-lm-0.6B** | `ACE-Step/acestep-5Hz-lm-0.6B` | 轻量级 0.6B 歌词/结构语言模型 | **8GB / RTX 4060 推荐**（省显存、加载快） |
| **acestep-5Hz-lm-1.7B** | `ACE-Step/acestep-5Hz-lm-1.7B` | 标准 1.7B 语言模型 | 12GB+ 显存用户推荐 |
| **acestep-5Hz-lm-4B** | `ACE-Step/acestep-5Hz-lm-4B` | 4B 大语言模型 | 24GB+ 显存旗舰显卡使用 |

### 修改模型保存位置（可选）

如需将模型保存到其他盘符（如磁盘空间更大的分区），可以通过设置环境变量 `ACESTEP_CHECKPOINTS_DIR`：

```powershell
# 在 PowerShell 中临时指定模型目录
$env:ACESTEP_CHECKPOINTS_DIR="E:\models\ace-step-checkpoints"
```

