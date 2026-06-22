# 协议与实现待决事项（bug / open issues）

> 本文档记录 `protocol.md` 与 ROS 侧实现（`usv_mqtt_bridge` 等）之间尚未对齐、或协议正文暂未收录但需讨论的事项。  
> 协议正文以 [protocol.md](./protocol.md) 为准；本文仅作跟踪，不构成对外交付规范。

---

## 1. productKey 代发策略（网关子设备 Topic 归属）

**状态：** 待讨论

**背景：** 已确认 MQTT 仅由 `jetson`、`mcu` 两个网关直连；其余设备为网关子设备，由对应网关代发。

**待明确：**

| 场景 | 选项 A | 选项 B |
| --- | --- | --- |
| Jetson 代发 `aivideo` 数据 | `/sys/aivideo/aivideo_01/...` | `/sys/jetson/jetson_01/...` |
| MCU 代发 `gps` 数据 | `/sys/gps/gps_01/...` | `/sys/mcu/mcu_01/...` |

`protocol.md` §2.2 / §2.3 的「对应实际 Topic」列目前采用**选项 A**（子设备独立 `productKey/deviceName`）。  
Jetson 侧 `usv_mqtt_bridge` 配置（`params.yaml`）对部分 Topic 已按选项 A（如 `aivideo`、`aivision`、`radar_nav_config`），但仍有大量 MCU 域 Topic 挂在 `jetson/jetson_01` 下（选项 B）。

**影响：** 云端订阅规则、ACL、物模型绑定、多船部署时的 Topic 模板。

**建议决议：** 在协议 §1.2 或 §2.1 增补一条硬性规则，并同步修改桥接配置（见 §8）。

---

## 2. Payload 外层格式（envelope）

**状态：** 待统一（桥接实现已定稿，协议正文示例未同步）

### 2.1 桥接实现（`usv_mqtt_bridge` 当前口径）

上行外发、下行 Service 解析均使用 **`timestamp` + `seq` + `data`** 三层结构：

```json
{
  "timestamp": 1703123456789,
  "seq": 1,
  "data": {}
}
```

- `timestamp`：毫秒级 Unix 时间戳（非 RFC3339 字符串）
- `seq`：按每个 MQTT Topic 独立递增，从 1 开始
- `data`：业务载荷（对象或数组）

旧的 `timestamps` / `device_id` / `msg_type` / `payload` envelope **已废弃**，桥接会拒绝含 `payload` 键的旧格式。

下行 Service 请求同样要求上述结构，业务字段放在 `data` 内，例如急停：

```json
{
  "timestamp": 1703123456789,
  "seq": 1,
  "data": {
    "estop": true,
    "src": "shore"
  }
}
```

### 2.2 协议正文（`protocol.md`）现状

§1.3.1 与 §3/§4 多数示例仍使用扁平 `"time": ...` 且**无 `seq`/`data` 包裹**，与桥接实现不一致。云端若按协议示例发下行指令，当前桥接会丢弃。

### 2.3 待决议

- 是否将 `protocol.md` 全部上下行示例统一为 `timestamp/seq/data`（推荐，与桥接 README 一致）
- 或桥接增加对扁平 `time` 格式的兼容层（不推荐，增加双轨维护成本）

---

## 3. `property/status`（整机/航行状态）

**状态：** 待讨论

**背景：** `usv_mqtt_bridge` 注册了 `property/status`，但 `protocol.md` 未定义该物模型；Jetson 侧已有 `property/status_jetson`（算力/硬件监控），MCU 侧有 `property/mcu_status`（链路/模式/设防等）。

**待明确：**

- 是否保留 `property/status` 作为「整机综合状态」？
- 若保留，与 `status_jetson`、`mcu_status` 的字段边界如何划分？
- 归属 Jetson 网关还是独立 productKey？

---

## 4. `service/manual_ctrl`（手动控制）

**状态：** 待讨论（已从 `protocol.md` 移除）

**移除原因：** 是否仍通过 MQTT 下发手动摇杆量尚未定论；暂不出现在对外协议中。

**原协议草案（供讨论）：**

**Topic:** `/sys/jetson/jetson_01/thing/service/manual_ctrl`

```json
{
  "time": 1703123456789,
  "x": 500,
  "y": 0,
  "z": 0,
  "r": 0,
  "buttons": 64
}
```

| 参数 | 类型 | 范围 | 说明 |
| --- | --- | --- | --- |
| x | int | -1000~1000 | 前进/后退 |
| y | int | -1000~1000 | 横移 |
| z | int | -1000~1000 | 备用油门 |
| r | int | -1000~1000 | 转向 |
| buttons | int | - | 按键位掩码 |

