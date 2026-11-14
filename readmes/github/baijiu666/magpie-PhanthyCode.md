# magpie-PhanthyCode

把 [code.phanthy.com](https://code.phanthy.com) 的模型接进 magpie：登录、模型目录、积分额度、每日签到、推理直连。不需要官方桌面端。

```
provider id   phanthycode
模型名        phanthycode/<id>
```

## 能做什么

- **两种登录**：浏览器 OAuth（PKCE，本机回环回调；默认用一次性隔离浏览器，不读日常 profile）；或从本机官方桌面端导入短效 access token（只读）。第二个账号登录时如果站点停在 `/account`（丢了 `next`），40 秒后会在同一隔离窗口补开「继续授权」页。
- **模型目录跟随账号**：登录后读取 `/api/phanthy_code/models`；接口失败退回上次快照，不返回空列表；价格未知的模型不列。
- **额度卡片**：积分余额、临期积分、套餐；临期信息挂在可路由窗口上。
- **每日签到**：每天一次，插件侧自动领取；卡片显示金额与连续天数（magpie 设置 → 用量可关）。设备身份未注册时，点签到会先完成注册，失败如实返回。
- **设备身份独立**：插件自己生成并保存设备密钥，不读取、不依赖桌面端。
- **邀请模式**（provider 级开关，默认关）：决定新注册账号使用谁的邀请链接。关闭 = 最新注册账号；开启 = magpie 账号页「Make first」的账号；没有可用源时用预留码，再没有就不带邀请。只对新注册账号生效，已有账号不能补绑。
- **错误不误杀账号**：模型级错误 404；套餐问题 403；积分用尽 402；账号被服务端禁用 403 `account_disabled`；非登录语义的 401/403 带 `X-Magpie-Sign-In: kept`；超时 504。

## 模型与计费

模型列表由云端目录下发，登录后用 `magpie plugin list` 查看该账号的真实列表；当前包含 DeepSeek、GLM、Kimi、Phana / Phano、Phanthy 自有档位，`Phanthy-fast` 免费。

倍率 = `input_credits_per_million_tokens / 3000`。计费按 token，小请求会抹零；除 `Phanthy-fast` 外没有免费模型。注册、邀请、签到赠送的积分（常见口径 3,000 + 3,000 + 300/天）以后端钱包为准，不会让套餐外的模型可用。

## 安装

magpie 插件 → 发现 → 搜 `PhanthyCode` → 安装；或：

```bash
magpie plugin add baijiu666/magpie-PhanthyCode
```

装完在插件设置里选登录方式：浏览器 OAuth（推荐，可自动续期）或从本机导入（只导入短效 access token）。

## 常见问题

| 现象 | 原因 / 处理 |
|---|---|
| 积分到账但模型不可用 | 模型不在当前套餐（404 `model_not_found`，正文带 `model_not_allowed`）。换模型或升级套餐；账号不会被停用。 |
| 所有模型都不可用 | 先看登录是否失效；再看是否 `device_binding_conflict`（账号已绑官方桌面端）；最后看套餐是否到期。 |
| 网页登录显示「此账户已被禁用」 | 账号被服务端禁用，返回 403 `account_disabled`；重登不会恢复，只能换号或联系客服。 |
| 额度显示未知 | 数据面接口失败；卡片会显示原因。 |

## 数据处理

- 提示词、补全、模型目录、额度、签到只发 `https://code.phanthy.com`；插件不中转、不记录对话。
- 请求头与官方桌面端一致（来源、客户端版本、设备指纹：platform / arch / osRelease / 本机 IPv4 候选），只含设备信息，不含对话。
- 本地只保存：OAuth 凭据（magpie 管理）、设备密钥（`~/.cache/magpie`，权限 0600）、目录 / 额度快照、邀请缓存。没有遥测。
- 从本机导入只读短效 access token，不复制 `refresh_token`、推理令牌、设备私钥，不会让桌面端下线。

## 设置

| 变量 | 作用 |
|---|---|
| `PHANTHY_ALLOW_DEVICE_REGISTER=0` | 禁止向云端注册设备身份（推理面不可用）。不设 = 允许。 |

## 已知限制

- 计费按 token，小请求抹零；除 `Phanthy-fast` 外没有免费模型；价格未知的模型不列。
- 从本机导入的 access token 有效期约 30 分钟，过期后改用浏览器 OAuth。
- 设备注册是写操作；服务端返回 409 表示账号已绑在别处，插件不再重试。
- 账号封禁是服务端行为，插件无法解除；请求头对齐只能降低被误判的概率。

## 文档

| 文件 | 内容 |
|---|---|
| [PROTOCOL-INFERENCE.md](./PROTOCOL-INFERENCE.md) | 推理协议与请求头 |
| [HOOKS.md](./HOOKS.md) | magpie 插件契约 |
| [AGENTS.md](./AGENTS.md) | 开发与排障入口 |
| [CHANGELOG.md](./CHANGELOG.md) | 更新记录 |

## 许可

MIT。见 [LICENSE](./LICENSE) 与 [NOTICE.md](./NOTICE.md)。
