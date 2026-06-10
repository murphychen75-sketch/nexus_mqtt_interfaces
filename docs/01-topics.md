## 2. 话题架构
### 2.1 系统主题模板
| 类型 | 主题模板 | 方向 | 默认 QoS |
| --- | --- | --- | --- |
| 服务下发 | `/sys/${productKey}/${deviceName}/thing/service/${identifier}` | 云 → 设备 | 1 |
| 服务回复 | `/sys/${productKey}/${deviceName}/thing/service/${identifier}_reply` | 设备 → 云 | 1 |
| 事件上报 | `/sys/${productKey}/${deviceName}/thing/event/${identifier}` | 设备 → 云 | 1 |
| 事件回复（保留，当前不使用） | `/sys/${productKey}/${deviceName}/thing/event/${identifier}_reply` | 云 → 设备 | 1 |
| 属性上报 | `/sys/${productKey}/${deviceName}/thing/property/${identifier}` | 设备 → 云 | 0 |
| 属性回复（保留，当前不使用） | `/sys/${productKey}/${deviceName}/thing/property/${identifier}_reply` | 云 → 设备 | 0 |


### 2.2 Jetson 处理域相关主题
> 注：本节按 Jetson 处理域归类，但实际 Topic 以“对应实际 Topic”列为准；视频、雷达、视觉、推进器、IMU、融合感知等可按独立 `productKey/deviceName` 上报。
>

#### 下行服务（云端 → Jetson）
| 主题模板 | 对应实际 Topic | 描述 | QoS |
| --- | --- | --- | --- |
| `/sys/${productKey}/${deviceName}/thing/service/estop` | `/sys/jetson/jetson_01/thing/service/estop` | 急停指令 | 1 |
| `/sys/${productKey}/${deviceName}/thing/service/arm` | `/sys/jetson/jetson_01/thing/service/arm` | 设防/解锁 | 1 |
| `/sys/${productKey}/${deviceName}/thing/service/mode` | `/sys/jetson/jetson_01/thing/service/mode` | 模式切换 | 1 |
| `/sys/${productKey}/${deviceName}/thing/service/manual_ctrl` | `/sys/jetson/jetson_01/thing/service/manual_ctrl` | 手动控制指令 | 1 |
| `/sys/${productKey}/${deviceName}/thing/service/auto_task` | `/sys/jetson/jetson_01/thing/service/auto_task` | 自动任务指令 | 1 |
| `/sys/${productKey}/${deviceName}/thing/service/params` | `/sys/jetson/jetson_01/thing/service/params` | 参数下发 | 1 |
| `/sys/${productKey}/${deviceName}/thing/service/aivideo_ctrl` | `/sys/aivideo/aivideo_01/thing/service/aivideo_ctrl` | 视频流控制 | 1 |
| `/sys/${productKey}/${deviceName}/thing/service/radar_nav_config` | `/sys/radar_nav/radar_nav_01/thing/service/radar_nav_config` | 导航雷达扫描配置查询 | 1 |


#### 上行服务回复（设备 → 云端）
| 服务 | 对应实际 Reply Topic | 描述 | QoS |
| --- | --- | --- | --- |
| estop | `/sys/jetson/jetson_01/thing/service/estop_reply` | 急停指令回复 | 1 |
| arm | `/sys/jetson/jetson_01/thing/service/arm_reply` | 设防/解锁回复 | 1 |
| mode | `/sys/jetson/jetson_01/thing/service/mode_reply` | 模式切换回复 | 1 |
| manual_ctrl | `/sys/jetson/jetson_01/thing/service/manual_ctrl_reply` | 手动控制指令回复 | 1 |
| auto_task | `/sys/jetson/jetson_01/thing/service/auto_task_reply` | 自动任务指令回复 | 1 |
| params | `/sys/jetson/jetson_01/thing/service/params_reply` | 参数下发回复 | 1 |
| aivideo_ctrl | `/sys/aivideo/aivideo_01/thing/service/aivideo_ctrl_reply` | 视频流控制回复 | 1 |
| radar_nav_config | `/sys/radar_nav/radar_nav_01/thing/service/radar_nav_config_reply` | 导航雷达扫描配置查询回复 | 1 |


#### 上行事件（Jetson → 云端）
| 主题模板 | 对应实际 Topic | 描述 | 频率 | QoS |
| --- | --- | --- | --- | --- |
| `/sys/${productKey}/${deviceName}/thing/event/alarm` | `/sys/jetson/jetson_01/thing/event/alarm` | 报警信息 | 实时 | 1 |
| `/sys/${productKey}/${deviceName}/thing/event/jetson_heartbeat` | `/sys/jetson/jetson_01/thing/event/jetson_heartbeat` | Jetson 心跳包 | 1Hz | 0 |
| `/sys/${productKey}/${deviceName}/thing/event/mission_delta` | `/sys/jetson/jetson_01/thing/event/mission_delta` | 航点信息更新 | 实时 | 1 |
| `/sys/${productKey}/${deviceName}/thing/event/aivideo_status` | `/sys/aivideo/aivideo_01/thing/event/aivideo_status` | 视频流状态 | 实时 | 1 |
| `/sys/${productKey}/${deviceName}/thing/event/aivision_targets` | `/sys/aivision/aivision_01/thing/event/aivision_targets` | 视觉识别目标 | 1-5Hz | 1 |


