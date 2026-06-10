## 5. 设备清单
### 5.1 IO 设备清单（MCU）
| 分类 | 设备名称 | 设备 ID | 类型 |
| --- | --- | --- | --- |
| 航行灯 | 左舷灯 | light_left | relay |
| 航行灯 | 右舷灯 | light_right | relay |
| 航行灯 | 前舷灯 | light_front | relay |
| 航行灯 | 后舷灯 | light_back | relay |
| 航行灯 | 桅灯 | light_mast | relay |
| 航行灯 | 尾灯 | light_stern | relay |
| 航行灯 | 告警灯 | light_signal | relay |
| 设备电源 | 毫米波雷达 | pwr_radar | relay |
| 设备电源 | 气象站 | pwr_weather | relay |
| 设备电源 | 测深仪 | pwr_depth | relay |
| 设备电源 | 水泵 | pump | relay |
| 执行机构 | 发动机起翘 | tilt_control | relay |
| 执行机构 | 主电源开关 | main_power | relay |
| 执行机构 | 空调开关 | air_conditioner | relay |
| 执行机构 | 执行电机 | execution_thruster | pwm |


### 5.2 对象类型定义
| object_type | 说明 |
| --- | --- |
| vehicle | 船舶/车辆 |
| pedestrian | 行人 |
| buoy | 浮标 |
| obstacle | 障碍物 |
| unknown | 未知类型 |


### 5.3 视觉目标类别定义
| class | 说明 |
| --- | --- |
| boat | 船只 |
| ship | 船舶 |
| buoy | 浮标 |
| obstacle | 障碍物 |
| person | 人员 |


