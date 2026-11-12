# 管家作业手册 · 电脑整理篇

> 这份文件是「亲爱的管家」的作业底稿：整理/改名/清理前先看这里，别再凭感觉动手。
> 2026-10-27 立。教训来源：本机一次 `YULE` 目录改名把 13 个开始菜单、6 个服务、10 个注册表项全部改断。

## 一、铁律（违反过，记终生）

1. **先查后动**：涉本机状态先采集。搬/改一个目录前，先查它被谁引用。
2. **先预览后动手**：给出清单，得到「准」再执行。
3. **能撤才动**：没有回滚方案的批量操作不做。
4. **不用障眼法**：隐藏入口、临时联接、假改名，一律不算解决。要么真改，要么不改。
5. **权限不够就说**：需要管理员/UAC 才能做干净的事，提前说明，不硬凑。
6. **失败要认**：办砸了先回滚，再复盘，不粉饰。

## 二、动程序目录前，必须查这四类引用

用 `tidy_apply kind=folders` 时插件会自动查（`detectDependencies`），手工核对时等价于：

```powershell
$p = 'D:\某个目录'
# 1) 服务
Get-CimInstance Win32_Service | Where-Object { $_.PathName -like "*$p*" }
# 2) 开机自启（HKCU/HKLM Run）
Get-CimInstance Win32_StartupCommand | Where-Object { $_.Command -like "*$p*" }
# 3) 计划任务
Get-ScheduledTask | Where-Object { $_.Actions.Execute -like "*$p*" }
# 4) 开始菜单快捷方式
(New-Object -ComObject WScript.Shell)  # 逐个 .lnk 读 TargetPath 比对
```

**还有第五类，最容易漏**：注册表卸载项 `HKLM\SOFTWARE\...\Uninstall\*` 里的 `InstallLocation` / `DisplayIcon` / `UninstallString`。

命中任意一类 → 改名/搬家就会让那部分功能断掉。要么先修引用（需管理员），要么别动。

## 三、Windows 上「安全可清理」与「绝不可碰」

| 可清（可重建） | 路径 |
|---|---|
| 用户临时 | `%LOCALAPPDATA%\Temp` |
| 系统临时 | `%SystemRoot%\Temp` |
| Windows 更新缓存 | `%SystemRoot%\SoftwareDistribution\Download` |
| 程序崩溃转储 | `%LOCALAPPDATA%\CrashDumps` |
| 系统小转储 | `%SystemRoot%\Minidump` |
| 回收站 | `C:\$Recycle.Bin`、`D:\$Recycle.Bin`（删除=永久，需点名） |

| 绝不可删 | 原因 |
|---|---|
| `%SystemRoot%\WinSxS` | 组件存储，只能用 `DISM /StartComponentCleanup` |
| `%SystemRoot%\Installer` | MSI 卸载/修复依赖 |
| `%SystemRoot%\SoftwareDistribution\DataStore` | 更新数据库 |
| `C:\Windows.old` | 需管理员或「磁盘清理」，删错影响回滚 |
| 程序安装目录（`Qingjian`、`Entertainment`、`Games` 之类） | 有服务/自启/注册表引用 |

## 四、查重与空目录的方法论

- **查重两段式**（学 fclones / czkawka）：①按文件大小分组 ②只对同尺寸候选算「前 64 KB 哈希」③命中再算全量哈希。
  这样 90% 的无关文件在第一轮就被排除，不读全盘。
- **空目录**：自底向上删，父目录会在子目录清空后变空，要循环几轮。
- **大文件**：按体量排序看，不要盲目删；先看是不是镜像/游戏/备份。
- **可回收空间** = 重复组 `size × (份数 − 1)`。

## 五、诊断工具箱（本机已验证可用）

| 工具 | 用途 |
|---|---|
| `handle64.exe`（Sysinternals，放在 `D:\AI\dsh-data\tidy\tools`） | 查「谁咬着这个文件/目录」——排除占用问题的第一把刀 |
| `Get-CimInstance Win32_Service/StartupCommand` | 查服务与自启 |
| `Get-ScheduledTask` | 查计划任务 |
| WScript.Shell `CreateShortcut` | 读写 `.lnk` 的目标/图标路径 |
| `Get-Volume` / `statfs` | 磁盘余量 |

## 六、BleachBit 的两条经验（清洁规则怎么写）

1. **路径用变量，不写死**：`%LocalAppData%`、`%ProgramData%`、`~`。换机器/换语言才不会废。
2. **想删某应用的数据，先让它跑一遍**：把功能都点开、日志打开，用 [Process Monitor](https://learn.microsoft.com/en-us/sysinternals/downloads/procmon) 看它到底写了哪些文件，再决定删什么。
3. 注册表里的应用痕迹一般在 `HKCU\Software\<应用名>`、`HKLM\SOFTWARE\<应用名>`。

## 七、事故复盘：YULE（2026-10-27）

- **做了什么**：把 `D:\YULE` 改名 `D:\Entertainment`，只修了桌面快捷方式。
- **漏了什么**：13 个系统级开始菜单、6 个服务、10 个注册表卸载项、1 个计划任务。
- **后果**：开始菜单图标全变白、卸载失灵、部分服务重启后起不来。
- **正确做法**：先用 `detectDependencies` 把四类引用查出来 → 列给用户 → 得到「连引用一起改」的批准 → 在有管理员权限时一次改完 → 逐项验证。
- **已固化**：插件 v0.4 起，`kind=folders` 命中引用默认拒绝执行（`blockers`），除非 `force:true`。
