# Changelog

本仓库基于 `dsh-purge` v1.3.5（MIT）。以下只记录本分支相对该版本的增量改动。

## [1.3.6] — Enhanced fork

### 新增

- **patch #38 `WEB_FETCH_FAKE_IP_TUNNEL`（TUN fake-ip 动态放行）**
  解决 Clash / mihomo 等以 fake-ip 模式接管系统 DNS 时，`dsh-web-fetch-http` 把每个域名都判成 SSRF 目标的问题（表现为 `URL hostname "..." resolves to a non-public IP address` / `WEB_BLOCKED_URL`）。
  上游只接受公网 unicast 地址，而 TUN 场景下解析结果落在 fake-ip 池内（`198.18.0.0/15`、`fdfe:dcba:9876::/48`），因此所有域名都被拒绝。
  本补丁**不写死白名单**，改为按需探测：仅当哨兵域名（`example.com` / `www.cloudflare.com` / `one.one.one.one`）此刻仍解析回 fake-ip 池内地址时，才放行池内地址；TUN 关闭后探测失败，立即恢复上游的拒绝行为。私网、环回、链路本地地址一概照旧拒绝。
  探测结果带 60 秒缓存，避免每个请求都做一次 DNS 解析。

### 修复

- **patch #22 幂等锚点缺失（`timeoutMs` 尾部换行）**
  `timeoutMs: 60000` 会命中 `timeoutMs: 600000` 的前缀，导致每次启动 apply 都再追加一个 `0`（曾把该值推到 `6e12`）。现在把行尾换行纳入锚点，marker 同步收紧为 `timeoutMs: 600000\n`，保证幂等。

- **patch #38 注入锚点缺失**
  只锚常量行会导致注入后该行仍在，每次 apply 再插一份 helper。现在锚点连上紧随其后的 JSDoc 起始行（`const IPV4ONLY_SENTINELS = ...` + `/**`），该组合在文件内唯一。

- **Node.js 24 兼容：`fs.existsSync` 类型守卫**
  上游 `state.files` 混合单路径与 `string[]` 列表（缺失时为 `[]`），Node 24 会对非路径参数抛出
  `Passing invalid argument types to fs.existsSync is deprecated`。
  现在在 `fsExists()` 内统一做类型把关，并新增 `shownPath()` 渲染，避免空列表打印空行。

### 增强

- **persona prefix 变体覆盖**
  上游补丁只匹配 `persona:` 键，遇到新版的 `prefix:` / `personaPrefix:` 变体时无法命中（表现为 Harness 默认身份残留）。
  现在三种键名均已覆盖，含 `>-` 与 `|-` 两种 YAML 块标量写法，以及 `prefix: You are a helpful software engineer assistant.` 的 minimal / sdk-minimal preset 变体。
  同时为对应 patch 增加 `markersAny: true`，任意一个 marker 命中即视为已应用。

- **`web-fetch-http` 文件路径映射**
  补充 `dsh-web-fetch-http/lib/index.js` 与源码树 `packages/web/web-fetch-http/lib/index.js` 两条解析路径，patch #38 在 Web 与源码启动两种形态下都能定位目标文件。

### 说明

- 本分支在功能上是上游的超集，安装方式与配置面完全一致。
- 上游引入的新特性在本分支合并前不会出现；如需上游最新能力，请切换回上游仓库。
