# NEUXS 无人船 MQTT 通信协议

NEUXS 无人船各子系统（Jetson、SOC/MCU 及传感器）与云端之间的 MQTT 通信协议规范。

| 项目 | 说明 |
| --- | --- |
| 传输协议 | MQTT 5.0 |
| Payload 格式 | JSON (UTF-8) |
| Topic 模型 | `/sys/{productKey}/{deviceName}/thing/{domain}/{action}` |

---

## 协议版本说明

本仓库同时保留多版协议文档，用途如下：

| 文档 | 版本 | 日期 | 定位 |
| --- | --- | --- | --- |
| [无人船通信协议1.1-629.md](无人船通信协议1.1-629.md) | **V2.0** | 2026-06-23 | **主参考**：全船完整协议，含统一消息信封、`inputParams`/`value` 结构及全部物模型 |
| [docs/protocol.md](docs/protocol.md) | V1.1 | 2026-06-01 | 历史版本，Topic 将业务标识写入路径（旧模型） |
| [docs/jetson_protocol.md](docs/jetson_protocol.md) | V1.1（命名规范对齐版） | 2026-06-29 | **当前维护中**：Jetson 处理域专项接口，按命名规范与 V2.0 消息体对齐 |
| [Nexus协议命名规范.md](Nexus协议命名规范.md) | — | — | Topic、字段、`identifier` 等命名约束 |

> **维护状态**：目前仅 [docs/jetson_protocol.md](docs/jetson_protocol.md) 在本项目中完成修订；其余文档保持原样，后续将按 Jetson 域实践逐步同步。

---

## 主要参考关系

```
Nexus协议命名规范.md          （命名约束）
        ↓
无人船通信协议 V2.0           （全船主协议、消息信封格式）
        ↓
docs/jetson_protocol.md       （Jetson 域落地版，当前唯一已改文档）
```

- **Topic 与命名**：以 [Nexus协议命名规范.md](Nexus协议命名规范.md) 为准。
- **消息体结构**：以 [无人船通信协议1.1-629.md](无人船通信协议1.1-629.md)（V2.0）为准——外层 `id` / `reportTime` / `deviceId` 等；服务用 `params.inputParams`，事件用 `params.value`。
- **Jetson 域实现**：以 [docs/jetson_protocol.md](docs/jetson_protocol.md) 为准（范围限于 `jetson`、`cam`、`vision`、`radar_mm`）。

---

## Jetson 域物模型速查

| 功能 | productKey | deviceName |
| --- | --- | --- |
| Jetson 主控 | `jetson` | `jetson_01` |
| 相机 | `cam` | `cam_01` |
| 视觉识别 | `vision` | `vision_01` |
| 毫米波雷达 | `radar_mm` | `radar_mm_01` |

完整接口定义见 **[docs/jetson_protocol.md](docs/jetson_protocol.md)**。

---

## 参与维护

- Jetson 域协议变更：编辑 [docs/jetson_protocol.md](docs/jetson_protocol.md)
- 全船协议变更：编辑 [无人船通信协议1.1-629.md](无人船通信协议1.1-629.md)
- 命名规范变更：编辑 [Nexus协议命名规范.md](Nexus协议命名规范.md)

通过 Pull Request 提交变更。
