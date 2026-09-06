# Video Prompt 模板

输入只读：Spec 起始摄影字段、入口／出口状态、action_design、camera_movement、lighting、asset_bindings 和视频约束。按 [转写规则](../references/prompt-building.md) 生成同义的中英正文。

## 中文正文顺序

1. 镜头时长、起始景别与机位、环境及光源；逐人交代稳定外观、起始位置、朝向、姿态和持物。
2. 按 Spec 的相对秒数描述动作阶段、对白、接触／释放和状态变化；并行动作明确“同时”，手别按角色自身。
3. 对齐同一秒轴说明摄像机起止、路径、速度、稳定程度和焦点变化；固定镜头明确全程固定。
4. 说明结束时人物位置、持物、衣物、道具及环境状态，承接下镜所需信息。
5. 指明全程保持的身份／场景特征与有意发生的变化；落实必须发生和禁止出现的内容。

## English body order

1. Duration, starting shot size and camera, environment and lighting; stable appearances and initial positions, orientations, poses and prop ownership.
2. Timestamped action phases, original dialogue, contact/release and state changes. Preserve simultaneity and anatomical hand identity.
3. Camera trajectory, speed, stability and focus changes on the same time axis. State when the camera is locked.
4. End positions, prop ownership, wardrobe and environment states needed for the next cut.
5. Persistent identity/layout constraints, intentional changes, required events and exclusions.

中英两版均保留剧本对白原文，不能让角色改说另一种语言。不开列软件配置，不新增音效或配乐。文字中的“开始”必须是镜头真实 t=0，不是中途代表帧。示例：0–1 秒保持持钥匙起态，1–2 秒右手到另一人的左手交接，2–4 秒承接者握紧、原持有人手空；不得重新播放递出动作或在视频末尾自动复原道具。
