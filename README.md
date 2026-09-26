# TH155-MOD

东方凭依华（Touhou 15.5 / AoCF）改版 pak —— **只需替换一个文件**，不含源码与实验记录。

当前生效的全部改动见 **[CHANGES.md](CHANGES.md)**。

## 下载

| 文件 | 大小 | 下载 |
|---|---|---|
| `th155b.pak` | 91.8 MB | [Releases → th155b.pak](https://github.com/KKHTAKOISHI/TH155-MOD/releases/download/pak-files/th155b.pak) |

**只需把游戏目录里的 `th155b.pak` 换成这一个，`th155.pak` 保持原版不动。**

`th155b.pak` 是覆盖包（游戏读取同名条目时以它为准），本项目的**全部改动都在它里面**——包括符卡数值，所以不需要动那个 1 GB 的 `th155.pak`。

## 校验

| 文件 | 大小 (B) | SHA256 |
|---|---|---|
| `th155b.pak` | 96291705 | `0c3669242500541cb3827dfe29adc692a18181c1a1fd30d52cd716488cfef7b1` |

> 校验值对应**本次发布**的这一版 pak；pak 一旦更新，本页与 Release 附件会同步刷新。

## 联机

双方必须使用**同一份 `th155b.pak`**：下载后核对 SHA256 一致即可（大小相同并不足以保证一致）。

Windows 下核对：
```
certutil -hashfile th155b.pak SHA256
```

## 说明

- 所有改动均为 **pak 内等长原地替换**：不重打包、不改 exe，用原版 `.bak` 覆盖即可完整还原。
- 游戏本体资源版权归 Twilight Frontier 所有，请自行持有正版游戏；本仓库仅提供修改后的归档文件。