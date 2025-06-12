# Docker Compose NuGet 私有服务器

基于 Docker Compose 的 NuGet 私有包管理服务器，支持 HTTPS 和域名绑定。

## 🚀 特性

- ✅ **开箱即用** - 一键部署 NuGet 私有服务器
- ✅ **HTTPS 支持** - 完整的 SSL/TLS 配置
- ✅ **反向代理** - Nginx 反向代理，支持域名绑定
- ✅ **数据持久化** - 本地数据存储，支持备份恢复
- ✅ **高性能** - 支持大文件上传（500MB+）
- ✅ **生产就绪** - 包含日志、监控和故障排除

## 📁 项目结构

```
docker-compose-nuget/
├── docker-compose.yml          # Docker Compose 配置
├── nginx/                      # Nginx 配置目录
│   ├── nginx.conf             # Nginx 主配置
│   └── conf.d/nuget.conf      # NuGet 站点配置
├── ssl/                       # SSL 证书目录（项目根目录）
│   ├── certificate.pem        # SSL 证书文件
│   └── private.key           # SSL 私钥文件
├── data/                      # 数据持久化目录
│   ├── db/                   # NuGet 数据库
│   └── packages/             # NuGet 包文件
└── README.md                  # 项目说明
```

## ⚡ 快速开始

### 1. 克隆项目

```bash
git clone <repository-url>
cd docker-compose-nuget
```

### 2. 准备环境

```bash
# 创建必要的目录
mkdir -p data/db data/packages ssl

# 准备 SSL 证书（用于 HTTPS，推荐生产环境使用）
# 将证书文件放入项目根目录的 ssl/ 目录：
# - ssl/certificate.pem  （SSL 证书文件）
# - ssl/private.key      （SSL 私钥文件）
```

### 3. 配置域名

编辑 `nginx/conf.d/nuget.conf`，将 `your-domain.com` 替换为您的实际域名。

### 4. 启动服务

```bash
# 启动所有服务
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f
```

### 5. 访问服务

- **HTTP**: http://your-domain.com
- **HTTPS**: https://your-domain.com

## 📦 使用 NuGet 服务器

### 配置 NuGet 客户端

```bash
# 添加私有 NuGet 源
dotnet nuget add source https://your-domain.com/nuget \
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
   - **源地址**: `https://your-domain.com/nuget`
   - **用户名**: 任意
   - **密码**: `8e5735ec-3eac-5f9d-5a1c-196c82d7cb3d`

## 🔧 配置说明

### 环境变量

- `NUGET_API_KEY`: NuGet API 密钥（默认: `8e5735ec-3eac-5f9d-5a1c-196c82d7cb3d`）

### 端口配置

- `80`: HTTP 端口（自动重定向到 HTTPS）
- `443`: HTTPS 端口

### 数据存储

- `./data/db`: NuGet 数据库文件
- `./data/packages`: NuGet 包文件存储

### SSL 证书配置

SSL 证书采用以下映射方式：
- **本地路径**: `./ssl/` （项目根目录）
- **容器路径**: `/etc/nginx/ssl/` （nginx 容器内）

**支持的证书格式**：
- `certificate.pem` - SSL 证书文件（PEM 格式）
- `private.key` - SSL 私钥文件（KEY 格式）

这样设计的优势：
- ✅ 证书文件独立管理，便于更新
- ✅ 不与系统证书路径冲突
- ✅ 符合 Docker 最佳实践
- ✅ 支持常见的 PEM/KEY 证书格式

## 🛠️ 维护操作

```bash
# 停止服务
docker-compose down

# 重启服务
docker-compose restart

# 查看日志
docker-compose logs -f

# 备份数据
tar -czf nuget-backup-$(date +%Y%m%d).tar.gz data/

# 更新镜像
docker-compose pull && docker-compose up -d
```

## 📋 系统要求

- Docker 19.03+
- Docker Compose 1.25+
- 至少 1GB 可用磁盘空间
- 可选：有效的 SSL 证书（用于 HTTPS）

## 🔒 安全建议

1. **更改默认 API Key** - 强烈建议修改默认的 API 密钥
2. **使用 HTTPS** - 在生产环境中启用 SSL 证书
3. **定期备份** - 定期备份 `data/` 目录
4. **访问控制** - 考虑添加 IP 白名单或防火墙规则
5. **监控日志** - 定期检查访问和错误日志

## 🐛 故障排除

### 常见问题

#### 1. 端口冲突错误
**错误信息**: `failed to set up container networking: driver failed programming external connectivity on endpoint nuget-nginx: Bind for 0.0.0.0:80 failed: port is already allocated`

**解决方案**:

**方案一**: 查找并停止占用端口80的进程
```powershell
# 查找占用端口80的进程
netstat -ano | findstr ":80"

# 停止占用的进程（PID替换为实际进程ID）
taskkill /PID <进程ID> /F
```

**方案二**: 修改端口映射（推荐）
```yaml
# 编辑 docker-compose.yml
ports:
  - "8080:80"    # 改为8080端口
  - "443:443"
```

**方案三**: 停止常见的Web服务
```powershell
# 停止IIS服务
net stop w3svc

# 停止Apache（如果安装了）
net stop apache2.4
```

#### 2. SSL 证书错误
- 检查证书文件路径和权限
- 验证证书是否有效
- 确保证书格式正确（PEM/KEY）

#### 3. 无法推送包
- 检查 API Key 是否正确
- 验证网络连接
- 查看 Nginx 错误日志：`docker-compose logs nginx`

#### 4. 服务无法启动
- 检查端口是否被占用
- 验证配置文件语法
- 查看 Docker 日志：`docker-compose logs`

### 日志查看

```bash
# 查看所有服务日志
docker-compose logs

# 查看特定服务日志
docker-compose logs nuget-server
docker-compose logs nginx

# 实时查看日志
docker-compose logs -f
```

## 📄 许可证

本项目基于 MIT 许可证开源。

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！
