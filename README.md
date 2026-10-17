# xcaddy

基于官方 Caddy 镜像构建，包含 `github.com/caddy-dns/cloudflare` DNS 插件。

- Caddy：`2.11.7`
- Cloudflare 插件：`v0.2.4`（支持 `cfut_` / `cfat_` 格式令牌）
- 平台：`linux/amd64`

## 构建与发布

工作流仅支持手动触发：在 Actions 中选择 **Build and publish xcaddy**，点击 **Run workflow**，或执行：

```sh
gh workflow run build.yml --repo sidecloudgroup/xcaddy --ref main
```

需要可供本仓库使用、带有 `self-hosted`、`linux`、`x64` 标签的 runner，且已安装 Docker 并允许 runner 用户使用 Docker。构建使用官方 `caddy:<版本>-builder` 镜像中的 xcaddy，运行镜像为官方 `caddy:<版本>-alpine`。

使用组织或仓库 Actions secrets：`DOCKERHUB_USERNAME` 和 `DOCKERHUB_TOKEN`。GHCR 使用工作流的 `GITHUB_TOKEN`。

镜像先构建并验证 Caddy、Cloudflare 模块及新令牌格式的配置兼容性，通过后将同一个镜像推送到：

- Docker Hub：`pplulee/xcaddy`
- GHCR：`ghcr.io/sidecloudgroup/xcaddy`

两个仓库均发布 `latest`、`2.11.7-cf0.2.4` 和 `sha-<提交前12位>` 标签。正式部署建议使用版本或 SHA 标签。工作流不会自动更新生产服务器。

升级时同时修改 Dockerfile 中的默认版本和 `.github/workflows/build.yml` 的版本变量，再手动触发构建。

## 本地构建

```sh
docker build --pull -t pplulee/xcaddy:local .
docker run --rm pplulee/xcaddy:local caddy list-modules
```

## 运行配置

Cloudflare API Token 仅在运行时通过环境变量注入；不要把真实令牌写入镜像或仓库。令牌应限制为需要的域名，并授予 Zone Read 和 DNS Write 权限。

```caddyfile
example.com {
    tls {
        dns cloudflare {env.CLOUDFLARE_API_TOKEN}
    }
    reverse_proxy alist:5244
}
```

升级镜像时保留 `/data` 和 `/config` 卷，以保留证书等运行数据。
