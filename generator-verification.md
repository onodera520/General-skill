# generator 验证记录

## 实现范围

实现入口、10份专业规则、5份 JSON Schema、3份输出模板和6份评测文档，共25个 Skill 文件。Image Prompt 与 Video Prompt 默认均为中英双语。开发验证代码和试运行产物在 `.generator-dev/`，不属于运行时 Skill，不参与媒体审查。

## 已执行的行为测试

- 无 Skill 基线：[原始产物](.generator-dev/baseline.md)。能保持主要交接、脱衣和反打关系，但缺少独立 Atomic Story Beats、完整逐镜契约、显式关键帧秒数和版本追踪。没有把它描述成已发生人物漂移的证据。
- Spec 只读转写：[首轮](.generator-dev/fidelity.md)、[首轮冲突报告](.generator-dev/fidelity-conflict.md)。发现分镜描述只覆盖关键帧，以及自然语言输入被套用不存在的 JSON Pointer。
- 修订后复测：[转写](.generator-dev/fidelity-v2.md)、[冲突报告](.generator-dev/fidelity-conflict-v2.md)。实际读取确认：中文分镜覆盖0–4秒；静态图为0.5秒交接前；视频为全过程；中英手别、时间、身份和固定摄影一致；自然语言输入使用源文定位。冲突变体不修改只读输入、不发布矛盾 Prompt。
- 三项局部测试：[实际产物](.generator-dev/targeted-evals.md)。正反打保留世界位置与西窗，双手持杯不消失；侧躲与抓腕并行，落刀不提前进入中途图像；缺图不假称观察，灰外套与缺口杯在时间省略和续批后保留。局部产物按测试要求只输出关键字段，不宣称通过完整生成包 Schema。

输入规则另明确：用户主动采用纯文字资产为 text_only；要求匹配但缺失的图片为 unavailable。未给实际路径时使用 unresolved: 逻辑引用，不伪造文件名。剧情时间差在 transition.explanation 记录，不混同屏幕时长。

## 结构检查

五份 Schema 已通过 JSON Schema Draft 2020-12 元模式与相互引用检查；全部本地 Markdown 链接可解析；官方 quick_validate.py 通过。开发验证器额外检查 ID、版本、时间区间、跨镜入口出口、覆盖率和 spec_paths，不能自动证明自然语言语义正确。

## 五镜完整链路与负例验证

独立完整试运行在前次会话中只保存了 Spec 中间产物，缺少五镜的分镜描述、Image Prompt 和 Video Prompt，首次完整包验证报15个缺项。原始文件保留为 [raw-generator-output.json](.generator-dev/forward/raw-generator-output.json)，没有将中断产物报告为一次通过。

恢复后由主执行者和独立转写执行者补齐五镜共25份文字，输出 [完整生成包](.generator-dev/forward/generator-output.json) 与 [可读分镜稿](.generator-dev/forward/storyboard.md)。修订了旧版文字资产的 unavailable/provisional 元数据，按当前 text_only 契约定稿；压缩了重复嵌入的状态快照文本，修正脱袖说明的一处手臂措辞，各镜版本升为2。机位、动作时刻及物理状态链未因此重新设计。这是经诊断、修订和续作后的通过，不是原始盲测一次通过。

2026-09-06 执行结果：5份 Schema、全部本地链接、5镜完整包的结构及跨记录检查通过。14项契约测试通过：1个真实完整包正例，以及缺失中文图像提示词、错误语言标签、缺失Blocking、缺失关键帧、未知状态引用、旧版转写、关键帧越界、运镜时间越界、不存在的字段指针、模型专属参数、旧版资产绑定、无依据的镜间瞬移、虚假剧情覆盖等13个变异负例。另验证纯文字资产必须携带文字说明，以及 unresolved 逻辑引用的合法结构。

人工逐镜读取了状态链与双语文字：第二镜末帧为接触未释放，第三镜首帧承接同一状态；握紧后林川空手、钥匙留阿岚左手；脱衣后外套留桌，北门离场后钥匙随阿岚在画外持续存在，林川留桌看门；双语不改变关键手别、时序或固定灯位。当前用例通过不代表所有未来剧本均无需迭代。

## 适用范围与限制

本轮资产输入是测试者提供的文字设定或声明缺失的图片，没有执行真实角色三视图和多视角场景图的识别测试，也没有生成或审查任何图片、视频。少量行为用例不是跨模型稳定性统计，更不能保证外部生图／视频软件完全服从 Prompt。
