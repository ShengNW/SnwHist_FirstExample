# AutoDL WebDAV 挂载与替代方案全程记录（从首次尝试到最终可落地方案）

> 记录目标：完整复盘本次从“按 `webdav.md` 直接挂载”到“验证可行替代方案”的全部调研、命令执行、失败原因、权限边界与最终结论。  
> 适用场景：AutoDL/SeetaCloud 这类容器化实例，出现 `fuse`/`/dev/fuse` 权限异常时。

---

## 1. 初始任务与输入信息

用户需求：

- 参考文档：`/home/administrator/SnwHist/FirstExample/webdav.md`
- 目标机：`ssh -p 26840 root@connect.westb.seetacloud.com`
- 目标：在该机器挂载 WebDAV
- WebDAV 认证：已提供（本记录中不明文重复）

关键预期：

- 在远端机器把 WebDAV 挂到本地目录（例如 `/mnt/webdav`）
- 能稳定读写（不是仅“挂上”）

---

## 2. 前置文档核对（本地）

先读取 `webdav.md`，确认文档主流程是：

1. 安装 `davfs2`
2. 在 `/etc/davfs2/secrets` 写入凭据
3. 使用 `mount -t davfs ... /mnt/webdav`
4. 验证挂载参数与读写

文档中强调“权限映射”和“编辑器锁文件”，但当前目标机是容器环境，后续实际阻塞点发生在更底层的 FUSE 权限。

---

## 3. 第一轮：按 davfs2 标准流程实操

### 3.1 登录并安装

在远端执行：

- `apt-get update`
- `apt-get install -y davfs2`
- `mkdir -p /mnt/webdav`
- 配置 `/etc/davfs2/secrets`

安装结果：`davfs2` 安装成功（后续也通过 `dpkg -s davfs2` 验证为 `install ok installed`）。

### 3.2 执行挂载

执行：

```bash
mount -t davfs https://webdav.yeying.pub/dav /mnt/webdav
```

首次关键报错：

```text
/sbin/mount.davfs: loading kernel module fuse
/sbin/mount.davfs: loading kernel module fuse failed
/sbin/mount.davfs: waiting for /dev/fuse to be created
/sbin/mount.davfs: can't open fuse device
```

结论：不是账号密码错误，而是 FUSE 设备/权限链路异常。

---

## 4. 第二轮：定位 FUSE 层权限问题（排除配置误差）

为确认是否是“缺文件”还是“权限被拦”，做了以下核查。

### 4.1 系统与设备状态

- `uname -a` 显示 Ubuntu 容器环境（AutoDL banner 明确容器实例）
- `/dev/fuse` 初始不存在
- `/proc/filesystems` 中存在 `fuse`、`fuseblk`、`fusectl`

说明：内核支持 FUSE 类型，但容器内设备节点与权限未就绪。

### 4.2 尝试手动创建设备节点

执行：

```bash
mknod -m 666 /dev/fuse c 10 229
```

随后再挂载，依旧报：

```text
/sbin/mount.davfs: can't open fuse device
```

说明：不是“节点文件不存在”的简单问题，而是“容器对打开该设备的能力被限制”。

### 4.3 补充核验

- `/proc/misc` 可看到 `229 fuse`
- 但容器内即使 root 也无法完成 mount 所需 FUSE 设备打开

这非常符合容器 runtime/cgroup/安全策略限制的表现：

- 设备节点可见（甚至可手建）
- 但 `open(/dev/fuse)` 仍被拒绝

---

## 5. 第三轮：按用户建议改走 rclone mount（绕过 davfs2）

用户提出“用 rclone 绕过”。随后进入实测。

### 5.1 rclone 安装与远端配置

执行：

- `curl -fsSL https://rclone.org/install.sh | bash`
- `rclone config create webdav webdav ...`
- `rclone listremotes`
- `rclone lsd webdav:/`

结果：

- `rclone v1.73.2` 安装成功
- 远端 `webdav:` 创建成功
- `rclone lsd webdav:/` 能列出目录（如 `apps`、`personal`）

关键判断：

- 网络通
- 认证通
- WebDAV 服务可访问

### 5.2 首次 rclone mount 失败（工具依赖缺失）

日志报错：

```text
Fatal error: failed to mount FUSE fs: fusermount: exec: "fusermount3": executable file not found in $PATH
```

处理：安装 `fuse3`（得到 `/usr/bin/fusermount3`）。

### 5.3 二次 rclone mount 失败（核心权限限制）

重新挂载后日志明确报：

```text
mount helper error: fusermount3: failed to open /dev/fuse: Operation not permitted
Fatal error: failed to mount FUSE fs
```

这是本次最关键证据。它与 davfs2 的失败根因一致：

- 不是 WebDAV 配置问题
- 不是 rclone 版本问题
- 是容器不允许使用 FUSE 挂载能力

---

## 6. 第四轮：只测“可行且不依赖 FUSE”的方案

根据用户进一步要求，不再重复明显不可行路径，转测非 FUSE 方案。

### 6.1 `rclone copy/sync` 全链路实测（成功）

测试 ID：`codex_autotest_20260315_052414`

执行动作：

1. 本地创建测试文件
2. `rclone copy` 上传到远端测试目录
3. 再 `rclone copy` 下载回本地
4. `cmp` 比较文件一致性
5. `rclone sync` 本地 -> 远端（`a.txt`、`b.txt`）
6. `rclone sync` 远端 -> 本地

结果：

- 上传成功
- 下载成功
- `cmp=OK`
- `sync` 双方向都成功（作为两次单向镜像）

结论：在当前 AutoDL 实例里，`copy/sync` 是稳定可用路径。

### 6.2 `rclone serve http`（成功）

