# 协议命名规范

> 适用范围：Topic、`productKey`、`deviceName`、`params` 内业务字段、`identifier`、枚举值、数组相关字段。

---

## 1. 边界说明

+ 最外层公共字段如 `id`、`reportTime`、`deviceId`、`tenantId`、`requestId`、`method`、`code`、`msg` 属于厂商协议信封层。
+ 这些字段的命名和结构直接遵循厂商协议，本规范不重新定义。
+ 本规范只约束 Topic 路径命名和业务层命名。
+ `productKey`、`deviceName` 来源于云端注册/同步结果，设备侧不应自行派生变体。

---

## 2. 总体规则

+ 同一层级只使用一种命名风格。
+ `params` 内新增业务字段统一使用 `snake_case`。
+ `identifier` 使用小写英文 `snake_case`。
+ 机器可解析的字符串取值使用英文编码，不使用中文字符串。
+ 历史字段允许兼容保留，新增内容不再沿用旧风格。

---

## 3. Topic 命名

### 3.1 固定格式

```text
/sys/{productKey}/{deviceName}/thing/{domain}/{action}
```
### 3.2 Topic 总表

| 类别             | 方向      | Topic 模板                                                  | Method                 |
| ---------------- | --------- | ----------------------------------------------------------- | ---------------------- |
| 属性上报         | 设备 → 云 | `/sys/{productKey}/{deviceName}/thing/property/post`        | `thing.property.post`  |
| 属性上报响应     | 云 → 设备 | `/sys/{productKey}/{deviceName}/thing/property/post_reply`  | —                      |
| 事件上报         | 设备 → 云 | `/sys/{productKey}/{deviceName}/thing/event/post`           | `thing.event.post`     |
| 事件上报响应     | 云 → 设备 | `/sys/{productKey}/{deviceName}/thing/event/post_reply`     | —                      |
| 服务调用（下行） | 云 → 设备 | `/sys/{productKey}/{deviceName}/thing/service/invoke`       | `thing.service.invoke` |
| 服务调用响应     | 设备 → 云 | `/sys/{productKey}/{deviceName}/thing/service/invoke_reply` | —                      |
| 配置下发         | 云 → 设备 | `/sys/{productKey}/{deviceName}/thing/config/push`          | `thing.config.push`    |
| 配置下发响应     | 设备 → 云 | `/sys/{productKey}/{deviceName}/thing/config/push_reply`    | —                      |
| 配置获取         | 设备 → 云 | `/sys/{productKey}/{deviceName}/thing/config/get`           | `thing.config.get`     |
| 配置获取响应     | 云 → 设备 | `/sys/{productKey}/{deviceName}/thing/config/get_reply`     | —                      |
| 子设备注册       | 设备 → 云 | `/sys/{productKey}/{deviceName}/thing/sub/register`         | `thing.sub.register`   |
| 子设备注册响应   | 云 → 设备 | `/sys/{productKey}/{deviceName}/thing/sub/register_reply`   | —                      |
| 拓扑获取         | 设备 → 云 | `/sys/{productKey}/{deviceName}/thing/topo/get`             | `thing.topo.get`       |
| 拓扑获取响应     | 云 → 设备 | `/sys/{productKey}/{deviceName}/thing/topo/get_reply`       | —                      |
| OTA 升级通知     | 云 → 设备 | `/sys/{productKey}/{deviceName}/thing/ota/upgrade`          | —                      |
| OTA 升级进度     | 设备 → 云 | `/sys/{productKey}/{deviceName}/thing/ota/progress`         | `thing.ota.progress`   |

### 3.2 各段含义

| 段位 | 含义 |
| ---- | ---- |
| `/sys` | 系统主题前缀 |
| `{productKey}` | 云端同步的产品标识 |
| `{deviceName}` | 云端同步的设备实例标识 |
| `{domain}` | 能力分类，如 `property`、`event`、`service`、`config`、`ota` |
| `{action}` | 动作语义，如 `post`、`get`、`push`、`invoke`、`upgrade`、`progress`、`get_reply` |

### 3.3 规则

+ 路径层级统一使用 `/`。
+ Topic 只表达“能力类别 + 动作”，不承载具体业务类型。
+ 具体业务类型通过 `method` 或 `params.identifier` 区分。
+ 回复类动作统一使用 `_reply` 后缀。
+ 设备不得自行拼接 `productKey` 或 `deviceName`，必须使用云端下发或同步的值。
+ 不允许在 Topic 路径中携带版本号，如 `/v1/`；版本信息应通过 `method` 或 `params.version` 承载。
+ 不允许在 Topic 路径中携带用户标识、租户标识等业务信息，保持路径纯通道语义。

---

## 4. 业务字段命名

### 4.1 基本要求

+ 字段名使用英文，不使用拼音。
+ 不使用无语义缩写，如 `temp`、`humi`、`num`、`ctr`。
+ 同一语义只保留一种写法，不同时出现 `taskId` 和 `task_id`。

### 4.2 常用后缀

