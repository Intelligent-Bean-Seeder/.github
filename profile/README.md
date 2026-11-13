# 揭榜挂帅 · 自主播种机器人

面向播种作业的自主移动机器人：树莓派上位机（ROS 2）负责感知、决策与调度，STM32 下位机负责电机与传感器，两端通过统一的接口仓通信。

## 仓库结构

| 仓库 | 职责 |
| --- | --- |
| `robot_interfaces` | 接口契约（`.msg`），版本以 tag 标记 |
| `RaspberryPi_SLAM` | 上位机：ROS 2 Jazzy，感知 / 规划 / 主控 |
| `STM32` | 下位机：STM32F407，micro-ROS 节点 |

三个库都是**私有库**，需要先被拉进组织（默认权限已是 write，加进来就能推）。

---

## 一、上手：克隆 + 装钩子

```bash
git clone --recurse-submodules https://github.com/Intelligent-Bean-Seeder/RaspberryPi_SLAM.git
cd RaspberryPi_SLAM
bash scripts/install-hooks.sh
```

三条都是必须的，尤其后两条：

- `--recurse-submodules` 少了 → `src/Robot_Interfaces` 是空目录，编不过；
- `install-hooks.sh` 少了 → 你的提交**不会被校验**，坏分支名也能提交，但到 PR 环节会被打回。

已经克隆过、发现接口库是空的：

```bash
git submodule update --init --recursive
```

---

## 二、cz-git：用菜单写提交信息

### 装

```bash
npm i -g cz-git
czg --version
```

装完就有 `czg` 命令。仓库根目录的 `.czrc` 已经写好了 type / scope 白名单，不用再初始化。

> 没有 Node 也行：手写 `git commit -m "feat(control): 增加摇杆遥控"`，格式完全一样，钩子只认格式不认工具。

### 用

```bash
git add -A
czg
```

按提示走：

```
? 选择你要提交的类型 › feat
? 选择一个模块范围 › control
? 填写简短描述 › 增加摇杆遥控
? 是否破坏性变更 › 否
```

生成 `feat(control): 增加摇杆遥控`。也可以跳过第一步：`czg feat`。

### type 怎么选

| type | 什么时候用 |
| --- | --- |
| `feat` | 新增功能、新增参数 |
| `fix` | 修 bug |
| `docs` | 只改文档、注释 |
| `refactor` | 不改行为的重构 |
| `perf` | 性能优化 |
| `test` | 测试 |
| `build` | 构建脚本、依赖版本 |
| `ci` | 流水线、钩子、配置 |
| `chore` | 杂务（删文件、格式化） |
| `proto` | **只改 `.msg`**（仅接口仓） |

三个最容易选错的：

- 改 `.msg` → 用 `proto`，不要用 `feat`；
- 升 `submodule` 指针 → 用 `deps`；
- 改 `pr-agent.yml` / 钩子 → 用 `ci`。

### scope 怎么选

`radar` `vision` `ml` `control` `hw` `proto` `fi` `bringup` `sim` `ci` `docs` `deps`

选不准就问自己：这次改动**主要**落在哪个模块。宁可粗，不要编一个新词——钩子只认这 12 个。

### 破坏性变更

`czg` 会问「是否破坏性变更」，选「是」会生成 `proto!: xxx`。接口仓里凡是删字段、改类型、改语义，都算破坏性，正文再补一句：

```
BREAKING CHANGE: ActuatorCommand 删除 velocity_ff，两端需同步升级到 v0.2.0
```

### 常见报错

| 报错 | 原因 |
| --- | --- |
| `✗ 提交信息不合规：update stuff` | 没按 `type(scope): 描述` 写 |
| `✗ 提交信息不合规：feat(nosuchscope)` | scope 不在 12 个白名单里 |
| `✗ 分支名不合规：dev` | 分支名没按规则，见下一节 |
| `czg: command not found` | npm 全局 bin 没进 PATH |

---

## 三、git 怎么同步

### 每天开工（三条）

```bash
git switch main
git pull --rebase origin main
git submodule update --init --recursive
```

### 开新分支

```bash
git switch -c feat/wm-control-add-joy-teleop origin/main
```

命名规则：`<type>/<initials>-<scope>-<slug>`。`initials` 是你名字的缩写（`wm` 吴冕、`wy` 王滢、`zth` 张天浩……完整表见 CONTRIBUTING），`slug` 用英文小写加连字符。

### 写代码期间，把主线的更新接进来

**不要 merge main 进你的分支**，用 rebase：

```bash
git fetch origin
git rebase origin/main
```

冲突了：

```bash
git status              # 看哪几个文件冲突
# 手动改完，删掉 <<<<<<< 标记
git add <冲突文件>
git rebase --continue

git rebase --abort      # 想放弃这次 rebase
```

### 推送

```bash
git push -u origin HEAD
```

如果 rebase 过、之前又推过，普通 push 会被拒：

```bash
git push --force-with-lease
```

`--force-with-lease` **只能对自己的分支用**。对 `main` 永远不要 force。

### 接口库升级了怎么办

`robot_interfaces` 发了新 tag 之后，只有协议负责人动指针：

```bash
cd src/Robot_Interfaces
git fetch --tags
git checkout v0.2.0
cd -
git add src/Robot_Interfaces
git commit -m "build(deps): 接口库升级到 v0.2.0"
```

其他人只需要：

```bash
git pull --rebase origin main
git submodule update --init --recursive
```

如果 `git status` 里出现 `modified: src/Robot_Interfaces (new commits)`，说明指针变了但没更新，跑一次 `git submodule update --init`。

### 三件绝对不要做的事

1. `git push -f origin main`
2. 在 `main` 上直接改（`pre-push` 会拒，但别试）
3. `git merge main` 进功能分支

### 真的搞乱了

```bash
git status              # 先读它说了什么，九成情况它会告诉你下一步
git stash               # 临时收起来
git log --oneline -5    # 看自己在哪
```

搞不定就**别硬来**，把 `git status` 的输出贴到群里——比 force push 安全得多。

---

## 四、提 PR

```bash
git push -u origin HEAD
gh pr create --fill     # 或者到网页上点
```

标题就是 squash 后的提交信息，必须符合 `type(scope): 描述`，否则 `PR Title` 检查会红。描述按模板五段填全。

合并用 **squash**，合并后删源分支。

---

## 五、协作

- 规范：[CONTRIBUTING.md](https://github.com/Intelligent-Bean-Seeder/.github/blob/main/CONTRIBUTING.md)
- 给个人 AI 助手的规则：[AI_AGENT_RULES.md](https://github.com/Intelligent-Bean-Seeder/.github/blob/main/AI_AGENT_RULES.md)
- 模块负责人：[OWNERS.md](https://github.com/Intelligent-Bean-Seeder/.github/blob/main/OWNERS.md)

## 约定速记

- 分支：`<type>/<initials>-<scope>-<slug>`，存活 ≤ 3 天
- 提交：Conventional Commits，`czg` 交互式填写
- 主线：只有 `main`，只接受 PR，squash 合并
- 接口：只改 `robot_interfaces`，三库联动、先后有序