先发现：当前 `rclone serve` 子命令不支持 `--daemon` 参数。改为 `nohup ... &` 后台运行。

验证：

- 启动 `rclone serve http webdav:/... --addr 127.0.0.1:18080`
- `curl` 访问 `http://127.0.0.1:18080/hello.txt`

结果：

- 返回 `HTTP/1.1 200 OK`
- 文件内容正确

结论：HTTP 服务转发路径可用。

### 6.3 `rclone serve webdav`（成功）

同样用 `nohup ... &` 启动。

验证：

- `PROPFIND http://127.0.0.1:18081/` with `Depth: 1`

结果：

- HTTP 状态码 `207 Multi-Status`
- 返回 DAV XML 列表

结论：WebDAV 服务转发路径可用。

### 6.4 Python 客户端验证

#### 6.4.1 `webdavclient3`（部分不可行）

- 安装后调用 `client.check()` 报：

```text
MethodNotSupported: Method 'check' not supported for https://webdav.yeying.pub/dav
```

说明：该库默认行为与当前服务端方法支持不完全兼容。

#### 6.4.2 `requests` 直调 DAV 方法（可行）

执行 `MKCOL/PUT/GET/DELETE` 实测：

- `MKCOL 201`
- `PUT 201`
- `GET 200`
- `DELETE_FILE 200`
- `DELETE_DIR 200`

结论：应用层直连 WebDAV 方法可行。

---

## 7. AutoDL 挂载限制与权限边界（重点）

这一节是本次最关键结论。

### 7.1 为什么“容器里 root”仍然挂不上

在容器中看到 `root` 并不等于拿到了宿主内核的全部能力。

FUSE 挂载通常要求：

- 能访问 `/dev/fuse`
- 具备对应 capability（常见需要 `SYS_ADMIN` 或 runtime 放行）
- 未被 seccomp/AppArmor/cgroup devices 拦截

当前实例表现是：

- `/dev/fuse` 即便手建，`open` 仍 `Operation not permitted`
- 即 runtime 层没有授予容器可用的 FUSE 设备权限

### 7.2 已观测到的具体限制

- 默认无可用 FUSE 挂载能力
- `davfs2`/`rclone mount` 同时失败，报错落点都在 `/dev/fuse`
- 容器内无法自行提升为特权容器
- 容器内无 `docker`/`nerdctl`，无法自助重启容器添加 `--cap-add SYS_ADMIN --device /dev/fuse`

### 7.3 直接影响

不可行：

- 在当前实例中使用任何依赖 FUSE 的“本地挂载”方案（`davfs2`、`rclone mount`）

可行：

- 非挂载方案：`rclone copy/sync`
- 代理服务方案：`rclone serve http`、`rclone serve webdav`
- 应用层 API 方案：`requests` 直接 WebDAV 方法

---

## 8. 可行/不可行总表

| 路径 | 结果 | 证据 | 说明 |
|---|---|---|---|
| `mount -t davfs ...` | 不可行 | `can't open fuse device` | FUSE 设备打开被拒绝 |
| `rclone mount ...` | 不可行 | `failed to open /dev/fuse: Operation not permitted` | 与 davfs2 同根因 |
| `rclone copy` | 可行 | 上传+下载+`cmp=OK` | 推荐用于低风险搬运 |
| `rclone sync` | 可行 | 双向单独执行成功 | 注意删除语义（镜像） |
| `rclone serve http` | 可行 | `HTTP/1.1 200 OK` | 适合只读分发 |
| `rclone serve webdav` | 可行 | `PROPFIND -> 207` | 适合 DAV 客户端语义 |
| `webdavclient3` 默认 check 流程 | 不稳定 | `MethodNotSupported` | 与服务端方法兼容性问题 |
| `requests` 直调 DAV 方法 | 可行 | `MKCOL/PUT/GET/DELETE` 全成功 | 应用层方案可落地 |

---

## 9. 额外过程记录（文档落地阶段）

后续在本地已挂载的 `/mnt/webdav` 路径写文档时，遇到过一次异常：

- 对某个中文文件名写入出现 `Input/output error`/`Permission denied`
- 同目录新建英文文件名可正常写入

处理方式：

- 改用新文件名落盘并完成内容写入

这属于 WebDAV 挂载端偶发 I/O 行为，不影响前述“AutoDL 远端容器内 FUSE 权限被拒”的主结论。

---

## 10. 最终结论（面向决策）

1. 当前 AutoDL 实例内，FUSE 挂载链路不可用，`davfs2` 与 `rclone mount` 均不可行。  
2. 根因是容器权限/设备限制，不是 WebDAV 账号、URL 或客户端命令写法。  
3. 现阶段可稳定落地的路线是“非挂载”：
   - 批处理：`rclone copy/sync`
   - 在线服务：`rclone serve http` / `rclone serve webdav`
   - 应用层：`requests` 直连 DAV 方法
4. 若必须实现“像本地盘一样挂载”，只能由平台侧提供容器运行时权限（如 `--cap-add SYS_ADMIN` + `/dev/fuse` 放行）后再测。

---

## 11. 建议执行顺序（可直接照做）

- 第一阶段（立即可用）：
  - 建立 `pull/push` 的 `rclone sync` 脚本
  - 所有高风险同步先 `--dry-run`
- 第二阶段（按需服务）：
  - 内网本机需求用 `serve http`（简单）
  - 需要 DAV 客户端语义时用 `serve webdav`
- 第三阶段（平台协同）：
  - 若业务必须挂载，向平台申请 FUSE 能力再复测 `rclone mount`

> 结论一句话：当前 AutoDL 实例里，“挂载”不可行，“传输/代理/API”可行；应把工程重心从 FUSE 挂载转到非挂载架构。
