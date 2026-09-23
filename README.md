# th155-mods — 东方凭依华(AoCF)解包 / 反编译 / Mod 管线

对《东方凭依华》(Touhou 15.5, tasofro, TFPK 引擎)的解包、Squirrel 脚本反编译、
以及基于字节码补丁的游戏修改工具集。所有修改均为**等长原地补丁**,不重打包、
不修改 exe,可通过备份文件随时完整还原。

> 仅用于学习研究。游戏资源版权归 Twilight Frontier 所有。

## 当前生效的修改(2026-09-23 复核)

> 详细"怎么改的 / 怎么回滚"见 `PATCHES.md`;逐项**实测现值**见 `TH155_mod_core_notes.md`。

**系统级**

| 修改 | 落点 |
|---|---|
| 受伤倍率恒定 **1.0** + 打康固定 **1.2**(guts 常量化 + 康分支 JCMP 翻转) | `battle_team.nut` / `battle_on_hit.nut` |
| 全角色投技伤害统一 **1100** | 各角色 pat motion 1802 |
| **投技独立按键 E**(scancode 18 → 输入槽 b5;键位设置菜单新增「投技 / Throw / 投げ」行) | `input.nut` / `input_command.nut` / `player_input.nut` / `key_config*.nut` 等 6 条目,**仅 th155b.pak** |
| 投技键也能**拆投**(读受害者设备槽 `input.b5`,**不读** `rsv_k5`) | th155b.pak,**仅 th155b**;详见 `PATCHES.md` §23 |
| LW(Climax) 伤害不再随灵力/符卡 cost 缩放(`atkRate_Pat` 改常量 1.0) | `player_spell.nut` |
| 狸子 6B 弹丸存活窗口 40 → 35 帧 | `mamizou_shot.nut` |
| 狸子符卡「変化「二ッ岩家の裁き」」消耗 800 → 1000 | `data/spell/mamizou.csv`(**只在 th155.pak**) |
| 华扇彩虹球伤害 −500(716/1216);老鹰只在球真命中时追击 | `kasen` pat / `kasen_eagle.nut` |

**招式级**

| 角色 | 改动 |
|---|---|
| 神子 | j5C 判定框缩短一半;DA 起手三套形态统一 9 帧(红袍 = `style 1` = motion 1302) |
| 哆来咪 | 4C(SP_Bound)、6B(Shot_Front) 回退 1.10 |
| 女苑 | LW 抓取判定框宽度 8.5 → 40.0 |
| 恋恋 | 6D 瞬移后取消后摇(按住方向仍连跑);A/B/C 系列「超反应」只在打空(含擦弹/打防)触发;8A・j8A 判定范围 +1/3 |
| 狸子 | 8A 每个攻击框处新增同款受击框(打人也掉叶子) |
| 堇子 | 6C 后摇压缩(第一下 31 → 16 帧) |
| 白蓮 | 符卡「大日如来の輝き」每次命中 −50(motion 7011 档0/档2) |
| 魔理沙 | 6D 前摇 9 → 4 帧 |
| 小碗 | 空中冲刺前摇 10/6 → 4 帧;符卡「小槌」前 4 段击退 + 前后双面判定(定稿数值见 `PATCHES.md` §22k) |

> 背景替换 / 血量上限等实验**已全部回滚**为原版。
> C 盘(桌面「东方凭依华1.21」)曾被打过补丁,**已完整还原**并经 MD5 校验与原版逐字节一致——那个目录不要再动。

## 目录结构

```
th155-mods/
├── scripts/              全部处理脚本(截至 2026-09-23 共 692 个,见下表)
├── nutcracker/
│   ├── NutCracker-patched/   NutCracker 源码(含为本项目做的兼容性修补)
│   ├── nutcracker.exe        zig cc 编译好的反编译器(可直接用)
│   └── build.bat             编译命令(需要 zig: pip install ziglang)
├── reference/            135tk 中 TFPK 格式的参考实现(C++)
├── PATCHES.md            ★ 全部补丁的字节级记录(模式/位置/效果/回滚)
├── TH155_mod_core_notes.md  ★ 核心资料总汇(当前状态 / 引擎机制 / 方法论 / 踩坑)
└── README.md
```

