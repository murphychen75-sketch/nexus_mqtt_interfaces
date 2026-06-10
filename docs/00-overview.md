# NEUXS 无人船通信协议 V1.1
> 版本日期：2026-06-01
>

## 1. 协议规范
### 1.1 基础信息
| 项目 | 规范 |
| --- | --- |
| 协议版本 | 1.1 |
| 传输协议 | MQTT 5.0 |
| 默认端口 | 1883 (TCP) / 8883 (TLS) |
| 数据编码 | UTF-8 |
| Payload 格式 | JSON |


### 1.2 产品与设备抽象
+ 产品是独立完整功能设备的抽象综合。
+ 产品分为：直连设备、网关设备、网关子设备。
+ 直连设备具备独立 IP，且不能外接设备。
+ 网关设备具备独立 IP，并可关联多个子设备。
+ 网关子设备没有独立 IP，通过网关设备代收发数据。
+ 设备必须从属于产品，并归属到总产品（船设备）。总设备即 M10 或 NEUXS 船舶总产品。
+ 本文的 `productKey/deviceName` 按可迁移的功能设备展开，例如 `gps/gps_01`、`io/io_01`、`ais/ais_01`。



### 1.3 Topic 规范
```plain
/sys/${productKey}/${deviceName}/thing/${type}/${identifier}
```

| 占位符 | 说明 | 示例 |
| --- | --- | --- |
| `${productKey}` | 产品唯一标识 | `aivideo` / `gps` / `io` |
| `${deviceName}` | 设备唯一名称 | `aivideo_01` / `gps_01` / `io_01` |
| `${type}` | 消息类型：`service` / `event` / `property` | `service` |
| `${identifier}` | 物模型定义的标识符 | `estop` |


---

#### 1.3.1 通用字段与时序约定
| 字段 | 约定 |
| --- | --- |
| `seq` | 按每个 Topic 独立递增，从1开始；接收端应按 Topic 分别统计丢包。 |
| 外层 `timestamp` | MQTT payload 生成/发布时间，单位：毫秒。 |
| `data.ts` | 传感器或执行器原始采样时间，单位：毫秒。 |
| `timestamps[].time` | 算法流水线阶段时间，单位：毫秒。 |
| `trajectories[].points[].timestamp` | 轨迹点对应的观测时间，单位：毫秒。 |


#### 1.3.2 服务回复约定
所有 `service` 下行指令均应有对应 `_reply` 主题，格式为：

```plain
/sys/${productKey}/${deviceName}/thing/service/${identifier}_reply
```

本版本保留 `event_reply` 与 `property_reply` 模板，但当前业务不使用、不要求设备实现。

#### 1.3.3 物模型表
| 功能 | productKey | deviceName |
| --- | ---: | ---: |
| Jetson 主控 | `jetson` | `jetson_01` |
| MCU 执行单元 | `mcu` | `mcu_01` |
| 推进器 | `thruster` | `thruster_01` |
| apm飞控 | `apm` | `apm_01` |
| 毫米波雷达 | `radar_mm` | `radar_mm_01` |
| nav导航雷达 | `radar_nav` | `radar_nav_01` |
| 视频 | `aivideo` | `aivideo_01` |
| 视觉识别 | `aivision` | `aivision_01` |
| GPS | `gps` | `gps_01` |
| 气象站 | `weather` | `weather_01` |
| 测深仪 | `depth` | `depth_01` |
| 电池 | `battery` | `battery_01` |
| 油箱 | `fuel` | `fuel_01` |
| IO 控制器 | `io` | `io_01` |
| AIS | `ais` | `ais_01` |


