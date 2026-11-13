# 开发守则 · 自主播种机器人

面向播种作业的自主移动机器人：树莓派上位机（ROS 2）负责感知、决策与调度，STM32 下位机负责电机与传感器，两端通过统一的接口仓通信。

> **30 秒速查**（老手只看这个）
> ```bash
> git switch main && git pull --rebase origin main && git submodule update --init
> git switch -c feat/wm-control-add-joy-teleop origin/main
> # ……写代码……
> czg && git push -u origin HEAD
> ```
> 然后到 GitHub 开 PR，等 AI 审查，squash 合并。

第一次参与的人，请从第 0 步开始，一步一步照做。每一步都写了**执行完应该看到什么**，看到的不对就停下问人。

---

## 第 0 步：先搞清楚三件事

### 0.1 三个库分别干什么

| 仓库 | 里面有什么 | 谁主要在这里干活 |
| --- | --- | --- |
| `robot_interfaces` | 只有 `.msg` 接口定义，发版用 tag | 协议负责人（张天浩） |
| `RaspberryPi_SLAM` | ROS 2 上位机：感知、规划、主控 | 上位机、视觉、机器学习、雷达 |
| `STM32` | 下位机固件：电机、传感器、micro-ROS | 下位机、电控 |

**三个库都是私有库**，需要先被拉进组织。

### 0.2 一次改动长什么样

```
拉最新 → 开分支 → 写代码 → 提交 → 推送 → 开 PR → AI 审查 → squash 合并 → 删分支
  ↑                                                              ↓
  └────────────────── 每天重复，分支活不过 3 天 ──────────────────┘
```

### 0.3 你在哪个模块

| 人 | 角色 | 常用 scope |
| --- | --- | --- |
| 吴冕、张天浩 | 上位机 / 协议 | `control` `bringup` `fi` `proto` `deps` |
| 王滢、张子路 | 下位机 | `fi` `hw` `deps` |
| 张笑然 | 电控 | `control` `hw` |
| 桂世燊 | 视觉 | `vision` |
| 邹嘉程、杜铭泽 | 机器学习 | `ml` |
| 叶思雨、杨武、郭靖尧 | 机械 | `sim` `docs` |

> 雷达目前没有负责人（`OWNERS.md` 里标了代管），涉及 `radar` 的改动先找张笑然。

---

## 第 1 步：拿到权限

1. 把 GitHub 账号发给负责人，被拉进 `Intelligent-Bean-Seeder` 组织；
2. 组织默认权限已经是 **write**，不需要单独加，进组织就能推。

**检查点**：打开 https://github.com/Intelligent-Bean-Seeder ，能看到三个仓库。

---

## 第 2 步：克隆

```bash
git clone --recurse-submodules https://github.com/Intelligent-Bean-Seeder/RaspberryPi_SLAM.git
cd RaspberryPi_SLAM
```

`--recurse-submodules` **不能省**。省了 `src/Robot_Interfaces` 就是个空目录，后面编译会报找不到 `robot_interfaces`。

**检查点**：

```bash
git submodule status
# 应该看到一行： b67a1eb... src/Robot_Interfaces (v0.1.0)
ls src/Robot_Interfaces/msg
# 应该看到：ActuatorCommand.msg  ActuatorState.msg  SystemStatus.msg
```

括号里是 `(v0.1.0)` 才对。如果**没有括号**、或者 `ls` 是空的，回去补：

```bash
git submodule update --init --recursive
```

---

## 第 3 步：装本地钩子（最容易忘，忘了等于裸奔）

```bash
bash scripts/install-hooks.sh
```

它做两件事：提交时检查「提交信息 + 分支名」，推送时拒绝直接推 `main`。

**检查点**（故意犯规，应该被拒）：

```bash
git commit --allow-empty -m "test"
# 期望输出：✗ 提交信息不合规：test
```

看到 `✗` 就说明装好了。没看到 `✗`、反而提交成功了，说明钩子没装，重跑上面的命令。

> 钩子存在你本机的 `.git/hooks` 里，**不会随仓库分发**。换电脑、重新克隆都要重装。

---

## 第 4 步：装 czg（写提交信息的菜单）

```bash
npm i -g czg
czg --version
```

装完就有 `czg` 命令，仓库里的 `.czrc` 已经配好白名单，不用初始化。

> 注意包名：**命令行的包叫 `czg`**，不是 `cz-git`。`cz-git` 是给 commitizen 用的适配器库，装错了只会在 `npm ls -g` 里多一行，`czg` 仍然找不到。

**没有 Node 也能干活**：手写 `git commit -m "feat(control): 增加摇杆遥控"`，格式一样，钩子只认格式不认工具。

**检查点**：`czg --version` 能打印版本号。报 `command not found` 就是 npm 全局 bin 没进 PATH。

---

## 第 5 步：每天开工（三条命令）

```bash
git switch main
git pull --rebase origin main
git submodule update --init
```

**检查点**：`git status` 显示 `nothing to commit, working tree clean`，且 `git log --oneline -1` 是最新提交。

> 这三条**每天第一次动代码前都要跑**。跳过它们，最后一天必冲突。

