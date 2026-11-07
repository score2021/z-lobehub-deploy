# GitHub App 与交付门禁

- GitHub 登录 SSO 和仓库 SCM App 是两套配置。按所选 LobeHub 版本的 [.env.example](https://github.com/lobehub/lobehub/blob/canary/.env.example) 和 UI 核对 App 路由、变量、事件及权限。常见自托管 SCM 路由：`/api/webhooks/github`（Webhook）、`/api/webhooks/github/oauth/callback`（授权回调）、`/api/webhooks/github/setup`（安装跳转）；都用用户实际访问的同一 HTTPS 域名。该版要求时启用安装时请求 OAuth 授权及更新时跳转。
- 在 [GitHub App 设置](https://github.com/settings/apps) 新建专用 App。记录 App ID、slug、Client ID，把 Client Secret、Webhook Secret 和私钥写入目标机私密配置；上游要求时将 PEM 仅做一次 Base64 转换，编码不等于加密。App 与服务器的 Webhook Secret 必须相同。
- 按 LobeHub 版本实际订阅（PR、审查、检查/工作流等）选最小权限；只读试点先授必要读取，执行者需要写分支/PR/合并时单独评估其身份权限。安装选择 **Only select repositories** 并核对实际安装可见仓库。App 安装、账号绑定和执行设备的 Git 推送凭据是三个检查点。按仓库策略设保护分支和所需检查。
- 在 LobeHub 连接仓库，创建受控测试 PR，查 GitHub App Recent Deliveries 的签名投递、响应、event/delivery ID，再到 LobeHub 核对关联的 PR/话题/运行与真实状态变更。2xx 只是收到。失败先看 DNS/TLS、路径/重定向、签名、订阅与范围，然后查该事件的目标绑定。回放前检查是否已处理，避免重复触发写操作。
- 审查记录要绑定候选 SHA；合并前读回 PR head、目标分支及该提交要求的 CI。合并后分别验证合并 SHA、文件树及该 SHA 的 CI，再部署该精确版本。候选变化时重新验收；合并或部署超时先查远端状态，不盲目重试。真实 CI 失败自动续跑、审查触发修复分别用受控缺陷验证。
