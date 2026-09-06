# 可读分镜稿模板

按此顺序输出，尖括号内容为模板变量，应以实际数据替换。数值和文字全部来自生成包，不在排版时补拍法。无适用内容明确写“无”；未知信息写清假设，不留空字段。结构化生成包同时交付。

## 项目概览

项目：<project_id>；交付状态：<status>；语言：中文分镜＋中文图像／视频提示词。

列出场次、风格／画幅决策、实体与资产版本、场景世界坐标和关键地标；列出影响执行的假设或缺失资产。给出 Beat→Shot 覆盖摘要；完整 Beats、Plan、状态和事件保存在生成包。

## SHOT <shot_id> · Spec v<spec_version> · <status>

| 项目 | 逐镜内容 |
|---|---|
| 场次／Beat | scene_id、beat_ids |
| 叙事目的与时长 | narrative_purpose、duration 秒 |
| 摄影 | shot_size、camera_angle、camera_height、camera_position、camera_direction、focus、visual_style、aspect_ratio |
| 构图 | composition 的前中后景、视觉层级、主体、遮挡、负空间及理由 |
| 角色 Blocking | 每人开始／结束世界位置、画面位置、朝向、视线、姿态、占比、遮挡和路径 |
| 动作与对白 | action_design 的秒数、顺序／并行、手别、接触、起止状态；对白原文 |
| 场景与道具 | environment、props、固定地标和物件当前状态 |
| 光线 | lighting 的来源、世界位置、方向、光质、色彩、补光、对比与变化 |
| 运镜 | camera_movement 的分段时间、起止、路径、速度、稳定性及动机 |
| 关键帧 | storyboard_keyframe 的秒数、用途、该时刻摄影／状态／构图 |
| 连续性 | 展开 entry／exit 状态关键内容；前后镜头、轴线、视线、动作匹配与转场 |
| 资产绑定 | entity_id、entity_version、asset_ids、view_ids、固定特征、状态覆盖 |
| 必须具备 | must_have，保留适用图像／视频及时间范围 |
| 禁止出现 | must_not_have，保留适用范围 |
| 假设与问题 | 本镜 assumptions 及关联 issues |

### 详细分镜描述

<中文：按时间顺序说明观众看到什么、空间与动作如何变化、摄影如何表达、镜尾如何承接；不只重复景别标签。>

### Image Prompt · 中文

分镜参考图：图<N>（本镜 Image Prompt 对应图号，N 沿用 Shot Plan 全局镜头顺序）；镜头：<shot_id>。关键帧：<time> 秒，<role>；Spec v<spec_version>。参考资产：<绑定清单，放在正文之外>。

```text
<image_prompt.zh_cn.text>
```

### Video Prompt · 中文

正文首行的分镜参考图号必须与本镜 Image Prompt 上方图号一致，且 shot_id、spec_version 与关键帧一致。时长：<duration> 秒；描述从镜头实际起点到终点。中途代表帧不作为完整视频的默认首帧。

```text
<video_prompt.zh_cn.text>
```

对所有镜头重复上述部分；不因镜数增加而省略字段。末尾如为 partial，明确剩余 Beat、next_shot_id 和续批输入，不将其标为完成。
