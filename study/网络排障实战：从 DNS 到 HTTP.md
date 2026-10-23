---
title: 网络排障实战：从 DNS 到 HTTP
date: 2026-10-23
tags:
  - 网络
  - 排障
  - Multipass
  - Clash
  - TCP
---

# 网络排障实战：从 DNS 到 HTTP

> 今天最重要的收获：**不要一上来就盯着“代理有没有开”，而要按 DNS → 路由 → TCP → TLS → HTTP 分层定位。每一层只回答一个问题，并用实际输出区分事实、假设和结论。**

## 一、今天发生了什么

今天围绕 macOS、Clash Verge、Multipass/QEMU 和 Ubuntu 虚拟机做了一次完整的网络排障。

故障中的核心现象是：

- 虚拟机里的请求出现 `no route` 一类错误；
- 抓包时没有看到预期的 `Flags [S]` 或 `Flags [S.]`；
- 抓到的却是 `192.168.252.1` 与旧虚拟机 `192.168.252.2:22` 之间的 `Flags [P.]`、`Flags [.]`；
- 多次重启仍没有快速定位，最后删除旧实例并使用本地缓存镜像创建了新实例 `net-lab`。

事后结合 Multipass 日志和当前运行状态，可以把过程还原为：

1. 旧实例 `ubuntu` 在 10:53 从暂停快照恢复时，QEMU 报过 `Error -1 while loading VM state`。
2. 10:55 改为正常启动，Multipass 等到了 SSH。
3. 15:21 重启；15:22 关机；15:23 再次启动并等到 SSH。
4. 15:43 旧实例被删除并清理（purge）。
5. 15:48 使用本地缓存的 Ubuntu 24.04 ARM64 镜像创建 `net-lab`，配置为 2 CPU、2 GiB 内存、10 GiB 磁盘。
6. 15:48:46 Multipass 建立了新实例的 SSH 控制连接。
7. 当前 `net-lab` 地址为 `192.168.252.3/24`，默认路由和 DNS 都指向 `192.168.252.1`，基本网络路径已经恢复。

这里要注意：**新实例恢复正常，不等于已经证明旧实例的唯一根因。**旧实例确实出现过快照恢复错误，也确实出现过 `no route`，但 Clash TUN、guest 路由、抓包接口和抓包时机都可能影响观察结果。

## 二、先建立分层排障思路

访问 `https://example.com/` 时，实际经历的链路可以简化为：

```text
域名
  ↓ DNS：域名解析成哪个 IP？
目标 IP
  ↓ Route：这个 IP 应该从哪张网卡、经哪个网关出去？
TCP 连接
  ↓ TLS：证书、SNI、协议协商能否完成？
加密连接
  ↓ HTTP：服务返回什么状态码和内容？
应用结果
```

排障时必须从上往下检查。上层依赖下层：路由都不存在时，继续分析 TLS 或 HTTP 没有意义。

### 1. DNS：名字解析成了什么

在 Ubuntu guest 中：

```bash
getent ahosts example.com
resolvectl status
```

- `getent ahosts` 使用系统实际的名称解析路径，适合确认应用大概率会拿到什么地址。
- `resolvectl status` 查看当前 DNS 服务器、对应网卡和默认 DNS 路由。
- 当前 `net-lab` 的 DNS 是 `192.168.252.1`，也就是 Multipass 的共享网络网关。

在 macOS host 中：

```bash
scutil --dns
dig +short example.com
```

- `scutil --dns` 展示 macOS 的系统 DNS 配置。
- `dig` 展示 DNS 查询结果；如果结果落在 `198.18.0.0/16`，要想到 Clash Fake-IP，而不是立刻把它当作网站真实 IP。

### 2. 路由：这个 IP 往哪里走

在 Ubuntu guest 中：

```bash
ip -brief address
ip route
ip route get 93.184.216.34
```

- `ip -brief address`：网卡是否为 `UP`、是否获得地址。
- `ip route`：是否存在默认路由。
- `ip route get <目标 IP>`：让内核直接回答“访问该 IP 将走哪张网卡、使用哪个源地址和下一跳”。

当前新实例的关键输出是：

```text
enp0s1  UP  192.168.252.3/24
default via 192.168.252.1 dev enp0s1
```

