# 🛠️ OpenClaw在wsl环境下的搭建与网络配置教程
记录在 Windows + WSL2 (镜像网络) 环境下，解决 OpenClaw (AI Agent) 本地代理穿透与 SSRF (Server-Side Request Forgery) 安全拦截的完整 Debug 过程。  

## 📌 1. 环境背景与问题现象  
在搭建基于 Windows + WSL2 的自动化 AI 实验室时，为了让 OpenClaw 能够“自主浏览”外部学术网络并连接 WhatsApp 节点，配置了本地代理工具（端口 7891）。但在实际运行中遇到了两个致命的网络阻碍：  
* **报错 A (通信层)**:WhatsApp 节点持续输出 `WebSocket Error (Opening handshake has timed out)`，无法建立连接。
* **报错 B (工具层)**:Agent 在调用 Web 抓取工具时，触发 `Blocked: resolves to private/internal/special-use IP address`，网页读取功能瘫痪。
 
## 🔍 2. 问题排查过程  
**1.基础网络排查**:在 WSL 终端内直接使用 `curl` 测试，证实基础网络畅通，排除系统级断网可能。  
   
**2.代理端口排查**:发现 Windows 代理软件监听的是 `7891` 端口，但通过 `Windows .bat` 脚本唤醒 WSL 时，拉起的是一个“纯净”的 Linux 会话，未能正确继承代理环境变量，导致流量“迷路”超时。  
  
**3.安全机制排查**:当尝试在终端手动指定代理 `127.0.0.1:7891` 后，触发了 `OpenClaw` 深层的` SSRF` 防御机制。系统将本地代理 IP 判定为“高危内网地址”并强行切断连接。  

## 🧠3.根本原因   
首先需要看整个项目的网络链路：首先明确几个网络概念
**1.网络访问逻辑**：计算机发送需要访问的域名给DNS,DNS查询ip地址并返回给电脑，电脑进行三次握手（TCP）   
* **a.SYN客户端发送**：客户端向服务端发送链接申请；
* **b.SYN-ACK服务器反馈**：服务器确认能收到客户端申请，服务器反馈申请，并询问是否能够链接；
* **c.ACK**：客户端确认能收到服务器反馈；
  
  之后，电脑向服务器ip发送HTTPget请求：我用什么协议、访问什么站点、我是什么系统的什么浏览器
  之后，服务器响应，按照TCP通道反馈信息，电脑收到二进制码，由浏览器解包渲染成画面
  
**2.VPN**：（whatsapp端会绕过加密打包缓解，直接走底层交互，被墙，无法发起请求）电脑先向dns访问，真正要去的ip是什么，得到真ip。电脑发送的数据包：请求+想要访问的真实ip地址；vpn将这个请求打包并加密，发送到海外的服务器（没有被墙）上，海外服务器解码，并访问，再将访问反馈的信息打包，返回，再由vpn解码。  

**3.TUN模式**：所有网络活动都会通向虚拟网卡，由虚拟网卡分配虚拟ip（一般都是内网，但是机场会给真实网址与虚拟ip做映射，方便解码访问），电脑先向DNS访问获取目标ip地址，得到虚拟ip。再将虚拟ip打包进访问数据包中，发起访问，机场将虚拟ip映射解码，用海外服务器访问真实ip，再加密返回，再本地解码。

**4.SSRF内网访问保护机制**：禁止访问内网，对http有效，对whatsapp的通信模块没有部署。  

**bug现象分析**：由于whatsapp通信模块不受阻，whatsapp顺利连接，但注意，此时电脑的联网活动全部被虚拟网卡截断，想要访问谷歌，DNS先挂掉，网卡给一个内网ip，电脑拿到内网ip作为网络访问目标，打包过程触发SSRF，无法发送给虚拟网卡，报错“访问被阻止”。  

## 💡 4. 针对性解决方案 (The Fix)  
无需修改 OpenClaw 源码或开启复杂的全局 TUN 模式，只需在唤醒 WSL 的启动脚本中，利用 bash -c 链式注入“安全特赦令”与“代理指向”。  
修改 `Windows` 桌面用于启动的 `.bat` 文件，将启动命令替换为以下单行代码：  
```bash
start "OpenClaw Gateway" wsl bash -c "export OPENCLAW_SECURITY_ALLOW_PRIVATE_IPS=true && export https_proxy='http://127.0.0.1:7891' && export http_proxy='http://127.0.0.1:7891' && openclaw gateway run --with-relay"
```
重新运行wsl环境  
**⚙️ 代码拆解与原理解析：**   
`bash -c "..."`：构建链式指令环境，确保后续所有环境变量都在同一个 Linux 会话周期内生效。  

`export OPENCLAW_SECURITY_ALLOW_PRIVATE_IPS=true`：核心解法 1。注入特权环境变量，从底层强行绕过 OpenClaw 的 SSRF 防御，允许请求通过本机的回环地址。  

`export https_proxy='http://127.0.0.1:7891'`：核心解法 2。精准对齐端口，为当前运行的 AI 网关挂载出海代理。  

`&& openclaw gateway run`：确保在“拿到安全特赦令”且“接通代理网线”两步均成功执行后，再正式启动 AI 引擎核心。  
