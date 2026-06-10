## 4. 上行数据格式（设备 → 云端）
### 4.1 属性数据
#### 4.1.1 Jetson 状态
**Topic:** `/sys/${productKey}/${deviceName}/thing/property/status_jetson`

**对应实际 Topic:** `/sys/jetson/jetson_01/thing/property/status_jetson`

```json
{
  "time": 1703123456789,
  
    "cpu_usage_percent": 42.5,
    "memory_usage_percent": 68.3,
    "gpu_usage_percent": 27.8,
    "temperature_c": 72.4,
    "uptime_ms": 86400000,
    "disk_usage_percent": 55.2
  
}
```

| 字段/位置 | 说明 |
| --- | --- |
| `cpu_usage_percent` | CPU 占用率 % |
| `memory_usage_percent` | 内存占用率 % |
| `gpu_usage_percent` | GPU 占用率 % |
| `temperature_c` | 温度(摄氏度) |
| `uptime_ms` | 上电时长（毫秒） |
| `disk_usage_percent` | 磁盘占用率 % |


#### 4.1.2 推进器数据
**Topic:** `/sys/${productKey}/${deviceName}/thing/property/thruster_status`

**对应实际 Topic:** `/sys/thruster/thruster_01/thing/property/thruster_status`

```json
{
  "time": 1703123456789,

   "id":"left",
   "rotate_speed_rpm":1300,
      "power_per":85,
        "torque_per":75,
        "electric_lift_per":65,
        "gear":0,
        "estop":0,
        "cooling_water_temp_C":36.5,
        "engine_oil_temp_C":36.5,
        "engine_oil_pressure_pa":1100,
        "battery_V":96  
  
}
```

