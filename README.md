# Docker Compose NuGet 私有服务器

基于 Docker Compose 的 NuGet 私有包管理服务器，使用 Nginx Proxy Manager 进行反向代理和 SSL 管理。

## 🚀 特性

- ✅ **开箱即用** - 一键部署 NuGet 私有服务器
- ✅ **可视化管理** - Nginx Proxy Manager 图形界面配置
- ✅ **自动 HTTPS** - Let's Encrypt 自动申请和续期 SSL 证书
- ✅ **数据持久化** - 本地数据存储，支持备份恢复
- ✅ **高性能** - 支持大文件上传（500MB+）
- ✅ **生产就绪** - 包含日志、监控和故障排除

## 📁 项目结构

```
docker-compose-nuget/
├── docker-compose.yml          # Docker Compose 配置
├── data/                       # 数据持久化目录
│   ├── db/                     # NuGet 数据库
│   └── packages/               # NuGet 包文件
└── README.md                   # 项目说明
```

## ⚡ 快速开始

### 1. 克隆项目

```bash
git clone https://github.com/blockchainiot/docker-compose-nuget.git
cd docker-compose-nuget
```

### 2. 准备环境

```bash
# 创建必要的目录
mkdir -p data/db data/packages
```

### 3. 启动服务

```bash
# 启动所有服务
docker compose up -d

# 查看服务状态
docker compose ps

# 查看日志
docker compose logs -f
```

### 4. 配置 Nginx Proxy Manager

1. 浏览器访问：`http://服务器IP:81`

2. 使用默认账号登录：
   - 邮箱：`admin@example.com`
   - 密码：`changeme`

3. 首次登录后会要求修改密码

4. 添加反向代理：
   - 点击 **Proxy Hosts** → **Add Proxy Host**
   - **Domain Names**: `nuget.your-domain.com`
   - **Forward Hostname**: `nuget-server`
   - **Forward Port**: `80`

5. 配置 SSL（可选但推荐）：
   - 在 **SSL** 标签页勾选 **Request a new SSL Certificate**
   - 勾选 **Force SSL** 和 **HTTP/2 Support**
   - 填写邮箱，勾选同意条款
   - 点击 **Save**

### 5. DNS 配置

在域名服务商处添加 A 记录：

```
nuget.your-domain.com → 服务器IP
```

## 📦 使用 NuGet 服务器

### 配置 NuGet 客户端

```bash
# 添加私有 NuGet 源
dotnet nuget add source https://nuget.your-domain.com/nuget \
  -n "Private NuGet" \
  -u "任意用户名" \
  -p "8e5735ec-3eac-5f9d-5a1c-196c82d7cb3d" \
  --store-password-in-clear-text

# 推送 NuGet 包
dotnet nuget push package.nupkg \
  -s "Private NuGet" \
  -k "8e5735ec-3eac-5f9d-5a1c-196c82d7cb3d"
```

### Visual Studio 配置

1. 打开 **工具** → **NuGet 包管理器** → **包管理器设置**
2. 选择 **包源** → **添加新源**
3. 配置信息：
   - **源地址**: `https://nuget.your-domain.com/nuget`
   - **用户名**: 任意
   - **密码**: `8e5735ec-3eac-5f9d-5a1c-196c82d7cb3d`

## 🔧 配置说明

### 环境变量

| 变量 | 默认值 | 说明 |
|------|--------|------|
| `NUGET_API_KEY` | `8e5735ec-3eac-5f9d-5a1c-196c82d7cb3d` | NuGet API 密钥 |

### 端口配置

| 端口 | 服务 | 说明 |
|------|------|------|
| 80 | Nginx Proxy Manager | HTTP（自动重定向到 HTTPS） |
| 443 | Nginx Proxy Manager | HTTPS |
| 81 | Nginx Proxy Manager | 管理界面 |

### 数据存储

- `./data/db`: NuGet 数据库文件
- `./data/packages`: NuGet 包文件存储
- Docker Volume `nuget-npm-data`: Nginx Proxy Manager 配置
- Docker Volume `nuget-npm-letsencrypt`: SSL 证书

## 🛠️ 维护操作

```bash
# 停止服务
docker compose down

# 重启服务
docker compose restart

# 查看日志
docker compose logs -f

# 备份数据
tar -czf nuget-backup-$(date +%Y%m%d).tar.gz data/

# 更新镜像
docker compose pull && docker compose up -d
```

## 📋 系统要求

- Docker 20.10+
- Docker Compose 2.0+
- 至少 1GB 可用磁盘空间
- 开放端口：80, 443, 81

## 🔒 安全建议

1. **更改默认 API Key** - 强烈建议修改默认的 API 密钥
2. **启用 HTTPS** - 在 Nginx Proxy Manager 中配置 SSL 证书
3. **定期备份** - 定期备份 `data/` 目录
4. **关闭管理端口** - 配置完成后可考虑限制 81 端口访问
5. **监控日志** - 定期检查访问和错误日志

## 🐛 故障排除

### 常见问题

#### 1. 端口冲突错误

**错误信息**: `Bind for 0.0.0.0:80 failed: port is already allocated`

**解决方案**:

```bash
# 查找占用端口的进程
sudo lsof -i :80
sudo lsof -i :443

# 停止占用的服务
sudo systemctl stop nginx
sudo systemctl stop apache2
```

#### 2. 无法访问管理界面

- 检查防火墙是否开放 81 端口
- 确认容器正常运行：`docker compose ps`

```bash
# 开放防火墙端口
sudo ufw allow 80/tcp
sudo ufw allow 443/tcp
sudo ufw allow 81/tcp
```

#### 3. SSL 证书申请失败

- 确保域名 DNS 已正确解析到服务器 IP
- 检查 80 端口是否可从外网访问（Let's Encrypt 验证需要）

#### 4. 无法推送包

- 检查 API Key 是否正确
- 验证网络连接
- 查看日志：`docker compose logs nuget-server`

### 日志查看

```bash
# 查看所有服务日志
docker compose logs

# 查看特定服务日志
docker compose logs nuget-server
docker compose logs nginx-proxy-manager

# 实时查看日志
docker compose logs -f
```
## 📄 许可证

本项目基于 MIT 许可证开源。

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！

- GitHub: https://github.com/blockchainiot/docker-compose-nuget