---

## 第 6 步：开分支

```bash
git switch -c feat/wm-control-add-joy-teleop origin/main
```

命名格式：`<type>/<initials>-<scope>-<slug>`

| 部分 | 规则 | 例 |
| --- | --- | --- |
| `type` | 10 个白名单之一（见第 8 步） | `feat` |
| `initials` | 你的缩写，2–3 个小写字母 | `wm` |
| `scope` | 12 个白名单之一（见第 8 步） | `control` |
| `slug` | 干什么，小写英文 + 连字符，2–30 字符 | `add-joy-teleop` |

姓名缩写：

| 缩写 | 姓名 | 缩写 | 姓名 |
| --- | --- | --- | --- |
| `zxr` | 张笑然 | `zjc` | 邹嘉程 |
| `wy` | 王滢 | `dmz` | 杜铭泽 |
| `zzl` | 张子路 | `ysy` | 叶思雨 |
| `gss` | 桂世燊 | `yw` | 杨武 |
| `wm` | 吴冕 | `gjy` | 郭靖尧 |
| `zth` | 张天浩 | `zzh` | 张展浩 |
| `wyq` | 王宇卿 | | |

**检查点**：`git branch --show-current` 显示你的新分支名；分支名里不能有中文、不能有空格。

> 分支从 `origin/main` 切，切完 **3 天内**必须合掉。做不完就拆成两个 PR，先合能跑的部分。

---

## 第 7 步：写代码 + 中途同步

写的过程中，每天至少把主线更新接进来一次：

```bash
git fetch origin
git rebase origin/main
```

**不要用 `git merge main`**，会把主线历史搅乱。

冲突了：

```bash
git status              # 看哪几个文件冲突
# 打开文件，删掉 <<<<<<< ======= >>>>>>> 这些标记，改成你要的结果
git add <冲突文件>
git rebase --continue
```

想放弃这次 rebase、回到动之前：

```bash
git rebase --abort
```

**检查点**：`git status` 不再显示 `rebase in progress`。

---

## 第 8 步：提交

```bash
git add -A
czg
```

菜单会依次问：**类型 → 模块 → 简短描述 → 是否破坏性变更**。选完自动生成 `feat(control): 增加摇杆遥控`。

也可以跳过第一步直接指定类型：`czg feat`。

### 8.1 type 怎么选（10 个）

| type | 什么时候用 | 典型改动 |
| --- | --- | --- |
| `feat` | 新增功能、新增参数 | 加一个遥控节点 |
| `fix` | 修 bug | 修串口丢帧 |
| `docs` | 只改文档、注释 | 改 README |
| `refactor` | 重构，但行为不变 | 拆函数、改命名 |
| `perf` | 性能优化 | 降采样、加缓存 |
| `test` | 加测试、改测试 | 补一个单测 |
| `build` | 构建脚本、依赖版本 | 改 CMakeLists |
| `ci` | 流水线、钩子、配置 | 改 `.pr_agent.toml` |
| `chore` | 杂务 | 删文件、格式化 |
| `proto` | **只改 `.msg`**（只在接口仓用） | 加一个字段 |

三个最容易选错的：

- 改 `.msg` → 用 `proto`，**不是** `feat`；
- 升 submodule 指针 → 用 `deps`；
- 改 `pr-agent.yml`、钩子、模板 → 用 `ci`。

### 8.2 scope 怎么选（12 个）

| scope | 含义 | 典型改动 |
| --- | --- | --- |
| `radar` | 雷达驱动、点云、LIO | 接 L2 驱动、改降采样 |
| `vision` | 视觉：相机、识别 | 加秧苗识别节点 |
| `ml` | 机器学习：训练、推理、模型 | 训练脚本、模型导出 |
| `control` | 运动控制、执行器指令 | 速度限幅、PID 参数 |
| `hw` | 硬件抽象、串口、RS-485/USB | 串口收发、帧编解码 |
| `proto` | 接口定义（**只在接口仓**） | 改 `.msg` 字段 |
| `fi` | 固件接口层（MCU 侧收发） | micro-ROS 端点注册 |
| `bringup` | 启动、参数、launch | 新增 launch 文件 |
| `sim` | 仿真：Gazebo、URDF | 改模型、世界文件 |
| `ci` | 流水线、钩子、模板 | 改 pr-agent 配置 |
| `docs` | 文档、注释 | 改本文件 |
| `deps` | 依赖、submodule、版本 | 升接口库 tag |

选不准就问自己：**这次改动主要落在哪个模块**。宁可粗，不要编新词——钩子只认这 12 个。

### 8.3 破坏性变更

删字段、改类型、改语义，都算破坏性。菜单里选「是」，生成 `proto!: xxx`，正文补一句：

```
BREAKING CHANGE: ActuatorCommand 删除 velocity_ff，两端需同步升级到 v0.2.0
```

### 8.4 常见报错

| 报错 | 原因 | 怎么办 |
| --- | --- | --- |
| `✗ 提交信息不合规：update stuff` | 没按 `type(scope): 描述` 写 | 用 `czg` 重来 |
| `✗ 提交信息不合规：feat(nosuchscope)` | scope 不在 12 个里 | 查 8.2 的表 |
| `✗ 分支名不合规：dev` | 分支名没按规则 | 改名，见下 |
| `czg: command not found` | npm 全局 bin 没进 PATH | 重装或手写提交信息 |

