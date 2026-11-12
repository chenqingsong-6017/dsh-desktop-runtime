# GitHub 学习笔记 · 电脑管家方向

> 2026-10-27。背景：本机 `github.com` / `raw.githubusercontent.com` 被 DNS 判为非公网地址，无法直接抓正文；以下基于检索摘要与既有认知整理，链接留作以后复核。

## 一、看了哪些

| 项目 | 定位 | 值得学的地方 |
|---|---|---|
| [ChrisTitusTech/winutil](https://github.com/ChrisTitusTech/winutil) | GUI 化 Windows 维护套件：装应用、系统调优、配置、更新、激活 | 每个 tweak **可撤销**；启动时**检测被系统还原掉的 tweak**；改配置前先记录原值 |
| [Cosmos-0118/TidyWindow](https://github.com/Cosmos-0118/TidyWindow) | 清理 / 诊断 / 优化 / 恢复四合一（WPF + PowerShell） | **启动项控制器**、诊断模块独立成块；恢复（recovery）与清理并列，不是附属 |
| [hasoftware/WinTrash](https://github.com/hasoftware/WinTrash) | 专扫 **16 类应用残留**：失效 PATH、孤儿目录、幽灵服务、损坏注册表、启动垃圾 | **逐项备份**再清理；按残留类型分类，不搞"一键全清" |
| [Sysinternals Suite](https://learn.microsoft.com/en-us/sysinternals/) | 微软官方工具箱 | Autoruns（全量启动项）、Process Explorer（句柄/进程树）、TCPView（网络连接）、Handle（本次查占用已用） |
| [BleachBit](https://github.com/bleachbit/bleachbit) | 系统清洁器 | 清洁规则用声明式清单；路径必须走变量 |
| [czkawka](https://github.com/qarmin/czkawka) | 重复/空目录/相似图/大文件 | 每类问题一个独立入口，结果可预览再处置 |
| [fclones](https://github.com/pkolczak/fclones) | 高效查重 | 大小分组 → 部分哈希 → 全量哈希，两段筛 |

## 二、成熟清理软件的共同套路

1. **先扫描、出清单、逐项选择、再执行**——没有一个是"一键全删"。
2. **每步可撤销 / 逐项备份**——winutil、WinTrash、BleachBit 都这么干。
3. **按"问题类型"组织入口**：残留、启动项、缓存、重复、大文件、网络。
4. **风险分级写清楚**：safe / caution / risky，用户自己能判断。
5. **GUI 只是壳，底层全是 PowerShell / 系统 API**——我们走插件路线是对的。
6. **检测"被打回"的改动**（winutil）：系统更新后某些改动会被还原，要能发现并提醒。

## 三、管家能力对照表

| 能力 | 业界做法 | 本机现状 | 差距 |
|---|---|---|---|
| 磁盘空间与垃圾盘点 | czkawka / BleachBit | ✅ `tidy_scan` | — |
| 空目录 / 查重 / 大文件 | czkawka / fclones | ✅ v0.7（查重可隔离） | 大文件排行未独立成入口 |
| 可撤销操作 | 全部 | ✅ 清单 + `tidy_undo` | 空目录删除暂不可撤 |
| 目录引用探查 | 少见，须手工 | ✅ `tidy_scan path` | 业界也没有，属本机特有教训 |
| **应用残留清理**（失效 PATH / 孤儿目录 / 幽灵服务） | WinTrash 16 类 | ❌ 无 | **高价值缺口** |
| **启动项管理**（读→停→删） | Autoruns / winutil | ⚠️ 只能读 | 缺启停（需管理员） |
| **系统体检**（事件日志错误 / 启动耗时 / 服务失败） | TidyWindow / ASMS | ❌ 无 | 中价值缺口 |
| 服务诊断（依赖树、启动失败原因） | Sysinternals | ⚠️ 只能看路径 | 中价值缺口 |
| 网络诊断（DNS / 代理 / 端口） | TCPView | ❌ 无 | 中价值缺口 |
| 更新管理 | winutil | ❌ 无 | 低优先级 |

## 四、下一步可做（按价值/风险排序，待陛下准）

1. **系统体检（只读，低风险）**：`tidy_scan` 加一段——近 7 天事件日志错误数、上次启动耗时、服务启动失败数、已装软件数。给"体检报告"用。
2. **应用残留探查（只读，中价值）**：查失效 PATH 项、已卸载软件的孤儿目录、指向不存在 exe 的服务/自启。只报告，不动手。
3. **启动项清单**：把服务/自启/计划任务整理成一张可读表（只读），后续再谈停用。
4. **大文件排行**：独立入口，按体积列前 20，附"疑似类型"。
5. 网络诊断：DNS 解析、代理设置、常用端口连通（只读）。

## 五、一句话总结

> 成熟清理软件的体面，不体现在"能删多少"，而体现在 **能不能撤**、**会不会把要删的东西先摊开给你看**。
>
> 本机这一条已经做到（清单 + 撤销 + 探查）；下一层比拼的是**覆盖面**：残留、启动项、体检、网络。

---

## 六、电脑环境（2026-10-27 补学）

### 看了什么

| 项目 | 是什么 | 学到什么 |
|---|---|---|
| [mise](https://mise.en.dev/dev-tools/) | 运行时/工具版本管理器（asdf 的后继） | 用 **shim** 按目录解析运行时版本；支持 `.tool-versions`、`.node-version` 这类"目录级约定" |
| [direnv](https://pkg.go.dev/github.com/direnv/direnv/v2@v2.36.0) | 按目录自动加载/卸载环境变量 | 项目级环境变量不该全塞进用户 PATH / profile；**进目录加载、出目录卸载** |
| winget / scoop / choco（从 hermes-agent 的提交里看到的实践） | 包管理器 | 装包会往 PATH 写项；**卸载后留下死路径**——正是我们刚清掉的那类残留 |
| hermes-agent 的 PATH 合并补丁 | 实际工程代码 | PATH 合并原则：**取 User + Machine 两个来源，大小写不敏感去重，保留原顺序**；再额外补 winget 的 Links 目录 |

### 提炼成管家的规矩

1. **PATH 是公共资源**：只加必要目录；卸载软件后要复查死路径与重复项。
2. **去重按小写比较**（Windows 不区分大小写），**保留首次出现的顺序**——顺序影响命令命中哪个版本。
3. **用户级与系统级分清**：用户级可撤（我们已能清），系统级要管理员（只报告）。
4. **环境变量优先用「按目录加载」**（direnv/mise 的思路），别把所有东西塞进全局 PATH。
5. **版本管理交给工具**（mise/asdf），别手工改 PATH 指到某个版本目录。
6. **PATH 有长度上限**，条目越多越容易踩坑；定期做一次"体检 + 瘦身"。

### 落进插件的改动（v0.9.1）

- 残留探查新增 **PATH 重复项**（用户级 / 系统级分开报）
- 残留处置在清理死路径的同时**按大小写不敏感去重**，只改用户级，仍写清单可撤销

### 本机现状（2026-10-27）

```
用户 PATH 长度 288 字符 · 系统 PATH 265 字符
用户 PATH 失效项 0 · 重复项 0 · 与系统 PATH 重叠 0
```
干净，暂无需处理。

---

## 七、系统运维与日志分析（2026-10-27 补学）

### 看了什么

| 项目 | 是什么 | 学到什么 |
|---|---|---|
| [TheLogRipper](https://github.com/tobiasGuta/TheLogRipper) | PowerShell 交互式 Windows 事件日志分析 | 读 `.evtx` 先**按来源+事件ID 归类计数**，再抽样例，最后才判断 |
| [Windows-Autorun-Process-Manager](https://github.com/elektronoide/Windows-Autorun-Process-Manager) | 启动项管理器 | 启动链路要全谱：Run / RunOnce / 服务 / 计划任务 / Winlogon / Explorer 扩展 |
| Sysinternals **Autoruns** | 微软官方启动项全量清单 | 与上面同一思路，作为交叉验证工具 |
| 服务管理实践（社区总结） | 哪些服务可停、哪些绝不可停 | 可停：DiagTrack、MapsBroker、Xbox 系列；**绝不可停**：wuauserv、WinDefend、EventLog、RpcSs、LSM、Schedule、Winmgmt、CryptSvc、BITS |
| `reg export` 备份做法 | 改服务/注册表前先导出 | **动注册表前先 `reg export` 该键**；批量备份可用 PowerShell 循环 |

### 提炼成管家的规矩

1. **看日志的顺序**：先归类计数 → 再抽样例 → 再判断；不要被单条吓到。
2. **区分"噪音"与"故障"**：DCOM 10010、Schannel 36876、Security-SPP 8198 这类多半是噪音，先查有没有伴随功能异常。
3. **动系统前三种备份**：`reg export` 具体键、系统还原点、清单留底（我们已有清单机制）。
4. **服务"绝不可停"名单**要刻在脑子里；停错了轻则更新失效，重则开不了机。
5. **启动项管理先只读**：列全谱 → 标出可疑 → 再谈禁用。

### 本机 7 天错误实际分析（2026-10-27）

**系统日志 106 条**，归类后只有三类：

| 次数 | 来源 | 判定 |
|---|---|---|
| 64 | `DistributedCOM` 10010「服务器没有在超时时间内向 DCOM 注册」 | **噪音**，常见于桌面组件，不影响使用 |
| 41 | `Schannel` 36876「证书验证失败，TLS 连接请求失败」 | 来自 **Watt Toolkit（BD.WTTS）反向代理**——它转发流量时遇到无效证书，属其工作特性 |
| 1 | `Service Control Manager` 7034「`IQIYI Video Platform Service` 意外停止」 | 单次，可忽略 |

**应用日志 88 条**：

| 次数 | 来源 | 判定 |
|---|---|---|
| 51 | `Security-SPP` 8198「许可证激活 slui.exe 失败」 | 激活检查噪音；系统已激活则无害 |
| 19 | `.NET Runtime`（`BD.WTTS.Services...RequestLoggingMiddleware`） | 同上，Watt Toolkit 代理 |
| 13 | `Application Error`：**Nexus.exe** 崩在 KERNELBASE/ntdll | 一部分是管家今天**强杀进程**造成；若自行崩溃频繁，考虑退出或更新该桌面美化程序 |

**结论**：没有磁盘、驱动、内存类错误——系统整体健康。真正值得留意的只有 `Nexus.exe` 本身是否稳定。

### 下次要做的

- 把"日志归类"做成插件的只读能力（`tidy_scan` 加 `events:true`，输出 Top 错误类别 + 判定建议）
- 系统级处置（服务/任务/注册表）加"改前自动 `reg export` 备份"
