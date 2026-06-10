## 3. 下行数据格式（云端 → 设备）
### 3.1 通用服务回复格式
设备对服务指令的回复通过对应 `_reply` 主题发送：

```json
{
  "time": 1703123456789,
   "code": 200,
   "message": "success"

}
```

### 3.2 急停
**Topic:** `/sys/${productKey}/${deviceName}/thing/service/estop`

**对应实际 Topic:** `/sys/jetson/jetson_01/thing/service/estop`

```json
{
  "time": 1703123456789,
    "estop": true,
    "src": "shore"
  
}
```

| 字段/位置 | 说明 |
| --- | --- |
| `estop` | 是否紧急停止,true=停机,false=恢复 |
| `src` | 来源端,例："shore"（岸端） |


### 3.3 设防/解锁
**Topic:** `/sys/${productKey}/${deviceName}/thing/service/arm`

**对应实际 Topic:** `/sys/jetson/jetson_01/thing/service/arm`

```json
{
  "time": 1703123456789,
  "armed": "arm"
  
}
```

| 字段/位置 | 说明 |
| --- | --- |
| `armed` | "arm" 表示设防（上锁/开启警戒）,"disarm" 表示解锁（解除警戒/撤防） |


### 3.4 模式切换
**Topic:** `/sys/${productKey}/${deviceName}/thing/service/mode`

**对应实际 Topic:** `/sys/jetson/jetson_01/thing/service/mode`

```json
{
  "time": 1703123456789,
    "mode": "auto"
  
}
```

| 字段/位置 | 说明 |
| --- | --- |
| `mode` | **<font style="color:rgb(15, 17, 21);">auto，manual,RTL,dock,follow,loiter,hold</font>** |


### 3.5 手动控制指令
**Topic:** `/sys/${productKey}/${deviceName}/thing/service/manual_ctrl`

**对应实际 Topic:** `/sys/jetson/jetson_01/thing/service/manual_ctrl`

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


| <font style="color:rgb(15, 17, 21);">消息字段</font> | <font style="color:rgb(15, 17, 21);">对应控制</font> | <font style="color:rgb(15, 17, 21);">正值含义</font> | <font style="color:rgb(15, 17, 21);">负值含义</font> |
| --- | --- | --- | --- |
| `**<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">x</font>**` | <font style="color:rgb(15, 17, 21);">油门/前进后退</font> | <font style="color:rgb(15, 17, 21);">前进</font> | <font style="color:rgb(15, 17, 21);">后退</font> |
| `**<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">y</font>**` | <font style="color:rgb(15, 17, 21);">横移（仅带侧推的船）</font> | <font style="color:rgb(15, 17, 21);">右移</font> | <font style="color:rgb(15, 17, 21);">左移</font> |
| `**<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">r</font>**` | <font style="color:rgb(15, 17, 21);">航向/转向</font> | <font style="color:rgb(15, 17, 21);">右转</font> | <font style="color:rgb(15, 17, 21);">左转</font> |
| `**<font style="color:rgb(15, 17, 21);background-color:rgb(235, 238, 242);">z</font>**` | <font style="color:rgb(15, 17, 21);">备用油门（通常不用）</font> | <font style="color:rgb(15, 17, 21);">正推力</font> | <font style="color:rgb(15, 17, 21);">负推力</font> |




### 3.6 自动任务
**Topic:** `/sys/${productKey}/${deviceName}/thing/service/auto_task`

**对应实际 Topic:** `/sys/jetson/jetson_01/thing/service/auto_task`

```json
{
  "time": 1703123456789,
    "cmd": "start",
    "task_id": "TASK_001",
    "waypoints_num": 2,
    "mission_type": 16,
    "homepoint":{
        "lat": 31.123456,
        "lon": 121.123456,
        "type": 1
    },
    "waypoints": [
      {
        "lat": 31.123456,
        "lon": 121.123456,
        "order": 1
      },
      {
        "lat": 31.123457,
        "lon": 121.123457,
        "order": 2
      }
    ],
    "mode": "auto"  
}
```

| 字段/位置 | 说明 |
| --- | --- |
| `cmd` | 任务操作："start"/"stop" |
| `task_id` | 任务流水号 |
| `waypoints_num` | 航点数量 |
| `mode` | 路径执行模式 |


### 3.7 参数下发
**Topic:** `/sys/${productKey}/${deviceName}/thing/service/params`

**对应实际 Topic:** `/sys/jetson/jetson_01/thing/service/params`

