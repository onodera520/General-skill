# 连续性：共享状态驱动逐镜设计

从 Shot Planning 起读取本文件，并贯穿 Spec 定稿。这里约束拍摄设计，不评审实际生成媒体。结构权威为 [output-schema](../templates/output-schema.json) 的全局档案、continuity_ledger 和 [shot-schema](../templates/shot-schema.json) 的 continuity。

## 状态权威与引用

- entity_registry 保存身份与固定外观；scene_registry 保存稳定世界布局。
- continuity_ledger.states 保存完整场景状态快照，state_id 唯一。每镜用 entry_state_id、exit_state_id 引用，不再复制另一份独立可改的起止状态。
- continuity_ledger.events 记录改变状态的事件：所属镜头、镜内秒数、实体、字段、before／after、剧本或执行细节依据。
- storyboard_keyframe.state 是指定时刻的完整快照，由 entry 状态和截至该时刻的事件推导；不是另一条剧情线。

连续镜头优先令下一镜 entry_state_id 直接复用上一镜 exit_state_id。同值的独立快照只允许在明确需要独立 ID 时使用，并逐字段比对。快照中的 visibility 按对应镜头更新是允许的投影差异，不能因此改变世界位置或持物。若复用状态而切镜改变 visibility，建立新快照并明确这是观察变化，持续物理字段必须相同。

完整快照保留本场已建立且影响连续性的所有角色、道具、门窗状态和光线，包括离画者、被遮挡的持物和遗留衣物。不可只列当前可见的人；没有变化的值仍继承，不用空数组代替“沿用”。

快照只在 ledger.states 和指定的关键帧 state 中结构化保存；action_design.state_before／state_after 写该动作涉及的局部状态短句，asset_bindings.state_overrides 写相对固定身份的穿着／持物等实际变化。不要将完整快照序列化成字符串塞入这些字段，否则会复制状态权威、掩盖重点并膨胀输出。

## 逐镜算法

1. 首镜从剧本与全局档案建立入口状态；连续后镜继承前镜出口。
2. 编排动作与对白区间。各状态变化产生事件；区间中间状态需要可执行的接触、握持和路径描述。
3. 按事件顺序推导出口状态；根据关键帧时间推导该时刻状态和投影，镜内动作未发生不能提前进入图像。
4. 检查下一镜的入口、机位与关注点是否能承接；不成立则在设计阶段修订。
5. 场景结束仍保留状态；角色再出现或回到旧场景时继承最近的相关状态。显式时间跳跃只允许有依据的变化，不自动让道具归位、伤口消失或衣物穿回。

## 必须追踪的细节

人物：世界位置、身体朝向、视线目标、姿态、左右手任务、服装／湿污／伤势。道具：唯一实体、持有者与身体自身手别、接触关系、世界位置、损坏／开关状态。场景：门窗开闭、家具位置、出入口路线、实际光源及环境变化。

props[].holders 是持物权威；characters[].hand_activity 描述动作但不得与之冲突。交接接触阶段可有两只手共同接触同一 PROP_，必须注明谁仍握持、谁仅承接；释放后清除原持有者。双手持杯用 hand=both。穿着衣物作为道具追踪时可用 hand=other、contact=穿着于躯干；脱下后改为实际手或无持有人，不能复制新外套。

## 轴线与投影

axis.definition 指明轴线由谁与谁或哪条运动路径构成；camera_side 使用世界参考定义。screen_direction 记录这一镜的投影。反打时重新计算投影，不能宣称“所有镜头中西门永远在画面左边”。人物移动改变轴线时重新建立关系；有意越轴在 crossing_justification 写出可见交代，不把越轴一概禁掉。

camera_direction 是镜头从哪里朝向哪里；character_blocking.facing 是人物身体朝向；gaze 是眼睛看向谁／什么。三者不得用同一个“朝左”含混代替。

## 转场、修订与续批

transition.kind 区分 continuous、time_ellipsis、scene_change、flashback、parallel、other。对非连续剪辑说明实际时间／空间关系及 source_beat_ids；事件需要新增执行细节时标 basis=execution_detail，不伪造剧本依据。闪回／平行线从对应时空的状态起步，不继承剪辑前另一条线的物理状态。

当前契约用 transition.explanation 保留剧情时间差的原文及可确定的换算，例如“十分钟后；剧情经过600秒”，scene_id 区分相应叙事场次。该值不是本镜 duration；不要把600秒误写成屏幕时长，也不要捏造未建立的绝对时刻。

修订前镜后沿状态依赖传播，直到后镜入口、动作和出口不再受影响；重新计算相关关键帧、绑定展开、版本与 Prompt。不可只修文本而留下旧的 ledger。续批保存全部档案、最近时空状态、镜头版本、已覆盖与未覆盖 Beat，不能凭上批摘要重建人物。

## 定稿门槛

引用存在且类型对应；事件时间、动作／运镜区间、关键帧均在 0..duration 内；先后依赖不成环；接触／释放符合时间；角色不能无路径瞬移；一件道具不无依据同时在两处；门窗、桌子与光源不随机换位；关键帧状态与时序一致。

Clip 边界继续使用同一状态链，依据 [Clip 分组](clip-planning.md) 检查场景入口全部在场人物、单人首镜和声音不重复。必要承接状态可重述，已发生事件不能重播；闪回和换场仍按对应时空处理。

这些跨记录语义不能仅靠 JSON Schema 证明。发现问题由拥有该决策的设计阶段修复；不调用 Reviewer，不生成或验收媒体。未消解输入冲突保留 provisional 与具体 issues，不无限循环修订，也不隐瞒冲突。
