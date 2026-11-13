# 负责人

> 私有仓库无法使用 GitHub 的 CODEOWNERS 强制评审（需要 Pro/Team），所以这份名单是**约定**：
> 改到对应范围时，在 PR 里 @ 对应的人，并等一句「我看过了」再合。

## 按模块

| 模块 | 负责人 | 缩写 | 主要仓库 | 说明 |
| --- | --- | --- | --- | --- |
| 总负责 / 电控 | 张笑然 | `zxr` | 全部 | 排期、跨仓库协调、最终合并 |
| 上位机主控 | 吴冕 | `wm` | `RaspberryPi_SLAM` | 调度、状态机、启动 |
| 上位机 / 协议 | 张天浩 | `zth` | `RaspberryPi_SLAM`、`robot_interfaces` | **接口仓唯一放行人** |
| 下位机 | 王滢 | `wy` | `STM32` | 固件、micro-ROS 节点 |
| 下位机 | 张子路 | `zzl` | `STM32` | 驱动、与 PCB 对照 |
| 视觉 | 桂世燊 | `gss` | `RaspberryPi_SLAM` | 相机、识别 |
| 机器学习 | 邹嘉程 | `zjc` | `RaspberryPi_SLAM` | 模型训练与部署 |
| 机器学习 | 杜铭泽 | `dmz` | `RaspberryPi_SLAM` | 数据集与评测 |
| 机械 | 叶思雨 | `ysy` | — | 结构 |
| 机械 | 杨武 | `yw` | — | 轻量化、强度 |
| 机械 | 郭靖尧 | `gjy` | — | 运动学仿真、有限元 |
| 雷达 / SLAM | **待定** | — | `RaspberryPi_SLAM` | ⚠️ 目前无人认领，见下方 |
| 待分配 | 张展浩 | `zzh` | — | |
| 待分配 | 王宇卿 | `wyq` | — | |

## 按路径（约定优先级高于上面的模块）

| 路径 | 必须 @ |
| --- | --- |
| `robot_interfaces/**`（任何 `.msg`） | `@zth` + 受影响端的负责人（`@wm` 或 `@wy`） |
| `STM32/src/**`、`STM32/*.ioc` | `@wy` 或 `@zzl` |
| `RaspberryPi_SLAM/src/seed_robot_control/**` | `@wm` |
| `RaspberryPi_SLAM/src/*vision*/**` | `@gss` |
| `RaspberryPi_SLAM/src/*ml*/**`、`ml/**` | `@zjc` 或 `@dmz` |
| `.github/**`、`scripts/**` | `@zxr` |

## 未决事项

- **雷达 / SLAM 无负责人。** 雷达是宇树 L2（3D 点云 + IMU），需要有人负责驱动、标定与 LIO 参数。
  在认领之前，涉及 `radar` scope 的 PR 由 `@zxr` 临时把关。
