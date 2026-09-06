# Prompt Builder：只读转写，不重新导演

读取已定稿或明确标注 provisional 的 [Shot Spec](../templates/shot-schema.json)，以及它引用的只读入口／出口状态。使用 [output-schema](../templates/output-schema.json) 封装输出：storyboard_description，image_prompt.zh_cn／en，video_prompt.zh_cn／en。每份文字包含 shot_id、spec_version、language、text、spec_paths。中文用 zh-CN，英文用 en。

## 权限与映射

Prompt Builder 不直接从剧本构思，不重新查看资产添加新特征，不从摄影 reference 临时选另一种拍法。完整生成流程必须先有可追溯的 Beats、Shot Plan、Spec。用户仅提交已定稿 Spec 要求转写时，接受其设计来源，不反向捏造 Beats；只交付转写内容，不声称产出了完整生成包。

两种入口的完整性要求不同：完整生成包必须满足 Schema 所有必填字段；只读转写已有自然语言 Spec 时，忠实转写已提供的摄影与视觉事实，未给的画幅、焦点或光质可不写，并在交付说明中列出。若缺失或矛盾会令主体身份、动作主体／对象、左右手、关键帧时刻或起止状态存在互斥解释，则该输出不能冒充可执行成品，返回设计阶段；不以自动补参数填满 Schema。Prompt Builder 发现输入问题只在独立交付说明／issues 中标明待解决，不修改原 Spec 的 status、版本或内容。

| 输出内容 | 唯一取值来源 |
|---|---|
| 人物／道具固定外观 | asset_bindings.identity_traits，结合同一 Spec 的 state_overrides |
| 风格与画幅 | visual_style、aspect_ratio |
| 静态景别、机位、焦点 | storyboard_keyframe.camera_state |
| 静态人物位置、动作、视线、持物 | storyboard_keyframe.blocking 与 state |
| 静态前中后景 | storyboard_keyframe.composition |
| 静态环境与光线 | environment 及 keyframe.state 中该时刻的状态 |
| 视频起始摄影、角色／道具状态 | 顶层摄影字段与 continuity.entry_state_id 对应状态 |
| 视频动作／对白／镜内变化 | action_design、camera_movement、lighting.changes、相关 events |
| 视频结束状态 | continuity.exit_state_id、最后运动段和 Blocking.end |
| 硬约束 | must_have／must_not_have 的 applies_to 与 time_range |

完整生成包中 spec_paths 是每份文字实际使用字段的 JSON Pointer 列表，根为该镜 spec，例如 /storyboard_keyframe/state、/asset_bindings、/duration。每个路径必须能在实际对象中解析，禁止写不存在的 /duration_seconds。它用于追踪而不是质量证明。入口／出口状态通过 /continuity/entry_state_id 等路径追溯到 ledger。只读自然语言 Spec 没有 JSON 对象时，在正文之外引用源文片段／段落定位，不伪造 spec_paths；该转写交付不套完整生成包 Schema。段外镜头标签、版本、关键帧时间和参考图映射不混入可复制 Prompt 正文。

详细分镜描述覆盖整个镜头的起态、动作时间线、摄影变化与终态，并说明所选关键帧；不能只描述关键帧而漏掉交接等后续动作。Image Prompt 才是单一时刻的画面描述。

## 中文与英文

两个语言版本都从同一个 Spec 事实清单展开；中文自然使用“画面右侧前景、角色自身左手”等明确术语，英文对应 “screen-right foreground, her left hand”。方向、手别、人数、道具数、否定项、秒数、景别、光源与动作先后必须一一一致。不能在英文版加“cinematic”后附赠浅景深、烟雾、手持或逆光。

稳定实体使用固定的中英外观短语；名字／内部 ID 不是唯一身份提示。对白在两个语言版本中均保持原文，可用英文介绍说话方式，但不能把实际台词译成英文后让角色改说英语。每个 Prompt 自包含，不能写“same as previous shot”“同上”要求生成软件读取上一镜。

## 静态约束与视频约束

Image Prompt 用 [图像模板](../templates/image-prompt-template.md) 描述一个静态瞬间，不写完整动作时间线。视频全镜 must_have 如果在关键帧尚未发生，就不能强行进入图像；仅 applies_to 包含 image 且 time_range 覆盖关键帧的约束应用于图像。time_range=null 表示对对应输出全程适用。发现某个 image 约束时间范围不含关键帧时，返回设计阶段明确适用范围，不能悄悄丢掉冲突。

Video Prompt 用 [视频模板](../templates/video-prompt-template.md) 说明起点、按秒动作和运镜、终点及持续不变的身份／场景特征。只有 Spec 给出的声音或对白才进入文本，不擅加音乐、脚步或环境声。

must_not_have 用自然语言“不要新增第三人／No additional person”，保留具体风险，避免冗长通用负面词库。不使用 --ar、LoRA、权重括号、专属 reference token、采样器或软件配置；画幅若已在 Spec 中确定，以自然语言表述。不能承诺某个平台必定执行。

## 转写检查与返回机制

先形成字段事实清单，再写三类文字，然后比对：Spec 版本、角色身份、画面位置、手别、道具、关键帧状态、光线、运镜、时间区间和中英等价。可删重复形容词，不可删必要事实。任何新摄影决定都意味着越权。

若发现 Spec 缺项或互相矛盾，记录冲突字段及影响，返回其设计阶段修订 Spec、ledger 和版本，再重建全部语言版本。provisional Spec 的已记录假设可忠实转写，交付层保留草案状态；不能把未解决的两个互斥状态都塞进同一个 Prompt。
