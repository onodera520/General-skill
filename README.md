# Smart Storyboard Generator

`generator` 是一个面向 Codex 的分镜设计 Skill。它将剧本和角色、场景、道具等视觉资产，转化为可执行的逐镜设计与中英双语生成提示词。

## 输出内容

每个镜头包含：

- 高精度 Shot Spec：景别、机位、摄影方向、构图、前中后景、Blocking、动作时序、环境、道具、光线、运镜与连续性。
- 中文详细分镜描述。
- Universal Image Prompt：中文与英文各一版，严格对应一个指定关键帧。
- Video Prompt：中文与英文各一版，描述镜头起止状态、动作时序与摄影机运动。

它只负责设计镜头与转写提示词，不生成或验收图片、视频。

## 核心流程

```text
剧本 + 视觉资产
        ↓
Story Beats
        ↓
Shot Planning
        ↓
Shot Spec 定稿
        ↓
分镜描述 + Image Prompt + Video Prompt
```

Prompt Builder 只从已定稿的 Shot Spec 转写，不能改变景别、构图、人物位置、手别、动作、光线或运镜。

## 连续性规则

Skill 在镜头规划阶段维护跨镜头状态：

- 人物固定身份、服装和状态由共享资产档案管理。
- 场景门窗、家具和光源使用固定世界位置；反打只改变画面投影。
- 人物位置、朝向、视线、持物、左右手、道具交接与损坏状态都在镜头入口和出口之间继承。
- 静态 Image Prompt 只写关键帧时刻；Video Prompt 写完整动作过程，避免把不同时间阶段混为一帧。

## 使用方式

在 Codex 中使用 `$generator`，并提供：

1. 剧本、场次或剧情段落。
2. 角色、场景和道具的资产图，或明确的文字资产设定。
3. 可选的风格、画幅、时长、镜头数和硬约束。

纯文字资产会标注为用户提供的信息；无法读取但要求匹配的图片会明确记录为未验证，而不是虚构观察结果。

## 项目结构

```text
generator/
├── SKILL.md
├── references/   # 剧本解析、拆镜、摄影、调度、连续性与提示词转写规则
├── templates/    # 输入、中间产物、Shot Spec 和最终输出的数据契约
└── evals/        # 场景、对话、动作、资产歧义与 Spec→Prompt 保真评测
```

## 验证

项目包含 JSON Schema、跨镜状态契约和行为评测用例。验证记录见 [generator-verification.md](generator-verification.md)。
