# 自己更新 Release 的步骤

适用仓库：`KKHTAKOISHI/TH155-MOD`
Release 页面：<https://github.com/KKHTAKOISHI/TH155-MOD/releases/tag/pak-files>

---

## 0. 先搞清楚：哪些文件要更新

| 你改了什么 | 需要更新 |
|---|---|
| `th155b.pak`（改脚本 / 数值 / 帧数条） | Release 里的 **`th155b.pak`** + README 的校验值 |
| `netcode.ini`（改帧数条位置、淡出时长） | Release 里的 **`netcode.ini`** + README 的校验值 |
| 换了 `Netcode.dll` / `th155r.exe` | 对应的那个附件 + README 的校验值 |
| 只是改了说明文字 | 只改 Release 说明 / README |

**只改 `th155.pak` 是绝对不要做的** —— 那个是原版主数据包，不要替换、不要上传。

---

## 1. 算出新文件的校验值

在文件所在目录打开 PowerShell（或 CMD），执行：

```powershell
certutil -hashfile th155b.pak SHA256
```

输出中间那一行就是 SHA256（**去掉空格、全部小写**再填进 README）。

多个文件一起算：

```powershell
Get-ChildItem th155b.pak, netcode.ini, Netcode.dll, th155r.exe |
  ForEach-Object { "{0,-14} {1,12:N0} B  {2}" -f $_.Name, $_.Length, (Get-FileHash $_ -Algorithm SHA256).Hash }
```

---

## 2. 替换 Release 里的附件（网页操作）

### 2.1 进入编辑

1. 打开 <https://github.com/KKHTAKOISHI/TH155-MOD/releases/tag/pak-files>
2. 右上角点 **✏️ Edit release**（铅笔图标）

### 2.2 删掉旧附件

在页面下方 **Attach binaries by dropping them here or selecting them.** 区域，会看到已有附件的列表：

```
th155b.pak     96,291,705 Bytes
netcode.ini     1,699 Bytes
...
```

- 鼠标移到 **`th155b.pak`** 那一行 → 右侧出现 **✗**（或 **Delete**）→ 点它删掉

> ⚠️ **必须先删掉旧的同名附件**。GitHub **不允许两个同名附件**，不删的话上传会失败（或变成 `th155b-1.pak`）。

### 2.3 上传新附件

- 把新的 `th155b.pak` **直接拖到** "Attach binaries by dropping them here" 区域
- 或者点 **selecting them** → 选择文件
- 92 MB 左右大概需要 **1～3 分钟**，页面上会出现进度条，等它变成文件名 + 大小才算完成

### 2.4 更新说明里的校验值

在 **Describe this release** 文本框里，把 `## 校验值 (SHA-256)` 那一段的 `th155b.pak` 一行改成新值：

```
th155b.pak   <新的 SHA256>
```

### 2.5 保存

- 点最下面的 **Update release** 按钮
- 回到 Release 页面，**刷新**，确认：
  - 附件列表里是**新的** `th155b.pak`，大小正确
  - 没有多余的 `th155b-1.pak` 之类

---

## 3. 更新 README.md 的校验值（网页操作）

1. 打开 <https://github.com/KKHTAKOISHI/TH155-MOD/blob/main/README.md>
2. 右上角点 **✏️**（Edit this file）
3. 找到这一段，改掉：

   ```
   | `th155b.pak` | 96291705 | `cdda2b06...` |
   ```
   → 大小改成新文件的字节数，SHA256 改成第 1 步算出来的值

4. 页面底部 **Commit changes** → 填一句说明（例如 `docs: 更新 th155b 校验值 (新hash前8位)`）→ **Commit changes**

---

## 4. 更新 CHANGES.md（有功能改动时才需要）

1. 打开 <https://github.com/KKHTAKOISHI/TH155-MOD/blob/main/CHANGES.md>
2. 点 **✏️**
3. 在对应章节里加一条改动说明（照现有格式写就行）
4. **Commit changes**

---

## 5. 通知联机的人

**联机双方必须用完全相同的 `th155b.pak`**。改完之后：

- 让对面**重新下载** Release 里的 `th155b.pak`
- 双方各自核对校验值一致：

  ```powershell
  certutil -hashfile th155b.pak SHA256
  ```

> 注意：**文件大小相同不代表内容相同**（pak 是等长替换，改脚本后大小几乎不变）。**必须比 SHA256。**

---

## 附：用命令行 / API 上传（可选，比网页快）

### A. 只推 README / CHANGES 到仓库

```powershell
cd D:\th155-mods
git fetch origin
git checkout main
git reset --hard origin/main      # 如果本地 main 落后
# 改完 README.md / CHANGES.md 之后:
git add README.md CHANGES.md
git commit -m "docs: 更新校验值"
git push
```

### B. 用 API 替换 Release 附件

需要一个有 `repo` 权限的 GitHub Token（Fine-grained 或 Classic 都行）。

```powershell
$token = "你的token"
$repo  = "KKHTAKOISHI/TH155-MOD"
$relId = 394736240        # 该 Release 的 id（见下方说明）
$H = @{ Authorization = "token $token"; "User-Agent" = "me" }

# 1) 查附件列表，拿到要删的附件 id
Invoke-RestMethod -Uri "https://api.github.com/repos/$repo/releases/$relId/assets" -Headers $H |
  Select-Object id, name, size

# 2) 删除旧附件
Invoke-RestMethod -Uri "https://api.github.com/repos/$repo/releases/assets/<附件id>" `
  -Method Delete -Headers $H

# 3) 上传新附件
Invoke-WebRequest -Uri "https://uploads.github.com/repos/$repo/releases/$relId/assets?name=th155b.pak" `
  -Method Post -Headers $H -ContentType "application/octet-stream" `
  -InFile "D:\th155\th155b.pak" -TimeoutSec 3600
```

Release 的 id 怎么查：

```powershell
Invoke-RestMethod -Uri "https://api.github.com/repos/KKHTAKOISHI/TH155-MOD/releases/tags/pak-files" -Headers $H |
  Select-Object id, name, tag_name
```

---

## 常见问题

| 现象 | 原因 / 解决 |
|---|---|
| 上传后变成 `th155b-1.pak` | 没先删旧附件。删掉两个，重新上传一个 |
| 上传卡住 / 失败 | 换浏览器或用命令行方式（附录 B）；92 MB 需要几分钟 |
| 校验值对不上 | `certutil` 输出里**有空格**，README 里要写成**连续小写**；确认算的是**新**文件 |
| 别人说"大小一样但联机不同步" | 双方 pak 不同。让双方都从 Release 重新下载并比 SHA256 |
| README 改完没生效 | 确认提交到了 **`main`** 分支（不是其他分支） |
| 想回退到上一版 pak | 提前把旧 pak 另存一份；Release 附件被替换后无法找回 |