```json
{
  "time": 1703123456789,
    "params": [
      {
        "name": "MAX_SPEED",
        "value": 5.2,
        "type": "float"
      },
      {
        "name": "SAFE_DISTANCE",
        "value": 10,
        "type": "int"
      }
    ]
  
}
```

### 3.8 视频控制
**Topic:** `/sys/${productKey}/${deviceName}/thing/service/aivideo_ctrl`

**对应实际 Topic:** `/sys/aivideo/aivideo_01/thing/service/aivideo_ctrl`

```json
{
  "time": 1703123456789,
    "cmd": "start",
    "camera_id": "front",
    "resolution": "1920x1080",
    "fps": 30,
    "bitrate_kbps": 4096
  
}
```

| 字段/位置 | 说明 |
| --- | --- |
| `cmd` | "start"/"stop" |
| `camera_id` | 摄像头标识 |
| `resolution` | 分辨率 |
| `fps` | 帧率 |
| `bitrate_kbps` | 码率（kbps） |


### 3.9 IO 设备控制（MCU）
**Topic:** `/sys/${productKey}/${deviceName}/thing/service/io_ctrl`

**对应实际 Topic:** `/sys/io/io_01/thing/service/io_ctrl`

```json
{
  "time": 1703123456789,

        "light_left_action": "on",
    
        "light_right_action": "on",
   
        "light_mast_action": "off",
  
        "light_stern_action": "on",
   
        "light_signal_action": "off",

        "tilt_control_action": "off",
  
   
        "thruster_left_action": "on",
        "thruster_right_action": "on",
   
        "air_conditioner_action": "on"
      
  
}
```

| 字段/位置 | 说明 |
| --- | --- |
| `id=light_left` | 左舷灯开关 |
| `id=light_right` | 右舷灯开关 |
| `id=light_mast` | 桅灯开关 |
| `id=light_stern` | 尾灯开关 |
| `id=light_signal` | 告警灯开关 |
| `id=tilt_control` | 发动机起翘控制 |
| `id=main_power` | 主电源开关,发电机 |
| `id=air_conditioner` | 空调开关 |


| 参数 | 类型 | 说明 |
| --- | --- | --- |
| id | string | 设备标识符（见设备清单） |
| action | string | on/off |


### 3.10 自检请求
**Topic:** `/sys/${productKey}/${deviceName}/thing/service/diag_request`

**对应实际 Topic:** `/sys/mcu/mcu_01/thing/service/diag_request`

```json
{
  "time": 1703123456789,

    "modules": [
      "all"
    ]
  
}
```

| 字段/位置 | 说明 |
| --- | --- |
| `modules` | 指定自检模块名数组,如 ["all"] |


| modules 值 | 说明 |
| --- | --- |
| all | 全部模块 |
| imu | IMU 模块 |
| gps | GPS 模块 |
| thruster | 推进器模块 |
| battery | 电池模块 |
| comms | 通信模块 |


### 3.11 导航雷达扫描配置查询（Jetson）
**Topic:** `/sys/${productKey}/${deviceName}/thing/service/radar_nav_config`

**对应实际 Topic:** `/sys/radar_nav/radar_nav_01/thing/service/radar_nav_config`

```json
{
  "time": 1703123456789,
    "cmd": "get_config"
  
}
```



### 3.12 导航雷达扫描配置查询回复
**Topic:** `/sys/${productKey}/${deviceName}/thing/service/radar_nav_config_reply`

**对应实际 Topic:** `/sys/radar_nav/radar_nav_01/thing/service/radar_nav_config_reply`

```json
{
  "time": 1703123456789,

    "code": 200,
    "message": "success",
    "angular_resolution_deg": 0.9,
    "max_range_m": 200.0
  
}
```

### 3.13 油机控制
**Topic:** `/sys/${productKey}/${deviceName}/thing/service/thruster_ctr`

**对应实际 Topic:** `/sys/thruster/thruster_01/thing/service/thruster_ctr`

```json
{
  "time": 1703123456789,
   "tiltSwitch":  1,
   "level": -1
    
    
  
}
```

| 字段/位置 | 说明 |
| --- | --- |
| `tiltSwitch` | 0：不变，1：起翘，2：落桨 ,3:启动，4关闭 |
| level | -4~+4，0:空挡,+1表示为前进1档，-1为后退1档，4档即100%最大 |




### 3.14 油机控制回复
**Topic:** `/sys/${productKey}/${deviceName}/thing/service/thruster_ctr_reply`

**对应实际 Topic:** `/sys/thruster/thruster_01/thing/service/thruster_ctr_reply`

```json
{
  "time": 1703123456789,
   "code": 200,
   "message": "success",
    "tiltSwitch": 2,
    "level": -2
      
  
}
```
