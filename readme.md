# 使用 Docker 部署 OpenVPN

## 项目简介

本项目旨在记录如何通过 Docker 快速部署 OpenVPN 服务，适用于需要便捷搭建 VPN 环境的用户。通过本项目的文档和脚本，用户可以在主流操作系统上高效完成 OpenVPN 的安装与配置。

## 适用场景

- 远程安全访问内网资源
- 科学上网
- 局域网穿透
- 个人或小型团队自建 VPN 服务

## 环境要求

- 操作系统：Windows / Linux / macOS（推荐使用 Linux 服务器）
- Docker 已安装并配置完成
- Docker Compose（可选，便于管理多个容器）

## 部署步骤

1. **克隆本项目**
   
2. **参考文档**
   
   - 请先阅读 `docker部署OpenVPN.md`，了解详细的部署流程和注意事项。
   - `客户端使用说明.md` 提供了 OpenVPN 客户端的安装与连接方法。
   
3. **拉取 OpenVPN 镜像**
   
   ```bash
   docker pull kylemanna/openvpn
   ```
   
4. **初始化 OpenVPN 配置**
   
   ```bash
   docker run -v /your/path/openvpn-data:/etc/openvpn --rm kylemanna/openvpn ovpn_genconfig -u udp://你的服务器IP
   docker run -v /your/path/openvpn-data:/etc/openvpn --rm -it kylemanna/openvpn ovpn_initpki
   ```
   
5. **启动 OpenVPN 服务**
   
   ```bash
   docker run -v /your/path/openvpn-data:/etc/openvpn -d -p 1194:1194/udp --cap-add=NET_ADMIN kylemanna/openvpn
   ```
   
6. **生成客户端配置文件**
   
   ```bash
   docker run -v /your/path/openvpn-data:/etc/openvpn --rm -it kylemanna/openvpn easyrsa build-client-full <客户端名称> nopass
   docker run -v /your/path/openvpn-data:/etc/openvpn --rm kylemanna/openvpn ovpn_getclient <客户端名称> > <客户端名称>.ovpn
   ```
   
7. **客户端下载与连接**
   
   - 使用 `openvpn-connect-3.7.3.4351_signed.msi` 安装 Windows 客户端，导入 `.ovpn` 配置文件即可连接。

## 常见问题

- 端口未开放：请确保服务器端口已放行。若是云服务器，还需要去控制台放行.
- 客户端无法连接：请检查服务器公网 IP、配置文件及防火墙设置。

## 参考资料

- [OpenVPN 官方文档](https://openvpn.net/)
- [kylemanna/openvpn 镜像文档](https://hub.docker.com/r/kylemanna/openvpn)

---

如有问题，欢迎提交 issue 或查阅相关文档。
