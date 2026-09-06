# 摄像机运动与静态关键帧

读取叙事目的、空间档案、Blocking 和动作时序。输出 [Shot Spec](../templates/shot-schema.json) 的 camera_movement、storyboard_keyframe，并协调摄影字段。

## 选择与描述

固定镜头是有效决定：kind=locked，segments 仍填写覆盖全镜的固定阶段，from=to，path 写无位移／无旋转，speed 写零；不能漏写。

推／拉改变摄像机位置；变焦改变视角；摇／俯仰改变朝向；跟拍、横移、升降、环绕改变路径；手持描述稳定程度。不要混用“推近”与“放大”而让空间关系不明确。

每段给 start/end、from/to、path、speed、stability、narrative_reason；时间区间与动作线共用秒轴。普通段顺序连接、覆盖 0..duration，无间隙或重叠；复合动作（推进同时轻摇）在同一段中说明，不写两条互相矛盾的摄像机轨迹。各段 from 承接上段 to。

开始／结束状态描述位置、看向目标、景别、焦点；速度明确匀速、加速、缓入缓出或保持，不用“慢慢动一下”。没有尺度时描述相对地标或景别变化，不说无法解释的“推进10%”。幅度采用设计坐标或明确起止视图，不能穿越墙、桌或人物。

每个运动有叙事作用：跟随移动保证路径可读、缓推突出已存在的信息、保持固定让观众看清交接。无必要时保持固定，不把所有镜头都设计成运动镜头。

## storyboard_keyframe

选择一个确切镜内秒数 time，标 role=start／representative／end 和 selection_reason。start 对应 t=0；end 对应 t=duration；中途为 representative。关键帧只能呈现一个时刻，不混合“挥刀、抓腕、刀已落地”等先后状态。

state 从入口状态和截止该时间的事件推导，camera_state 从运动轨迹推导，blocking／composition 从该状态与机位投影得到。关键帧不必总选起始帧；若选中途代表帧，明确不能直接作为完整 Video Prompt 的首帧输入。

Video Prompt 默认描述完整镜头，起点来自顶层摄影字段和 entry_state，终点来自最后运动段和 exit_state。未来用户明确需要“从代表帧续拍”时，返回设计阶段新建或修订区间与 Spec，再转写；Prompt Builder 不能自行截掉前半段。
