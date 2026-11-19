# 服务器优选工具

一个简化版的Cloudflare Workers优选工具，用来生成订阅链接。

GitHub: https://github.com/byJoey/yx-auto

## 主要功能

- 优选域名：内置了一些常用的优选域名
- 优选IP：从动态 IP 接口获取，支持IPv4/IPv6
- GitHub优选：内置多个 GitHub 定时更新源，也可以自定义仓库地址
- 多协议支持：VLESS、Trojan、VMess
- 多客户端格式：Clash、Surge、Quantumult X等
- 运营商筛选：可以按移动/联通/电信筛选

## 默认优选IP来源

`piu` 参数留空时，会并行拉取以下 GitHub 定时更新源，结果缓存 10 分钟，并在生成节点时去重、限制每个源的条数：

| 来源 | 更新频率 | 说明 |
| --- | --- | --- |
| [ahang39/router](https://github.com/ahang39/router) | 每日 | 上海电信实测优选 |
| [yuanxiawan/cfipv4db](https://github.com/yuanxiawan/cfipv4db) | 约 30 分钟 | 移动/联通/电信三网，按时延排序，受运营商筛选控制 |
| [ymyuuu/IPDB](https://github.com/ymyuuu/IPDB) | 每日多次 | 反代优选 IP 库 |
| [joname1/BestCFip](https://github.com/joname1/BestCFip) | 每日多次 | 自动采集优选 IP |
| [qwer-search/bestip](https://github.com/qwer-search/bestip) | 不定期 | 原有的默认列表 |

传入 `piu` 后只使用自定义来源（支持单个 URL 或多行文本混合）。

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
- `epi` - 启用优选IP（默认yes）
- `egi` - 启用GitHub优选（默认yes）
- `piu` - 自定义IP来源URL（可选，留空使用内置的多个定时更新源）
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
