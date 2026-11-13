# 揭榜挂帅 · 自主播种机器人

面向播种作业的自主移动机器人：树莓派上位机（ROS 2）负责感知、决策与调度，STM32 下位机负责电机与传感器，两端通过统一的接口仓通信。

## 仓库结构

| 仓库 | 职责 |
| --- | --- |
| `robot_interfaces` | 接口契约（`.msg`），版本以 tag 标记 |
| `RaspberryPi_SLAM` | 上位机：ROS 2 Jazzy，感知 / 规划 / 主控 |
| `STM32` | 下位机：STM32F407，micro-ROS 节点 |

## 协作

- 规范：[CONTRIBUTING.md](https://github.com/Intelligent-Bean-Seeder/.github/blob/main/CONTRIBUTING.md)
- 给个人 AI 助手的规则：[AI_AGENT_RULES.md](https://github.com/Intelligent-Bean-Seeder/.github/blob/main/AI_AGENT_RULES.md)
- 模块负责人：[OWNERS.md](https://github.com/Intelligent-Bean-Seeder/.github/blob/main/OWNERS.md)

## 约定速记

- 分支：`<type>/<initials>-<scope>-<slug>`，存活 ≤ 3 天
- 提交：Conventional Commits，支持 `czg` 交互式填写
- 主线：只有 `main`，只接受 PR，squash 合并
- 接口：只改 `robot_interfaces`，三库联动、先后有序
