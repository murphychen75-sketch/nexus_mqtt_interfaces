# Jetson 处理域 MQTT 接口规范

> 版本：1.1（命名规范对齐版）  
> 日期：2026-06-29  
> 依据：[Nexus协议命名规范.md](../Nexus协议命名规范.md)  
> 来源：从 [protocol.md](protocol.md) Jetson 处理域章节抽取并按命名规范调整

---

## 1. 适用范围

本文档覆盖 **Jetson 处理域** 及其关联子设备在 MQTT 上的接口定义，包括：

| productKey | deviceName | 说明 |
| --- | --- | --- |
| `jetson` | `jetson_01` | Jetson 主控 |
| `cam` | `cam_01` | 相机 |
| `vision` | `vision_01` | 视觉识别 |
| `radar_mm` | `radar_mm_01` | 毫米波雷达 |

### 1.1 与主协议的关系

+ Topic 路径遵循命名规范：`/sys/{productKey}/{deviceName}/thing/{domain}/{action}`，业务类型通过 `method` 与 `params.identifier` 区分，**不再**将业务标识写入 Topic 路径。
+ 消息体采用与 [无人船通信协议 V2.0](无人船通信协议1.1-629.md) 一致的统一外层包裹格式。
+ 服务调用业务参数置于 `params.inputParams`；事件业务参数置于 `params.value`；属性业务参数直接置于 `params`（可含 `identifier` 区分属性类型）。
+ `inputParams`、`value` 及属性 `params` 内业务字段统一使用 `snake_case`；枚举与状态值使用小写英文。

### 1.2 统一消息结构

所有 MQTT 消息采用统一外层包裹格式：

