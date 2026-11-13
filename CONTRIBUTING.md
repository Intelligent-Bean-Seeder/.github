# 协作规范

「揭榜挂帅 · 自主播种机器人」项目。所有规则用 **R** 编号，评审时可以直接引用（例如「违反 R2-3」）。

| 仓库 | 职责 | 主要语言 |
| --- | --- | --- |
| `robot_interfaces` | 接口契约：所有 `.msg` | `.msg` |
| `RaspberryPi_SLAM` | 上位机：树莓派 5 / Ubuntu 24.04 / ROS 2 Jazzy | C++ / Python |
| `STM32` | 下位机：STM32F407 / micro-ROS | C |

> 本文只写「规则」。**怎么操作**（克隆、装钩子、cz-git、日常同步、冲突处理）见组织主页的上手教程：https://github.com/Intelligent-Bean-Seeder

---

## R1 仓库边界

- **R1-1【必须】** `.msg` 只能存在于 `robot_interfaces`。业务仓库里出现 `.msg` 一律视为放错仓库。
- **R1-2【必须】** 业务仓库用 submodule 引用接口库，并且锁定 **tag**，不是分支。
  `git submodule status` 应显示 `(vX.Y.Z)`；只有 SHA 没有 tag 名，说明锁错了。
- **R1-3【必须】** 升级 submodule 指针时，一次提交只升一个 tag，不允许跨版本跳。
- **R1-4【禁止】** 提交构建产物：`build/`、`install/`、`log/`、`__pycache__/`、`*.hex`、`*.bin`、`*.o`、`*.map`。

## R2 分支

- **R2-1【必须】** 分支名格式：`<type>/<initials>-<scope>-<slug>`
  例：`feat/wm-control-add-joy-teleop`、`fix/wy-hw-uart-baud-921600`、`proto/zth-add-torque-field`
- **R2-2【必须】** 从最新主线切出：`git switch -c <分支名> origin/main`
- **R2-3【禁止】** 直接 `push` 到 `main`。主线只接受 PR。
- **R2-4【必须】** 分支存活 **≤ 3 天**，单个 PR **≤ 400 行**。做不完就拆，先合能跑的部分。
- **R2-5【必须】** 每天 `git fetch && git rebase origin/main`，减少最后一天的冲突。
- **R2-6【禁止】** 把 `main` merge 进功能分支（会污染历史）。用 `rebase`。
- **R2-7【必须】** `proto/` 分支全组织同时只允许 **一个**，由协议负责人（`zth`）开，合并后立即删除。

### type 白名单（11 个）

`feat` `fix` `docs` `refactor` `perf` `test` `build` `ci` `chore` `proto`

> `proto` 专门用于接口仓的 `.msg` 变更，必须配合 `!` 或 `BREAKING CHANGE` 说明影响。

### scope 白名单（12 个）

| scope | 含义 | scope | 含义 |
| --- | --- | --- | --- |
| `radar` | 雷达驱动与点云 | `fi` | 固件接口层 |
| `vision` | 视觉 | `bringup` | 启动与配置 |
| `ml` | 机器学习 | `sim` | 仿真 |
| `control` | 运动控制 | `ci` | 流水线与钩子 |
| `hw` | 硬件抽象 / 串口 | `docs` | 文档 |
| `proto` | 接口定义 | `deps` | 依赖与 submodule |

### initials（提交者缩写，冲突时加第 3 个字母）

| 缩写 | 姓名 | 缩写 | 姓名 |
| --- | --- | --- | --- |
| `zxr` | 张笑然 | `zjc` | 邹嘉程 |
| `wy` | 王滢 | `dmz` | 杜铭泽 |
| `zzl` | 张子路 | `ysy` | 叶思雨 |
| `gss` | 桂世燊 | `yw` | 杨武 |
| `wm` | 吴冕 | `gjy` | 郭靖尧 |
| `zth` | 张天浩 | `zzh` | 张展浩 |
| `wyq` | 王宇卿 | | |

## R3 提交

- **R3-1【必须】** 格式：`<type>(<scope>): <subject>`，subject 用中文，不超过 50 字。
- **R3-2【必须】** type / scope 取自上方白名单，不在表里就说明你选错了。
- **R3-3【必须】** 破坏性变更加 `!`，并在正文写 `BREAKING CHANGE:` 说明影响。
- **R3-4【禁止】** subject 只写「修改」「更新」「修复 bug」这类无信息量的词。
- **R3-5【必须】** 已安装本地钩子（`bash scripts/install-hooks.sh`）。没装的话，提交会被钩子拒绝；`--no-verify` 绕过视为违规。

## R4 PR 与评审

- **R4-1【必须】** 一个 PR 只做一件事；标题即 squash 后的提交信息，同样要符合 R3-1。
- **R4-2【必须】** PR 描述按模板五段填全，缺段视为未完成。
- **R4-3【必须】** 等 pr-agent 出评论后，对「阻塞级」意见逐条回应：改，或写明不改的理由。
- **R4-4【重要】** **AI 批准 ≠ 免责。** 合并者对合进去的每一行负责。
- **R4-5【必须】** 合并方式用 **squash**，合并后删除源分支。

## R5 协议变更（三库联动）

- **R5-1【必须】** 顺序固定：`robot_interfaces` PR → 打 tag → `STM32` PR → `RaspberryPi_SLAM` PR。
- **R5-2【必须】** 三个 PR 的描述里都写 `Part n/3 of #<issue 编号>`，方便串起来。
- **R5-3【必须】** 说明四件事：谁发、谁收、周期、超时与断链行为。
- **R5-4【必须】** 版本号规则：新增字段 → minor；删除字段或改语义 → major；纯注释 → patch。

## R6 安全与环境

- **R6-1【禁止】** 任何 key / token / 密码进代码、注释或日志。
- **R6-2【必须】** API key 只放 GitHub Secrets，变量名统一 `OPENAI_KEY`。
- **R6-3【必须】** 行尾由仓库内 `.gitattributes` 决定，不要在本机设 `core.autocrlf=true` 覆盖它。

## R7 追责

- **R7-1** 每次合并留下三元组：**谁写的**（squash 提交作者）、**谁审的**（pr-agent 评论 + 人工回应）、**谁合的**（merged by）。
- **R7-2** 每周由负责人导出一次合并清单（`git log --since="1 week ago"`），在周会过一遍。
- **R7-3** 违规提交由合并者在 24 小时内修正（`rebase` 或 `revert`），并在 PR 里说明原因。
- **R7-4** 同一人同一类违规出现 3 次，在周会做 5 分钟说明；不是惩罚，是把规则讲清楚。

---

## 常用命令

```bash
# 1. 切新分支（从最新主线）
git fetch origin && git switch -c feat/wm-control-add-joy-teleop origin/main

# 2. 提交（交互式，自动带 type/scope 选项）
czg

# 3. 推送并开 PR
git push -u origin HEAD

# 4. 安装钩子（每个仓库一次）
bash scripts/install-hooks.sh
```

装 `czg`：`npm i -g czg`（注意是 `czg`；`cz-git` 只是适配器库，没有命令行）。不装也能用 `git commit -m "feat(control): ..."`，钩子会照样校验。
