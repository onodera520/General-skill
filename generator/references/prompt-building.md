# Prompt Builder：只读转写，不重新导演

读取已定稿或明确标注 provisional 的 [Shot Spec](../templates/shot-schema.json)，以及它引用的只读入口／出口状态。使用 [output-schema](../templates/output-schema.json) 封装输出：storyboard_description，image_prompt.zh_cn，video_prompt.zh_cn。每份文字包含 shot_id、spec_version、language、text、spec_paths。三类文字均使用 zh-CN，不生成英文版本。

## 权限与映射

Prompt Builder 不直接从剧本构思，不重新查看资产添加新特征，不从摄影 reference 临时选另一种拍法。完整生成流程必须先有可追溯的 Beats、Shot Plan、Spec。用户仅提交已定稿 Spec 要求转写时，接受其设计来源，不反向捏造 Beats；只交付转写内容，不声称产出了完整生成包。

两种入口的完整性要求不同：完整生成包必须满足 Schema 所有必填字段；只读转写已有自然语言 Spec 时，忠实转写已提供的摄影与视觉事实，未给的画幅、焦点或光质可不写，并在交付说明中列出。若缺失或矛盾会令主体身份、动作主体／对象、左右手、关键帧时刻或起止状态存在互斥解释，则该输出不能冒充可执行成品，返回设计阶段；不以自动补参数填满 Schema。Prompt Builder 发现输入问题只在独立交付说明／issues 中标明待解决，不修改原 Spec 的 status、版本或内容。

| 输出内容 | 唯一取值来源 |
|---|---|
| 人物／道具固定外观 | asset_bindings.identity_traits，结合同一 Spec 的 state_overrides |
| 风格与画幅 | visual_style、aspect_ratio |
| 静态景别、机位、焦点 | storyboard_keyframe.camera_state |
| 静态人物位置、动作、视线、持物 | storyboard_keyframe.blocking 与 state |
| 每个可见人物相对于镜头的身体朝向，以及有差异的头脸朝向 | storyboard_keyframe.blocking[].facing，由调度阶段按关键帧机位确定 |
| 静态前中后景 | storyboard_keyframe.composition |
| 静态环境与光线 | environment 及 keyframe.state 中该时刻的状态 |
| 视频起始摄影、角色／道具状态 | 顶层摄影字段与 continuity.entry_state_id 对应状态 |
| 视频动作／对白／镜内变化 | action_design、camera_movement、lighting.changes、相关 events |
| 视频环境声与音效 | environment.atmosphere 中明确的声音说明、action_design.description／delivery 中已设计的声音与时间；不从画面物件推测 |
| 视频分镜参考图注释 | 本镜 image_prompt.zh_cn 的 shot_id／spec_version 配对关系、共享全局图号，及 storyboard_keyframe.time／role；实际图像引用仅来自已提供的文件 |
| 视频结束状态 | continuity.exit_state_id、最后运动段和 Blocking.end |
| 硬约束 | must_have／must_not_have 的 applies_to 与 time_range |

完整生成包中 spec_paths 是每份文字实际使用字段的 JSON Pointer 列表，根为该镜 spec，例如 /storyboard_keyframe/state、/asset_bindings、/duration。每个路径必须能在实际对象中解析，禁止写不存在的 /duration_seconds。它用于追踪而不是质量证明。入口／出口状态通过 /continuity/entry_state_id 等路径追溯到 ledger。只读自然语言 Spec 没有 JSON 对象时，在正文之外引用源文片段／段落定位，不伪造 spec_paths；该转写交付不套完整生成包 Schema。版本、内部 shot_id 和资产绑定清单保留在正文外；Video Prompt 按模板在正文首行保留分镜参考图号与关键帧时间／用途，随后保留供用户阅读的“镜头N”顺序编号，由 Shot Plan 的全局顺序对应到 shot_id，续批不重置。

详细分镜描述覆盖整个镜头的起态、动作时间线、摄影变化与终态，并说明所选关键帧；不能只描述关键帧而漏掉交接等后续动作。Image Prompt 才是单一时刻的画面描述。

## 中文正文

中文正文从同一个 Spec 事实清单展开，使用“画面右侧前景、角色自身左手”等明确术语。方向、手别、人数、道具数、否定项、秒数、景别、光源与动作先后必须与 Spec 一致，不能为“电影感”添加浅景深、烟雾、手持或逆光。

稳定实体使用固定的中文外观短语；名字／内部 ID 不是唯一身份提示。对白保持剧本原文，中文提示词不意味着翻译或改写角色实际台词。每个 Prompt 自包含，不能写“same as previous shot”“同上”要求生成软件读取上一镜。

## 静态约束与视频约束

Image Prompt 用 [图像模板](../templates/image-prompt-template.md) 描述一个静态瞬间，不写完整动作时间线。视频全镜 must_have 如果在关键帧尚未发生，就不能强行进入图像；仅 applies_to 包含 image 且 time_range 覆盖关键帧的约束应用于图像。time_range=null 表示对对应输出全程适用。发现某个 image 约束时间范围不含关键帧时，返回设计阶段明确适用范围，不能悄悄丢掉冲突。

Video Prompt 按 [视频模板](../templates/video-prompt-template.md) 输出“【分镜参考图】…／镜头N，【时长】…，【镜头设计】…／【镜头内容】…／【音效】<…>”四行结构。【镜头设计】承载摄影与运镜，【镜头内容】详细展开起态、按秒动作、可见表情、光线、终态及持续身份／场景约束；详尽程度取决于 Spec 中的可执行细节，不以重复形容词凑字数。抽象情绪按调度阶段已定稿的具体表现转写，若只给“无助”等词且缺少可见表现，返回设计阶段补齐，不由 Prompt Builder 自行加跪地、哭泣或抱头。只有 Spec 给出的声音或对白才进入文本；音效未指定时按模板明确“未指定”，不把它当作静音指令，不擅加音乐、脚步或环境声。

must_not_have 用自然语言“不要新增第三人”，保留具体风险，避免冗长通用负面词库。不使用 --ar、LoRA、权重括号、专属 reference token、采样器或软件配置；画幅若已在 Spec 中确定，以自然语言表述。不能承诺某个平台必定执行。

## 转写检查与返回机制

先形成字段事实清单，再写三类文字，然后比对：Spec 版本、角色身份、画面位置、手别、道具、关键帧状态、光线、运镜、时间区间与正文语言。逐镜核对 Video Prompt 的分镜参考图注释与本镜 Image Prompt 的图号、shot_id、spec_version 和关键帧一致；完整生成包引用关键帧信息时 spec_paths 包含实际使用的 /storyboard_keyframe/time 与 /storyboard_keyframe/role。对中文 Image Prompt，按关键帧实际可见人物逐一核对是否明确写出相对于镜头的身体朝向，并保留头脸与身体的朝向差异；前景、背景及局部遮挡但仍可见的人物均不能遗漏，完全画外或完全遮住的人物不因此加入画面。仅写“画面右侧”“朝西”“看着甲”不满足此项；侧身也不等于直视镜头。若缺少该关系，沿用下述返回设计阶段的机制，Prompt Builder 不自行选择朝向或改变机位；裁切或遮挡不因补写朝向而改变。可删重复形容词，不可删必要事实。任何新摄影决定都意味着越权。

若发现 Spec 缺项或互相矛盾，记录冲突字段及影响，返回其设计阶段修订 Spec、ledger 和版本，再重建受影响的三类中文输出。provisional Spec 的已记录假设可忠实转写，交付层保留草案状态；不能把未解决的两个互斥状态都塞进同一个 Prompt。
