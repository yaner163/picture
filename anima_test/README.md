# anima_test —— 《终幕之前》Anima 美术试验图集

本目录为《终幕之前》Anima 底模/版本对照的图片归档（私有仓库，仅个人非商业研究用）。

## 目录

- `references/`：用户提供的 6 张参考原图（R04 为主参考；版权归原作者，仅内部对照）
- `generated/base/`：Anima-Base v1.0 生成图（A/B 提示词 × 2 种子，768×1152，30 步，CFG 4，无 LoRA）
- `generated/aesthetic/`：Anima-Aesthetic v1.1 同参数生成图
- `comparison/`：带标注的对比图（A/B、与 R04、Base vs Aesthetic）
- `records/`：参数与运行记录、ComfyUI API 工作流（复现用）

## 命名

- `A-` 用户原始标签；`B-` 追加鼻部结构描述
- `24681357` / `97531864` 种子；`AE-` Aesthetic v1.1

## 训练数据注意

6 张参考图是拼版/带水印/带字合集，**不能直接作为 LoRA 训练集**；需拆单图、去水印、去重，目标画风 60–120 张、角色 40–80 张。
