## OpenVPN Docker 部署到 8888/tcp 端口完整流程

以下文档演示如何使用 `kylemanna/openvpn` 镜像，将 OpenVPN 服务部署在 `8888/tcp` 端口上。包含配置生成、密钥初始化、客户端证书生成、客户端配置导出、容器启动及 Docker Compose 文件示例。

------

### 1. 环境准备

1. 确保已拉取最新镜像：

   ```bash
   docker pull kylemanna/openvpn
   ```

2. 在宿主机创建配置与数据目录：

   ```bash
   mkdir -p /root/openvpn
   ```

3. 放行 `8888/tcp` 端口：

   - 云厂商安全组或防火墙：放行 `8888/tcp`。

   - 本地防火墙（以 UFW 为例）：

     ```bash
     sudo ufw allow 8888/tcp
     ```

------

### 2. 生成服务端配置文件 (`ovpn_genconfig`)

在目录 `/root/openvpn` 下，通过 `ovpn_genconfig` 指定使用 `tcp://` 协议和自定义端口：

```bash
docker run --rm -v /root/openvpn:/etc/openvpn \
  kylemanna/openvpn ovpn_genconfig \
    -u tcp://YOUR_PUBLIC_IP:8888
```

- `--rm`：容器运行结束后自动删除。
- `-v /root/openvpn:/etc/openvpn`：映射配置卷。
- `ovpn_genconfig`：生成初始 `server.conf`，内部仍监听 `1194/tcp`。
- `-u tcp://...:8888`：客户端连接使用 `tcp` 协议及 `8888` 端口。

------

### 3. 初始化 PKI 密钥库 (`ovpn_initpki`)

第一次部署需初始化证书与密钥：

```bash
docker run --rm -it -v /root/openvpn:/etc/openvpn \
  kylemanna/openvpn ovpn_initpki
```

- `-it`：交互模式，需输入 CA 密码等。
- 生成 CA 根证书、服务器证书、DH 参数等。

------

### 4. 生成客户端证书 (`easyrsa build-client-full`)

为客户端生成无密码证书（可选 `nopass`）：

```bash
docker run --rm -it -v /root/openvpn:/etc/openvpn \
  kylemanna/openvpn easyrsa build-client-full openvpn-client nopass
```

- `openvpn-client`：客户端名字，可自定义。
- `nopass`：不需要输入证书密码，便于自动连接。

------

### 5. 导出客户端配置 (`ovpn_getclient`)

导出包含证书与所有配置的一体化 `.ovpn` 文件：

```bash
docker run --rm -v /root/openvpn:/etc/openvpn \
  kylemanna/openvpn ovpn_getclient openvpn-client \
    > /root/openvpn/openvpn-client.ovpn
```

- 输出文件位于主机 `/root/openvpn/openvpn-client.ovpn`。
- 使用图形或命令行客户端导入该文件即可连接。

------

### 6. 启动 OpenVPN 服务容器 (`docker run`)

将宿主机 `8888/tcp` 映射到容器内部的 `1194/tcp`：

```bash
docker run -d \
  --name openvpn-server \
  --cap-add=NET_ADMIN \
  -v /root/openvpn:/etc/openvpn \
  -p 8888:1194/tcp \
  kylemanna/openvpn
```

- `-d`：后台运行。
- `--cap-add=NET_ADMIN`：授予网络管理能力。
- `-p 8888:1194/tcp`：端口映射。
- 容器内部仍监听 `1194/tcp`，客户端配置中已指向 `8888/tcp`。

------

### 7. Docker Compose 配置示例

在 `/root/openvpn` 下创建 `docker-compose.yml`：

```yaml
version: '3.8'
services:
  openvpn:
    image: kylemanna/openvpn
    container_name: openvpn-server
    cap_add:
      - NET_ADMIN
    volumes:
      - ./:/etc/openvpn
    ports:
      - "8888:1194/tcp"
    restart: unless-stopped
```

启动/停止服务：

```bash
  docker compose up -d
  docker compose down
```

------

至此，OpenVPN 已成功部署在 `8888/tcp` 端口。客户端导入 `openvpn-client.ovpn`，并连接到 `YOUR_PUBLIC_IP:8888` 即可。