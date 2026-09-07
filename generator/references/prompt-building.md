# Prompt Builder：只读转写，不重新导演

读取已定稿或明确provisional的时间线、Clip设计及其引用的 [Shot Spec](../templates/shot-schema.json) 和状态。默认按 [MD模板](../templates/storyboard-output-template.md) 保存一个中文文件，每个Clip只有详细镜头描述、逐Shot Image Prompts、一份整体智能镜头 Prompt（原Video Prompt）。内部Shot文字绑定shot_id+spec_version；视频绑定clip_id+clip_version与source_shots，不生成英文版本。

仅明确请求结构化导出时使用 [output-schema](../templates/output-schema.json) v2.0：shots保留spec、storyboard_description、image_prompt.zh_cn；clips保留clip_id、clip_version、video_prompt.zh_cn。旧shots[].video_prompt不再输出。兼容现有契约，内部clips[].video_prompt字段名不改；对外标题统一为“智能镜头 Prompt”。Clip正文的source_paths指向整个包的真实JSON Pointer，source_shots必须精确列出本Clip全部Shot及版本；Shot文字仍用相对本镜Spec的spec_paths。默认不展示这些内部字段，也不构造完整JSON。

## 权限与映射

Prompt Builder 不直接从剧本构思，不重新查看资产添加新特征，不从摄影 reference 临时选另一种拍法。完整生成流程必须先有Beats、定稿时间线、Clip分组、Shot Plan与Spec。用户仅提交已定稿设计要求转写时接受来源，不反向捏造Beats。只给一个Shot时可按其既定时长机械封装为一个Clip；不补拍法或假称完整规划。多个Shot缺少明确区间、声音或分组且需重新决策时返回设计阶段；已有自然语言来源保留源文定位，不伪造JSON路径。

两种入口的完整性要求不同：完整生成包必须满足 Schema 所有必填字段；只读转写已有自然语言 Spec 时，忠实转写已提供的摄影与视觉事实，未给的画幅、焦点或光质可不写，必要时在该镜详细描述中简短注明。若缺失或矛盾会令主体身份、动作主体／对象、左右手、关键帧时刻或起止状态存在互斥解释，则该输出不能冒充可执行成品，返回设计阶段；不以自动补参数填满 Schema。Prompt Builder 发现输入问题只在该镜详细描述中简短注明待解决，结构化导出时写入 issues，不修改原 Spec 的 status、版本或内容。

| 输出内容 | 唯一取值来源 |
|---|---|
| 人物固定形象与动态状态 | 已清晰绑定角色使用 asset_bindings 的对应资产引用并声明严格继承，不展开 identity_traits；无清晰绑定时使用已确认 identity_traits，动态状态读取适用的 state_overrides，规则见 [资产绑定](asset-binding.md) |
| 道具外观与状态 | asset_bindings.identity_traits，结合同一 Spec 的 state_overrides |
| 风格与画幅 | visual_style、aspect_ratio |
| 静态景别、机位、焦点 | storyboard_keyframe.camera_state |
| 静态人物位置、动作、视线、持物 | storyboard_keyframe.blocking 与 state |
| 每个可见人物相对于镜头的身体朝向，以及有差异的头脸朝向 | storyboard_keyframe.blocking[].facing，由调度阶段按关键帧机位确定 |
| 静态前中后景 | storyboard_keyframe.composition |
| 静态环境与光线 | environment 及 keyframe.state 中该时刻的状态 |
| 视频起始摄影、角色／道具状态 | 顶层摄影字段与 continuity.entry_state_id 对应状态 |
| 视频动作／镜内变化 | 各Shot的action_design、camera_movement、lighting.changes、相关events，内部按shot_span.start换算，正文按镜号写先后过程，不输出起止秒数区间 |
| 视频分镜参考图注释 | Clip.shot_spans对应的全部Shot与image_prompt版本、全片图号和各storyboard_keyframe.time／role |
| 视频台词／旁白／音效 | timeline.audio_events及Clip.audio_spans；原文片段与起止时间精确对应，不由各Shot重复播放 |
| 视频场景起态与全局补充 | Clip.scene_entries关联状态、visual_style、global_details |
| 视频结束状态 | continuity.exit_state_id、最后运动段和 Blocking.end |
| 硬约束 | must_have／must_not_have 的 applies_to 与 time_range |

完整生成包中 Shot文字的spec_paths是实际使用字段的JSON Pointer列表，根为该镜 spec，例如 /storyboard_keyframe/state、/asset_bindings、/duration。每个路径必须能在实际对象中解析，禁止写不存在的 /duration_seconds。它用于追踪而不是质量证明。入口／出口状态通过 /continuity/entry_state_id 等路径追溯到 ledger。只读自然语言 Spec 没有 JSON 对象时，在工作记录中保留源文片段／段落定位，不伪造 spec_paths；该转写交付不套完整生成包 Schema。版本、内部 shot_id 和资产绑定清单保留在工作记录中，默认不对外展开；智能镜头 Prompt在每个镜头段前列对应图号与关键帧用途；需要标时只写单点关键帧时间。全片镜头编号续批不重置。

