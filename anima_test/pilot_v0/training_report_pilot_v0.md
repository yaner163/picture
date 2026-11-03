# Anima LoRA 训练可行性报告 + pilot v0 结果

生成时间：2026-11-03 19:32–19:48（UTC+08:00）
机器：RTX 5070 Laptop 8 GB / 32 GB RAM / Windows
结论先说：**8GB 显存可以训练 Anima LoRA**（实测峰值 GPU 显存 3.29 GB，1.34 秒/步），pilot v0 已真实训练完成并用它出图，风格向 R04 有可见偏移。但 **pilot 数据只有 18 张从 6 张拼版参考切出的单图，不能当作最终画风 LoRA**；正式训练仍等 60–120 张单图。

## 1. 训练环境（隔离，未动 ComfyUI / Z-Image）

| 项目 | 值 |
|---|---|
| 训练器 | kohya-ss/sd-scripts（main 分支 tarball，含 `anima_train_network.py`、`networks/lora_anima.py`） |
| 代码位置 | `_work/train/sd-scripts/`（tarball 解压；跳过 `.ai/.github` 隐藏目录） |
| Python | 复用 ComfyUI standalone Python 3.13.12 + torch 2.12.1+cu130 |
| 依赖 | 装到任务私有目录 `_work/train/pylibs/`：accelerate 1.15.0、diffusers 0.40.0、voluptuous、imagesize、ftfy、wcwidth、importlib_metadata（PyPI 手动拉轮子解包；pip 的临时目录在本沙箱被拒，venv 的 ensurepip 也被拒） |
| 全局改动 | **无**。没有升级 ComfyUI 环境、没有改 E: 盘、没有动 Z-Image |
| 显存策略 | `--blocks_to_swap 22`（28 block 中换出 22 个）+ `--block_swap_h2d_only` + `--gradient_checkpointing` + bf16 + 缓存 latent / text encoder（落盘） |

## 2. 冒烟测试（30 步，8 张图，512 桶）

- 30 步耗时约 30 秒，**1.0 秒/步**；GPU 峰值显存 **2292 MiB**；无 OOM。
- LoRA 保存成功（rank 8，45.9 MB）。
- 证明训练脚本、权重加载（`net.` 前缀）、VAE/TE 缓存、block swap、保存格式全链路可用。

## 3. pilot v0 正式训练（600 步）

**数据**：6 张参考图 → 切成 18 张单图（R01/R02/R03/R06 各 4 格 + R04/R05 单张；拼版不直接进训练集）。
**字幕**：统一触发词 `r04style` + 画风/内容 tag 串；`num_repeats=8`（144 张/轮，5 轮）。
**桶**：配置 768；实际 512×896（128 张）、576×1024（16 张）。

| 参数 | 值 |
|---|---|
| network | lora_anima，rank 32，alpha 16，仅 DiT（`--network_train_unet_only`） |
| 学习率 | 2e-5，cosine，warmup 30 |
| 优化器 | AdamW（不用 bitsandbytes） |
| 精度 | bf16 |
| 采样器/时间步 | sigmoid，sigmoid_scale 1.0 |
| 步数 | 600，每 150 步存盘 |
| 耗时 | **13 分 26 秒**（19:32:06 → 19:45:34），≈ **1.34 秒/步** |
| GPU 峰值 | **3290 MiB**（nvidia-smi 3 秒采样），远低于 8 GB |
| loss | 约 0.18 起，末段 0.13 左右，无 NaN |
| 产物 | `pilot_out/anima_style_pilot_v0{,-step150/300/450/600}.safetensors`，每个 **183.6 MB**（fp32） |

**LoRA 转换**：`networks/convert_anima_lora_to_comfy.py` → `anima_style_pilot_v0_comfy.safetensors`（183.6 MB，ComfyUI 可加载）。

## 4. pilot v0 出图验证（真实推理）

- 用 ComfyUI 0.36.0 进程内 API 加载 Base + LoRA（strength 1.0），同参数（768×1152、30 步、CFG 4、euler/simple、denoise 1）跑 A/B × 2 种子共 4 张。
- LoRA 加载成功（0.0 s），出图正常。

**观感对比（相对 Base）**：

| 维度 | Base v1.0 | Pilot v0 LoRA |
|---|---|---|
| 光影 | 平光、整体偏亮 | 出现明显的侧光/明暗分组（尤其 B/97531864），更接近 R04 的戏剧光 |
| 材质 | 塑料高光、赛璐璐 | 高光收敛、发丝分层更多、脸部过渡更柔和 |
| 角色细节 | 耳饰/颈圈简单 | 耳饰带金边、颈圈纹样更复杂，胸花更接近蜘蛛百合 |
| 构图 | 换种子漂移 | 仍然漂移；LoRA 不解决构图 |
| 身份 | 眼罩方向不稳 | 仍不稳；pilot 数据未专门覆盖角色一致性 |

结论：**学习信号成立**，LoRA 确实把底模往 R04 的画风拉（至少是光影/细节层面）。但这只是 18 张图的 v0，**不能宣称画风已复现**：数据量不足、含拼版水印、字幕只有触发词，过拟合风险高。

## 5. 对 8GB 的实测结论

- 可以训练：600 步 13.5 分钟，峰值 3.3 GB，速度 1.34 秒/步。
- 2000 步约 **45 分钟**；3000 步约 **67 分钟**，本地算力足够。
- 显存还有余量：正式训练可以尝试 1024 桶 / 减少 `blocks_to_swap`（如 16–18）换速度，或加 batch（先用 1 稳妥）。
- 不能训练的只是“网页服务模式”，训练器本身是纯 CLI/库模式，不受回环限制影响。

## 6. 正式训练的改动清单（等你补数据即可开跑）

1. 数据：60–120 张单图（同一画师/画风，不同角色、构图、光照；去水印/文字/拼版），40–80 张角色图（第二轮）。
2. 字幕：触发词统一（如 `r04style`），每张写内容 tag + 一句自然语言；不要只写触发词。
3. 参数：rank 32、alpha 16、lr 2e-5（可试 1e-5–5e-5）、cosine、warmup 5%、1500–2500 步、每 250 步存盘、768–1024 桶、blocks_to_swap 18–22。
4. 评估：固定 A/B 提示词与 4 个种子，LoRA 强度 0 / 0.4 / 0.65 / 0.85，保留失败图；同时跑 Base 与 Aesthetic 两个底模验证迁移。
5. 数据到位前不继续堆步数：18 张图训练越久越过拟合，没有意义。

## 7. 交付文件

- `原始出图/`：PILOT-A/B × 2 种子（4 张）
- `对比图_Base对Pilot.png`、`对比图_目标与Pilot.png`
- `anima_style_pilot_v0_comfy.safetensors`（ComfyUI 用）、`anima_style_pilot_v0_sdscripts.safetensors`（训练格式）
- `Pilot出图参数与运行记录.json`
- 训练日志：`_work/logs/`、`_work/train/pilot_logs/`（工作区内）

## 8. 说明与边界

- 参考图版权归原作者（pixiv 128661355），仅个人非商业试验；LoRA 不公开发布。
- 训练数据来自拼版切图，pilot 仅验证管线，不代表最终质量。
- 本轮**没有**启动付费云 GPU、没有改全局依赖、没有动其他项目。