| 语义 | 后缀 | 示例 |
| ---- | ---- | ---- |
| 标识 | `_id` | `task_id` |
| 名称 | `_name` | `task_name` |
| 类型 | `_type` | `object_type` |
| 状态 | `_status` | `system_status` |
| 动作 | `_action` | `generator_action` |
| 数量 | `_count` | `points_count` |
| 顺序 | `order` | `points[].order` |

规则：

+ 数量统一使用 `_count`。
+ 不再新增 `_num`、`_number`、`_quantity`、`_len`。
+ 有顺序语义的点、边、步骤，统一使用 `order`。

### 4.3 布尔字段

+ 布尔字段统一使用 `true/false`。
+ 状态字段可直接使用 `online`、`armed`、`connected` 这类语义名。
+ 开关控制动作优先使用 `on/off`，不强行写成布尔值。

### 4.4 开关值建议

+ 对设备控制类开关字段，推荐使用字符串枚举值 `on/off`。
+ 对设备状态反馈类开关字段，如需和控制动作保持一致，也推荐使用 `on/off`。
+ 只有在字段语义天然是布尔判断时，才使用 `true/false`。

推荐：

+ `light_left_action: "on"`
+ `light_left_status: "off"`
+ `generator_action: "on"`
+ `camera_stream_action: "off"`

不推荐：

+ `light_left_action: true`
+ `light_left_status: 1`
+ `generator_action: "open"`

---

## 5. 单位与取值

### 5.1 单位后缀

+ 数值字段建议直接带单位后缀，且统一小写。
+ 推荐：`_ms`、`_s`、`_m`、`_mps`、`_deg`、`_dps`、`_rpm`、`_percent`、`_c`、`_v`、`_a`、`_w`、`_pa`、`_hpa`、`_bar`、`_kbps`、`_l`。

示例：

+ `temperature_c`
+ `speed_mps`
+ `uptime_ms`
+ `voltage_v`

### 5.2 字符串取值

+ 枚举值、模式值、状态值、通道值统一使用小写英文。
+ 多单词值使用 `snake_case`。
+ 机器可解析字段不使用中文字符串。
+ 展示或说明类字段，如 `msg`、`message`、`remark`、`task_name`，按业务需要使用 UTF-8 文本。

### 5.3 时间与坐标约定

+ 绝对时间统一使用 Unix 时间戳，单位毫秒。
+ 时间字段命名建议带 `_ms` 后缀；如外层厂商字段已固定命名，则沿用厂商字段。
+ 持续时长字段按单位使用 `_ms` 或 `_s`，同一语义不混用。
+ 经纬度统一使用十进制度数。
+ `lat` 表示纬度，`lon` 表示经度，不使用 `lng`。
+ 经纬度建议保留 6 到 8 位小数；常规导航场景建议至少 7 位小数。
+ 不建议使用time这种无单位后缀。
+ 不推荐使用 date、datetime、timestamp作为字段名，除非厂商协议强制。

推荐：

+ `report_time_ms`
+ `capture_time_ms`
+ `uptime_ms`
+ `lat: 31.1234567`
+ `lon: 121.1234567`

说明：

+ 厂商信封层已有字段如 `reportTime`、`params.time`，按既有协议使用，但单位仍应明确为毫秒。
+ 轨迹点、航点、围栏点等涉及定位的数据，建议统一使用同一套经纬度精度。

---

## 6. 参数嵌套

+ 能平铺表达清楚的字段，优先平铺，其属性参数为保证后台的可读性，尽量平铺。
+ 字段天然属于同一对象时，再考虑嵌套分组。
+ `params` 内建议控制在 2 到 3 层以内，避免过深嵌套。
+ 数组优先使用对象数组，不使用多个并列数组表达同一组数据。
+ 不建议增加无语义中间层，如 `data`、`info`、`object`。

推荐：

+ `orientation.yaw_deg`
+ `point_status.current_point_index`
+ `targets[].range_m`

不推荐：

+ `data.base.info.status.value`
+ `lat_list` + `lon_list` + `speed_list`

---

## 7. 数组命名

+ 数组字段使用复数名词，如 `targets`、`points`、`modules`、`subs`。
+ 不使用多个并列数组表达同一对象集合。
+ 动态长度且需要校验的数组，增加 `<array_name>_count`。
+ 点位、边界、步骤等有顺序语义的数组元素，建议增加 `order`。

## 8. identifier 命名

+ 服务和事件的 `identifier` 统一使用小写英文 `snake_case`。
+ `identifier` 表达业务动作或事件语义，不表达传输方式。

示例：

+ `manual_ctrl`
+ `light_ctrl`
+ `task_dispatch`
+ `task_status_report`
+ `diag_result`

---

## 9. 存量字段收敛方向

| 当前写法 | 建议写法 |
| -------- | -------- |
| `taskId` | `task_id` |
| `taskName` | `task_name` |
| `routeId` | `route_id` |
| `cronExpression` | `cron_expression` |
| `executeStatus` | `execute_status` |
| `pointStatus` | `point_status` |
| `currentPointIndex` | `current_point_index` |
| `arriveTime` | `arrive_time_ms` |
| `targets_num` | `targets_count` |
| `tra_num` | `trajectories_count` |
| `battery_quantity` | `battery_count` |
| `temp` | `temperature_c` |
| `humi` | `humidity_percent` |