分支名写错了要改（还没推过）：

```bash
git branch -m fix/wy-hw-uart-baud-921600
```

---

## 第 9 步：推送 + 开 PR

```bash
git push -u origin HEAD
```

推不上去、提示 `non-fast-forward`（因为你 rebase 过）：

```bash
git push --force-with-lease
```

`--force-with-lease` 只能对**自己的分支**用。对 `main` 永远不要 force。

然后开 PR：

```bash
gh pr create --fill
```

或到网页上点。**PR 标题就是最终提交信息**，必须符合 `type(scope): 描述`，否则 `PR Title` 检查会红。描述按模板填五段：

| 段 | 写什么 |
| --- | --- |
| 1. 做了什么 | 一句话 + 关键改动 |
| 2. 影响范围 | 勾选受影响的模块 |
| 3. 接口变更 | 有没有动 `.msg`；动了就按第 10 步的三库联动 |
| 4. 验证证据 | 你怎么证明它能跑（日志、截图、`colcon test` 输出） |
| 5. 回滚方式 | revert 就行 / 需要同时回滚哪几个库 |

**检查点**：PR 页面上 `PR Title` 是绿的。红了就改标题。

---

## 第 10 步：等 AI 审查、合并

1. 等两分钟左右，`AI PR Review Agent` 会留一条评论（`## PR Reviewer Guide`）；
2. 对「阻塞级」意见逐条回应：改，或者写明不改的理由；
3. 自己确认没问题后 **squash 合并**，合并后删源分支。

**改了 `.msg` 的话，顺序是固定的**：

```
robot_interfaces PR → 合入并打 tag（如 v0.2.0）
        ↓
STM32 PR（升 submodule 指针 + 重新生成 typesupport）
        ↓
RaspberryPi_SLAM PR（升 submodule 指针 + 适配代码）
```

三个 PR 的描述里都写 `Part n/3 of #<issue 编号>`，方便串起来看。

**检查点**：合并后 `git switch main && git pull --rebase`，你的改动出现在 `git log` 里。

---

## 第 11 步：收尾

```bash
git switch main
git pull --rebase origin main
git branch -d feat/wm-control-add-joy-teleop
git submodule update --init
```

---

## 出问题了怎么办

| 现象 | 原因 | 解决 |
| --- | --- | --- |
| `src/Robot_Interfaces` 是空目录 | 克隆时没带 submodule | `git submodule update --init --recursive` |
| `git status` 显示 `modified: src/Robot_Interfaces (new commits)` | 主线改了指针，你没同步 | `git submodule update --init` |
| 推不上去，`non-fast-forward` | rebase 过 | `git push --force-with-lease`（仅自己分支） |
| `pre-push` 拒绝推 `main` | 你在 main 上直接推 | 开分支走 PR |
| 提交被拒，`✗ 提交信息不合规` | 格式不对 | 用 `czg` |
| 提交被拒，`✗ 分支名不合规` | 分支名不对 | `git branch -m 新名字` |
| rebase 到一半乱了 | 冲突没解完 | `git rebase --abort` 重来 |
| 完全不知道现在什么状态 | —— | 先 `git status`，把输出贴到群里，**不要 force push** |

---

## 三件绝对不要做的事

1. `git push -f origin main`
2. 在 `main` 上直接改（`pre-push` 会拒，别试）
3. `git merge main` 进功能分支（用 `rebase`）

---

## 名词表

| 词 | 意思 |
| --- | --- |
| `main` | 主线，唯一长期分支，永远能构建 |
| 分支 | 你干活的地方，从 `main` 切，3 天内合回去 |
| `rebase` | 把你分支上的提交「挪到」最新主线之上，历史保持一条直线 |
| `squash` | 把 PR 里的多个提交压成一个再合入，主线历史干净 |
| PR | Pull Request，请求把你的分支合进 `main`，也是评审发生的地方 |
| `tag` | 给某个提交起的固定名字，如 `v0.1.0`。接口库靠它锁定版本 |
| submodule | 一个仓库里嵌另一个仓库，这里用来引用 `robot_interfaces` |
| pr-agent | 自动审查 PR 的机器人，读完代码留一条评论 |
| CI | 自动跑检查的流水线。本项目只跑「标题校验」和「AI 审查」 |

---

## 规则在哪

- **规则**（MUST / 禁止，用 R 编号）：[CONTRIBUTING.md](https://github.com/Intelligent-Bean-Seeder/.github/blob/main/CONTRIBUTING.md)
- **给个人 AI 助手的规则**：[AI_AGENT_RULES.md](https://github.com/Intelligent-Bean-Seeder/.github/blob/main/AI_AGENT_RULES.md)
- **模块负责人**：[OWNERS.md](https://github.com/Intelligent-Bean-Seeder/.github/blob/main/OWNERS.md)

本文只写「怎么做」，规则以 CONTRIBUTING 为准。
