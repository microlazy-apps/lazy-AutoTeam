# lazy-AutoTeam

懒猫微服 (LazyCat) wrapper around upstream
[`cnitlrt/AutoTeam`](https://github.com/cnitlrt/AutoTeam) — ChatGPT Team
账号自动轮转 / Codex 认证同步工具。

## 安装

应用市场搜 `AutoTeam` 一键安装。安装时会要求：

| 字段 | 必填 | 说明 |
|---|---|---|
| **API Key** | 是 | Web 面板登录密钥；建议 16+ 位随机字符串 |
| 邮箱提供者 | 否 | `cloudmail` / `cf_temp_email`，默认 `cloudmail` |
| Playwright 代理 URL | 否 | Chromium 出口代理（住宅 IP 体验更佳） |

装好后浏览器访问 `https://autoteam.{你的微服域名}` 即可。

## 装好后

1. 用安装时填写的 **API Key** 登录 Web 面板
2. 进「设置」继续填写邮箱凭据 / 远端同步（CPA / Sub2API）/ 代理等
3. 配置都自动写入 `/lzcapp/var/persist/.env`，重启 / 升级保留

详细使用说明见上游
[`docs/getting-started.md`](https://github.com/cnitlrt/AutoTeam/blob/dev/docs/getting-started.md)
和 [`docs/configuration.md`](https://github.com/cnitlrt/AutoTeam/blob/dev/docs/configuration.md)。

## 数据持久化

`/lzcapp/var/persist/` 下保留：

- `.env` —— 运行时配置
- `accounts.json` —— 账号池
- `state.json` —— 调度状态
- `auths/` —— 各账号 Codex 认证文件
- `screenshots/` —— Playwright 调试截图

## 免责声明

上游 README 注明：使用本工具可能违反 OpenAI 服务条款（包括但不限于
自动化操作、多账号管理）。使用者需自行承担账号封禁、IP 限制等后果。
本仓库仅做安装层封装，不改动上游逻辑。仅供学习研究。

## 链接

- 上游项目: <https://github.com/cnitlrt/AutoTeam>
- 本仓库: <https://github.com/microlazy-apps/lazy-AutoTeam>

## 许可证

跟随上游 — MIT。详见 [`LICENSE`](./LICENSE)。
