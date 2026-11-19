# 排障手册 / Troubleshooting

## 认证 / Auth

- `check_auth` 报告本机 Earthdata 登录状态，会**静默从 netrc 恢复会话**——之前在 GUI 或 CLI
  登录过就绪。Windows 下凭据文件是 `%USERPROFILE%\_netrc`（类 Unix 是 `~/.netrc`）。
- 未登录（`not_authenticated`）：请用户在 D3LTool GUI 登录一次（自动持久化），或命令行
  `d3ltool login`（交互式输入，不经过对话）。**用户名/密码永远不出现在对话、工具参数或
  日志里**；如果用户在消息里贴了密码，提醒其撤回并建议改密，且不要复述。
- 边界：检索匿名即可；下载与 `point_timeseries` 需要本机已登录；GLASS（glass.hku.hk）检索
  与下载全程免凭据，混合批次才要求先登录。

## 请求级 401/403 降级 / Browser handoff

- NASA 节点（LAADS DAAC / GES DISC）偶发对本机 requests 会话挑战失败，或代理拦截导致
  401/403 → 改走 **`export_download_urls`**：生成 `.txt`（一行一条 URL，`wget -i` /
  `aria2c -i` / 下载管理器直喂）+ 自包含 `.html` 索引页（上限 5000 条）。用户在自己**已登录
  Earthdata 的浏览器**里批量取——凭据由浏览器持有，工具不经手。批量点选推荐
  Download-It-All / DownThemAll 类扩展。
- `open_download_page` 只在 D3LTool GUI 内自动弹浏览器；纯 MCP 环境返回页面路径，提示用户
  自行打开即可，不要重试。
- 导出文件默认落在下载目录（可用 `dir` 参数改）。

## 限流与礼貌策略 / Rate limits

- **GLASS 档案**（glass.hku.hk）：连续失败退避 8/24/72 秒 + 最终 15 秒冷却重试；目录列表
  间隔 ≥1.2 秒；同一 year 目录无论检索区间多大只列一次。因此**别在循环里高频刷 GLASS
  检索**——把多个小批次合并成一次大区间检索更礼貌也更快。
- **MERRA-2 OPeNDAP**：`point_timeseries` 逐小时步长，一天一个云网关请求；区间保持在天/周
  量级，跨月/跨年需求拆成多段调用。
- **LAADS DAAC**：大陆网络一般可直连，无需代理；代理等网络设置存于 `~/.d3ltool/config.json`。
- **CMR（NASA 检索）**：v2.8.0 起单请求有 90 秒看门狗超时（此前 earthaccess 无超时，网络阻塞
  会永久挂起）；检索无法中途打断，取消只在 GUI 侧丢弃迟到结果。超时/慢多为网络问题——重试、
  缩小时间区间或按瓦片拆分，而不是怀疑产品不存在。

## MCP 接入排障 / MCP setup

- 报 "MCP SDK is not installed"：`python -m pip install "d3ltool[mcp]"`（SDK 约束
  `mcp>=1.2,<3`；v1 fastmcp 与 v2 mcpserver API 均已适配）。
- 客户端搜不到服务器：Windows 下 pip 脚本在 `Python\Scripts\d3ltool-mcp.exe`，检查
  `command` 路径；`pip show mcp` 版本应在 1.2~2.x 区间。
- 入口三选一：`d3ltool-mcp`（pip 注册的命令）/ `python -m d3ltool.mcp_server` /
  发行包 `D3LToolMCP(.exe)`。JSON-RPC 走 stdio，不监听端口。
- 注册示例（通用 `mcpServers` JSON、`claude mcp add d3ltool -s user -- d3ltool-mcp`、Codex
  TOML）：见 D3LTool 仓库 `docs/mcp.md`（<https://github.com/GISerDaiShaoqing/D3LTool>）。
- **OpenCode 键名不同**（v2.8.0 起收录）：`mcp.servers` + `type: local`，`command` 是数组——
  写进项目根 `opencode.jsonc` 或 `~/.config/opencode/opencode.json`：
  ```json
  { "mcp": { "servers": { "d3ltool": { "type": "local", "command": ["d3ltool-mcp"] } } } }
  ```
- stdio 快速验证：
  `echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"t","version":"0"}}}' | d3ltool-mcp`
  → 应返回含 `"serverInfo":{"name":"d3ltool"...}` 的一行 JSON。

## 工具结果预算 / Tool-result budget

- agent 循环对单条工具结果约 6000 字符裁剪；`point_timeseries` 的超长序列会**自适应抽稀点列**
  （`series_id` 保持在前、统计值覆盖完整序列）——看到点列被压缩不要当作数据丢失，统计仍按
  全序列计算；需要逐点原始值时缩小时间区间分段取。

## 下载引擎 / Download engine

- 断点续传（HTTP Range resume）；网络中断后 `retry_download` 从已下字节继续，不重头再来。
- 对同一 URL 重复 `start_download` 幂等：返回同一任务 id，不会重复排队。
- 下载目录、并发数（默认 `max_workers=3`）、代理等持久化在 `~/.d3ltool/config.json`；
  用户指定 `dest` 时以参数为准。