**Reply Topic:** `/sys/jetson/jetson_01/thing/service/manual_ctrl_reply`（格式同 §3.1 通用服务回复）

**实现侧：** `usv_mqtt_bridge` 仍注册 `manual_ctrl` / `manual_ctrl_reply`；`usv_interfaces/srv/ManualControl` 已定义。

**待决议：** 是否恢复进协议；若否，是否从桥接层删除相关映射（见 §8）。

---

## 5. 协议正文内部字段不一致（待修订）

**状态：** 待修订（与实现无关的文档质量问题）

| 位置 | 问题 |
| --- | --- |
| §4.2.2 `mission_delta` | JSON 示例 `waypoint` 对象括号不匹配 |
| §4.2.4 `aivision_targets` | 字段表写 `timestamps[].time`、`targets[].class`、`rel_ang`；示例用 `time_ms`、`class_name`、`rel_ang_deg` |
| §3.5 `auto_task` | `mission_type` 示例值为 `16`，无枚举说明 |
| §3/§4 全局 | 示例使用 `time`，桥接使用 `timestamp/seq/data`（见 §2） |

---

## 6. `event/task_prog`（任务进度反馈）

**状态：** 待补充进 `protocol.md`

**背景：** `auto_task` 异步链路在实现侧为 Goal / Feedback / Result 三段：

| 链路 | MQTT Topic | 实现 |
| --- | --- | --- |
| Goal | `service/auto_task` | 协议已收录 |
| Feedback | `event/task_prog` | **协议 §2.2 未收录** |
| Result | `service/auto_task_reply` | 协议有 reply 表，无独立 payload 节 |

`usv_interfaces/TaskProgress.msg` 与 `usv_mqtt_bridge` 已对齐 `task_prog`。

**建议：** 在 §2.2 上行事件表增加 `event/task_prog`，并新增 §4.2.x payload 说明。

---

## 7. 实现侧仍注册但协议已删除/未收录的映射

以下项在 `usv_mqtt_bridge`（Jetson 配置）中仍存在，与当前 `protocol.md` 不一致，需在协议定稿或实现收敛时处理：

| 类别 | 桥接层 Topic / 链路 | 说明 |
| --- | --- | --- |
| 已删协议 | `property/motor`、`property/imu`（`apm_imu`） | 推进器/APM 已移出协议 |
| 已删协议 | `property/radar_mm`、`property/radar_nav` | 雷达扫描数据不再 MQTT 上传 |
| 域错误 | `property/gps_status` 由 Jetson 桥接发布 | GPS 仅 MCU 网关代发 |
| 域错误 | `event/mcu_heartbeat` 挂在 `jetson` Topic 下 | MCU 心跳仅 MCU 网关发布 |
| 未收录 | `property/status` | 见 §3 |
| 未收录 | `event/task_prog` | 见 §6 |
| 待讨论 | `service/manual_ctrl` | 见 §4 |
| 架构 | 多条 MCU 域 Topic 挂在 `jetson` 前缀下 | 见 §1、§8 |

---

## 8. `usv_mqtt_bridge` Jetson 配置待修改清单（`params.yaml`）

**状态：** 已记录，**暂未改配置文件**（按当前决策仅文档跟踪）。

以下针对 `src/usv_comm/usv_mqtt_bridge/config/params.yaml` 中 **Jetson 网关实例** 的收敛项。MCU 网关应单独部署一份配置，仅承载 §2.3 域 Topic。

### 8.1 应删除或清空的链路（协议已明确不走 Jetson / 不再上传）

| 配置项 | 当前值（摘要） | 修改建议 |
| --- | --- | --- |
| `ros_topics.gps_status_input_topic` | `/sensors/gps/data` | 置空 `""`；GPS 改由 MCU 网关配置 |
| `topics.gps_status` | `/sys/jetson/jetson_01/thing/property/gps_status` | 删除或注释；MCU 侧重定向 `/sys/gps/gps_01/...` |
| `ros_topics.radar_mm_input_topic` | `/perception/radar/mmw/objects` | 置空 `""` |
| `topics.radar_mm` | `/sys/radar_mm/radar_mm_01/thing/property/radar_mm` | 删除或注释 |
| `ros_topics.radar_nav_input_topic` | `/sensors/radar/nav/sector` | 置空 `""` |
| `topics.radar_nav` | `/sys/jetson/jetson_01/thing/property/radar_nav` | 删除或注释 |
| `publish_rates.radar_nav_hz` | `10.0` | 删除（无对应上行） |
| `ros_topics.mcu_heartbeat_input_topic` | `/usv/monitor/heartbeat` | 置空 `""`；Jetson 桥不接 MCU 心跳 |
| `topics.mcu_heartbeat` | `/sys/jetson/jetson_01/thing/event/mcu_heartbeat` | 删除或注释；MCU 网关发布至 `/sys/mcu/mcu_01/thing/event/mcu_heartbeat` |