```json
{
  "id": "msg-uuid-xxx",
  "reportTime": 1703123456789,
  "deviceId": "jetson.jetson_01",
  "tenantId": "tenant_xxx",
  "requestId": "req-uuid-xxx",
  "method": "thing.service.invoke",
  "params": {}
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `id` | string | 消息唯一标识（UUID） |
| `reportTime` | int64 | 消息生成/发布时间戳，单位：毫秒 |
| `deviceId` | string | 设备唯一标识，格式 `{productKey}.{deviceName}` |
| `tenantId` | string | 租户标识 |
| `requestId` | string | 请求唯一标识（UUID）；属性/事件上报可为空字符串 |
| `method` | string | 方法名，见各 Topic 对应 method |
| `params` | object | 业务参数，不同 method 对应不同结构 |

**响应消息额外字段**：

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `code` | int | 状态码，`0` 或 `200` 表示成功 |
| `msg` | string | 响应描述信息 |

### 1.3 通用约定

| 字段 | 约定 |
| --- | --- |
| `seq` | 按每个 Topic 独立递增，从 1 开始 |
| `reportTime` | Payload 生成/发布时间，单位：毫秒 |
| `params.inputParams.sample_time_ms` | 传感器或执行器原始采样时间，单位：毫秒 |
| `params.value.timestamps[].time_ms` | 算法流水线阶段时间，单位：毫秒 |
| 经纬度 | 十进制度数，`lat` 表纬度，`lon` 表经度，建议 7~8 位小数 |

---

## 2. Topic 总览

### 2.1 Jetson 主控（`jetson/jetson_01`）

#### 下行服务（云 → 设备）

| identifier | Topic | Method | QoS | 说明 |
| --- | --- | --- | --- | --- |
| `estop` | `/sys/jetson/jetson_01/thing/service/invoke` | `thing.service.invoke` | 1 | 急停指令 |
| `arm` | 同上 | 同上 | 1 | 设防/解锁 |
| `mode` | 同上 | 同上 | 1 | 模式切换 |
| `auto_task` | 同上 | 同上 | 1 | 自动任务 |
| `apm_params` | 同上 | 同上 | 1 | APM 参数下发 |

#### 上行服务回复（设备 → 云）

| identifier | Topic | QoS | 说明 |
| --- | --- | --- | --- |
| `estop` | `/sys/jetson/jetson_01/thing/service/invoke_reply` | 1 | 急停指令回复 |
| `arm` | 同上 | 1 | 设防/解锁回复 |
| `mode` | 同上 | 1 | 模式切换回复 |
| `auto_task` | 同上 | 1 | 自动任务回复 |
| `apm_params` | 同上 | 1 | APM 参数下发回复 |

#### 上行事件（设备 → 云）

| identifier | Topic | Method | 频率 | QoS | 说明 |
| --- | --- | --- | --- | --- | --- |
| `alarm` | `/sys/jetson/jetson_01/thing/event/post` | `thing.event.post` | 实时 | 1 | 报警信息 |
| `heartbeat` | 同上 | 同上 | 1 Hz | 0 | Jetson 心跳 |
| `mission_delta` | 同上 | 同上 | 实时 | 1 | 航点信息更新 |

#### 上行属性（设备 → 云）

| identifier | Topic | Method | 频率 | QoS | 说明 |
| --- | --- | --- | --- | --- | --- |
| `status_jetson` | `/sys/jetson/jetson_01/thing/property/post` | `thing.property.post` | 中低频 | 0 | Jetson 系统状态 |
| `perception_trajectory` | 同上 | 同上 | 实时 | 0 | 融合感知轨迹 |

### 2.2 关联子设备

#### 相机（`cam/cam_01`）

##### 下行服务（云 → 设备）

| identifier | Topic | Method | QoS | 说明 |
| --- | --- | --- | --- | --- |
| `cam_ctrl` | `/sys/cam/cam_01/thing/service/invoke` | `thing.service.invoke` | 1 | 相机流控制 |

##### 上行服务回复（设备 → 云）

| identifier | Topic | QoS | 说明 |
| --- | --- | --- | --- |
| `cam_ctrl` | `/sys/cam/cam_01/thing/service/invoke_reply` | 1 | 相机流控制回复 |

##### 上行事件（设备 → 云）

| identifier | Topic | Method | QoS | 说明 |
| --- | --- | --- | --- | --- |
| `cam_status` | `/sys/cam/cam_01/thing/event/post` | `thing.event.post` | 1 | 相机流状态 |

#### 视觉识别（`vision/vision_01`）

| 方向 | identifier | Topic | Method | QoS | 说明 |
| --- | --- | --- | --- | --- | --- |
| 上行 | `vision_detections` | `/sys/vision/vision_01/thing/event/post` | `thing.event.post` | 1 | 视觉识别目标（1~5 Hz） |

#### 毫米波雷达（`radar_mm/radar_mm_01`）

| 方向 | identifier | Topic | Method | QoS | 说明 |
| --- | --- | --- | --- | --- | --- |
| 上行 | `radar_mm` | `/sys/radar_mm/radar_mm_01/thing/property/post` | `thing.property.post` | 0 | 毫米波目标（20 Hz） |

---

## 3. 下行数据格式（云 → 设备）

### 3.1 通用服务调用格式

**Topic：** `/sys/{productKey}/{deviceName}/thing/service/invoke`

```json
{
  "id": "req-uuid-001",
  "reportTime": 1703123456789,
  "deviceId": "jetson.jetson_01",
  "tenantId": "tenant_xxx",
  "requestId": "req-uuid-001",
  "method": "thing.service.invoke",
  "params": {
    "identifier": "estop",
    "inputParams": {
      "estop": true,
      "source_type": "shore"
    }
  }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `identifier` | string | 服务标识符，区分不同指令类型 |
| `inputParams` | object | 服务输入参数，由具体 identifier 决定结构 |

### 3.2 通用服务回复格式

**Topic：** `/sys/{productKey}/{deviceName}/thing/service/invoke_reply`

```json
{
  "id": "resp-uuid-001",
  "reportTime": 1703123456800,
  "deviceId": "jetson.jetson_01",
  "tenantId": "tenant_xxx",
  "requestId": "req-uuid-001",
  "method": "thing.service.invoke",
  "params": {
    "identifier": "estop",
    "inputParams": {
      "estop": true,
      "source_type": "shore"
    }
  },
  "code": 0,
  "msg": "success"
}
```

### 3.3 急停（`estop`）

**设备：** `jetson/jetson_01`

```json
{
  "id": "req-xxx",
  "reportTime": 1703123456789,
  "deviceId": "jetson.jetson_01",
  "tenantId": "tenant_xxx",
  "requestId": "req-xxx",
  "method": "thing.service.invoke",
  "params": {
    "identifier": "estop",
    "inputParams": {
      "estop": true,
      "source_type": "shore"
    }
  }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `inputParams.estop` | bool | `true` 紧急停止，`false` 恢复 |
| `inputParams.source_type` | string | 来源端，如 `shore`（岸端）、`cloud`（云端） |

### 3.4 设防/解锁（`arm`）

**设备：** `jetson/jetson_01`

```json
{
  "id": "req-xxx",
  "reportTime": 1703123456789,
  "deviceId": "jetson.jetson_01",
  "tenantId": "tenant_xxx",
  "requestId": "req-xxx",
  "method": "thing.service.invoke",
  "params": {
    "identifier": "arm",
    "inputParams": {
      "armed_action": "arm"
    }
  }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `inputParams.armed_action` | string | `arm` 设防（上锁/开启警戒），`disarm` 解锁（解除警戒/撤防） |

### 3.5 模式切换（`mode`）

**设备：** `jetson/jetson_01`

```json
{
  "id": "req-xxx",
  "reportTime": 1703123456789,
  "deviceId": "jetson.jetson_01",
  "tenantId": "tenant_xxx",
  "requestId": "req-xxx",
  "method": "thing.service.invoke",
  "params": {
    "identifier": "mode",
    "inputParams": {
      "mode": "auto"
    }
  }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `inputParams.mode` | string | `auto`、`manual`、`rtl`、`dock`、`follow`、`loiter`、`hold` |

### 3.6 自动任务（`auto_task`）

**设备：** `jetson/jetson_01`

```json
{
  "id": "req-xxx",
  "reportTime": 1703123456789,
  "deviceId": "jetson.jetson_01",
  "tenantId": "tenant_xxx",
  "requestId": "req-xxx",
  "method": "thing.service.invoke",
  "params": {
    "identifier": "auto_task",
    "inputParams": {
      "cmd": "start",
      "task_id": "TASK_001",
      "waypoints_count": 2,
      "mission_type": 16,
      "home_lat": 31.1234567,
      "home_lon": 121.1234567,
      "home_type": 1,
      "waypoints": [
        {
          "lat": 31.1234567,
          "lon": 121.1234567,
          "order": 1
        },
        {
          "lat": 31.1234570,
          "lon": 121.1234570,
          "order": 2
        }
      ],
      "mode": "auto"
    }
  }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `inputParams.cmd` | string | 任务操作：`start` / `stop` |
| `inputParams.task_id` | string | 任务流水号 |
| `inputParams.waypoints_count` | int | 航点数量 |
| `inputParams.mission_type` | int | 任务类型 |
| `inputParams.home_lat` / `home_lon` | float | 返航点经纬度 |
| `inputParams.home_type` | int | 返航点类型 |
| `inputParams.waypoints` | array | 航点列表，元素含 `lat`、`lon`、`order` |
| `inputParams.mode` | string | 路径执行模式 |

### 3.7 APM 参数下发（`apm_params`）

**设备：** `jetson/jetson_01`

```json
{
  "id": "req-xxx",
  "reportTime": 1703123456789,
  "deviceId": "jetson.jetson_01",
  "tenantId": "tenant_xxx",
  "requestId": "req-xxx",
  "method": "thing.service.invoke",
  "params": {
    "identifier": "apm_params",
    "inputParams": {
          "param_id": "MAX_SPEED",
          "value":
            {
              "integer": 0,
              "real": 5.0
            }
        }
    }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `inputParams.param_id` | string | 参数名 |
| `inputParams.value[].integer` | int | 整数值 （参数为整数时real字段写0）|
| `inputParams.value[].real` | float | 浮点数 （参数为浮点数时integer字段写0） |

### 3.8 相机控制（`cam_ctrl`）

**设备：** `cam/cam_01`

```json
{
  "id": "req-xxx",
  "reportTime": 1703123456789,
  "deviceId": "cam.cam_01",
  "tenantId": "tenant_xxx",
  "requestId": "req-xxx",
  "method": "thing.service.invoke",
  "params": {
    "identifier": "cam_ctrl",
    "inputParams": {
      "cmd": "start",
      "camera_id": "front",
      "resolution": "1920x1080",
      "fps": 30,
      "bitrate_kbps": 4096
    }
  }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `inputParams.cmd` | string | `start` / `stop` |
| `inputParams.camera_id` | string | 相机标识 |
| `inputParams.resolution` | string | 分辨率 |
| `inputParams.fps` | int | 帧率 |
| `inputParams.bitrate_kbps` | int | 码率（kbps） |

---

## 4. 上行数据格式（设备 → 云）

### 4.1 通用属性上报格式

**Topic：** `/sys/{productKey}/{deviceName}/thing/property/post`

```json
{
  "id": "prop-xxx",
  "reportTime": 1703123456789,
  "deviceId": "jetson.jetson_01",
  "tenantId": "tenant_xxx",
  "requestId": "",
  "method": "thing.property.post",
  "params": {
    "identifier": "status_jetson",
    "cpu_usage_percent": 42.5,
    "memory_usage_percent": 68.3,
    "gpu_usage_percent": 27.8,
    "temperature_c": 72.4
  }
}
```

### 4.2 通用事件上报格式

**Topic：** `/sys/{productKey}/{deviceName}/thing/event/post`

```json
{
  "id": "evt-xxx",
  "reportTime": 1703123456789,
  "deviceId": "jetson.jetson_01",
  "tenantId": "tenant_xxx",
  "requestId": "",
  "method": "thing.event.post",
  "params": {
    "identifier": "alarm",
    "value": {
      "event_id": "evt-001",
      "error_name": "SENSOR_AIS_FAIL",
      "error_level": "critical"
    }
  }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `identifier` | string | 事件标识符 |
| `value` | object | 事件数据 |

> 事件发生时间使用外层 `reportTime`，`params` 内不再重复携带 `time` 字段。

### 4.3 Jetson 系统状态（`status_jetson`）

**设备：** `jetson/jetson_01`

```json
{
  "id": "prop-xxx",
  "reportTime": 1703123456789,
  "deviceId": "jetson.jetson_01",
  "tenantId": "tenant_xxx",
  "requestId": "",
  "method": "thing.property.post",
  "params": {
    "identifier": "status_jetson",
    "cpu_usage_percent": 42.5,
    "memory_usage_percent": 68.3,
    "gpu_usage_percent": 27.8,
    "temperature_c": 72.4,
    "uptime_ms": 86400000,
    "disk_usage_percent": 55.2
  }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `cpu_usage_percent` | float | CPU 占用率（%） |
| `memory_usage_percent` | float | 内存占用率（%） |
| `gpu_usage_percent` | float | GPU 占用率（%） |
| `temperature_c` | float | 温度（℃） |
| `uptime_ms` | int | 上电时长（毫秒） |
| `disk_usage_percent` | float | 磁盘占用率（%） |

### 4.4 毫米波雷达（`radar_mm`）

**设备：** `radar_mm/radar_mm_01`

```json
{
  "id": "prop-xxx",
  "reportTime": 1779272255654,
  "deviceId": "radar_mm.radar_mm_01",
  "tenantId": "tenant_xxx",
  "requestId": "",
  "method": "thing.property.post",
  "params": {
    "targets": [
      {
        "x": 12.2,
        "y": 23.2,
        "z": 1.23,
        "width": 1.2,
        "length": 3.45,
        "heigth": 1.23,
        "xvel_abs": 1.2,
        "yvel_abs": 2.3,
        "xacc_abs": 0.1,
        "yacc_abs": 0.8,
        "heading_angle": 24.4,
        "classify_type": 1,
        "classfiy_prob": 0.89,
        "objmotion_status": 1,
        "obstacle_prob": 0.123,
        "track_id": 11
      }
    ]
  }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `targets[].x` | float | 目标中心纵向位置 (m) |
| `targets[].y` | float | 目标中心横向位置 (m) |
| `targets[].z` | float | 目标中心垂向位置 (m) |
| `targets[].width` | float | 目标包围框宽度（m） |
| `targets[].length` | float | 目标包围框长度（m） |
| `targets[].height` | float | 目标包围框高度（m）|
| `targets[].xvel_abs` | float | 目标纵向绝对速度 (m/s) |
| `targets[].yvel_abs` | float | 目标横向绝对速度 (m/s) |
| `targets[].xacc_abs` | float | 目标纵向绝对加速度 (m/s/s) |
| `targets[].yacc_abs` | float | 目标横向绝对加速度 (m/s/s) |
| `targets[].heading_angle` | float | 目标航向角（+左 -右） |
| `targets[].classify_type` | float | #目标类别  0未知目标 1行人 2自行车 3小汽车 4大卡车 (移植于车载雷达，相关目标分类算法还未完成船端迁移) |
| `targets[].classify_prob` | float | 目标分类概率 |
| `targets[].objmotion_status` | float | # 动静状态: 0 静止, 1 运动 |
| `targets[].obstacle_prob` | float | 障碍物概率 |
| `targets[].track_id` | float | 跟踪 ID |

### 4.5 融合感知轨迹（`perception_trajectory`）

**设备：** `jetson/jetson_01`

```json
{
  "id": "prop-xxx",
  "reportTime": 1703123456789,
  "deviceId": "jetson.jetson_01",
  "tenantId": "tenant_xxx",
  "requestId": "",
  "method": "thing.property.post",
  "params": {
    "identifier": "perception_trajectory",
    "trajectories_count": 1,
    "trajectories": [
      {
        "track_id": 101,
        "object_type": "vehicle",
        "points": [
          {
            "lat": 31.1256789,
            "lon": 121.1256789,
            "capture_time_ms": 1703123456000,
            "speed_mps": 5.2,
            "heading_deg": 90.0,
            "order": 1
          }
        ]
      }
    ]
  }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `trajectories_count` | int | 轨迹数量 |
| `trajectories[].track_id` | int | 目标跟踪 ID |
| `trajectories[].object_type` | string | 目标类型（见附录） |
| `trajectories[].points[].lat` / `lon` | float | 轨迹点经纬度 |
| `trajectories[].points[].capture_time_ms` | int | 观测时间（毫秒） |
| `trajectories[].points[].speed_mps` | float | 速度（m/s） |
| `trajectories[].points[].heading_deg` | float | 航向角（°） |
| `trajectories[].points[].order` | int | 点序号 |

### 4.6 报警事件（`alarm`）

**设备：** `jetson/jetson_01`

```json
{
  "id": "evt-xxx",
  "reportTime": 1703123456789,
  "deviceId": "jetson.jetson_01",
  "tenantId": "tenant_xxx",
  "requestId": "",
  "method": "thing.event.post",
  "params": {
    "identifier": "alarm",
    "value": {
      "event_id": "evt-001",
      "error_name": "SENSOR_AIS_FAIL",
      "error_level": "critical"
    }
  }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `value.event_id` | string | 事件 ID |
| `value.error_name` | string | 故障名 |
| `value.error_level` | string | `info` / `warning` / `critical` |

### 4.7 心跳（`heartbeat`）

**设备：** `jetson/jetson_01`

```json
{
  "id": "evt-xxx",
  "reportTime": 1703123456789,
  "deviceId": "jetson.jetson_01",
  "tenantId": "tenant_xxx",
  "requestId": "",
  "method": "thing.event.post",
  "params": {
    "identifier": "heartbeat",
    "value": {
      "online": true,
      "unit_type": "jetson"
    }
  }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `value.online` | bool | 在线状态 |
| `value.unit_type` | string | 单元标识：`jetson` |

### 4.8 航点信息更新（`mission_delta`）

**设备：** `jetson/jetson_01`

```json
{
  "id": "evt-xxx",
  "reportTime": 1703123456789,
  "deviceId": "jetson.jetson_01",
  "tenantId": "tenant_xxx",
  "requestId": "",
  "method": "thing.event.post",
  "params": {
    "identifier": "mission_delta",
    "value": {
      "mission_id": "M20250122_001",
      "operation": "update",
      "waypoint_index": 2,
      "waypoint_lat": 31.1256789,
      "waypoint_lon": 121.1256789,
      "radius_m": 5.0,
      "speed_mps": 4.0
    }
  }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `value.mission_id` | string | 任务 ID |
| `value.operation` | string | 操作类型，如 `update` |
| `value.waypoint_index` | int | 航点索引 |
| `value.waypoint_lat` / `waypoint_lon` | float | 航点经纬度 |
| `value.radius_m` | float | 到达半径（m） |
| `value.speed_mps` | float | 航点速度（m/s） |

### 4.9 相机流状态（`cam_status`）

**设备：** `cam/cam_01`

```json
{
  "id": "evt-xxx",
  "reportTime": 1703123456789,
  "deviceId": "cam.cam_01",
  "tenantId": "tenant_xxx",
  "requestId": "",
  "method": "thing.event.post",
  "params": {
    "identifier": "cam_status",
    "value": {
      "streaming": true,
      "camera_id": "front",
      "stream_url": "rtmp://push.example.com/live/boat001"
    }
  }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `value.streaming` | bool | 是否正在推流 |
| `value.camera_id` | string | 相机标识 |
| `value.stream_url` | string | 推流地址 |

### 4.10 视觉识别目标（`vision_detections`）

**设备：** `vision/vision_01`

```json
{
  "id": "evt-xxx",
  "reportTime": 1703123456789,
  "deviceId": "vision.vision_01",
  "tenantId": "tenant_xxx",
  "requestId": "",
  "method": "thing.event.post",
  "params": {
    "identifier": "vision_detections",
    "value": {
      "timestamps": [
        { "stage_name": "image_capture", "time_ms": 1703123456000 },
        { "stage_name": "preprocessing_start", "time_ms": 1703123456100 },
        { "stage_name": "inference_end", "time_ms": 1703123456780 }
      ],
      "targets_count": 1,
      "targets": [
        {
          "class_name": "buoy",
          "confidence": 0.96,
          "bbox":
            {
              "x": 120,
              "y": 10,
              "width": 980,
              "height": 10,
            },
          "rel_ang_deg": 31.1257
        }
      ]
    }
  }
}
```

| 字段 | 类型 | 说明 |
| --- | --- | --- |
| `value.timestamps[].stage_name` | string | 处理阶段名称 |
| `value.timestamps[].time_ms` | int | 阶段时间（毫秒） |
| `value.targets_count` | int | 目标数量 |
| `value.targets[].class_name` | string | 目标类别（见附录） |
| `value.targets[].confidence` | float | 置信度 0~1 |
| `value.targets[].bbox.x` | int | 边界框x像素坐标 |
| `value.targets[].bbox.y` | int | 边界框y像素坐标 |
| `value.targets[].bbox.width` | int | 边界框宽度 |
| `value.targets[].bbox.height` | int | 边界框高度 |
| `value.targets[].rel_ang_deg` | float | 相对角度（°） |

---

## 5. 附录

### 5.1 对象类型（`object_type`）

| 值 | 说明 |
| --- | --- |
| `vehicle` | 船舶/车辆 |
| `pedestrian` | 行人 |
| `buoy` | 浮标 |
| `obstacle` | 障碍物 |
| `unknown` | 未知类型 |

### 5.2 视觉目标类别（`class_name`）

| 值 | 说明 |
| --- | --- |
| `boat` | 船只 |
| `ship` | 船舶 |
| `buoy` | 浮标 |
| `obstacle` | 障碍物 |
| `person` | 人员 |

### 5.2 毫米波雷达目标类别（`classify_name`）

| 值 | 说明 |
| --- | --- |
| `0` | 未知目标 |
| `1` | 行人 |
| `2` | 自行车 |
| `3` | 小汽车 |
| `4` | 大卡车 |


### 5.3 通信频率建议

| 数据类型 | 建议频率 | QoS |
| --- | --- | --- |
| 视觉识别目标 | 1~5 Hz | 1 |
| 毫米波雷达 | 20 Hz | 0 |
| 融合感知轨迹 | 实时 | 0 |
| 报警事件 | 触发时立即上报 | 1 |
| 心跳 | 1 Hz | 0 |
