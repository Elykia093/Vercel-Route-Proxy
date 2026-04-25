# Vercel-Route-Proxy

把目标 URL 编进路径里，交给 Vercel 的路由规则转发。

这个项目没有后端函数、没有 Node.js 运行时，也没有额外依赖。仓库里真正承担转发逻辑的只有 [`vercel.json`](./vercel.json)，[`index.html`](./index.html) 只是一个方便手动生成代理地址的静态页面。

## 这个项目适合拿来做什么

- 给接口、页面、静态资源做一个临时转发入口
- 在不写服务端代码的前提下验证 Vercel 路由转发能力
- 自建一个轻量的“URL -> 代理地址”转换工具页
- 作为个人使用或小范围内部使用的代理壳

## 不适合什么场景

- 面向公网开放、无人值守的通用代理服务
- 需要鉴权、限流、审计、缓存策略的正式网关
- 对稳定性、可观测性、容灾有明确 SLA 的生产入口
- 需要对请求头、Cookie、响应内容做深度改写的场景

## 核心机制

Vercel 读取如下规则后，会把请求路径中的协议、主机和剩余路径重新拼成目标地址：

```json
{
  "routes": [
    {
      "src": "/(?<protocol>[a-z][a-z0-9+.-]*)/(?<host>[^/]+)(?:/(?<path>.*))?",
      "dest": "$protocol://$host/$path"
    }
  ]
}
```

也就是把：

```text
/https/api.github.com/repos/vercel/vercel
```

转成：

```text
https://api.github.com/repos/vercel/vercel
```

## 地址格式

代理地址格式：

```text
https://你的域名/<协议>/<主机>/<路径>?<query>
```

示例：

| 目标地址 | 代理地址 |
| --- | --- |
| `https://example.com` | `/https/example.com` |
| `https://example.com/docs/getting-started` | `/https/example.com/docs/getting-started` |
| `https://example.com:8443/api/users?id=1` | `/https/example.com:8443/api/users?id=1` |
| `http://httpbin.org/get?from=vercel` | `/http/httpbin.org/get?from=vercel` |

## 首页怎么用

部署完成后直接访问站点根路径：

1. 输入完整目标 URL，也可以只输入域名，页面会默认补成 `https://`。
2. 点击“立即代理”。
3. 页面会在新标签页打开生成后的代理地址。

如果目标 URL 里带查询参数，页面会一并保留。

## 部署

### 方式一：导入到 Vercel

1. Fork 这个仓库。
2. 在 Vercel 中导入该仓库。
3. 保持默认配置完成部署。
4. 访问分配到的域名。

### 方式二：本地改完后推送

这个项目是纯静态文件 + `vercel.json` 配置，只要仓库连接到 Vercel，推送后就会自动重新部署。

## 本地文件说明

```text
.
├── index.html    # 静态输入页，用来生成代理地址
├── vercel.json   # 路由转发规则
├── README.md
└── LICENSE
```

## 使用边界与注意事项

- 公开部署有被滥用的风险，至少应配合访问控制或只在私有环境使用。
- 目标站点是否允许被转发，取决于目标站点自身策略以及 Vercel 的网络行为。
- 这个项目的能力上限就是 Vercel 路由规则本身，不包含自定义业务逻辑。
- 页面里的 `#fragment` 属于浏览器侧片段标识，不参与服务端请求，但会保留在最终打开的 URL 中。

## 二次修改方向

如果你准备继续扩展，这个仓库比较自然的演进方向有：

- 增加访问密码或白名单，限制滥用
- 为常用目标生成预设快捷入口
- 给首页增加复制链接、历史记录、最近访问
- 按自己的域名策略补充更多路由规则

## 许可证

[AGPL-3.0](./LICENSE)
