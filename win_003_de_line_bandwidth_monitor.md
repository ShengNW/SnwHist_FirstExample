# Win 德国专线被动带宽监测（不改配置版）

## 0. 结论先看

你现在可以在 **不主动测速**、只正常使用浏览器的前提下，统计德国专线的：

1. 峰值带宽（Mbps）
2. 秒级带宽时间线（CSV）
3. 平均带宽

当前规则下，`browser-chrome.173-249-49-54.sslip.io -> de_proxy(:32699)`，所以“德国专线带宽”基本等价于该站点带宽。

---

## 1. 已落地文件（绝对路径）

1. 监测脚本（Windows 侧）  
`D:\exe\Snw_vwRayN\v2rayN-windows-64\scripts\Measure-DeLineBandwidth.ps1`
2. 本文档（WSL 侧）  
`\\wsl.localhost\Ubuntu\home\administrator\SnwHist\FirstExample\win_003_de_line_bandwidth_monitor.md`

---

## 2. 一键复现流程（小白步骤）

### 2.1 打开管理员 PowerShell

必须“以管理员身份运行”，因为 `pktmon` 抓包需要管理员权限。

### 2.2 进入脚本目录

```powershell
cd D:\exe\Snw_vwRayN\v2rayN-windows-64\scripts
```

### 2.3 开始采集（先开采集再正常使用）

```powershell
powershell -ExecutionPolicy Bypass -File .\Measure-DeLineBandwidth.ps1 -Action start
```

然后正常用 Brave/Chrome 访问你的目标站，持续你想观察的时间（例如 10~30 分钟）。

### 2.4 停止采集并自动出报告

```powershell
powershell -ExecutionPolicy Bypass -File .\Measure-DeLineBandwidth.ps1 -Action stop
```

脚本会自动完成：

1. 停止 `pktmon`
2. ETL 转 PCAPNG
3. 用 `tshark` 统计 `:32699` 秒级流量
4. 输出峰值/平均值/CSV

---

## 3. 输出结果在哪看

默认输出目录：`C:\Users\<你的用户名>\de-line-monitor`

核心文件：

1. `report.txt`：峰值 Mbps、平均 Mbps、总流量
2. `bytes_per_second.csv`：每秒带宽时间线（可用 Excel 作图）
3. `capture.pcapng`：原始抓包（可复查）
4. `session.json`：本次目标主机/端口上下文

---

## 4. 原理（为什么这样做）

1. 采集层：`pktmon` 被动抓包，不发测试流量，不改变路由策略。
2. 识别层：按德国专线端口 `32699` 过滤数据包。
3. 统计层：`tshark` 导出每包时间戳和包长，按“每秒”聚合，得到 Mbps。
4. 结论层：秒级最大值就是你的峰值带宽。

---

## 5. 注意事项

1. 需要安装 Wireshark/tshark（脚本统计阶段依赖 `tshark`）。
2. 统计结果是“该端口上的真实业务流量峰值”，不是跑测速站的理论极限。
3. 如果未来你把别的网站也分流到 `de_proxy`，那结果会变成“德国专线总带宽”，不再只代表当前站点。

---

## 6. 常见报错

1. `This action requires an elevated PowerShell window`  
重新用管理员 PowerShell 执行。
2. `Command not found: tshark`  
安装 Wireshark 并勾选 `tshark`，或把 `tshark.exe` 加入 PATH。
3. `No packets matched filter`  
说明采集期间没有命中 `:32699` 流量，检查是否确实访问了目标站、或端口是否改变。
