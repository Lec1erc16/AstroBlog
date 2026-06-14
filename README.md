# Blog Service

这个目录是 `blog.example.cn` 的 Astro 博客项目。

## Position In Workspace

在当前架构里，这个博客已经纳入 `E:\aliyun` 根目录统一编排：

- 根目录 [docker-compose.yml](E:/aliyun/docker-compose.yml) 负责统一启动 `gateway + blog`
- 根目录 `gateway` 负责公网 `80/443` 入口
- 本目录只负责博客应用自身的构建、内容和容器镜像

## Recommended Usage

### Local Astro development

```powershell
npm install
npm run dev
```

适合日常写文章、调样式、做页面开发。

### Local container validation

```powershell
docker compose up -d --build
```

这里调用的是本目录自己的 [docker-compose.yml](E:/aliyun/blog/docker-compose.yml)，会把博客直接映射到本机 `8080`，适合单独验证镜像是否正常。

### Unified production-style run

请在根目录执行：

```powershell
cd E:\aliyun
docker compose up -d --build
```

这是当前推荐的正式部署方式。

## Files In This Directory

- [Dockerfile](E:/aliyun/blog/Dockerfile)
  - 构建 Astro 静态站点并交给容器内 Nginx 提供服务
- [nginx/default.conf](E:/aliyun/blog/nginx/default.conf)
  - 博客容器内部静态站点配置
- [docker-compose.yml](E:/aliyun/blog/docker-compose.yml)
  - 单项目本地容器验证用
- [docker-compose.prod.yml](E:/aliyun/blog/docker-compose.prod.yml)
  - 早期“博客自带 gateway”的方案，建议仅保留作参考
- [DEPLOY_ALIBABA_CLOUD_LINUX.md](E:/aliyun/blog/DEPLOY_ALIBABA_CLOUD_LINUX.md)
  - 旧版单项目部署说明，后续建议逐步迁移为根目录统一说明

## Suggested Cleanup Later

后续架构稳定后，可以考虑：

1. 删除 `blog/docker-compose.prod.yml`
2. 删除 `blog/deploy/nginx/conf.d/blog.conf.template`
3. 重写 `DEPLOY_ALIBABA_CLOUD_LINUX.md`，改为引用根目录部署文档
