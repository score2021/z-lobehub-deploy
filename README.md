# z-lobehub-deploy

在新 Linux 机器安装或迁移自托管 LobeHub 的 Codex Skill。覆盖容器与依赖、域名和 HTTPS、设备接入、GitHub App、企业微信长任务回执、Claude Code 认证与试点验收。默认从上游发行物开始，提供操作指南与验收合同。

## 安装

```sh
git clone https://github.com/score2021/z-lobehub-deploy.git
mkdir -p "$HOME/.agents/skills"
target="$HOME/.agents/skills/z-lobehub-deploy"
legacy="${CODEX_HOME:-$HOME/.codex}/skills/z-lobehub-deploy"
if [ -e "$target" ] || [ -L "$target" ] || [ -e "$legacy" ] || [ -L "$legacy" ]; then
  printf '%s\n' "已有同名安装，请先比对版本：$target" >&2
else
  mkdir "$target" && cp -R z-lobehub-deploy/SKILL.md z-lobehub-deploy/references z-lobehub-deploy/LICENSE "$target/"
fi
```

`z-` 是便于筛选的个人命名前缀。安装需要 Git 和 Codex；实装需要目标主机权限及相应平台授权。开启新 Codex 会话并使用 `$z-lobehub-deploy`。

## 样例

以下使用合成域名与仓库，无需凭据或真实主机。要求只生成本地计划：

```text
用 $z-lobehub-deploy 为一台 Ubuntu 24.04 新机规划部署。公开域名是 agents.example.com，GitHub 试点仓库是 example-org/hello-pilot，企业微信只允许一个试点成员单聊，使用独立 Claude Code 审查。输出 plan.md；不要联网、登录或改任何外部配置。
```

## 成果展示

以上样例已在独立 Codex CLI 0.162.1 会话生成 `plan.md`。以下是实际方案摘录；它是部署计划，表内验收尚未执行。

| 验收项 | 计划要求的证据 | 本次状态 |
| --- | --- | --- |
| HTTPS | A/必要 AAAA、独立网络证书链、正确跳转与公开签名 URL | 未运行 |
| Webhook | 签名 delivery/event ID 对应正确 PR、话题、运行及状态推进 | 未运行 |
| 独立 Claude | 实际设备真实调用、独立认证/会话/候选、权限边界及审查输出 | 未运行 |
| 短任务回执 | 真实入站、绑定确认、匹配 operation 终态、原单聊实际收到 | 未运行 |

本 Skill 从一次实际试点提炼。历史试点观察到：超过 270 秒的任务在桥重启后仍能自动送达；另一项新需求用专用 Claude 入口完成独立审查、CI、合并、测试部署和企业微信回传。发布包不含真实环境代码、日志或凭据。该经验不等于各版本、各云平台的完整重装或所有故障模式均已验证。

入口：[SKILL.md](SKILL.md) · [验收与故障判断](references/acceptance.md)
