<p align="center">
  <img src="docs/banner.svg" alt="dsh-purge" width="640">
</p>

# dsh-purge（增强版）

DeepSeek Harness 破甲插件。改写本机已安装的 `@deepseek-ai/*` 包里的默认身份、权限策略与工具上限，并在设置页提供「规则设定」：分组查看补丁、应用 / 还原、维护多套 `AGENTS.md` / `CLAUDE.md` 规则集。启动时会自动检查并重新应用，npm 覆盖 `node_modules` 后不必手工重跑。

对准 **dsh 0.1.2-rc.1**，Node.js ≥18。其它版本原文对不上时显示待应用 / 跳过，不会乱改文件。

本仓库是增强分支，相比上游 `v1.3.5`：

- 新增 **patch #38** — TUN fake-ip 动态放行。Clash / mihomo 以 fake-ip 接管 DNS 时，上游会把每个域名判成 SSRF 目标（`resolves to a non-public IP address`）；本补丁按哨兵探测放行池内地址，TUN 关闭后自动恢复拒绝，不写死白名单。
- 修复 patch #22 的 `timeoutMs` 锚点——此前每次 apply 都会再追加一个 `0`。
- 修复 patch #38 锚点，保证 helper 只注入一次。
- 兼容 Node.js 24：`fsExists()` 增加类型守卫，消除 `fs.existsSync` 弃用警告与空行输出。
- 补全 persona 的 `prefix` / `personaPrefix` 键名与 `>-` / `|-` 块标量写法，并新增 `web-fetch-http` 路径映射。

完整改动见 [CHANGELOG.md](CHANGELOG.md)。

## 安装

装完必须**完全退出并重启** DeepSeek Harness，设置页才会出现「规则设定」；随后点「应用」才会真正改动 `@deepseek-ai` 包。

```sh
# Web profile
dsh plugin --profile web add github:mortyandtimo/dsh-purge-plus

# 桌面 profile
dsh plugin --profile default add github:mortyandtimo/dsh-purge-plus
```

也可以从本地目录安装：

```sh
git clone https://github.com/mortyandtimo/dsh-purge-plus.git
cd dsh-purge-plus
dsh plugin --profile web add .
```

或者把本 README 整段交给本机 coding agent，让它直接安装、应用补丁并在必要时重启。

### 安装后

1. 重启 dsh 进程，否则设置页与聊天命令都不会加载。
2. 应用补丁：聊天里 `/purge apply`，或设置页「规则设定 → 应用」，或 `dsh-purge --apply`。
3. 验收：`/purge status` 能打印 `DSH_HOME` 与补丁列表；设置页出现「规则设定」卡片（有缓存时 Ctrl+F5）。
4. `#20` / `#21` 依赖 `dsh-web-fetch-http`，`#28` / `#29` 依赖 `dsh-liangshen`，未安装时跳过属正常，不必重试。

## 使用

```sh
# CLI
dsh-purge --status | --apply | --revert | --edit

# 聊天
/purge status | apply | revert | edit | help
/rules list | use <id> | create <id> | delete <id> | reset

# 模型工具
purge_status   purge_apply   purge_revert
```

「应用」完成后需重启才会加载已改的包文件；点设置页的「重启」才会重启，不会自动重启。

## 还原与卸载

- 每个目标文件在应用前备份为 `<文件>.dshpurge.bak`；「还原」或 `/purge revert` 用备份覆盖回去并删除备份。重复应用是幂等的。
- `prompt-inject.md` 是用户文件，还原时保留。
- 卸载：

```sh
# 先在设置页点「还原」，再
dsh plugin --profile web remove dsh-purge
dsh plugin --profile default remove dsh-purge
```

## 说明

- 改动范围限于本机 `@deepseek-ai/*` 包内的渲染文案、默认策略与执行逻辑，以及 `$DSH_HOME` 下的覆盖文件与规则集；不修改 Harness 源码仓库。
- 不写死盘符：按 `$DSH_HOME` / `DSH_BASE` → dsh 启动器旁的 `.dsh` → 系统默认 `~/.dsh` 依次探测。找不到目标时会提示设置 `DSH_BASE`，不改文件。
- 第三方插件不在改动范围内（Windows 下 CMD 无感属运行时补丁，不写它们的源仓库）。
- 升级后原文对不上会报 `pattern_not_found` 或显示待应用，不会乱改。
- 许可证：MIT，见 [LICENSE](LICENSE)。

## 本地校验

```sh
node --check lib/index.js
node --check lib/core.js
node --check client.js
```