这表示 guest 已获得地址，并且有默认网关。如果这里报 `Network is unreachable` 或查不到路径，问题还在路由层，数据包通常不会进入 TCP 握手。

在 macOS host 中：

```bash
netstat -rn -f inet
route -n get 93.184.216.34
```

- `netstat -rn` 查看完整 IPv4 路由表。
- `route -n get` 查看某个目标最终匹配的路由。

今天的 host 路由表中存在 `utun4`，并有大量目标网段指向 `198.18.0.1`，这说明 Clash 的 TUN 数据面存在；不能只根据“系统代理已关闭”判断流量没有经过 Clash。

### 3. TCP：能不能建立连接

```bash
nc -vz -w 5 example.com 443
```

它只测试 TCP 端口，不发送完整 HTTP 请求：

- 成功：至少说明 DNS、路由和 TCP 三次握手基本可用。
- `Connection refused`：包到达了对端，但端口没有服务或被主动拒绝。
- 超时：可能是包被丢弃、防火墙过滤、回程路由异常，也可能抓错了目标。
- `No route to host` / `Network is unreachable`：优先返回路由层检查。

### 4. TLS：加密握手能不能完成

```bash
openssl s_client \
  -connect example.com:443 \
  -servername example.com \
  -brief </dev/null
```

- `-servername` 发送 SNI；同一 IP 承载多个 HTTPS 网站时非常重要。
- 若 TCP 成功而 TLS 失败，重点检查证书、系统时间、SNI、TLS 版本、代理中间人和证书信任链。

### 5. HTTP：应用协议返回了什么

```bash
curl -v --connect-timeout 5 https://example.com/
```

`curl -v` 会依次显示解析、连接、TLS 和 HTTP 过程。重点看：

- `Trying ...`：准备连接的 IP；
- `Connected to ...`：TCP 已建立；
- TLS 版本和证书信息：TLS 是否完成；
- `< HTTP/... 200/301/403/500`：已经进入 HTTP 层。

一旦收到了 HTTP 状态码，DNS、路由、TCP、TLS 通常都已走通，后续应分析 URL、请求头、鉴权、反向代理和服务端逻辑。

## 三、怎么看懂今天的 TCP 抓包

常见标志：

| tcpdump 标志 | 含义 |
|---|---|
| `Flags [S]` | SYN，发起 TCP 连接 |
| `Flags [S.]` | SYN + ACK，对方同意建立连接 |
| `Flags [.]` | ACK，确认收到数据 |
| `Flags [P.]` | PSH + ACK，连接中正在传输应用数据 |
| `Flags [F.]` | FIN + ACK，正常关闭连接 |
| `Flags [R]` / `[R.]` | RST，连接被立即重置 |

今天抓到的是：

```text
192.168.252.1:51183 > 192.168.252.2:22  Flags [P.]
192.168.252.2:22 > 192.168.252.1:51183  Flags [P.]
双方随后以 Flags [.] 确认
```

这能证明：

- 当时 `192.168.252.1` 与旧 guest `192.168.252.2:22` 之间已有一条建立完成的 TCP 连接；
- 连接正在双向传输数据；
- 端口 22 表明它很可能是 Multipass 用来控制虚拟机的 SSH 流量。

它不能证明：

- 目标网站请求已经从 guest 发出；
- Clash 一定导致了故障；
- TCP 三次握手从未发生。

没有看到 `[S]` / `[S.]` 可能有三种原因：

1. 抓包开始时 SSH 连接早已建立，只能看到后续数据。
2. 抓错接口、主机、端口或流量方向，看到的是 Multipass 控制流量，不是测试请求。
3. guest 在路由层已经报错，目标 TCP SYN 根本没有生成，自然抓不到。

因此正确做法不是漫无目的地抓全量包，而是先固定五个条件：**测试时间、发起位置、目标域名/IP、目标端口、抓包接口。**

例如在 guest 内只看目标 TCP：

```bash
sudo tcpdump -nn -i enp0s1 'host 93.184.216.34 and tcp port 443'
```

在 host 上观察 Multipass guest：

```bash
sudo tcpdump -nn -i bridge102 'host 192.168.252.3 and tcp port 443'
```

