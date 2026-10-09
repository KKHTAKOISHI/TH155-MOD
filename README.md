# TH155-MOD

东方凭依华（Touhou 15.5 / AoCF）改版 pak —— **只需替换一个文件**，不含源码与实验记录。

当前生效的全部改动见 **[CHANGES.md](CHANGES.md)**。

## 下载

全部文件都在同一个 Release 里：**[Releases → pak-files](https://github.com/KKHTAKOISHI/TH155-MOD/releases/tag/pak-files)**

| 文件 | 大小 | 说明 |
|---|---|---|
| [`th155b.pak`](https://github.com/KKHTAKOISHI/TH155-MOD/releases/download/pak-files/th155b.pak) | 91.8 MB | **生效包** —— 联机与帧数条增强都靠它 |
| [`netcode.ini`](https://github.com/KKHTAKOISHI/TH155-MOD/releases/download/pak-files/netcode.ini) | 1.7 KB | 155r 网络模组设置（帧数条位置 / 淡出时长）|
| [`Netcode.dll`](https://github.com/KKHTAKOISHI/TH155-MOD/releases/download/pak-files/Netcode.dll) | 657 KB | 155r 网络模组（帧数条 / 逐帧 / 输入显示）|
| [`th155r.exe`](https://github.com/KKHTAKOISHI/TH155-MOD/releases/download/pak-files/th155r.exe) | 110 KB | 配套启动器（**必须用它启动**）|
| [`README.md`](https://github.com/KKHTAKOISHI/TH155-MOD/releases/download/pak-files/README.md) | 3.4 KB | 安装说明（含校验值）|

### 要下载哪些？

- **只想联机**（已经装过 155r 网络模组）：只需 **`th155b.pak`**
- **还想用练习模式的两条帧数条**：需要 **`th155b.pak` + `netcode.ini` + `Netcode.dll` + `th155r.exe`** 四个
- 用 **`th155r.exe`** 启动游戏（不要用 `th155.exe`）

**`th155.pak`（约 1 GB）保持原版不动。**

`th155b.pak` 是覆盖包（游戏读取同名条目时以它为准），本项目的**全部改动都在它里面**——包括符卡数值，所以不需要动那个 1 GB 的 `th155.pak`。

## 校验

| 文件 | 大小 (B) | SHA256 |
|---|---|---|
| `th155b.pak` | 96291705 | `cdda2b066252a592481e7b441192695417bf243e79a8aab44d79df45005f6095` |
| `netcode.ini` | 1699 | `de7fd99504e1acc096fe1418e42fbddea79d1c0374bd9515f1f11a7db3d83758` |
| `Netcode.dll` | 656896 | `4d07174606c4a6a1a38118b898c66a6799a8378a4513e7ad4dbc7bab7d4d8c07` |
| `th155r.exe` | 110080 | `a711c4b3f9b1128386f8176491af7128d4d5ec6374e08b1715529d71385bab96` |

> 校验值对应**本次发布**的这一版；pak 一旦更新，本页与 Release 附件会同步刷新。

## 联机

双方必须使用**同一份 `th155b.pak`**：下载后核对 SHA256 一致即可（大小相同并不足以保证一致）。

Windows 下核对：
```
certutil -hashfile th155b.pak SHA256
```

## 录像快进 / 快退

播放录像（Replay）时，**把方向键 / 摇杆推到最右或最左**即可跳转：

| 操作 | 效果 |
|---|---|
| 推到**最右**（`x >= +0.5`） | **快进 5 秒**（450 帧）|
| 推到**最左**（`x <= -0.5`） | **快退 5 秒**（450 帧）|

- 每**拨动一次**跳 450 帧（约 5 秒，游戏逻辑 90 帧/秒）；松开再拨可连续跳
- **快进是瞬时的**：直接多跑几次战斗更新，没有额外开销
- **快退需要重算**：战斗状态无法回滚，只能**从第 0 帧重新模拟到目标帧**，所以**目标越靠后越慢**
- 跳转过程中即使碰到录像结尾也**不会中断播放**（有专门守卫，避免跳转途中把场景拆掉）

> 代码位置：触发在 `battle_team.nut`（每帧读方向输入），执行在 `battle_on_hit.nut`，结尾守卫在 `battle_replay.nut` —— 都在 `th155b.pak` 内。

## 练习模式帧数条

配合 155r 网络模组使用，练习模式下会显示**两条**帧数条：

**P1 帧数条（上方，`netcode.ini` 里 `y=110`）**
- **绿** = 发生帧 / **红** = 持续帧 / **蓝** = 后摇帧
- 已修正：不再被其他角色（例如你正在操作的 2P）的动作打乱

**P2 帧数条（下方，位置 = `y + 420`）**
- **灰** = P1 出招后、命中前的帧
- **紫** = P2 **打防硬直**帧（**冻结帧不计**）
- 硬直结束后保持不变，直到 P1 **下一次出招**才整条刷新
- 淡出时长与 P1 那条一致（`netcode.ini` 里 `timer=240`）

**两条都从 2 帧开始计数**，逐帧同步。

**录制 / 播放宏（record / play）期间，两条帧数条会自动隐藏**，停止后恢复。

想调整位置或时长，改 `netcode.ini` 的 `[frame_data_display]` 段（改完**重开游戏**生效）。

## 说明

- 所有改动均为 **pak 内等长原地替换**：不重打包、不改 exe，用原版 `.bak` 覆盖即可完整还原。
- 帧数条依赖 155r 网络模组（`Netcode.dll` / `th155r.exe`），请使用本 Release 提供的版本以保证接口一致。
- 打防成功的第一帧会有一小段停顿，这是游戏本身的**命中冻结（hitstop）**，属原作手感，非本模组引入。
- 游戏本体资源版权归 Twilight Frontier 所有，请自行持有正版游戏；本仓库仅提供修改后的归档文件。
