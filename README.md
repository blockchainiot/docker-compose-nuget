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
│   ├── certificate.crt        # SSL 证书文件
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
# - ssl/certificate.crt  （SSL 证书文件）
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

这样设计的优势：
- ✅ 证书文件独立管理，便于更新
- ✅ 不与系统证书路径冲突
- ✅ 符合 Docker 最佳实践

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

## 📖 详细文档

更多详细的配置和故障排除信息，请参考 [DOCKER_COMPOSE_README.md](./DOCKER_COMPOSE_README.md)。

## 🐛 问题反馈

如果您遇到问题，请检查：

1. Docker 和 Docker Compose 是否正确安装
2. 端口 80 和 443 是否被占用
3. SSL 证书文件是否正确配置
4. 查看容器日志：`docker-compose logs`

## 📄 许可证

本项目基于 MIT 许可证开源。

## 🤝 贡献

欢迎提交 Issue 和 Pull Request！
