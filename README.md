# NEUXS 无人船 MQTT 通信协议

NEUXS 无人船各子系统（Jetson、MCU、APM 及传感器）与云端之间的 MQTT 通信协议规范。

| 项目 | 说明 |
| --- | --- |
| 协议版本 | **1.1** |
| 版本日期 | 2026-06-01 |
| 传输协议 | MQTT 5.0 |
| Payload 格式 | JSON (UTF-8) |

## 协议文档

完整协议见 **[docs/protocol.md](docs/protocol.md)**，包含协议规范、话题架构、上下行数据格式、设备清单及附录。

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

## 参与维护

修改协议内容请直接编辑 [docs/protocol.md](docs/protocol.md)，通过 Pull Request 提交变更。
