# 主机、应用、域名与执行设备

1. 在目标 Linux 核对系统/架构、容器运行时、磁盘、端口、出站 DNS 与 HTTPS、时间同步。为 LobeHub 使用专用服务账号和目录，记录项目名及持久卷。若用 Podman，单独验证 Compose、网络名解析、卷权限和重启行为；默认按上游 Docker Compose 路径。
2. 从 [官方自托管指南](https://github.com/lobehub/lobehub/blob/canary/docs/self-hosting/start.mdx) 找到安装入口，再按选定 release 下载并审阅 Compose 与环境模板；记录版本/镜像摘要。使用该版支持的 PostgreSQL 与扩展、Redis、S3 兼容存储，初始化 bucket，验证应用实际拥有读写权限。应用、数据库和存储凭据文件限制为服务账号可读；授权和加密密钥跨重启保留。分别备份数据库、对象存储、配置和密钥。先运行 `docker compose config --quiet`、`docker compose up -d`、`docker compose ps`；按组件查脱敏日志和就绪。空库安装与旧库迁移分开。
3. 准备如 `agents.example.com` 的子域名，核对域名管辖和主机所在地域要求。按云厂商和 DNS 管理面配置 A（实际有 IPv6 才配 AAAA），用 `dig +short A agents.example.com` 和独立网络验证解析、入口连通与证书链。按 ACME 验证方式开放必要端口；内网数据库、Redis、Agent Gateway 及管理端口不直接暴露。反向代理示意（假定应用已绑定宿主回环 3210）：

   ```caddyfile
   agents.example.com {
       reverse_proxy 127.0.0.1:3210
   }
   ```

   用 `caddy validate --config /etc/caddy/Caddyfile` 检查，再按所用服务管理方式重载。核对 `curl -sSI https://agents.example.com/`、真实网页登录、反代转发的公开协议/主机、S3 浏览器可达的 HTTPS 签名 URL 和上传/下载。若版本支持区分内部与公开 URL，分别配置：内部供容器通信，公开供用户、OAuth 和签名 URL 使用。对象存储只开放确需的入口，管理台和 bucket 维持受控。不要为测试上传把 bucket 改为公开。
4. [官方 CLI](https://www.npmjs.com/package/@lobehub/cli) 按版本要求装 Node 和固定版 CLI，确认服务进程看到的 Node 与交互 shell 一致。用 `lh login --help`、`lh connect --help` 核对自托管实例登录与设备连接；网关地址和令牌采用同版本上游定义。连接设备以后从 LobeHub 下发只读 smoke，核对设备、账号、工作目录和真实命令结果，再按版本支持的方式交给 systemd 监督。用户级服务要跨注销/重启运行时，核查 linger 及开机启动需求。
5. 切换域名时同步应用公开 URL、GitHub 回调、浏览器 Cookie/会话、CLI 登录地址和公开对象 URL。观察每个跳转的 Location；出现内部 IP、localhost 或 HTTP 时查应用 URL 与转发头。先修源配置；兼容性重写仅限已定位的具体路径，复验跳转和 Webhook，记录移除条件。

快速区分：网页登录正常、设备离线时，优先检查设备进程的登录实例、网关可达性、出站代理和常驻状态；设备在线、运行失败时再查真实账号、工作目录、PATH 和模型认证。曾遇到的 Node 版本不匹配、反代跳转到容器内部地址，应通过进程版本和 Location 证实后再修复。
