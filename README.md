# NEUXS 无人船 MQTT 通信协议

NEUXS 无人船各子系统（Jetson、MCU、APM 及传感器）与云端之间的 MQTT 通信协议规范。

| 项目 | 说明 |
| --- | --- |
| 协议版本 | **1.1** |
| 版本日期 | 2026-06-01 |
| 传输协议 | MQTT 5.0 |
| Payload 格式 | JSON (UTF-8) |

## 文档目录

| 文档 | 内容 |
| --- | --- |
| [协议规范](docs/00-overview.md) | 基础信息、产品与设备抽象、Topic 规范、物模型表 |
| [话题架构](docs/01-topics.md) | 系统主题模板，Jetson / MCU 处理域 Topic 清单 |
| [下行数据格式](docs/02-downlink-messages.md) | 云端 → 设备：服务指令与回复 Payload 定义 |
| [上行数据格式](docs/03-uplink-messages.md) | 设备 → 云端：属性、事件 Payload 定义 |
| [设备清单](docs/04-device-catalog.md) | IO 设备清单、对象类型与视觉目标类别 |
| [附录](docs/05-appendix.md) | 错误码、通信频率建议、MQTT 配置建议 |

## Topic 模板

```
/sys/${productKey}/${deviceName}/thing/${type}/${identifier}
```

- `${type}`：`service` / `event` / `property`
- 所有 `service` 下行指令均应有对应的 `${identifier}_reply` 回复主题

## 物模型速查

| 功能 | productKey | deviceName |
| --- | --- | --- |
| Jetson 主控 | `jetson` | `jetson_01` |
| MCU 执行单元 | `mcu` | `mcu_01` |
| 推进器 | `thruster` | `thruster_01` |
| APM 飞控 | `apm` | `apm_01` |
| GPS | `gps` | `gps_01` |
| IO 控制器 | `io` | `io_01` |

完整物模型表见 [协议规范](docs/00-overview.md)。

## 参与维护

修改协议内容请直接编辑 `docs/` 目录下对应文档，通过 Pull Request 提交变更。
