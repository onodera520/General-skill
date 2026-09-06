# Blocking 与可执行动作

读取场景空间档案、镜头入口状态和 [摄影语言](cinematic-language.md)，填写 character_blocking、action_design 以及关键帧 blocking。结构遵循 [shot-schema](../templates/shot-schema.json)。

## 三种关系分开写

- world_position：相对固定地标的位置，角色在镜间物理连续性的依据。
- screen_position／depth：当前摄像机下的画面投影；按观众视角定义左右与前中后景。
- facing／gaze：身体朝向和眼睛的目标分别记录。看门不要求身体已经转向门，转头也不等于整个人换位。对关键帧中的每个可见人物，调度阶段根据该时刻的人物位置、身体朝向和实际机位，在 storyboard_keyframe.blocking[].facing 中保留世界朝向，并明确相对于镜头呈正面、侧面、背面或四分之三正面／背面；头脸与身体朝向不一致时分别说明。相机运动时使用关键帧机位，不沿用起始机位；此补充不改变状态账本的世界朝向。

每人填写镜头开始和结束 placement、movement_path。画面占比用 frame_height_fraction 表示设计估计，范围 0..1；出画者用 null，depth=offscreen，不胡写可见占比。occlusion 指明被谁或什么遮挡、遮哪部分；关键动作参与手部不能被肩膀、桌沿或其他角色挡住。

例：阿岚在桌东侧（世界位置），画面右侧中景，身体朝西，视线看林川右手，左掌向上准备承接；右手空闲。比“阿岚站右边，气氛紧张”更能执行。

## 动作时间线

action_design 为每个动作给出镜内 start/end 秒、主体、对象、接触、路径、前后状态、depends_on 和 overlaps_with。对白另用 kind=dialogue，原文放 dialogue，delivery 描述说话者／画内外和语速。没有对白用 null。

起止秒数非负且 start < end <= duration。静止、停顿也可作为执行阶段，避免时间空洞导致模型无依据乱动。depends_on 指必须完成的前置 action_id；并行动作用 overlaps_with，且区间真实相交。不能同时把一个动作设为另一个的已完成前置与重叠动作。

将剧本中有依据的抽象情绪落实为当前景别可见的表情、视线、姿态或动作节奏，写入 action_design.description、状态与 placement.pose／gaze；注明属于设计的 execution_detail。例如“无助”可根据情境设计为“肩膀下垂，抬眼看向对方，嘴唇张开又合上”，不是固定要求跪地或双手抱头。表现强度、占用的手、身体姿态、用时及前后状态须与剧情和持物一致，不新增伤势、哭喊或无依据的大幅动作。Prompt Builder 只转写这些已定稿的可见表现。

故事动作标 story_action，服务执行的开门、让路、调整握持标 execution_detail。执行细节挂到相关 beat_ids，不新增冲突剧情。重要状态改变另登记 ledger.events；同一事件跨镜时只发生一次。

## 接触与手别

所有左右手按角色自身。交接写清“林川右手仍握 → 阿岚左掌接触 → 阿岚握紧 → 林川释放”，接触可短时重叠但实体数量只有一件。动作的描述与 props.holders、hand_activity 一致。

一手被钥匙占用时，脱外套必须设计可完成的衣袖退出路径；若需要暂时放下或换手，写入时间线和状态事件，然后按剧情恢复目标状态，不能省略持物变化。双手持杯时不另安排同一只手开门，除非先说明释放或其他可执行方式。

人物离画后沿世界路径继续，角色再次入画必须能从先前位置到达。路径避开固定障碍，不能穿过桌子、封闭门或彼此身体。相机移动造成的屏幕位移不写成角色移动。

关键帧 placement 从该时刻的动作阶段推导，不把起始 standing 姿态套到结束 walking 状态。提交 [构图](composition.md) 统合前中后景和视觉焦点。