随后另开一个终端，只执行一次测试请求并记录时间：

```bash
date
curl -v --connect-timeout 5 https://example.com/
```

安全提醒：不要随意加 `-A`、`-X` 或抓全网卡全流量；它们可能把 Cookie、Authorization 和其他敏感数据打印到终端或文件。

## 四、macOS 系统代理、环境变量代理与 Clash TUN/Fake-IP

这几种“代理”不在同一层。

### 1. macOS 系统代理

查看命令：

```bash
scutil --proxy
```

它是 macOS 提供给“愿意读取系统代理配置”的应用使用的。浏览器和部分 GUI 应用通常会读取，但命令行工具不一定读取。

今天检查时，HTTP、HTTPS、SOCKS 和 PAC 系统代理均为关闭状态。

### 2. 环境变量代理

查看当前进程环境：

```bash
env | grep -iE '^(http|https|all|no)_proxy='
```

常见变量有：

```text
HTTP_PROXY / http_proxy
HTTPS_PROXY / https_proxy
ALL_PROXY / all_proxy
NO_PROXY / no_proxy
```

它们只影响当前进程及其子进程，并不会自动写入 macOS 系统代理。`curl`、包管理器和各语言 HTTP 库是否支持、优先级如何，也要看各自实现。

临时绕过 `curl` 能识别的显式代理：

```bash
curl -v --noproxy '*' https://example.com/
```

但这只能绕过 `curl` 的代理选择，**不能绕过 TUN**，因为 TUN 在更低的 IP 层接管流量。

### 3. Clash TUN

TUN 会创建虚拟网卡并修改路由，让不懂代理协议的应用也能把 IP 流量交给 Clash。

本机当前证据：

- Clash Verge 设置中 `enable_tun_mode: true`；
- 系统代理设置为关闭；
- 生成配置使用 gVisor TUN；
- 路由表中存在经 `utun4`、下一跳 `198.18.0.1` 的路由。

所以：**“系统代理关闭”与“TUN 正在工作”可以同时成立。**

### 4. Fake-IP

Fake-IP 模式下，Clash 的 DNS 会先给域名返回一个合成地址，再在收到去往该地址的连接时还原域名并匹配规则。

本机配置使用：

```text
enhanced-mode: fake-ip
fake-ip-range: 198.18.0.1/16
```

看到 `198.18.x.x` 时，不要把它当成网站真实公网 IP。Fake-IP 要求 DNS 响应和后续流量都进入同一套 Clash 处理链；如果 DNS 被接管但路由没有跟上，就可能出现“解析有结果，但连接失败”的错觉。

### 5. Multipass guest 为什么不同

Multipass/QEMU 使用 `vmnet-shared`：

```text
Ubuntu guest 192.168.252.3
        ↓ enp0s1
Multipass 网关 192.168.252.1 / bridge102
        ↓ macOS 转发/NAT
物理网卡或 Clash TUN
        ↓
目标网络
```

guest 不会自动继承：

- macOS 系统代理设置；
- host shell 的 `HTTP_PROXY` 等环境变量；
- host 应用里的代理选择。

但 guest 流量在 host 转发/NAT 之后，仍可能被 host 的 TUN 路由接管。是否接管取决于实时路由、TUN 配置、排除网段和实现细节，不能只凭 UI 开关猜测。

## 五、证据、假设与尚未证明的结论

| 类型 | 内容 |
|---|---|
| 已证实 | 旧实例抓包中存在 `192.168.252.1 ↔ 192.168.252.2:22` 的 SSH 数据流 |
| 已证实 | 抓到的 `P.` / `.` 是已建立连接的数据与确认，不是三次握手 |
| 已证实 | 用户观察到目标操作报 `no route`，且未看到目标 SYN/SYN-ACK |
| 已证实 | 旧实例 QEMU 从暂停状态恢复时出现过 VM state 加载错误 |
| 已证实 | 旧实例经过重启、关机、再启动，最终被 purge |
| 已证实 | 新实例从本地缓存 Ubuntu 镜像创建，当前默认路由与 DNS 正常 |
| 已证实 | macOS 系统代理关闭，但 Clash TUN/Fake-IP 配置和 `utun4` 路由存在 |
| 合理假设 | 抓包时看到的是 Multipass 控制 SSH，而不是目标网站请求 |
| 合理假设 | 目标请求可能在 guest 路由层失败，因此根本没有生成 SYN |
| 合理假设 | 没看到 SSH 的 SYN 也可能只是因为抓包开始得太晚 |
| 未证明 | Clash TUN 是旧实例故障的唯一根因 |
| 未证明 | 删除实例是唯一修复方法；它只是这次最快恢复了可用环境 |

