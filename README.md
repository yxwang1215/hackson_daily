# hackson_daily

这是一个带后端 API、SQLite 数据库和本地上传目录的 Next.js 应用。它不能像
`/Users/wyx/Code/yxwang1215.github.io` 那种 Jekyll 静态站一样直接丢到
GitHub Pages；GitHub Pages 只能托管静态文件，不能运行 `/api/*`、Prisma 或图片上传。

推荐部署方式是：代码放 GitHub，服务跑在一台 VPS/云服务器上，用 Docker Compose
启动。公网访问可以先用 `http://服务器IP:3000`，之后再接域名和 HTTPS 反向代理。

## 本地运行

```bash
npm install
npm run db:setup
npm run dev
```

打开 http://localhost:3000。

## 公网部署

服务器需要：

- 一个公网 IP
- 安全组/防火墙放行 TCP `3000`
- 安装 Docker 和 Docker Compose

假设仓库地址是
`https://github.com/yxwang1215/hackson_daily.git`：

```bash
git clone https://github.com/yxwang1215/hackson_daily.git
cd hackson_daily
cp .env.production.example .env.production
```

如果要启用真实 AI 分析，编辑 `.env.production`，填入 `LLM_API_KEY`。不填也能运行，
应用会使用本地兜底文案。

启动：

```bash
docker compose up -d --build
```

访问：

```text
http://服务器公网IP:3000
```

如果访问不了，先检查云服务器控制台的安全组是否放行 `3000`，再检查系统防火墙。

更新代码后：

```bash
git pull
docker compose up -d --build
```

## 数据持久化

`docker-compose.yml` 会把服务器当前目录的 `./data` 挂载到容器内 `/app/data`：

- SQLite 数据库：`data/dev.db`
- 上传图片：`data/uploads/`

备份时直接备份 `data/` 目录即可。

## 域名和 HTTPS

上线后建议把域名 A 记录指向服务器公网 IP，再用 Nginx 或 Caddy 反向代理到
`127.0.0.1:3000`。如果只是临时公开测试，可以先开放服务器安全组/防火墙的 3000 端口。

本仓库已经带了 `Caddyfile.example`。有域名后可以复制成 `Caddyfile`，把里面的
`your-domain.example.com` 改成你的域名，再把 `docker-compose.yml` 里的 `caddy`
服务取消注释。