详细镜头描述按Clip串联其各Shot的storyboard_description，覆盖所有镜头的起态、动作时间线、摄影变化与终态，并保留关键帧对应；不能只描述关键帧而漏掉交接等后续动作。Image Prompt 才是单一时刻的画面描述。

## 中文正文

中文正文从同一个 Spec 事实清单展开，使用“画面右侧前景、角色自身左手”等明确术语。方向、手别、人数、道具数、否定项、秒数、景别、光源与动作先后必须与 Spec 一致，不能为“电影感”添加浅景深、烟雾、手持或逆光。

清晰绑定的角色按 [资产绑定](asset-binding.md) 仅写角色与参考图的明确对应及严格继承声明，不重复固定外貌与服装；资产不足的角色才使用已确认的必要中文形象描述。对白保持剧本原文，中文提示词不意味着翻译或改写角色实际台词。新写画面文字使用具体名字／稳定标签，禁止代词替代角色；原台词中的代词不改。每个 Prompt 包含独立可用的本镜指令和明确资产引用；清晰绑定时与对应资产图配合使用，不为“自包含”再次列举固定形象，也不能写“same as previous shot”“同上”要求软件读取上一镜。

## 静态约束与视频约束

Image Prompt 用 [图像模板](../templates/image-prompt-template.md) 描述一个静态瞬间，不写完整动作时间线。视频全镜 must_have 如果在关键帧尚未发生，就不能强行进入图像；仅 applies_to 包含 image 且 time_range 覆盖关键帧的约束应用于图像。time_range=null 表示对对应输出全程适用。发现某个 image 约束时间范围不含关键帧时，返回设计阶段明确适用范围，不能悄悄丢掉冲突。

智能镜头 Prompt 按 [智能镜头模板](../templates/video-prompt-template.md) 一次覆盖完整Clip：必要风格与资产／站位、逐镜参考图注释、逐镜编号和单独时长、详细画面、连续声音与结尾生成要求。正文及详细镜头描述不输出“0–2秒”等起止区间；需要独立镜号的动作阶段必须先由设计阶段拆成真实Shot并配套Image Prompt，Builder不能把一个Spec偷偷拆成多个镜号。镜内自然连续的小幅动作使用先后词，不因省略区间而漏动作。保留所有适用约束；单Shot限制只作用于其Clip内区间，不能扩大到其他镜头。声音和图像时间轴分别描述但相互对齐。长对白切镜不重播；无原文旁白不新增V.O.。具体表演来自已定稿action_design，只有抽象情绪而缺少必要可见表现时返回设计阶段，不由Builder临时添加。

must_not_have 用自然语言“不要新增第三人”，保留具体风险，避免冗长通用负面词库。不使用 --ar、LoRA、权重括号、专属 reference token、采样器或软件配置；画幅若已在 Spec 中确定，以自然语言表述。不能承诺某个平台必定执行。

## 转写检查与返回机制

先形成字段事实清单，再写三类文字；清晰绑定角色核对参考图指向准确、继承声明齐全、固定特征没有重复展开或与资产冲突，本镜动态状态与朝向没有因省略形象而丢失。若资产与 Spec 固定形象冲突，按资产绑定规则返回设计阶段，不能以忠实转写为由复制冲突文字。然后比对：Spec 版本、角色身份、画面位置、手别、道具、关键帧状态、光线、运镜、时间区间与正文语言。逐Clip核对全部Shot图号和版本、局部／全局时间换算、声音区间原文覆盖及scene_entries全员站位；Clip输出source_paths引用实际使用的Clip、timeline和Shot字段，不能把Shot相对spec_paths混作包根路径。再检查智能镜头Prompt逐镜时长合计等于Clip时长、镜号与Image Prompt一一对应、没有起止秒数区间；声音使用<>、台词使用{}且只在首次声音条目出现一次；全局补充每条确实贯穿Clip，逐镜光向／焦点和出口状态归回对应画面段。对中文 Image Prompt，按关键帧实际可见人物逐一核对是否明确写出相对于镜头的身体朝向，并保留头脸与身体的朝向差异；前景、背景及局部遮挡但仍可见的人物均不能遗漏，完全画外或完全遮住的人物不因此加入画面。仅写“画面右侧”“朝西”“看着甲”不满足此项；侧身也不等于直视镜头。若缺少该关系，沿用下述返回设计阶段的机制，Prompt Builder 不自行选择朝向或改变机位；裁切或遮挡不因补写朝向而改变。可删重复形容词，不可删必要事实。任何新摄影决定都意味着越权。

若发现 Spec 缺项或互相矛盾，记录冲突字段及影响，返回其设计阶段修订 Spec、ledger 和版本，再重建受影响的Shot图像／描述及整段Clip视频。provisional Spec 的已记录假设可忠实转写，交付层保留草案状态；不能把未解决的两个互斥状态都塞进同一个 Prompt。
