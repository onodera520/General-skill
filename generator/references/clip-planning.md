# Clip 分组与定稿

读取已 locked 的 [时间线](../templates/timeline-schema.json)、Beats 和资产状态。按 [Clip Plan](../templates/clip-plan-schema.json) 组织一次生成片段。Clip 可包含多个 Shot 与场景；Shot 保持原有连续摄影和单一关键帧语义。

## 优先级与边界

剧情与台词完整可理解 → 紧密因果段完整 → 不扩写、不拖时 → 优先10–15秒。

先列出已估定时间线的安全边界：事件阶段或台词语义边界，不能切入 indivisible_groups。即时刺激与反应（喃喃自语→猛然睁眼）不可拆；一般剧情上的因果不自动绑定整场戏。分组只移动边界，不改变 total_duration、blocks、audio_events 或语速。用户分镜符号、段落、换行不是 Clip 边界依据。

在安全边界优先合并：可在15秒内容纳的连续内容不主动拆成多个 Clip，允许换场在同一 Clip 内发生。优先让各段10–15秒；不足10秒先尝试与相邻段合并或重分边界。全文自然不足10秒允许 short_total；不可继续合并或重分的尾段允许 short_tail。28秒有合适边界时可15+13；17秒可在实际安全边界分为10+7，不延长成20秒。

不可拆段自身超过15秒时允许 causal_overrun，并写 group_ids 和原因；超长 Clip 只包含该不可拆段所需范围，不借例外随意拼接无关内容。孤立微动作不独立包装成 Clip；天然只有一个动作的输入可作为完整短剧情，不能为了凑长度发明内容。无法兼顾剩余约束时使用 exception.kind=conflict，保留 provisional，简短说明，不伪装合规。

## 两阶段设计

第一步仅确定 Clip 全局 start/end、duration、例外与 timeline_version；第二步在固定区间内设计 Shot 并补全 shot_spans、audio_spans、scene_entries，最后赋 clip_version 并定稿。不得为匹配某个想要的镜头倒改既定时长。

shot_spans 使用 Clip 内秒数，含 shot_id、spec_version、start/end；全部顺序覆盖0..duration，无重复、重叠或空洞，长度等于该 Shot.duration。Shot 与图号按全片顺序延续，每个 Shot 恰好属于一个 Clip；某次摄影跨 Clip 边界时建立两个 Shot，衔接原状态而非重播动作。

audio_spans 引用 audio_id，使用 Clip 内start/end与该音轨text的text_start/text_end。全局播放区间=Clip.start+局部时间，必须与音轨真实重叠区间相等；声音跨 Shot 不必拆 audio_span，跨 Clip 才分文字播放片段。Shot 覆盖哪些声音由 shot_spans 与 audio_spans 相交推导，不复制整句到每个 Shot。

scene_entries 在每次进入或返回叙事场景时记录 scene_id、Clip 内 time、对应入口 state_id、全部 present_character_ids 与静态 staging。不同场景分别建项；同场景跨 Clip 也必须重述仍在场的人物，不因画外而删人。起始姿势写站、坐、跪等，列地标、朝向角度和相对距离，无尺度依据不编米数。其后动态动作在画面时间线写。

## Clip 衔接与摄影

同场景连续续接 Clip 的首镜必须是单人中景、近景、特写或空镜，禁止多人开场。人物首镜明确“某某单人镜头”。完整在场站位是空间说明，不是要求全部入画；画外人物仍有状态。即使该段中途换场，规则只针对实际连续续接处。显式换场、闪回或时间跳跃依原文转场，不冒充连续状态。

禁止直视镜头，互动人物不能全部正面朝镜头；为每个可见人物写清相对于镜头的身体朝向、头脸方向与独立视线。不能仅靠裁切掩盖物理站位错误。

前 Clip 出口与后 Clip 入口按 [连续性](continuity.md) 衔接。不得重复已完成动作／台词，但应重述“右手仍握钥匙”等起态。延续方向、运镜速度与停止／起动节奏要有合理承接。不要把不同时空的出口状态继承给闪回或新场景。

## 版本与迁移

定稿 Clip 的任何 Shot 引用、声音区间、站位或全局补充变化均增加 clip_version，重建整段 Video Prompt；变化 Shot 增加 spec_version并更新 Image Prompt。timeline_version 变化使相关 Clip Plan 失效，需要重新核对全片分组和声音覆盖。

旧1.x包读取后先保留资产、场景、原文、Shot与ledger，再按当前估时原则建立时间线和Clip层；兼容旧时长时直接复用Shot，若旧时长或开场不符合新设计则在设计阶段修订受影响Shot与版本。旧逐镜Video Prompt不能拼接为Clip正文，须按新Clip设计重新转写。输入契约仍1.1，Shot契约仍1.0，可选生成包升2.0。默认仅交付一个Markdown文件。