![](https://cdn.nlark.com/yuque/0/2026/png/35874052/1780043564913-71910752-f156-44b7-b4cf-b3aa9e8f85e8.png)

#### 4.1.3 IMU 数据
**Topic:** `/sys/${productKey}/${deviceName}/thing/property/apm_imu`

**对应实际 Topic:** `/sys/apm/apm_01/thing/property/apm_imu`

```json
{
  "time": 1703123456789,

    "orientation": {
      "yaw_deg": 120.1,
      "roll_deg": 1.2,
      "pitch_deg": 0.5
    },
    "angular_velocity": {
      "yaw_rate_dps": 0.5,
      "roll_rate_dps": 0.1,
      "pitch_rate_dps": 0.05
    },
    "linear_acceleration": {
      "x_mps2": 0.01,
      "y_mps2": 0.02,
      "z_mps2": 9.81
    }
  
}
```

#### 4.1.4 毫米波雷达数据
**Topic:** `/sys/${productKey}/${deviceName}/thing/property/radar_mm`

**对应实际 Topic:** `/sys/radar_mm/radar_mm_01/thing/property/radar_mm`

```json
{
  "time": 1779272255654,


    "targets": [
      {
        "x": 3.2899999618530273,
        "y": -0.10999999940395355,
        "v_x": 0,
        "v_y": 0,
        "size_w": 0.8899999856948853,
        "size_l": 2.8299999237060547,
        "size_h": 0.3400000035762787,
        "objmotion_status": 0,
        "track_id": 11
      },
      {
        "x": 6.489999771118164,
        "y": 1,
        "v_x": 0,
        "v_y": 0,
        "size_w": 0.6800000071525574,
        "size_l": 1.4499999284744263,
        "size_h": 0.3499999940395355,
        "objmotion_status": 0,
        "track_id": 4
      }
    ]
  
}
```

#### 4.1.5 导航雷达扫描数据
**Topic:** `/sys/${productKey}/${deviceName}/thing/property/radar_nav`

**对应实际 Topic:** `/sys/radar_nav/radar_nav_01/thing/property/radar_nav`

```json
{
  "time": 1703123456789,
    "timestamps": [
      {
        "name": "scan_start",
        "time": 1703123456000
      },
      {
        "name": "scan_end",
        "time": 1703123456700
      },
      {
        "name": "signal_processing_end",
        "time": 1703123456789
      }
    ],
    "targets_num": 2,
    "targets": [
      {
        "range_m": 45.2,
        "bearing_deg": 32.5,
        "intensity": 0.92,
        "velocity_mps": 6.8
      },
      {
        "range_m": 78.0,
        "bearing_deg": 120.3,
        "intensity": 0.45,
        "velocity_mps": -2.3
      }
    ]
  
}
```

| 字段/位置 | 说明 |
| --- | --- |
| `timestamps` | 流程各时间戳数组 |
| `timestamps[].name` | 流程阶段名称 |
| `timestamps[].time` | 流程阶段时间，单位：毫秒 |
| `targets_num` | 目标数量 |
| `targets` | 追踪目标列表 |
| `targets[].range_m` | 目标距离，单位：米 |
| `targets[].bearing_deg` | 目标方位角，单位：度 |
| `targets[].intensity` | 回波/目标强度 |
| `targets[].velocity_mps` | 目标径向速度，单位：m/s |


#### 4.1.6 导航雷达地图
**Topic:** `/sys/${productKey}/${deviceName}/thing/property/radar_nav_map`

**对应实际 Topic:** `/sys/radar_nav/radar_nav_01/thing/property/radar_nav_map`

```json
{
  "time": 1703123456789,
    "map_id": "radar_local_001",
    "frame_id": "base_link",
    "width": 200,
    "height": 200,
    "resolution_m": 0.5,
    "origin": {
      "x": -50.0,
      "y": -50.0
    },
    "encoding": "rle",
    "cells_len": 100,
    "cells": "AAECAwQFBgc..."
  
}
```

| 字段/位置 | 说明 |
| --- | --- |
| `map_id` | 地图编号 |
| `frame_id` | 坐标系 |
| `width` | 栅格宽度 |
| `height` | 栅格高度 |
| `resolution_m` | 栅格分辨率（米/格） |
| `origin` | 原点位置 |
| `encoding` | 数据编码方式: "raw"/"rle" |
| `cells` | 栅格数据（可压缩后字符串） |


| 参数 | 类型 | 说明 |
| --- | --- | --- |
| map_id | string | 地图编号 |
| frame_id | string | 坐标系 |
| width | int | 栅格宽度 |
| height | int | 栅格高度 |
| resolution_m | float | 栅格分辨率（米/格） |
| origin | object | 原点位置 |
| encoding | string | 数据编码方式：raw/rle |
| cells | string | 栅格数据（可压缩后字符串） |


#### 4.1.7 融合轨迹
**Topic:** `/sys/${productKey}/${deviceName}/thing/property/perception_trajectory`

**对应实际 Topic:** `/sys/jetson/jetson_01/thing/property/perception_trajectory`

```json
{
  "time": 1703123456789,
    "tra_num": 1,
    "trajectories": [
      {
        "track_id": 101,
        "object_type": "vehicle",
        "points": [
          {
            "lat": 31.1256789,
            "lon": 121.1256789,
            "timestamp": 1703123456000,
            "speed_mps": 5.2,
            "heading_deg": 90.0
          }
        ]
      }
    ]
  
}
```

| 字段/位置 | 说明 |
| --- | --- |
| `tra_num` | 轨迹数量 |
| `trajectories` | 轨迹列表 |
| `trajectories[].track_id` | 目标跟踪 ID |
| `trajectories[].object_type` | 目标类型 |
| `trajectories[].points` | 轨迹点集 |
| `trajectories[].points[].lat` | 轨迹点纬度 |
| `trajectories[].points[].lon` | 轨迹点经度 |
| `trajectories[].points[].timestamp` | 轨迹点对应观测时间，单位：毫秒 |
| `trajectories[].points[].speed_mps` | 轨迹点速度，单位：m/s |
| `trajectories[].points[].heading_deg` | 轨迹点航向角，单位：度 |


| 参数 | 类型 | 说明 |
| --- | --- | --- |
| track_id | int | 目标跟踪ID |
| object_type | string | 目标类型（vehicle/pedestrian/buoy/obstacle/unknown） |
| points | array | 轨迹点集 |


#### 4.1.8 GPS 状态数据（MCU）
**Topic:** `/sys/${productKey}/${deviceName}/thing/property/gps_status`

**对应实际 Topic:** `/sys/gps/gps_01/thing/property/gps_status`

```json
{
  "time": 1703123456789,

    "fix_type": 4,
    "satellites": 24,
    "hdop": 0.8,
    "vdop": 1.2,
    "pdop": 1.5,
    "diff_age": 0.5,
    "lat": 32.12345678,
    "lon": 118.1234567,
    "alt_m": 5.123,
    "heading_deg": 78.9,
    "ground_speed_mps": 2.4,
    "knots": 4.6
  
}
```

| fix_type | 说明 |
| --- | --- |
| 0 | 无效解 |
| 1 | 单点定位 |
| 2 | 伪距差分 |
| 4 | RTK 固定解 |
| 5 | RTK 浮点解 |


#### 4.1.9 气象站状态数据（MCU）
**Topic:** `/sys/${productKey}/${deviceName}/thing/property/weather_status`

**对应实际 Topic:** `/sys/weather/weather_01/thing/property/weather_status`

```json
{
  "time": 1703123456789,
    "temp_c": 25.9,
    "humidity_percent": 67.1,
    "pressure_hpa": 1000.2,
    "wind_speed_mps": 3.5,
    "wind_direction_deg": 120.0
  
}
```

| 字段/位置 | 说明 |
| --- | --- |
| `temp_c` | 温度(°C) |
| `humidity_percent` | 湿度(%) |
| `pressure_hpa` | 气压(hPa) |
| `wind_speed_mps` | 风速(m/s) |
| `wind_direction_deg` | 风向(度) |


#### 4.1.10 测深仪状态数据（MCU）
**Topic:** `/sys/${productKey}/${deviceName}/thing/property/depth_status`

**对应实际 Topic:** `/sys/depth/depth_01/thing/property/depth_status`

```json
{
  "time": 1703123456789,
 

    "position": {
      "lat": 30.1234567,
      "lon": 114.1234567,
      "alt_m": 0
    },
    "water_depth": {
      "depth_m": 12.37,
      "offset_m": 0.45,
      "confidence": 0.98
    }
  
}
```

| 字段/位置 | 说明 |
| --- | --- |
| `lat` | 纬度 |
| `lon` | 经度 |
| `alt_m` | 高程,可为null,单位：m |
| `depth_m` | 水深(米) |
| `offset_m` | 偏移(米) |
| `confidence` | 置信度 |


| 参数 | 类型 | 说明 |
| --- | --- | --- |
| depth_m | float | 相对于换能器的水深 |
| offset_m | float | 吃水深度 |
| confidence | float | 置信度 |


#### 4.1.11 电池状态数据（MCU）
**Topic:** `/sys/${productKey}/${deviceName}/thing/property/battery_status`

**对应实际 Topic:** `/sys/battery/battery_01/thing/property/battery_status`

```json
{
  "time": 1703123456789,


    "battery_quantity": 3,
    "battery_info": [
      {
        "battery_id": 1,
        "battery_name": "main_battery",
        "current_a": 18.62,
        "voltage_v": 48.34,
        "power_w": 900.68
      },
      {
        "battery_id": 2,
        "battery_name": "core_battery",
        "current_a": 6.62,
        "voltage_v": 48.34,
        "power_w": 300.68
      },
      {
        "battery_id": 3,
        "battery_name": "power_battery",
        "current_a": 6.62,
        "voltage_v": 48.34,
        "power_w": 300.68
      }
    ]
  
}
```

#### 4.1.12 油箱状态数据（MCU）
**Topic:** `/sys/${productKey}/${deviceName}/thing/property/fuel_status`

**对应实际 Topic:** `/sys/fuel/fuel_01/thing/property/fuel_status`

```json
{
  "time": 1703123456789,
    "level_percent": 56,
    "volume_liter": 45,
    "capacity_liter": 60,
    "temperature_c": 23.9,
    "pressure_bar": 1.01
  
}
```

#### 4.1.13 MCU 系统状态（MCU）
**Topic:** `/sys/${productKey}/${deviceName}/thing/property/mcu_status`

**对应实际 Topic:** `/sys/mcu/mcu_01/thing/property/mcu_status`

```json
{
  "time": 1703123456789,
    "version": "1.1.0",
    "uptime_s": 86400,
    "rssi_dbm": -68,
    "current_link": "5G",
    "gnss_status": "valid",
    "control_mode": "auto",
    "armed_status": "armed",
    "estop": false,
    "mqtt_online": true,
    "temp":26.5,
    "humi":85.0
  
}
```

| 字段/位置 | 说明 |
| --- | --- |
| `version` | 协议版本 |
| `uptime_s` | 上电时长（秒） |
| `rssi_dbm` | 信号强度（dBm） |
| `current_link` | 当前链路 |
| `gnss_status` | GNSS 状态 |
| `control_mode` | 当前模式 |
| `armed_status` | 设防状态 |
| `estop` | 是否急停 |
| `mqtt_online` | MQTT 是否在线 |
| temp | 温度 |
| humi | 湿度 |




#### 4.1.14 AIS 数据（MCU）
**Topic:** `/sys/${productKey}/${deviceName}/thing/property/ais`

**对应实际 Topic:** `/sys/ais/ais_01/thing/property/ais`

```json
{
  "time": 1703123456789,
    "ais_stream_num": 99,
    "ais_stream": "!AIVDM,1,1,,A,15Muq@?P00PD;88MD5MTDwwT0<0u,0*5C"
  
}
```

#### 4.1.15 IO 设备状态（MCU）
**Topic:** `/sys/${productKey}/${deviceName}/thing/property/io_status`

**对应实际 Topic:** `/sys/io/io_01/thing/property/io_status`

```json
{
  "time": 1703123456789,


   
        "light_left_status": "on",
      
        "light_right_status": "on",

        "light_mast_status": "on",
    
        "light_stern_status": "on",
   
        "light_signal_status": "off",
  
        "tilt_control_status": "off",
    
        "thruster_left_status": "on",
        "thruster_right_status": "on",
   
        "air_conditioner_status": "on"
      
    
  
}
```

#### 4.1.16 心跳状态
**Jetson Topic:** `/sys/${productKey}/${deviceName}/thing/event/jetson_heartbeat`

**Jetson 对应实际 Topic:** `/sys/jetson/jetson_01/thing/event/jetson_heartbeat`

**Jetson 心跳：**

```json
{
  "time": 1703123456789,
    "online": true,
    "unit": "jetson"
  
}
```

**MCU Topic:** `/sys/${productKey}/${deviceName}/thing/event/mcu_heartbeat`

**MCU 对应实际 Topic:** `/sys/mcu/mcu_01/thing/event/mcu_heartbeat`

**MCU 心跳：**

```json
{
  "time": 1703123456789,
    "online": true,
    "unit": "mcu"

}
```

**APM飞控 Topic:** `/sys/${productKey}/${deviceName}/thing/event/apm_heartbeat`

**APM飞控 对应实际 Topic:** `/sys/apm/apm_01/thing/event/apm_heartbeat`

**APM飞控  心跳：**

```json
{
  "time": 1703123456789,
    "online": true,
    "unit": "apm"

}
```

| 参数 | 类型 | 说明 |
| --- | --- | --- |
| online | bool | 在线状态 |
| unit | string | 单元标识：jetson/mcu |


#### 4.1.17 APM飞控状态
**Topic:** `/sys/${productKey}/${deviceName}/thing/property/status_apm`

**对应实际 Topic:** `/sys/apm/apm_01/thing/property/status_apm`

```json
{
  "time": 1703123456789,

    "connected": "false"
    "armed": "false"
    "guided": "false"
    "manual_input": "false"
    "mode": "hode"
    "system_status": 0
  
}
```

![](https://cdn.nlark.com/yuque/0/2026/png/35874052/1779184347846-bf7bf51e-9880-4f97-867a-6a5974c851d5.png)





### 4.2 事件数据
#### 4.2.1 报警
**Topic:** `/sys/${productKey}/${deviceName}/thing/event/alarm`

**对应实际 Topic:**

+ Jetson：`/sys/jetson/jetson_01/thing/event/alarm`
+ MCU：`/sys/mcu/mcu_01/thing/event/alarm`

```json
{
  "time": 1703123456789,
    "event_id": "evt-001",
    "error_name": "SENSOR_AIS_FAIL",
    "error_level": "critical"
  
}
```

| 字段/位置 | 说明 |
| --- | --- |
| `event_id` | 事件ID |
| `error_name` | 故障名 |
| `error_level` | 故障等级 |


故障等级表

| error_level | 说明 |
| --- | --- |
| info | 信息提示 |
| warning | 警告，可继续运行但需要关注 |
| critical | 严重故障，需要立即处理 |


#### 4.2.2 航点信息更新
**Topic:** `/sys/${productKey}/${deviceName}/thing/event/mission_delta`

**对应实际 Topic:** `/sys/jetson/jetson_01/thing/event/mission_delta`

```json
{
  "time": 1703123456789,

    "mission_id": "M20250122_001",
    "operation": "update",
    "waypoint": {
      "index": 2,
      "lat": 31.1256789,
      "lon": 121.1256789,
      "radius_m": 5.0,
      "speed_mps": 4.0
    
  }
}
```

#### 4.2.3 视频流状态
**Topic:** `/sys/${productKey}/${deviceName}/thing/event/aivideo_status`

**对应实际 Topic:** `/sys/aivideo/aivideo_01/thing/event/aivideo_status`

```json
{
  "time": 1703123456789,
    "streaming": true,
    "camera_id": "front",
    "url": "rtmp://push.example.com/live/boat001"
  
}
```

#### 4.2.4 视觉目标
**Topic:** `/sys/${productKey}/${deviceName}/thing/event/aivision_targets`

**对应实际 Topic:** `/sys/aivision/aivision_01/thing/event/aivision_targets`

```json
{
  "time": 1703123456789,

    "timestamps": [
      {
        "name": "image_capture",
        "time_ms": 1703123456000
      },
      {
        "name": "preprocessing_start",
        "time_ms": 1703123456100
      },
      {
        "name": "inference_end",
        "time_ms": 1703123456780
      }
    ],
    "targets_num": 1,
    "targets": [
      {
        "class_name": "buoy",
        "confidence": 0.96,
        "bbox": {
          "x": 120,
          "y": 80,
          "width": 200,
          "height": 150
        },
        "rel_ang_deg": 31.1257
      }
    ]
  
}
```

| 字段/位置 | 说明 |
| --- | --- |
| `timestamps` | 处理流程主要时间戳数组 |
| `timestamps[].name` | 处理阶段名称 |
| `timestamps[].time` | 处理阶段时间，单位：毫秒 |
| `targets_num` | 目标数量 |
| `targets` | 检测目标列表 |
| `targets[].class` | 目标类别 |
| `targets[].confidence` | 置信度，范围 0~1 |
| `targets[].bbox` | 目标边界框 |
| `targets[].rel_ang` | 相对角度，单位：度 |


| 参数 | 类型 | 说明 |
| --- | --- | --- |
| timestamps | array | 处理流程各阶段时间戳 |
| targets_num | int | 目标数量 |
| targets | array | 检测目标列表 |
| class | string | 目标类别（buoy/ship/boat/obstacle/person） |
| confidence | float | 置信度（0-1） |
| bbox | object | 边界框（x,y,width,height） |
| rel_ang | float | 相对角度（度） |


#### 4.2.5 自检结果（MCU）
**Topic:** `/sys/${productKey}/${deviceName}/thing/event/diag_result`

**对应实际 Topic:** `/sys/mcu/mcu_01/thing/event/diag_result`

```json
{
  "time": 1703123456789,
 
    "result": "pass",
    "modules": [
      {
        "name": "imu",
        "status": "pass",
        "message": ""
      },
      {
        "name": "gps",
        "status": "fail",
        "message": "RTK固定解未收敛"
      },
      {
        "name": "thruster",
        "status": "pass",
        "message": ""
      }
    ],
    "summary": {
      "total": 3,
      "pass": 2,
      "fail": 1
    }
  
}
```

---
