# 服务器优选工具

一个简化版的Cloudflare Workers优选工具，用来生成订阅链接。

GitHub: https://github.com/byJoey/yx-auto

## 主要功能

- 优选域名：内置了一些常用的优选域名
- 优选IP：内置多个 GitHub 定时源，支持IPv4/IPv6
- GitHub优选：可自定义一个或多个来源 URL，与内置源合并去重
- 多协议支持：VLESS、Trojan、VMess
- 多客户端格式：Clash、Surge、Quantumult X等
- 运营商筛选：可以按移动/联通/电信筛选

## 默认优选IP来源

「启用优选IP」开启时会并行拉取以下来源（`egi=yes` 且 `piu` 留空时也会兜底使用），结果缓存 10 分钟，每个动态源 raw 优先、超时后自动改用 jsDelivr 镜像，并在生成节点时去重、限制每个源的条数：

| 来源 | 更新频率 | 说明 |
| --- | --- | --- |
| [yuanxiawan/cfipv4db](https://github.com/yuanxiawan/cfipv4db) | 约 30 分钟 | 移动/联通/电信三网，按时延排序，受运营商筛选控制 |
| [LancelotRar/best-cf-ips](https://github.com/LancelotRar/best-cf-ips) | 每日扫描 | 实测高速扫描结果 top50，多数条目可跑到 78–127 Mbps |
| 实测高速（内置静态） | 随版本更新 | 2026-11-25 用 10MB 单线程实测的 IPv4 快照，均 ≥78 Mbps |
| 实测高速v6（内置静态） | 随版本更新 | 同日实测的 IPv6 快照，均 ≥116 Mbps |

> 2026-11-25 用 10MB 单线程复测后调整了源列表：短距离的 512KB 测速会受 TCP 慢启动影响严重低估实际带宽。上海电信、IPDB（v4/v6）、BestCFip（v4/v6）的条目实测普遍达不到 50 Mbps，已移除；BestCFip IPv4 在更早一轮已移除。高速条目集中在少数网段，保留的静态列表用于在动态源失效时兜底，若后续实测变慢可直接删除或更新。

传入 `piu` 后使用自定义来源，支持多个 URL（每行一个或英文逗号分隔）与直接粘贴的 `IP:端口#备注`；所有来源合并后按 IP+端口去重，重名节点自动加序号。`egi=no` 时不添加自定义来源。

内置优选域名已清理为 8 个实测仍解析到 Cloudflare 的域名。

节点名统一为「来源/运营商 + 两位序号」，例如 `上海电信01`、`联通03`、`IPDB05`，不再携带源列表里的速度、延迟等原始备注。

## 部署

1. 去Cloudflare Dashboard，找到Workers & Pages
2. 创建个新Worker
3. 把`_worker.js`的代码复制进去
4. 保存部署就完事了

## 使用

打开你的Worker地址，会看到一个界面：

1. 填域名和UUID
2. 选协议（VLESS/Trojan/VMess）
3. 选客户端类型
4. 点按钮生成订阅链接

就这么简单。

## 订阅链接格式

生成的链接大概长这样：
```
https://your-worker.workers.dev/{UUID}/sub?domain=your-domain.com&epd=yes&epi=yes&egi=yes
```

可以通过`&target=`参数指定输出格式：
- `base64` - 默认格式
- `clash` - Clash配置
- `surge` - Surge配置  
- `quantumult` - Quantumult配置

## URL参数

所有配置都通过URL参数控制，不需要环境变量。

主要参数：
- `domain` - 你的域名（必填）
- `epd` - 启用优选域名（默认yes）
- `epi` - 启用优选IP（内置多源，默认yes）
- `egi` - 启用GitHub优选/自定义来源（默认yes）
- `piu` - 自定义来源（可选，支持多个 URL，每行一个或英文逗号分隔；需要 `egi=yes`）
- `ev` - 启用VLESS（默认yes）
- `et` - 启用Trojan（默认no）
- `mess` - 启用VMess（默认no，注意不是vm，会被屏蔽）
- `ipv4/ipv6` - IP版本选择（默认都开启）
- `ispMobile/ispUnicom/ispTelecom` - 运营商筛选（默认都开启）
- `target` - 输出格式（base64/clash/surge/quantumult）

## 注意事项

- 这只是一个订阅生成工具，不提供代理服务
- 生成的节点需要配合你自己的服务器使用
- UUID必须是标准格式（xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx）
- VMess参数用的是`mess`不是`vm`，因为`vm`会被某些地方屏蔽
