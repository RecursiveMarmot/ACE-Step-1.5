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
2. 执行上述启动命令。
3. 终端显示 `Running on local URL: http://127.0.0.1:7860` 后，在浏览器中打开该地址。
4. 输入音乐描述，点击**创建样本**后点击**生成音乐**即可。