### 8.2 协议已删除物模型（应收敛）

| 配置项 | 修改建议 |
| --- | --- |
| `ros_topics.motor_input_topic` / `topics.motor` | 保持空/删除 |
| `ros_topics.imu_input_topic`（`/mavros/imu/data`）/ `topics.imu`（`apm_imu`） | 置空并删除 APM Topic 映射 |
| `topics.status_apm`（已注释） | 彻底移除 |
| `topics.manual_ctrl` / `manual_ctrl_reply` 及相关 `ros_topics.*` | 待 §4 决议后删除或保留 |

### 8.3 productKey / Topic 前缀（待 §1 决议后批量修正）

当前 Jetson 配置中下列 MCU 域 Topic 仍使用 `jetson/jetson_01` 前缀，与协议 §2.3 不一致，应在 **MCU 网关配置** 中按子设备真实路径发布，并从 Jetson 配置中移除：

- `topics.weather_status`、`depth_status`、`battery_status`、`fuel_status`
- `topics.mcu_status`、`ais`、`io_status`
- `topics.diag_result`、`io_ctrl`、`diag_request` 及其 `_reply`
- `topics.alarm`（MCU 报警应走 `/sys/mcu/mcu_01/...`，Jetson 报警保留 `/sys/jetson/jetson_01/...`）

已符合协议、可保留在 Jetson 侧的示例：

- `topics.aivideo_status`、`aivideo_ctrl` → `aivideo/aivideo_01`
- `topics.vision_targets` → `aivision/aivision_01`
- `topics.radar_nav_config` → `radar_nav/radar_nav_01`
- `topics.radar_nav_map`（启用时）→ `radar_nav/radar_nav_01`

### 8.4 其他实现问题（配置层无法单独解决）

| 问题 | 说明 |
| --- | --- |
| `auto_task` 下行未接通 | `ros_topics.auto_task_output_topic` 为空，需 `action_json_adapter_node` 或桥接内置 Action |
| `mode` 双路径 | 同时配置 `mode_output_topic`（raw）与默认 `SetMode` 服务，可能重复触发 |
| 明文 `broker.password` | 应迁出仓库，使用环境变量或本地 overlay 配置 |
| `protocol.py` 仍注册已删物模型 | 收敛配置后应同步删减 `MSG_TYPE_MOTOR`、`MSG_TYPE_IMU`、`MSG_TYPE_STATUS_APM`、`MSG_TYPE_RADAR_MM`、`MSG_TYPE_RADAR_NAV` 等（代码变更，非本次 params 修改范围） |

---

## 附录 A：`protocol.md` 已完成的修订摘要

> 本节汇总 2026-06-22 前后对 [protocol.md](./protocol.md) 的协议收敛修改，便于与 `usv_mqtt_bridge` 实现对照。  
> **未改动的部分**：协议版本号仍为 V1.1、版本日期仍为 2026-06-01；§3/§4 中 JSON 示例的扁平 `"time"` 字段尚未统一为桥接层 `timestamp/seq/data`（见 §2）。

### A.1 架构与网关边界（§1.2、§2）

| 修订项 | 修改前（概要） | 修改后 |
| --- | --- | --- |
| MQTT 直连设备 | 未明确仅两网关 | 明确仅 **`jetson`**、**`mcu`** 两个网关 MQTT 直连 |
| 子设备代发 | 笼统描述 | 其余设备为网关子设备，由对应网关代发 |
| GPS 归属 | 未区分网关 | **GPS 仅 MCU 网关代发，不经 Jetson** |
| 雷达扫描上行 | §2.2 含 `radar_mm`、`radar_nav` 属性上报 | **原始扫描数据不上传 MQTT**；保留 `radar_nav_map` 地图与 `radar_nav_config` 服务 |
| §2.2 / §2.3 注释 | 「上报」表述 | 改为「网关代发」，并注明 Jetson 域不含雷达扫描上行 |

### A.2 物模型表（§1.3.3）

**已删除条目：**

- `thruster` / `thruster_01`（推进器产品）
- `apm` / `apm_01`（APM 飞控产品）

**保留并标注：**

- `jetson`、`mcu` 行增加「（网关）」说明

**仍保留（硬件仍存在，仅扫描属性不上传）：**

- `radar_mm`、`radar_nav`（用于配置服务 / 地图等，非扫描 property 上报）

### A.3 Jetson 处理域 Topic（§2.2）