> **工作流**:`D:\memory` 是当天的实盘工作目录(临时脚本 + `_*_backup` 备份 + 整包快照),
> 每次"保存"时把其中的 `.py` 脚本同步进 `scripts/`、更新 `PATCHES.md`、然后 commit。
> 路径依赖:`scripts/` 里多数脚本硬编码 `D:\tool\th155_scripts`(工具链)、
> `D:\th155\*.pak`(游戏)、`D:\memory\_*`(备份/临时)。

## 脚本索引(scripts/)

**核心管线**
| 脚本 | 用途 |
|---|---|
| `tfpk_extract.py` | TFPK v1 解包器:RSA 头部块 + 每文件 XOR 解密 + 归档内文件名表。也提供 `uncrypt_block` 等供其他脚本 import |
| `harvest3.py` | 文件名还原:从全文件内容(含 SJIS 字节串)收割 `data/...` 路径 + 编号/扩展名枚举,按 FNV 哈希匹配 |
| `apply_names.py` | 把匹配到的哈希重命名为真实路径 |
| `batch_decompile.py` | 用 nutcracker.exe 批量反编译全部 .nut(成功 698/710) |
| `integrate_decomp.py` | 整合社区 th155-decomp 的文件名清单(额外命名 106 个) |

**Mod 补丁**
| 脚本 | 用途 |
|---|---|
| `mod_guts.py` | guts 减伤移除(当前生效),内含验证 |
| `apply_throwkey.py` / `verify_throwkey.py` | 投技独立按键 E + 键位菜单新行(6 条目,仅 th155b) |
| `mod_throwbreak_key.py` / `verify_throwbreak.py` | 投技键也能拆投(仅 th155b) |
| `fix_lw_scaling.py` | LW 伤害去灵力/符卡 cost 缩放(幂等) |
| `mod_throw_all_1100.py` / `check_throw_both_paks.py` | 全角色投技伤害 1100 |
| `mod_j5c_shrink.py` / `mod_miko_da_red_fix.py` / `verify_miko_da_red.py` | 神子 j5C / DA 起手 |
| `mod_jyoon_lw_width.py` / `verify_jyoon_lw_width.py` | 女苑 LW 抓取框宽度 |
| `mod_koishi_dash_recovery.py` / `mod_koishi_auto_v4.py` / `mod_koishi_8c_v4.py` / `mod_koishi_8a_range.py` / `mod_koishi_j8a_range.py` | 恋恋 6D 后摇 / A·B 超反应 / 8C / 8A・j8A 范围 |
| `mod_mamizou_8a_identical.py` / `verify_mamizou_8a_identical.py` / `find_mergeable_frames.py` | 狸子 8A 受击框(含"腾字节"用的相同帧合并) |
| `mod_mamizou_card_cost.py` / `mod_mamizou_6b*` | 狸子符卡消耗 / 6B 弹丸窗口 |
| `mod_usami_6c*.py` / `verify_usami_6c.py` | 堇子 6C 后摇(pat + 脚本) |
| `mod_hijiri_spellB_dmg*.py` / `verify_hijiri_spellB.py` | 白蓮符卡伤害 |
| `mod_marisa_dash.py` / `mod_sm_airdash.py` | 魔理沙 6D / 小碗空中冲刺前摇 |
| `mod_sm_kotuchi_dmg_knock.py` / `verify_sm_kotuchi.py` | ★ 小碗符卡「小槌」定稿(幂等:从原版快照重来) |
| `mod_sm_kobito_final.py` / `mod_sm_kobito_shot_push.py` | 小碗符卡「小人」接触框击退(当前已回原版) |
| `mod_doremy_4c_6b_lw.py` / `sqir_walk.py` / `strip_lineinfo.py` | 哆来咪回退 1.10 的"函数段级字节替换"工具链 |
| `mod_eagle_no_graze.py` | 华扇老鹰只在真命中时追击 |
| `revert_*.py` / `restore_*.py` | 各补丁的一键还原(详见 PATCHES.md 各章末尾) |
| `mod_background.py` / `mod_all_dds.py` / `mod_conservative_bg.py` | 背景替换范例(已回滚,保留作参考) |
| `restore_d.py` / `restore_c.py` | 从 .bak 一键还原 pak |

