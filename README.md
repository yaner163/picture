# picture —— 《终幕之前》Anima 美术试验图集（私有）

本仓库为**私有**仓库，用于《终幕之前》的本地美术技术验证与后续 LoRA 训练素材整理。

- nima_test/references/：6 张参考原图（R04 为主参考；版权归原作者，仅内部对照）
- nima_test/generated/base/：Anima-Base v1.0 生成图（A/B × 2 种子，768×1152，30 步，CFG 4，无 LoRA）
- nima_test/generated/aesthetic/：Anima-Aesthetic v1.1 同参数生成图
- nima_test/comparison/：带标注的对比图
- nima_test/records/：参数与运行记录、ComfyUI API 工作流

## 训练数据注意
6 张参考图是拼版/带水印合集，**不能直接作为 LoRA 训练集**；需拆单图、去水印、去重，目标画风 60–120 张、角色 40–80 张。