**下行服务 — 已删除：**

- `service/manual_ctrl` / `manual_ctrl_reply`（移至本 bug 文档 §4 待讨论）

**下行服务 — 已删除（油机控制）：**

- 原 §3.13–3.14 `service/thruster_ctr`（整节删除，油机控制不走 Jetson）

**上行属性 — 已删除：**

- `property/thruster` / `thruster_status`（推进器数据）
- `property/apm_imu`（IMU 数据）
- `property/radar_mm`（毫米波雷达扫描）
- `property/radar_nav`（导航雷达扫描）

**上行属性 — 保留：**

- `status_jetson`、`radar_nav_map`、`perception_trajectory`
- 视频 / 视觉相关 event、service 及子设备 `productKey` 路径

**上行事件 — 未在 Jetson 域新增：**

- `mcu_heartbeat` 仅出现在 §2.3 MCU 域（`/sys/mcu/mcu_01/...`），Jetson 不得代发

### A.4 MCU 处理域（§2.3）

- 结构未大改；`gps_status`、`mcu_heartbeat` 等仍归属 MCU 及对应子设备 `productKey`
- 与修订一致：**GPS、MCU 心跳不由 Jetson 网关承载**

### A.5 下行数据格式（§3）

| 原章节 | 处置 |
| --- | --- |
| §3.5 `manual_ctrl` | **删除**，草案记入 bug §4 |
| §3.13–3.14 `thruster_ctr` / 回复 | **删除** |
| §3.6–3.12 | 重编号为 §3.5–3.11（自动任务、参数、视频、IO、自检、雷达配置） |
| §3.10 自检 `modules` 枚举 | 删除 `thruster` 模块项 |

### A.6 上行数据格式（§4.1）

**已删除整节：**

- 原 §4.1.2 推进器数据（`property/thruster_status`）
- 原 §4.1.3 IMU 数据（`property/apm_imu`）
- 原 §4.1.2–4.1.3 毫米波 / 导航雷达**扫描**数据格式（第二轮删除）
- 原 §4.1.17 APM 飞控状态（`property/status_apm`）
- §4.1.16 中 APM 心跳（`event/apm_heartbeat`）段落

**重编号后当前 §4.1 结构（4.1.1–4.1.12）：**

1. Jetson 状态  
2. 导航雷达地图  
3. 融合轨迹  
4. GPS（MCU）  
5. 气象站（MCU）  
6. 测深仪（MCU）  
7. 电池（MCU）  
8. 油箱（MCU）  
9. MCU 系统状态  
10. AIS（MCU）  
11. IO 设备状态（MCU）  
12. 心跳（Jetson + MCU，MCU 路径为 `mcu/mcu_01`）

**其他：**

- `diag_result` 示例中删除 `thruster` 自检模块项

### A.7 附录频率建议（§6.2）

**已删除行：**

- IMU 数据、推进器数据（第一轮）
- 雷达扫描数据（第二轮）

**保留：**

- GPS、雷达**地图**、感知轨迹、视觉目标等

### A.8 配套文档（非 protocol 正文）

| 文件 | 修改 |
| --- | --- |
| [bug.md](./bug.md) | 新建；记录待决事项、envelope 说明、`params.yaml` 待改清单（§8） |
| [README.md](../README.md) | 增加 bug.md 链接；物模型速查删除 thruster/apm；描述改为双网关架构 |

### A.9 明确未纳入本次 protocol 修订的项

以下仍记在 bug 文档其它章节，**尚未写入 protocol 正文**：

- `event/task_prog` 任务进度（§6）
- `property/status` 整机状态（§3）
- `service/manual_ctrl` 是否恢复（§4）
- Payload 示例统一为 `timestamp/seq/data`（§2）
- `usv_mqtt_bridge` / `params.yaml` 与协议对齐（§7、§8）

### A.10 IO 域未删内容（说明）

以下含 `thruster_*` 字样的内容**保留**，属于 MCU **IO 继电器设备名**，非 MQTT 推进器物模型：

- §3.8 `io_ctrl` 中 `thruster_left_action` / `thruster_right_action`
- §4.1.11 `io_status` 中对应 status 字段
- §5.1 设备清单中 `execution_thruster`

---

## 变更记录

| 日期 | 说明 |
| --- | --- |
| 2026-06-22 | 初版：thruster/apm/manual_ctrl 协议删减后的待决事项汇总 |
| 2026-06-22 | 更新 envelope 描述；删除雷达扫描/GPS/mcu_heartbeat Jetson 代发；新增 §8 params 待改清单 |
| 2026-06-22 | 新增附录 A：`protocol.md` 修订摘要 |