#### 上行属性（Jetson → 云端）
| 主题模板 | 对应实际 Topic | 描述 | 频率 | QoS |
| --- | --- | --- | --- | --- |
| `/sys/${productKey}/${deviceName}/thing/property/status_jetson` | `/sys/jetson/jetson_01/thing/property/status_jetson` | Jetson 系统状态 | 中低频 | 0 |
| `/sys/${productKey}/${deviceName}/thing/property/thruster` | `/sys/thruster/thruster_01/thing/property/thruster` | 推进器数据 | 10Hz | 1 |
| `/sys/${productKey}/${deviceName}/thing/property/apm_imu` | `/sys/apm/apm_01/thing/property/apm_imu` | IMU 数据 | 20-50Hz | 0 |
| `/sys/${productKey}/${deviceName}/thing/property/radar_mm` | `/sys/radar_mm/radar_mm_01/thing/property/radar_mm` | 毫米波雷达数据 | 20Hz | 0 |
| `/sys/${productKey}/${deviceName}/thing/property/radar_nav` | `/sys/radar_nav/radar_nav_01/thing/property/radar_nav` | 导航雷达扫描数据 | 1-2Hz | 0 |
| `/sys/${productKey}/${deviceName}/thing/property/radar_nav_map` | `/sys/radar_nav/radar_nav_01/thing/property/radar_nav_map` | 导航雷达地图 | 实时 | 1 |
| `/sys/${productKey}/${deviceName}/thing/property/perception_trajectory` | `/sys/jetson/jetson_01/thing/property/perception_trajectory` | 融合感知轨迹 | 实时 | 0 |


### 2.3 MCU 处理域相关主题
> 注：本节按 MCU 处理域归类，但实际 Topic 以“对应实际 Topic”列为准；GPS、气象、测深、电池、油箱、IO、AIS 等可按独立 `productKey/deviceName` 上报。
>

#### 下行服务（云端 → MCU）
| 主题模板 | 对应实际 Topic | 描述 | QoS |
| --- | --- | --- | --- |
| `/sys/${productKey}/${deviceName}/thing/service/io_ctrl` | `/sys/io/io_01/thing/service/io_ctrl` | IO 设备控制 | 1 |
| `/sys/${productKey}/${deviceName}/thing/service/diag_request` | `/sys/mcu/mcu_01/thing/service/diag_request` | 自检请求 | 1 |


#### 上行服务回复（设备 → 云端）
| 服务 | 对应实际 Reply Topic | 描述 | QoS |
| --- | --- | --- | --- |
| io_ctrl | `/sys/io/io_01/thing/service/io_ctrl_reply` | IO 设备控制回复 | 1 |
| diag_request | `/sys/mcu/mcu_01/thing/service/diag_request_reply` | 自检请求回复 | 1 |


#### 上行属性（MCU → 云端）
| 主题模板 | 对应实际 Topic | 描述 | 频率 | QoS |
| --- | --- | --- | --- | --- |
| `/sys/${productKey}/${deviceName}/thing/property/mcu_status` | `/sys/mcu/mcu_01/thing/property/mcu_status` | MCU 系统状态 | 1Hz | 0 |
| `/sys/${productKey}/${deviceName}/thing/property/gps_status` | `/sys/gps/gps_01/thing/property/gps_status` | GPS 定位数据 | 1-5Hz | 0 |
| `/sys/${productKey}/${deviceName}/thing/property/weather_status` | `/sys/weather/weather_01/thing/property/weather_status` | 气象站数据 | 1Hz | 1 |
| `/sys/${productKey}/${deviceName}/thing/property/depth_status` | `/sys/depth/depth_01/thing/property/depth_status` | 测深仪数据 | 1Hz | 1 |
| `/sys/${productKey}/${deviceName}/thing/property/battery_status` | `/sys/battery/battery_01/thing/property/battery_status` | 电池数据 | 1Hz | 1 |
| `/sys/${productKey}/${deviceName}/thing/property/fuel_status` | `/sys/fuel/fuel_01/thing/property/fuel_status` | 油箱数据 | 1Hz | 1 |
| `/sys/${productKey}/${deviceName}/thing/property/io_status` | `/sys/io/io_01/thing/property/io_status` | IO 设备状态 | 1Hz | 0 |
| `/sys/${productKey}/${deviceName}/thing/property/ais` | `/sys/ais/ais_01/thing/property/ais` | AIS 数据 | 1Hz | 0 |


#### 上行事件（MCU → 云端）
| 主题模板 | 对应实际 Topic | 描述 | 频率 | QoS |
| --- | --- | --- | --- | --- |
| `/sys/${productKey}/${deviceName}/thing/event/alarm` | `/sys/mcu/mcu_01/thing/event/alarm` | 报警信息 | 实时 | 1 |
| `/sys/${productKey}/${deviceName}/thing/event/mcu_heartbeat` | `/sys/mcu/mcu_01/thing/event/mcu_heartbeat` | MCU 心跳包 | 1Hz | 0 |
| `/sys/${productKey}/${deviceName}/thing/event/diag_result` | `/sys/mcu/mcu_01/thing/event/diag_result` | 自检结果 | 实时 | 1 |


---