**分析 / 诊断**(过程记录,可按需查阅)
`dump_all_boxes.py` / `find_melee_boxes.py` / `pair_sm_takes.py` / `map_spell_takes.py`(判定框与档位定位)、
`diff_sm_pat.py`(与快照逐字段 diff)、`mod_sm_fingerprint.py`(数字指纹探针)、
`find_damage_calc.py`、`inventory_dds.py`、`dbg_*.py`、`inspect_*.py`、`find_*.py`、
`validate_hash.py`、`count_patterns.py` 等。

## 快速上手

```bat
:: 解包(需要 numpy;游戏目录以 D:\th155 为例)
python scripts\tfpk_extract.py D:\th155\th155.pak D:\out\th155

:: 反编译单个 .nut 字节码
nutcracker\nutcracker.exe D:\out\th155\data\actor\reimu.nut

:: 批量反编译
python scripts\batch_decompile.py

:: 重新编译 nutcracker.exe(pip install ziglang 之后)
cd nutcracker\NutCracker-patched && ..\build.bat
```

> 脚本里的 pak 路径多为写死的 `D:\th155` / 桌面 1.21 目录,复用时按需改路径。
> 桌面的 1.21 是在玩的版本,脚本不要指向它。

## 格式要点(TFPK v1,TH145/TH155)

- 头部:`TFPK` + 版本字节 `01`;其后所有元数据按 **64 字节 RSA 块**存储
  (TH145/155 公钥见 `tfpk_extract.py` 的 `KEYS_N`,e=0x10001,
  填充 `00 01 FF..FF 00`,payload 在块尾)。
- 块序列:目录数 → 目录项 → 文件名表(zlib 压缩的 SJIS 路径)→ 文件数 →
  每文件 3 块(size/offset、nameHash、16 字节 XOR key)。
- 文件数据:`offset ^= key[1]`,`size ^= key[0]`,`nameHash ^= key[2]`,
  key 四个 DWORD 取负后作 16 字节 XOR 链密钥(解密/加密为不同方向算法)。
- 文件名只存 **FNV-1 哈希**(ASCII 小写、`/`→`\`,结果 ×-1);真实文件名靠
  归档内文件名表 + 内容引用分析还原(本项目恢复 2400+)。

## 已知边界

- 8 个 .nut 字节码 NutCracker 无法反编译(工具本身限制,社区同样失败):
  `data/script/const.nut`、`const_key.nut`(两包)、
  `data/actor/nitori_boss_shot.nut`、`actor_update.nut`、`effect_member.nut`、
  `practice_function.nut`。
- 约 1.6 万资源文件保留哈希名,类型靠魔数推断;清单在各包 `extracted/*/​_files.txt`。
- 引擎 exe(C++)内的硬编码不可数据层修改:血条按 10000 刻度绘制、
  伤害为最大生命百分比制——因此"提高血量上限"类修改会产生伤害同比放大、
  血条卡满的副作用(已实验验证并回滚,详见 PATCHES.md)。

## 还原 / 回滚

三个状态各自的实测 SHA256(2026-09-23 复核,三份互不相同,别混用):

| 状态 | 位置 | th155.pak | th155b.pak |
|---|---|---|---|
| 当前生效 | `D:\th155\*.pak` | `71A5F762…654665B8` | `FAF1C9BF…E85E1D35` |
| 中途快照(09-22 02:46) | `D:\memory\_pak_snapshots\` | `4391215A…26176676` | `23081197…C0D9E886` |
| **原版**(2023-09-24 安装副本) | `D:\th155\*.pak.bak` | `4E754599…1AF1EB10A9` | `D8FC55E1…F2F1E69ED705` |

- 整包回退:用 `.bak` 覆盖回去 = 完全原版;只想退到"某个补丁之前"则用
  `D:\memory\_*_backup\` 里对应的**单个条目**(它们存的是改动前的整条目)。
- 单项回滚:多数补丁脚本自带 `--revert` / `--pristine`(见 `PATCHES.md` 各章末尾);
  血量类见 `revert_hp.py`;小碗符卡见 `mod_sm_kotuchi_dmg_knock.py --pristine`。
- ⚠️ 回滚后**必须从 pak 读回复检**(重新解析 pat / 反编译 nut 逐字段对比)——
  曾出现 `--revert` 逻辑写错而静默失败的情况(PATCHES.md §7.9)。
