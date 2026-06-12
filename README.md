# 🔐 OpenVPN 服务器搭建教程 / OpenVPN Server Setup Guide

> 使用 Docker 和原生方式快速搭建 OpenVPN 服务器 | Quick setup of OpenVPN server using Docker and native methods

## 📖 目录 / Table of Contents

- [📖 目录 / Table of Contents](#-目录--table-of-contents)
- [一、概述 / Overview](#一概述--overview)
- [二、OpenVPN 工作原理 / How OpenVPN Works](#二openvpn-工作原理--how-openvpn-works)
- [三、部署方式对比 / Deployment Comparison](#三部署方式对比--deployment-comparison)
- [四、前置条件 / Prerequisites](#四前置条件--prerequisites)
- [五、方式一：Docker 部署（推荐） / Method 1: Docker Deployment (Recommended)](#五方式一docker-部署推荐--method-1-docker-deployment-recommended)
  - [5.1 安装 Docker / Install Docker](#51-安装-docker--install-docker)
  - [5.2 使用 kylemanna/openvpn 镜像 / Using kylemanna/openvpn Image](#52-使用-kylemannaopenvpn-镜像--using-kylemannaopenvpn-image)
  - [5.3 生成客户端配置 / Generate Client Configuration](#53-生成客户端配置--generate-client-configuration)
  - [5.4 客户端连接 / Client Connection](#54-客户端连接--client-connection)
- [六、方式二：原生安装（Ubuntu/Debian） / Method 2: Native Installation (Ubuntu/Debian)](#六方式二原生安装ubuntudebian--method-2-native-installation-ubuntudebian)
  - [6.1 安装 OpenVPN 与 EasyRSA / Install OpenVPN and EasyRSA](#61-安装-openvpn-与-easyrsa--install-openvpn-and-easyrsa)
  - [6.2 初始化 PKI / Initialize PKI](#62-初始化-pki--initialize-pki)
  - [6.3 生成服务器证书 / Generate Server Certificate](#63-生成服务器证书--generate-server-certificate)
  - [6.4 生成 Diffie-Hellman 参数 / Generate DH Parameters](#64-生成-diffie-hellman-参数--generate-dh-parameters)
  - [6.5 生成 TLS 加密密钥 / Generate TLS Crypt Key](#65-生成-tls-加密密钥--generate-tls-crypt-key)
  - [6.6 配置 OpenVPN 服务器 / Configure OpenVPN Server](#66-配置-openvpn-服务器--configure-openvpn-server)
  - [6.7 配置 IP 转发与防火墙 / Configure IP Forwarding and Firewall](#67-配置-ip-转发与防火墙--configure-ip-forwarding-and-firewall)
  - [6.8 启动 OpenVPN 服务 / Start OpenVPN Service](#68-启动-openvpn-服务--start-openvpn-service)
- [七、客户端配置生成 / Client Configuration Generation](#七客户端配置生成--client-configuration-generation)
  - [7.1 生成客户端证书 / Generate Client Certificate](#71-生成客户端证书--generate-client-certificate)
  - [7.2 创建 .ovpn 配置文件 / Create .ovpn Configuration File](#72-创建-ovpn-配置文件--create-ovpn-configuration-file)
- [八、常用命令 / Common Commands](#八常用命令--common-commands)
- [九、客户端使用指南 / Client Usage Guide](#九客户端使用指南--client-usage-guide)
  - [9.1 Windows / Windows](#91-windows--windows)
  - [9.2 macOS / macOS](#92-macos--macos)
  - [9.3 Linux / Linux](#93-linux--linux)
  - [9.4 iOS / Android / iOS / Android](#94-ios--android--ios--android)
- [十、高级配置 / Advanced Configuration](#十高级配置--advanced-configuration)
  - [10.1 用户密码认证（auth-user-pass）/ Username/Password Authentication](#101-用户密码认证auth-user-pass--usernamepassword-authentication)
  - [10.2 路由推送（push routing）/ Push Routing](#102-路由推送push-routing--push-routing)
  - [10.3 DNS 推送 / DNS Push](#103-dns-推送--dns-push)
  - [10.4 多客户端隔离 / Multi-Client Isolation](#104-多客户端隔离--multi-client-isolation)
  - [10.5 UDP 与 TCP 双协议 / Dual Protocol (UDP + TCP)](#105-udp-与-tcp-双协议--dual-protocol-udp--tcp)
- [十一、安全加固 / Security Hardening](#十一安全加固--security-hardening)
- [十二、常见问题 / FAQ](#十二常见问题--faq)
- [十三、参考资源 / References](#十三参考资源--references)
- [☕ 支持 / Support](#-支持--support)

---

## 一、概述 / Overview

### 中文

OpenVPN 是一个开源的虚拟专用网络（VPN）解决方案，基于 SSL/TLS 协议实现安全加密通信。它支持多种认证方式（证书、用户名密码、双因素认证）和灵活的传输协议（UDP/TCP）。本教程将详细介绍使用 Docker 和原生方式搭建 OpenVPN 服务器的完整步骤。

OpenVPN 广泛应用于以下场景：
- **远程办公**：员工从外部网络安全访问公司内网资源
- **隐私保护**：加密网络流量，防止 ISP 或中间人窃听
- **跨地域组网**：将多个地理位置的分支机构连接成虚拟局域网
- **绕过地理限制**：安全访问受限内容（需遵守法律法规）

### English

OpenVPN is an open-source Virtual Private Network (VPN) solution that implements secure encrypted communication based on the SSL/TLS protocol. It supports multiple authentication methods (certificates, username/password, two-factor authentication) and flexible transport protocols (UDP/TCP). This guide details the complete steps for setting up an OpenVPN server using both Docker and native installation methods.

OpenVPN is widely used for:
- **Remote Work**: Employees securely access corporate intranet resources from external networks
- **Privacy Protection**: Encrypt network traffic to prevent ISP or man-in-the-middle eavesdropping
- **Cross-Region Networking**: Connect branch offices across multiple geographic locations into a virtual LAN
- **Bypassing Geo-Restrictions**: Safely access restricted content (comply with local laws and regulations)

---

## 二、OpenVPN 工作原理 / How OpenVPN Works

### 中文

| 组件 | 说明 |
|------|------|
| **OpenVPN 服务器** | 运行 OpenVPN 服务端进程，监听 VPN 连接请求 |
| **OpenVPN 客户端** | 运行 OpenVPN 客户端进程，向服务器发起连接 |
| **CA 证书** | 证书颁发机构，用于签发服务器和客户端证书 |
| **服务器证书** | 服务器的身份凭证，客户端通过它验证服务器 |
| **客户端证书** | 客户端的身份凭证，服务器通过它验证客户端 |
| **TLS 加密密钥** | 用于 TLS 通道的额外加密层 |
| **TUN/TAP 接口** | TUN（Layer 3，IP层）或 TAP（Layer 2，以太网层）虚拟网络接口 |

### English

| Component | Description |
|-----------|-------------|
| **OpenVPN Server** | Runs the OpenVPN daemon, listens for VPN connection requests |
| **OpenVPN Client** | Runs the OpenVPN client process, initiates connection to the server |
| **CA Certificate** | Certificate Authority that issues server and client certificates |
| **Server Certificate** | Server identity credential, verified by clients |
| **Client Certificate** | Client identity credential, verified by the server |
| **TLS Crypt Key** | Additional encryption layer for the TLS channel |
| **TUN/TAP Interface** | TUN (Layer 3, IP) or TAP (Layer 2, Ethernet) virtual network interface |

### 中文

OpenVPN 的工作流程：

1. **握手阶段**：客户端向服务器发起 SSL/TLS 握手，建立加密通道
2. **证书验证**：服务器和客户端互相验证对方的证书（双向认证）
3. **密钥交换**：通过 TLS 通道协商会话密钥
4. **数据加密**：所有 VPN 流量通过 AES-256-GCM 等加密算法加密
5. **隧道建立**：创建 TUN/TAP 虚拟网卡，分配虚拟 IP 地址
6. **数据传输**：客户端通过加密隧道访问目标网络资源

TUN（路由模式）工作在 IP 层（Layer 3），是最常用的模式。TAP（桥接模式）工作在以太网层（Layer 2），适用于需要广播协议的场景。

### English

OpenVPN workflow:

1. **Handshake Phase**: Client initiates an SSL/TLS handshake with the server to establish an encrypted channel
2. **Certificate Verification**: Server and client mutually verify each other's certificates (mutual authentication)
3. **Key Exchange**: Session keys are negotiated through the TLS channel
4. **Data Encryption**: All VPN traffic is encrypted using AES-256-GCM and similar algorithms
5. **Tunnel Establishment**: A TUN/TAP virtual network interface is created with a virtual IP address assigned
6. **Data Transmission**: The client accesses target network resources through the encrypted tunnel

TUN (routing mode) operates at the IP layer (Layer 3) and is the most commonly used mode. TAP (bridging mode) operates at the Ethernet layer (Layer 2) and is suitable for scenarios requiring broadcast protocols.

---

## 三、部署方式对比 / Deployment Comparison

### 中文

| 特性 | Docker 部署 | 原生部署 |
|------|-----------|---------|
| 部署速度 | ⚡ 快速（几分钟） | 🐢 较慢（需编译/安装依赖） |
| 配置复杂度 | 简单 | 中等 |
| 可维护性 | 高（容器化管理） | 中（手动管理服务） |
| 资源占用 | 低（容器化） | 极低（原生运行） |
| 升级/回滚 | 一键操作 | 需手动处理 |
| 自定义程度 | 高 | 极高（完全控制） |
| 适合场景 | 快速部署、开发测试 | 生产环境、需要深度定制 |

### English

| Feature | Docker Deployment | Native Deployment |
|---------|------------------|-------------------|
| Speed | ⚡ Fast (minutes) | 🐢 Slower (dependencies) |
| Complexity | Simple | Moderate |
| Maintainability | High (container management) | Medium (manual service management) |
| Resource Usage | Low (containerized) | Very low (native) |
| Upgrade/Rollback | One-click | Manual handling |
| Customization | High | Very high (full control) |
| Best For | Quick deployment, dev/test | Production, deep customization |

---

## 四、前置条件 / Prerequisites

### 中文

- **一台 Linux 服务器**（推荐 Ubuntu 20.04+ 或 Debian 11+）
- **公网 IP**（或具备端口转发的内网环境）
- **开放端口**：UDP 1194（默认端口，可按需更改）
- **域名**（可选，用于证书和方便连接）
- **Docker**（仅 Docker 部署方式需要）
- **基本的 Linux 命令行知识**

### English

- **A Linux server** (Ubuntu 20.04+ or Debian 11+ recommended)
- **Public IP** (or an intranet environment with port forwarding)
- **Open port**: UDP 1194 (default, configurable)
- **Domain name** (optional, for certificates and convenient connections)
- **Docker** (only needed for Docker deployment)
- **Basic Linux command line knowledge**

---

## 五、方式一：Docker 部署（推荐） / Method 1: Docker Deployment (Recommended)

### 5.1 安装 Docker / Install Docker

#### 中文

```bash
# Ubuntu/Debian
apt update && apt install -y docker.io
systemctl enable --now docker

# CentOS/RHEL
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce docker-ce-cli containerd.io
systemctl enable --now docker

# 验证安装
docker --version
```

#### English

```bash
# Ubuntu/Debian
apt update && apt install -y docker.io
systemctl enable --now docker

# CentOS/RHEL
yum install -y yum-utils
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum install -y docker-ce docker-ce-cli containerd.io
systemctl enable --now docker

# Verify installation
docker --version
```

### 5.2 使用 kylemanna/openvpn 镜像 / Using kylemanna/openvpn Image

#### 中文

`kylemanna/openvpn` 是最流行的 OpenVPN Docker 镜像之一，它将 OpenVPN 和 EasyRSA 打包在一起，并提供便捷的管理脚本。

```bash
# 设置变量（替换为你的值）
OVPN_DATA="/opt/openvpn-data"
OVPN_PORT="1194"
OVPN_PROTO="udp"
OVPN_SERVER="vpn.example.com"  # 替换为你的服务器 IP 或域名

# 创建数据卷
mkdir -p "${OVPN_DATA}"

# 初始化配置（替换 OVPN_SERVER 为实际值）
docker run -v "${OVPN_DATA}:/etc/openvpn" --rm kylemanna/openvpn \
  ovpn_genconfig -u "${OVPN_PROTO}://${OVPN_SERVER}:${OVPN_PORT}"

# 初始化 PKI（证书基础设施）
docker run -v "${OVPN_DATA}:/etc/openvpn" --rm -it kylemanna/openvpn \
  ovpn_initpki nopass

# 启动 OpenVPN 服务器
docker run -d \
  --name openvpn \
  --restart always \
  --cap-add NET_ADMIN \
  -v "${OVPN_DATA}:/etc/openvpn" \
  -p "${OVPN_PORT}:${OVPN_PORT}/${OVPN_PROTO}" \
  kylemanna/openvpn

# 检查容器运行状态
docker ps | grep openvpn
```

#### English

`kylemanna/openvpn` is one of the most popular OpenVPN Docker images. It bundles OpenVPN with EasyRSA and provides convenient management scripts.

```bash
# Set variables (replace with your values)
OVPN_DATA="/opt/openvpn-data"
OVPN_PORT="1194"
OVPN_PROTO="udp"
OVPN_SERVER="vpn.example.com"  # Replace with your server IP or domain

# Create data volume
mkdir -p "${OVPN_DATA}"

# Initialize configuration (replace OVPN_SERVER with actual value)
docker run -v "${OVPN_DATA}:/etc/openvpn" --rm kylemanna/openvpn \
  ovpn_genconfig -u "${OVPN_PROTO}://${OVPN_SERVER}:${OVPN_PORT}"

# Initialize PKI (Certificate Infrastructure)
docker run -v "${OVPN_DATA}:/etc/openvpn" --rm -it kylemanna/openvpn \
  ovpn_initpki nopass

# Start OpenVPN Server
docker run -d \
  --name openvpn \
  --restart always \
  --cap-add NET_ADMIN \
  -v "${OVPN_DATA}:/etc/openvpn" \
  -p "${OVPN_PORT}:${OVPN_PORT}/${OVPN_PROTO}" \
  kylemanna/openvpn

# Check container status
docker ps | grep openvpn
```

### 5.3 生成客户端配置 / Generate Client Configuration

#### 中文

```bash
# 为客户端生成证书
CLIENT_NAME="myphone"  # 替换为客户端名称
docker run -v "${OVPN_DATA}:/etc/openvpn" --rm -it kylemanna/openvpn \
  easyrsa build-client-full "${CLIENT_NAME}" nopass

# 导出客户端配置文件
docker run -v "${OVPN_DATA}:/etc/openvpn" --rm kylemanna/openvpn \
  ovpn_getclient "${CLIENT_NAME}" > "/root/${CLIENT_NAME}.ovpn"

# 查看生成的配置文件
ls -la "/root/${CLIENT_NAME}.ovpn"
```

#### English

```bash
# Generate certificate for client
CLIENT_NAME="myphone"  # Replace with client name
docker run -v "${OVPN_DATA}:/etc/openvpn" --rm -it kylemanna/openvpn \
  easyrsa build-client-full "${CLIENT_NAME}" nopass

# Export client configuration file
docker run -v "${OVPN_DATA}:/etc/openvpn" --rm kylemanna/openvpn \
  ovpn_getclient "${CLIENT_NAME}" > "/root/${CLIENT_NAME}.ovpn"

# Verify generated configuration
ls -la "/root/${CLIENT_NAME}.ovpn"
```

### 5.4 客户端连接 / Client Connection

#### 中文

将生成的 `.ovpn` 文件安全地传输到客户端设备（通过 SCP、Airdrop、USB 等），然后使用 OpenVPN 客户端软件导入连接。

#### English

Securely transfer the generated `.ovpn` file to your client device (via SCP, Airdrop, USB, etc.), then import it into an OpenVPN client application to connect.

---

## 六、方式二：原生安装（Ubuntu/Debian） / Method 2: Native Installation (Ubuntu/Debian)

### 6.1 安装 OpenVPN 与 EasyRSA / Install OpenVPN and EasyRSA

#### 中文

```bash
# 更新系统并安装
apt update && apt upgrade -y
apt install -y openvpn easy-rsa

# 创建 EasyRSA 工作目录
make-cadir /etc/openvpn/easy-rsa
cd /etc/openvpn/easy-rsa
```

#### English

```bash
# Update system and install
apt update && apt upgrade -y
apt install -y openvpn easy-rsa

# Create EasyRSA working directory
make-cadir /etc/openvpn/easy-rsa
cd /etc/openvpn/easy-rsa
```

### 6.2 初始化 PKI / Initialize PKI

#### 中文

```bash
cd /etc/openvpn/easy-rsa

# 初始化 PKI 目录
./easyrsa init-pki

# 构建 CA 证书（设置一个 Common Name）
./easyrsa build-ca nopass

# 输出示例：
# ...
# Common Name (eg: your user, host, or server name) [Easy-RSA CA]:
# CA creation complete and you may now import and sign cert requests.
```

#### English

```bash
cd /etc/openvpn/easy-rsa

# Initialize PKI directory
./easyrsa init-pki

# Build CA certificate (set a Common Name)
./easyrsa build-ca nopass

# Sample output:
# ...
# Common Name (eg: your user, host, or server name) [Easy-RSA CA]:
# CA creation complete and you may now import and sign cert requests.
```

### 6.3 生成服务器证书 / Generate Server Certificate

#### 中文

```bash
cd /etc/openvpn/easy-rsa

# 生成服务器证书请求并签署（nopass 表示无密码保护）
./easyrsa gen-req server nopass
./easyrsa sign-req server server

# 生成 Diffie-Hellman 参数（可能需要较长时间）
./easyrsa gen-dh

# 复制证书和密钥到 OpenVPN 目录
cp pki/ca.crt /etc/openvpn/
cp pki/issued/server.crt /etc/openvpn/
cp pki/private/server.key /etc/openvpn/
cp pki/dh.pem /etc/openvpn/
```

#### English

```bash
cd /etc/openvpn/easy-rsa

# Generate server certificate request and sign it
./easyrsa gen-req server nopass
./easyrsa sign-req server server

# Generate Diffie-Hellman parameters (may take a while)
./easyrsa gen-dh

# Copy certificates and keys to OpenVPN directory
cp pki/ca.crt /etc/openvpn/
cp pki/issued/server.crt /etc/openvpn/
cp pki/private/server.key /etc/openvpn/
cp pki/dh.pem /etc/openvpn/
```

### 6.4 生成 Diffie-Hellman 参数 / Generate DH Parameters

#### 中文

```bash
# 如果上一步没有生成，可以单独生成
cd /etc/openvpn/easy-rsa
openssl dhparam -out /etc/openvpn/dh.pem 2048
```

#### English

```bash
# If not generated in the previous step, generate separately
cd /etc/openvpn/easy-rsa
openssl dhparam -out /etc/openvpn/dh.pem 2048
```

### 6.5 生成 TLS 加密密钥 / Generate TLS Crypt Key

#### 中文

TLS 加密密钥为控制通道提供额外的加密层，可防范未授权探测和 DDoS 攻击。

```bash
cd /etc/openvpn/easy-rsa
openvpn --genkey secret /etc/openvpn/tls-crypt.key
```

#### English

The TLS crypt key provides an additional encryption layer for the control channel, protecting against unauthorized probing and DDoS attacks.

```bash
cd /etc/openvpn/easy-rsa
openvpn --genkey secret /etc/openvpn/tls-crypt.key
```

### 6.6 配置 OpenVPN 服务器 / Configure OpenVPN Server

#### 中文

创建服务器配置文件 `/etc/openvpn/server.conf`：

```ini
# 监听端口和协议
port 1194
proto udp
dev tun

# 证书和密钥路径
ca /etc/openvpn/ca.crt
cert /etc/openvpn/server.crt
key /etc/openvpn/server.key
dh /etc/openvpn/dh.pem
tls-crypt /etc/openvpn/tls-crypt.key

# VPN 虚拟 IP 地址池
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist /etc/openvpn/ipp.txt

# 推送路由（如果需要访问服务器所在局域网，取消注释）
;push "route 192.168.1.0 255.255.255.0"

# 推送 DNS 服务器
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 1.1.1.1"

# 安全设置
cipher AES-256-GCM
auth SHA256
tls-version-min 1.2
tls-cipher TLS-ECDHE-RSA-WITH-AES-256-GCM-SHA384

# 客户端配置
client-config-dir /etc/openvpn/ccd
topology subnet

# 保持连接
keepalive 10 120

# 其他设置
persist-key
persist-tun
status /etc/openvpn/openvpn-status.log
log-append /var/log/openvpn.log
verb 3
explicit-exit-notify 1

# 最多并发客户端数
max-clients 100
```

#### English

Create server configuration file `/etc/openvpn/server.conf`:

```ini
# Port and protocol
port 1194
proto udp
dev tun

# Certificate and key paths
ca /etc/openvpn/ca.crt
cert /etc/openvpn/server.crt
key /etc/openvpn/server.key
dh /etc/openvpn/dh.pem
tls-crypt /etc/openvpn/tls-crypt.key

# VPN virtual IP address pool
server 10.8.0.0 255.255.255.0
ifconfig-pool-persist /etc/openvpn/ipp.txt

# Push routes (uncomment if accessing server LAN)
;push "route 192.168.1.0 255.255.255.0"

# Push DNS servers
push "dhcp-option DNS 8.8.8.8"
push "dhcp-option DNS 1.1.1.1"

# Security settings
cipher AES-256-GCM
auth SHA256
tls-version-min 1.2
tls-cipher TLS-ECDHE-RSA-WITH-AES-256-GCM-SHA384

# Client configuration
client-config-dir /etc/openvpn/ccd
topology subnet

# Keep alive
keepalive 10 120

# Other settings
persist-key
persist-tun
status /etc/openvpn/openvpn-status.log
log-append /var/log/openvpn.log
verb 3
explicit-exit-notify 1

# Maximum concurrent clients
max-clients 100
```

### 6.7 配置 IP 转发与防火墙 / Configure IP Forwarding and Firewall

#### 中文

```bash
# 启用 IP 转发
echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
sysctl -p

# 配置 iptables NAT（根据实际网络接口调整 eth0）
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE

# 保存 iptables 规则（Ubuntu 使用 netfilter-persistent）
apt install -y iptables-persistent
netfilter-persistent save

# 开放防火墙端口
ufw allow 1194/udp
# 如果使用 firewalld（CentOS/RHEL）
# firewall-cmd --permanent --add-port=1194/udp
# firewall-cmd --reload
```

#### English

```bash
# Enable IP forwarding
echo 'net.ipv4.ip_forward = 1' >> /etc/sysctl.conf
sysctl -p

# Configure iptables NAT (adjust eth0 to actual network interface)
iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE

# Save iptables rules (Ubuntu uses netfilter-persistent)
apt install -y iptables-persistent
netfilter-persistent save

# Open firewall port
ufw allow 1194/udp
# If using firewalld (CentOS/RHEL)
# firewall-cmd --permanent --add-port=1194/udp
# firewall-cmd --reload
```

### 6.8 启动 OpenVPN 服务 / Start OpenVPN Service

#### 中文

```bash
# 启动并启用开机自启
systemctl enable --now openvpn@server

# 检查服务状态
systemctl status openvpn@server

# 查看日志
tail -f /var/log/openvpn.log
```

#### English

```bash
# Start and enable OpenVPN service
systemctl enable --now openvpn@server

# Check service status
systemctl status openvpn@server

# View logs
tail -f /var/log/openvpn.log
```

---

## 七、客户端配置生成 / Client Configuration Generation

### 7.1 生成客户端证书 / Generate Client Certificate

#### 中文

```bash
cd /etc/openvpn/easy-rsa
CLIENT_NAME="client1"  # 客户端名称

# 生成客户端证书（无密码）
./easyrsa gen-req "${CLIENT_NAME}" nopass
./easyrsa sign-req client "${CLIENT_NAME}"
```

#### English

```bash
cd /etc/openvpn/easy-rsa
CLIENT_NAME="client1"  # Client name

# Generate client certificate (no password)
./easyrsa gen-req "${CLIENT_NAME}" nopass
./easyrsa sign-req client "${CLIENT_NAME}"
```

### 7.2 创建 .ovpn 配置文件 / Create .ovpn Configuration File

#### 中文

创建客户端配置文件，手动编写或使用脚本生成：

```bash
# 手动创建客户端 .ovpn 文件
cat > "/root/${CLIENT_NAME}.ovpn" << EOF
client
dev tun
proto udp
remote YOUR_SERVER_IP_OR_DOMAIN 1194
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
cipher AES-256-GCM
auth SHA256
verb 3

<ca>
$(cat /etc/openvpn/ca.crt)
</ca>

<cert>
$(sed -n '/-----BEGIN CERTIFICATE-----/,/-----END CERTIFICATE-----/p' /etc/openvpn/easy-rsa/pki/issued/${CLIENT_NAME}.crt)
</cert>

<key>
$(cat /etc/openvpn/easy-rsa/pki/private/${CLIENT_NAME}.key)
</key>

<tls-crypt>
$(cat /etc/openvpn/tls-crypt.key)
</tls-crypt>
EOF

echo "配置文件已生成: /root/${CLIENT_NAME}.ovpn"
```

#### English

Create client configuration file, either manually or using a script:

```bash
# Manually create client .ovpn file
cat > "/root/${CLIENT_NAME}.ovpn" << EOF
client
dev tun
proto udp
remote YOUR_SERVER_IP_OR_DOMAIN 1194
resolv-retry infinite
nobind
persist-key
persist-tun
remote-cert-tls server
cipher AES-256-GCM
auth SHA256
verb 3

<ca>
$(cat /etc/openvpn/ca.crt)
</ca>

<cert>
$(sed -n '/-----BEGIN CERTIFICATE-----/,/-----END CERTIFICATE-----/p' /etc/openvpn/easy-rsa/pki/issued/${CLIENT_NAME}.crt)
</cert>

<key>
$(cat /etc/openvpn/easy-rsa/pki/private/${CLIENT_NAME}.key)
</key>

<tls-crypt>
$(cat /etc/openvpn/tls-crypt.key)
</tls-crypt>
EOF

echo "Configuration file generated: /root/${CLIENT_NAME}.ovpn"
```

---

## 八、常用命令 / Common Commands

### 中文

| 命令 | 说明 |
|------|------|
| `systemctl start openvpn@server` | 启动 OpenVPN 服务 |
| `systemctl stop openvpn@server` | 停止 OpenVPN 服务 |
| `systemctl restart openvpn@server` | 重启 OpenVPN 服务 |
| `systemctl status openvpn@server` | 查看服务状态 |
| `systemctl enable openvpn@server` | 设置开机自启 |
| `journalctl -u openvpn@server -f` | 查看实时日志 |
| `docker logs -f openvpn` | 查看 Docker 容器的实时日志 |
| `docker restart openvpn` | 重启 Docker 容器 |
| `docker exec openvpn ovpn_listclients` | 查看已连接的客户端（Docker 方式） |
| `cat /etc/openvpn/openvpn-status.log` | 查看客户端连接状态 |
| `tail -f /var/log/openvpn.log` | 查看 OpenVPN 日志 |
| `netstat -tulpn \| grep 1194` | 检查端口监听状态 |

### English

| Command | Description |
|---------|-------------|
| `systemctl start openvpn@server` | Start OpenVPN service |
| `systemctl stop openvpn@server` | Stop OpenVPN service |
| `systemctl restart openvpn@server` | Restart OpenVPN service |
| `systemctl status openvpn@server` | Check service status |
| `systemctl enable openvpn@server` | Enable auto-start on boot |
| `journalctl -u openvpn@server -f` | View real-time logs |
| `docker logs -f openvpn` | View Docker container real-time logs |
| `docker restart openvpn` | Restart Docker container |
| `docker exec openvpn ovpn_listclients` | List connected clients (Docker) |
| `cat /etc/openvpn/openvpn-status.log` | View client connection status |
| `tail -f /var/log/openvpn.log` | View OpenVPN logs |
| `netstat -tulpn \| grep 1194` | Check port listening status |

---

## 九、客户端使用指南 / Client Usage Guide

### 9.1 Windows / Windows

#### 中文

1. 下载 OpenVPN Connect 客户端：https://openvpn.net/client/
2. 安装客户端并运行
3. 点击 **Import Profile** > **File**，选择 `.ovpn` 文件
4. 点击 **Connect** 连接
5. 连接成功后状态栏显示绿色图标

#### English

1. Download OpenVPN Connect client: https://openvpn.net/client/
2. Install and run the client
3. Click **Import Profile** > **File**, select the `.ovpn` file
4. Click **Connect** to connect
5. A green icon appears in the system tray when connected

### 9.2 macOS / macOS

#### 中文

1. 下载 Tunnelblick（免费开源）：https://tunnelblick.net/
2. 安装 Tunnelblick
3. 双击 `.ovpn` 文件导入配置
4. 点击菜单栏 Tunnelblick 图标 > **Connect**
5. 或下载 OpenVPN Connect for macOS

#### English

1. Download Tunnelblick (free and open-source): https://tunnelblick.net/
2. Install Tunnelblick
3. Double-click the `.ovpn` file to import the configuration
4. Click Tunnelblick icon in menu bar > **Connect**
5. Alternatively, download OpenVPN Connect for macOS

### 9.3 Linux / Linux

#### 中文

```bash
# 安装 OpenVPN 客户端
apt install -y openvpn

# 使用 .ovpn 配置文件连接（需要 root 权限）
sudo openvpn --config /path/to/client.ovpn

# 或者使用 systemd 方式
sudo systemctl start openvpn-client@client  # 需要将 .ovpn 放在 /etc/openvpn/client/
```

#### English

```bash
# Install OpenVPN client
apt install -y openvpn

# Connect using .ovpn configuration file (requires root)
sudo openvpn --config /path/to/client.ovpn

# Or using systemd
sudo systemctl start openvpn-client@client  # Place .ovpn in /etc/openvpn/client/
```

### 9.4 iOS / Android / iOS / Android

#### 中文

| 平台 | 客户端 | 下载方式 |
|------|--------|---------|
| iOS | OpenVPN Connect | App Store 搜索 "OpenVPN Connect" |
| Android | OpenVPN Connect | Google Play 或 F-Droid |

步骤：
1. 安装 OpenVPN Connect 应用
2. 将 `.ovpn` 文件传输到手机（通过邮件、网盘、Airdrop 等）
3. 在应用中导入配置文件
4. 点击连接

#### English

| Platform | Client | Download |
|----------|--------|----------|
| iOS | OpenVPN Connect | Search "OpenVPN Connect" on App Store |
| Android | OpenVPN Connect | Google Play or F-Droid |

Steps:
1. Install the OpenVPN Connect app
2. Transfer the `.ovpn` file to your phone (email, cloud storage, Airdrop, etc.)
3. Import the configuration file in the app
4. Tap Connect

---

## 十、高级配置 / Advanced Configuration

### 10.1 用户密码认证（auth-user-pass）/ Username/Password Authentication

#### 中文

除了证书认证，还可以额外启用用户名密码认证：

1. 创建密码文件 `/etc/openvpn/passwd`（格式：用户名 密码的 SHA512 哈希）
2. 安装 `openvpn-auth-pam` 插件

```bash
# 安装 PAM 认证插件
apt install -y openvpn-auth-pam

# 在 server.conf 中添加
echo 'plugin /usr/lib/openvpn/openvpn-auth-pam.so openvpn' >> /etc/openvpn/server.conf
echo 'client-cert-not-required' >> /etc/openvpn/server.conf  # 可选：双因素认证
```

#### English

In addition to certificate authentication, you can enable username/password authentication:

1. Create a password file `/etc/openvpn/passwd` (format: username SHA512-hashed-password)
2. Install the `openvpn-auth-pam` plugin

```bash
# Install PAM authentication plugin
apt install -y openvpn-auth-pam

# Add to server.conf
echo 'plugin /usr/lib/openvpn/openvpn-auth-pam.so openvpn' >> /etc/openvpn/server.conf
echo 'client-cert-not-required' >> /etc/openvpn/server.conf  # Optional: two-factor authentication
```

### 10.2 路由推送（push routing）/ Push Routing

#### 中文

让客户端自动添加特定网段的路由：

```ini
# 在 server.conf 中配置
push "route 192.168.1.0 255.255.255.0"
push "route 10.0.0.0 255.0.0.0"

# 按客户端推送（在 /etc/openvpn/ccd/client1 文件中）
iroute 192.168.2.0 255.255.255.0
```

#### English

Automatically add routes for specific network segments on the client:

```ini
# Configure in server.conf
push "route 192.168.1.0 255.255.255.0"
push "route 10.0.0.0 255.0.0.0"

# Per-client push (in /etc/openvpn/ccd/client1 file)
iroute 192.168.2.0 255.255.255.0
```

### 10.3 DNS 推送 / DNS Push

#### 中文

```ini
# 推送自定义 DNS 服务器
push "dhcp-option DNS 208.67.222.222"
push "dhcp-option DNS 208.67.220.220"
push "dhcp-option DOMAIN example.com"

# 阻止客户端 DNS 泄漏（Windows）
push "block-outside-dns"
```

#### English

```ini
# Push custom DNS servers
push "dhcp-option DNS 208.67.222.222"
push "dhcp-option DNS 208.67.220.220"
push "dhcp-option DOMAIN example.com"

# Prevent DNS leaks (Windows)
push "block-outside-dns"
```

### 10.4 多客户端隔离 / Multi-Client Isolation

#### 中文

默认情况下，VPN 客户端之间可以互相通信。如果需要隔离，添加：

```ini
# 在 server.conf 中添加
client-to-client  # 允许客户端间通信（默认注释掉即禁止）
# 注释掉 client-to-client 即可隔离客户端
```

如果希望更严格的隔离，使用 iptables：

```bash
# 禁止客户端间通信
iptables -I FORWARD -i tun0 -o tun0 -j DROP
```

#### English

By default, VPN clients can communicate with each other. To isolate them:

```ini
# Add to server.conf
client-to-client  # Allow client-to-client communication (commented out by default disables it)
# Commenting out client-to-client isolates clients
```

For stricter isolation, use iptables:

```bash
# Block client-to-client communication
iptables -I FORWARD -i tun0 -o tun0 -j DROP
```

### 10.5 UDP 与 TCP 双协议 / Dual Protocol (UDP + TCP)

#### 中文

同时监听 UDP 和 TCP 端口，提高连接兼容性：

1. 启动两个 OpenVPN 实例（两个配置文件）

```bash
# /etc/openvpn/server-udp.conf - 使用 UDP 1194
# /etc/openvpn/server-tcp.conf - 使用 TCP 443（类似 HTTPS 流量）
cp /etc/openvpn/server.conf /etc/openvpn/server-tcp.conf
```

编辑 `server-tcp.conf`，修改以下内容：

```ini
port 443
proto tcp
explicit-exit-notify 0  # TCP 模式下不需要此选项
```

```bash
# 启动两个服务
systemctl enable --now openvpn@server
systemctl enable --now openvpn@server-tcp
```

#### English

Listen on both UDP and TCP ports for better connection compatibility:

1. Start two OpenVPN instances (two configuration files)

```bash
# /etc/openvpn/server-udp.conf - Uses UDP 1194
# /etc/openvpn/server-tcp.conf - Uses TCP 443 (mimics HTTPS traffic)
cp /etc/openvpn/server.conf /etc/openvpn/server-tcp.conf
```

Edit `server-tcp.conf` and modify:

```ini
port 443
proto tcp
explicit-exit-notify 0  # Not needed for TCP mode
```

```bash
# Start both services
systemctl enable --now openvpn@server
systemctl enable --now openvpn@server-tcp
```

---

## 十一、安全加固 / Security Hardening

### 中文

| 安全措施 | 说明 | 配置 |
|---------|------|------|
| 使用 tls-crypt | 加密控制通道，防止主动探测 | `tls-crypt /path/to/key` |
| 限制加密算法 | 仅使用强加密套件 | `cipher AES-256-GCM` |
| 限制 TLS 版本 | 禁用旧版 TLS | `tls-version-min 1.2` |
| 证书撤销 | 吊销泄露的客户端证书 | `./easyrsa revoke CLIENT_NAME` |
| 双因素认证 | 证书 + 用户名密码 | `plugin openvpn-auth-pam.so` |
| 防火墙限制 | 仅允许 VPN 流量 | iptables 白名单 |
| 日志监控 | 监控异常连接 | 分析 `/var/log/openvpn.log` |
| 定期证书更新 | 设置证书有效期 | `./easyrsa renew CLIENT_NAME` |
| 禁用重复连接 | 防止同一证书多处使用 | `duplicate-cn` 保持注释 |
| 限制并发连接数 | 防止资源耗尽 | `max-clients 50` |

### English

| Security Measure | Description | Configuration |
|-----------------|-------------|---------------|
| Use tls-crypt | Encrypt control channel, prevent probing | `tls-crypt /path/to/key` |
| Restrict ciphers | Only use strong cipher suites | `cipher AES-256-GCM` |
| Restrict TLS version | Disable older TLS | `tls-version-min 1.2` |
| Certificate revocation | Revoke compromised client certs | `./easyrsa revoke CLIENT_NAME` |
| Two-factor auth | Certificate + username/password | `plugin openvpn-auth-pam.so` |
| Firewall restrictions | Allow only VPN traffic | iptables whitelist |
| Log monitoring | Monitor abnormal connections | Analyze `/var/log/openvpn.log` |
| Regular cert renewal | Set certificate validity period | `./easyrsa renew CLIENT_NAME` |
| Disable duplicate CN | Prevent multi-use of same cert | Keep `duplicate-cn` commented |
| Limit connections | Prevent resource exhaustion | `max-clients 50` |

---

## 十二、常见问题 / FAQ

### 中文

**Q1: 客户端连接失败，日志显示 "TLS Error: TLS key negotiation failed"**

A: 最常见的原因是 tls-crypt 密钥不匹配。确保服务器和客户端使用相同的 tls-crypt.key 文件。

**Q2: 连接成功但不能访问互联网**

A: 检查 IP 转发是否启用：`sysctl net.ipv4.ip_forward`。检查 iptables MASQUERADE 规则是否正确配置。确认客户端配置中没有将流量全部排除。

**Q3: 如何吊销客户端证书？**

```bash
cd /etc/openvpn/easy-rsa
./easyrsa revoke CLIENT_NAME
./easyrsa gen-crl
# 在 server.conf 中添加（如果还没有）
echo 'crl-verify /etc/openvpn/easy-rsa/pki/crl.pem' >> /etc/openvpn/server.conf
systemctl restart openvpn@server
```

**Q4: UDP 连接超时，怎么办？**

A: 网络环境可能限制了 UDP 流量。可以改用 TCP 协议（端口 443 通常不会被限制），参考 10.5 节。

**Q5: 如何查看当前在线客户端？**

```bash
cat /etc/openvpn/openvpn-status.log
# 或使用命令
systemctl status openvpn@server | grep "Connected"
```

**Q6: 端口被占用怎么办？**

A: 更改 server.conf 中的 `port` 参数，确保防火墙开放对应端口。

**Q7: 如何备份 OpenVPN 配置？**

```bash
# 备份整个配置目录
tar -czf openvpn-backup.tar.gz /etc/openvpn/
# Docker 方式备份数据卷
docker run --rm -v openvpn-data:/etc/openvpn -v /root:/backup alpine tar czf /backup/openvpn-backup.tar.gz /etc/openvpn/
```

**Q8: OpenVPN 支持 IPv6 吗？**

A: 是的，支持。需要在 server.conf 中添加 `server-ipv6 2001:db8:0:123::/64` 和 `push "route-ipv6 2001:db8:0:123::/64"`。

### English

**Q1: Client connection fails with "TLS Error: TLS key negotiation failed"**

A: The most common cause is a mismatched tls-crypt key. Ensure both server and client use the same tls-crypt.key file.

**Q2: Connected but can't access the internet**

A: Check if IP forwarding is enabled: `sysctl net.ipv4.ip_forward`. Check if iptables MASQUERADE rule is correctly configured. Verify that the client configuration doesn't exclude all traffic.

**Q3: How to revoke a client certificate?**

```bash
cd /etc/openvpn/easy-rsa
./easyrsa revoke CLIENT_NAME
./easyrsa gen-crl
# Add to server.conf (if not already present)
echo 'crl-verify /etc/openvpn/easy-rsa/pki/crl.pem' >> /etc/openvpn/server.conf
systemctl restart openvpn@server
```

**Q4: UDP connection times out, what to do?**

A: The network environment may be blocking UDP traffic. Try switching to TCP protocol (port 443 is usually not blocked), see section 10.5.

**Q5: How to see currently connected clients?**

```bash
cat /etc/openvpn/openvpn-status.log
# Or using
systemctl status openvpn@server | grep "Connected"
```

**Q6: What if the port is already in use?**

A: Change the `port` parameter in server.conf and ensure the firewall allows the new port.

**Q7: How to backup OpenVPN configuration?**

```bash
# Backup entire configuration directory
tar -czf openvpn-backup.tar.gz /etc/openvpn/
# Docker method: backup data volume
docker run --rm -v openvpn-data:/etc/openvpn -v /root:/backup alpine tar czf /backup/openvpn-backup.tar.gz /etc/openvpn/
```

**Q8: Does OpenVPN support IPv6?**

A: Yes, it does. Add `server-ipv6 2001:db8:0:123::/64` and `push "route-ipv6 2001:db8:0:123::/64"` to server.conf.

---

## 十三、参考资源 / References

### 中文

- [OpenVPN 官方文档](https://openvpn.net/community-resources/)
- [kylemanna/openvpn Docker 镜像](https://github.com/kylemanna/docker-openvpn)
- [EasyRSA 文档](https://github.com/OpenVPN/easy-rsa)
- [OpenVPN HOWTO](https://openvpn.net/community-resources/how-to/)
- [OpenVPN 安全配置指南](https://community.openvpn.net/openvpn/wiki/Hardening)

### English

- [OpenVPN Official Documentation](https://openvpn.net/community-resources/)
- [kylemanna/openvpn Docker Image](https://github.com/kylemanna/docker-openvpn)
- [EasyRSA Documentation](https://github.com/OpenVPN/easy-rsa)
- [OpenVPN HOWTO](https://openvpn.net/community-resources/how-to/)
- [OpenVPN Security Hardening Guide](https://community.openvpn.net/openvpn/wiki/Hardening)

---

## ☕ 支持 / Support

如果这个教程对你有帮助，欢迎请我喝杯咖啡：

**USDT (TRC20)**
```
TVbQerV1SF4MXB1JCcAzQxarewHwEPYTKm
```
