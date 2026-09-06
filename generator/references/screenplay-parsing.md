# 剧本解析：先确定发生了什么

## 读取时机与产物

流程第一阶段。输入为 [input-schema](../templates/input-schema.json) 的 screenplay，输出遵循 [story-beats-schema](../templates/story-beats-schema.json)。不决定景别、机位、构图、灯光、镜头数量或 Prompt；原文明确的摄影要求保存在 inherited_camera_requirements，交给后续设计。

## 解析方法

按场次定位时间、地点、人物、道具、进出场、对白、行动、环境变化与信息揭示。source_id 对应原始段落，locator 记录场次／行号／段落位置，text 保留对应原文，不能用自己的概括替代证据。场次改变不一定改变物理场景档案：同一厨房的早晚场次应引用同一场景实体，保留不同 scene_id 的时间状态。

每个 Atomic Story Beat 是一个对叙事有意义的动作或状态变化，描述 subject_ids、action、object_ids、state_before、state_after。原文对白逐字保留，不能增写心理独白、台词或剧情解释。情绪有依据才提取；没有时用 null，不将推测动机记为事实。关键 Beat 标 essential；气氛或支撑信息可标 supporting。

“挥刀刺向林川，林川侧身躲开，同时抓住对方手腕”拆出刺、躲、抓三个 Beat；用 temporal_relations 表示躲与抓重叠。不能改成顺次完成，也不能压缩为“双方打斗”。同一动作的纯修辞重述不再造一个 Beat。长动作可在 Shot Plan 中跨镜，但不要为切镜伪造新的剧情事件。

required_visual_information 写观众必须获得的信息，例如“钥匙已经从右手交到另一人的左手”，而不是预先指定“手部特写”。对白、画外声音或明确时间省略也可承担信息，不能强制所有 Beat 都成为屏幕内动作。

## 边界情况

- 进入／离开区分场景边界与画面边界；离开画面仍可能留在场景。
- 相邻段落的动作关系不清晰时采用最少必要时间假设，记录到生成包 assumptions，不改源文。
- 剧本漏写可执行细节（开门、绕桌、衣袖释放）不补为源文 Beat；后续 action_design 标 execution_detail，并挂到所服务 Beat。
- 显式闪回、换场、十分钟后分别保留 transition Beat；省略不等于允许任意改变道具归属或人物身份。

完成门槛：每个事实性 Beat 可定位到原文；所有关键状态变化和对白均有承载；ID 唯一且续批稳定；时间关系无因果循环。随后读取 [资产绑定](asset-binding.md) 和 [镜头规划](shot-planning.md)。
