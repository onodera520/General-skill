# 资产绑定与共享档案

输入为已解析实体和用户资产。输出结构由 [output-schema](../templates/output-schema.json) 的 entity_registry、scene_registry 定义；不要在这里另创字段。

## 建立稳定身份

人物、场景、道具分别分配 CHAR_、ENV_、PROP_ 前缀的稳定 ID；同名实体需结合剧本身份和用户标注消歧，不能只依赖文件名。scene_id 标识叙事场次，场景实体 ID 标识物理地点；同地点的不同场次复用实体与布局版本。

同一个人的正面、侧面、背面是一个实体的不同 view_id；一张三视图按图内区域记录视图，一张图不等于一个人物。衣物最初可作为角色外观，发生脱下、交接或遗留时同时建立可追踪 PROP_ 实体，不能制造第二件衣物。

固定特征保存在 fixed_traits：发型／发色、可见面部标志、体型轮廓、服装款式、道具外形、场景结构。每项记录 knowledge_status 与 evidence_ref：

- observed：实际看过图片，证据指向 asset_id/view_id。
- user_provided：用户文字说明，不能改称图像观察。
- script：剧本明确事实，证据指向 source_id 或 beat_id。
- assumed：为执行而选定的最少必要设计，指向 assumption_id。
- unknown：确实无法判断，不捏造补全为已知。

衣服款式是固定特征，是否穿着、湿污和破损是动态状态。外套出画不删除其身份，脱下不改变其款式。禁止从图片推断性格、背景、身份关系或隐藏动机。

## 场景空间档案

选择一次稳定坐标约定，定义原点、方向及地标的相对位置。没有尺度依据时用“东墙近北角”等拓扑位置，不伪造厘米精度。登记主要门窗、家具、出入口、实际光源的位置与依据。entrances 和 light_sources 引用相应 landmark_id。

场景图可能只展示一面墙；画外结构保持 unknown，必要时设计并标 assumed。两个视角必须解释为同一布局，不为迎合反打把窗移到另一面墙。光源的世界位置保持固定，画面光向随摄像机视角重新计算。

## 绑定到每镜

asset_bindings 包含本镜人物、场景及相关道具的实体版本、asset_ids、view_ids、稳定 identity_traits 和当前 state_overrides。该字段是全局档案的只读展开，不是新的身份来源。不可见特征不强行写进静态图可见要求；仍在档案中保留。

Prompt Builder 读取 Spec 中已展开的绑定与状态，不自行重新查看资产并增添细节。关键身份描述采用稳定英文短语，避免每镜换用可能改变发色、年龄感、服装的同义描述。外部模型不认识内部 ID：输出需要可理解的外观文字，同时在生成包保留引用映射。

## 缺失、冲突与替换

区分两类输入：用户明确以文字设定作为资产依据且没有需要读取的图片时，资产记录用 availability=text_only、reference=user-text:某个稳定标识，user_description 保存原文，views 为空，特征为 user_provided；文字已满足本镜设计时可以定稿，不因“没有图片”自动全场 provisional。若用户提供或要求匹配某张图片而该图缺失／不可读，availability=unavailable，不能声称完成了视觉匹配，受影响身份采用临时设计并标 provisional。缺失手背纹理等无关细节不必虚构。

用户明确指定替换资产时增加对应实体版本，重新展开受影响 Spec 绑定、增加其 spec_version 并重建文字输出；不能只替换图片路径而留下旧外观 Prompt。

用户声称有附件但连文件名或路径都未提供时，可用 reference=unresolved:ASSET_001 这类逻辑占位引用，availability=unavailable，并在 issues 写明缺失；它不是文件路径，不能尝试按此打开或声称已读。正侧背若只是用户声明，views 只记录声明的视角与 region=unknown，不伪造图像区域。

剧本规定动作与状态，可用资产规定外观；如用户显式规定另一套服装则视为覆盖要求。不能用“资产里有外套”否定剧本脱外套，也不能用“反打更好看”覆盖场景图地标。无法同时满足的关键要求记 issues，保留明确采用的暂定解释。