这张表体现了网络排障最重要的习惯：**“我看到了什么”与“我认为为什么”必须分开写。**

## 六、一套以后可以直接照着走的 SOP

在 guest 中按顺序执行：

```bash
# 0. 明确执行位置和时间
hostname
date

# 1. 网卡与地址
ip -brief address

# 2. DNS 配置与解析结果
resolvectl status
getent ahosts example.com

# 3. 路由表与精确选路
ip route
ip route get 93.184.216.34

# 4. TCP 端口
nc -vz -w 5 example.com 443

# 5. TLS
openssl s_client -connect example.com:443 -servername example.com -brief </dev/null

# 6. HTTP
curl -v --connect-timeout 5 https://example.com/
```

与此同时在 host 检查：

```bash
# macOS 系统代理
scutil --proxy

# 当前 shell 环境变量代理
env | grep -iE '^(http|https|all|no)_proxy='

# 系统 DNS
scutil --dns

# 路由表及指定目标选路
netstat -rn -f inet
route -n get 93.184.216.34

# Multipass 状态
multipass list
multipass info net-lab
```

判断原则：

1. DNS 错，就先修 DNS，不分析 TLS。
2. 没有路由，就先修地址、网关和路由，不盲目抓应用层包。
3. TCP 失败，再用精确过滤的 tcpdump 判断 SYN 发到哪里、有没有回包。
4. TCP 成功而 TLS 失败，检查 SNI、证书、时间和代理。
5. 已有 HTTP 状态码，就转向请求与服务端逻辑。
6. 每次只改变一个变量，并记录前后输出。

## 七、Multipass 常用安全命令

查看状态：

```bash
multipass version
multipass get local.driver
multipass list
multipass info net-lab
```

进入实例或执行只读诊断：

```bash
multipass shell net-lab
multipass exec net-lab -- ip route
```

普通重启：

```bash
multipass restart net-lab
```

新建实验实例：

```bash
multipass launch 24.04 \
  --name net-lab \
  --cpus 2 \
  --memory 2G \
  --disk 10G
```

本次日志证明 Multipass 命中了本地缓存镜像，因此没有必要手动寻找 qcow2 路径再直接启动 QEMU。让 Multipass 管理磁盘、cloud-init、网卡和 SSH 密钥更安全。

删除前先停机并确认实例名：

```bash
multipass stop net-lab
multipass list
```

`multipass delete` 和 `multipass purge` 会删除实例数据，属于破坏性操作；除非明确不需要实例内文件，否则不要把它们当作第一排障手段。

## 八、下一步练习

- [ ] 在 `net-lab` 内分别记录 DNS、路由、TCP、TLS、HTTP 五层的成功输出。
- [ ] 选择一个固定测试域名，同时在 guest 的 `enp0s1` 和 host 的 `bridge102` 抓同一次请求，对照包经过了哪些接口。
- [ ] 分别在 Clash TUN 开/关时查看 `netstat -rn -f inet`，只比较路由差异，不先猜结论。
- [ ] 用 `curl --noproxy '*'` 验证“绕过应用代理但不能绕过 TUN”的区别。
- [ ] 下次发生 `no route` 时，先保存 `ip -brief address`、`ip route`、`ip route get <IP>`，再考虑重启或删除实例。
- [ ] 若问题能稳定复现，保留失败实例，创建第二个干净实例做 A/B 对照；不要先销毁唯一证据。

## 最后的方法论

网络问题难，不是因为命令少，而是因为路径上有很多层。最有效的做法是：

> **固定测试条件，沿 DNS → 路由 → TCP → TLS → HTTP 逐层推进；抓包只验证明确假设；运行态证据优先于配置文件和 UI 开关；恢复可用与找到根因是两件事。**
