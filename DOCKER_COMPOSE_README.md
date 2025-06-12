# Docker Compose NuGet 服务器部署指南

这个 Docker Compose 配置提供了一个完整的 NuGet 私有服务器解决方案，包含：
- NuGet 服务器 (sunside/simple-nuget-server)
- Nginx 反向代理（支持 HTTPS）

## 目录结构

```
docker-compose-nuget/
├── docker-compose.yml          # 主配置文件
├── nginx/
│   ├── nginx.conf             # Nginx 主配置
│   └── conf.d/
│       └── nuget.conf         # NuGet 站点配置
├── ssl/                       # SSL 证书目录
│   ├── certificate.crt        # SSL 证书文件
│   └── private.key           # SSL 私钥文件
└── data/                     # 数据持久化目录
    ├── db/                   # NuGet 数据库
    └── packages/             # NuGet 包文件
```

## 部署步骤

### 1. 准备 SSL 证书

将您的 SSL 证书文件放在 `ssl` 目录下：
```bash
mkdir -p ssl
# 将您的证书文件复制到:
# ssl/certificate.crt  - SSL 证书
# ssl/private.key      - SSL 私钥
```

### 2. 配置域名

编辑 `nginx/conf.d/nuget.conf`，将 `your-domain.com` 替换为您的实际域名。

### 3. 创建数据目录

```bash
mkdir -p data/db data/packages
```

### 4. 启动服务

```bash
# 启动所有服务
docker-compose up -d

# 查看服务状态
docker-compose ps

# 查看日志
docker-compose logs -f
```

## 配置说明

### NuGet 服务器配置

- **API Key**: `8e5735ec-3eac-5f9d-5a1c-196c82d7cb3d`
- **端口**: 内部使用 80 端口，通过 Nginx 代理
- **数据存储**: `./data/db` 和 `./data/packages`

### Nginx 配置

- **HTTP**: 端口 80（自动重定向到 HTTPS）
- **HTTPS**: 端口 443
- **上传限制**: 500MB（支持大包文件）
- **SSL**: 支持 TLS 1.2 和 1.3

## 使用方法

### 1. 配置 NuGet 客户端

```bash
# 添加私有源
dotnet nuget add source https://your-domain.com/nuget -n "Private NuGet" -u "任意用户名" -p "8e5735ec-3eac-5f9d-5a1c-196c82d7cb3d" --store-password-in-clear-text

# 推送包
dotnet nuget push package.nupkg -s "Private NuGet" -k "8e5735ec-3eac-5f9d-5a1c-196c82d7cb3d"
```

### 2. Visual Studio 配置

在 Visual Studio 中：
1. 工具 → NuGet 包管理器 → 包管理器设置
2. 包源 → 添加新源
3. 源地址：`https://your-domain.com/nuget`
4. 用户名：任意
5. 密码：`8e5735ec-3eac-5f9d-5a1c-196c82d7cb3d`

## 维护操作

```bash
# 停止服务
docker-compose down

# 重新构建并启动
docker-compose up -d --build

# 查看容器状态
docker-compose ps

# 查看实时日志
docker-compose logs -f nuget-server
docker-compose logs -f nginx

# 备份数据
tar -czf nuget-backup-$(date +%Y%m%d).tar.gz data/

# 清理未使用的镜像
docker system prune -f
```

## 安全建议

1. **更改 API Key**: 建议修改默认的 API Key
2. **防火墙配置**: 确保只开放必要的端口（80, 443）
3. **定期备份**: 定期备份 `data` 目录
4. **SSL 证书**: 使用有效的 SSL 证书，定期更新
5. **访问控制**: 考虑添加 IP 白名单或其他访问控制

## 故障排除

### 常见问题

1. **SSL 证书错误**
   - 检查证书文件路径和权限
   - 验证证书是否有效

2. **无法推送包**
   - 检查 API Key 是否正确
   - 验证网络连接
   - 查看 Nginx 错误日志

3. **服务无法启动**
   - 检查端口是否被占用
   - 验证配置文件语法
   - 查看 Docker 日志

### 日志查看

```bash
# NuGet 服务器日志
docker-compose logs nuget-server

# Nginx 日志
docker-compose logs nginx

# 系统日志
docker-compose logs
``` 