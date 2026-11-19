---
name: d3ltool
description: 编排 d3ltool MCP 工具检索与下载 NASA MODIS / VIIRS / MERRA-2 / GLASS 数据、采样点位时间序列。Orchestrate the d3ltool MCP tools (mcp__d3ltool__*) to search & download NASA MODIS/VIIRS/MERRA-2/GLASS data or sample point time series. Use whenever the user mentions NASA / Earthdata / 卫星数据 / 遥感数据下载, MODIS, VIIRS, MERRA-2, GLASS, 夜光 nighttime lights, NDVI, EVI, LAI, LST, GPP, albedo, 积雪, 土地覆盖, 火烧迹地, or asks for a point time series / trend / plot at a location — even if they never say "d3ltool".
---

# d3ltool：NASA 数据检索与下载 · Agent 使用指南

**定位**：d3ltool 是 NASA 数据检索与下载工具（search & download 是主功能）——检索与下载
MODIS / VIIRS（LAADS DAAC）、MERRA-2（GES DISC）、GLASS（glass.hku.hk 公开档案），另提供
轻量的**点位时间序列采样与绘图**（OPeNDAP 直采像素，辅助功能）。不要把它表述为"数据分析"
工具。项目：<https://github.com/GISerDaiShaoqing/D3LTool>（完整工具表见仓库内 `docs/mcp.md`）。

本文对齐 d3ltool v2.8.0（13 个 MCP 工具）。工具集与产品目录会随版本增长——动手实测，
不要按本文数字下结论。

## 第一步：判断需求类型 / Route first

| 用户需求 | 走哪条流程 |
| --- | --- |
| 要**文件**（granule、影像、nc/hdf） | 流程 A |
| 要**点位时间序列 / 趋势 / 绘图**（"这个点这些年的 NDVI/温度/夜光变化"） | 流程 B |
| 下载 401/403 失败，或想把批量下载交给用户浏览器 | 流程 C |

### 流程 A：文件检索与下载

1. **`check_auth`** —— 先确认本机 Earthdata 登录状态。检索匿名即可，**下载**需要本机已
   持久化登录（`~/.netrc`，Windows `_netrc`）。未登录 → 请用户在 D3LTool GUI 登录一次或跑
   `d3ltool login`；密码永远不出现在对话或工具参数里。
2. **定位产品** —— 产品名不确定：`list_products`（`group` 子串筛选，如 `group='GLASS'`；
   单次输出上限 80 行，超出附 hint，按 hint 再筛）。已知经纬度不知瓦片：`point_to_tile` 得
   (h, v)；反查瓦片范围用 `tile_bounds`。产品速查见 `references/products.md`。
3. **`search_granules`** —— `short_name` + `start`/`end`（YYYY-MM-DD）+ `tiles`（如
   `h04v03`）或 `bbox`（west,south,east,north）。**默认 `limit=10`**，用户要"全部/全年"时调大。
   返回文件名/日期/大小/URL。NASA 产品走 CMR；GLASS 变体 id（如 `LAI_MODIS_250M`）自动改走
   公开档案，免登录。
4. **`start_download`** —— 收 `search_granules` 返回的 URL 或 granule 行，立即返回任务 id
   （断点续传；重复提交同一 URL 幂等）。`dest` 省略则进已配置下载目录（默认 `~/Downloads`）。
   批量前先向用户确认数量与总体积。
5. **轮询 `download_status`** 直到全部 done。失败任务 `retry_download`（Range 续传）；
   排错的 `cancel_download`。

### 流程 B：点位时间序列

- **直接 `point_timeseries`**，绝不为"要一条序列"去 search + download 整景文件——那是单像素
  OPeNDAP 直采，一次调用返回点列 + 全序列统计 + `series_id`。
- `product` 必须是 CMR 真实 short_name（NDVI/EVI 用 `MOD13Q1`/`MYD13Q1`）；`layer` 可用简称
  （`'NDVI'` 命中 `_250m_16_days_NDVI`）或留空自动选层；简称需 ≥3 字符，过短会被拒绝并列出
  可用图层。
- 数值为原生单位，结果自带 `units`（MERRA-2 T2M 是开尔文，°C = K − 273.15）。
- **MERRA-2 逐小时**：一天一个 OPeNDAP 网关请求，区间保持在天/周量级，长区间拆段。
- 仅覆盖 MODIS/VIIRS/MERRA-2；GLASS 不支持 `point_timeseries`。
- 纯 MCP 环境：把点列自己画成图/表。`plot_timeseries` 只在 D3LTool 桌面软件内可用，纯 MCP
  调用返回 `not_in_gui`——不要反复重试。

### 流程 C：导出清单 / 浏览器辅助下载

- 请求级 401/403（本机凭据对节点挑战失败、代理拦截）→ **`export_download_urls`**：写 `.txt`
  一行一条（`wget -i` / `aria2c -i` 直喂）+ 自包含 `.html` 索引页（上限 5000 条）。让用户在
  已登录 Earthdata 的浏览器里批量取（Download-It-All / DownThemAll 类扩展）——凭据留在
  浏览器，工具不经手。
- `open_download_page` 仅 GUI 内自动弹浏览器；纯 MCP 返回页面路径，让用户自己打开。
- `urls` 可直接收 `search_granules` 返回的 granule 行，不必手工转 URL。

## 硬约束 / Hard rules

- **密码不经手**：Earthdata 用户名/密码永远不出现在对话、工具参数或日志里；凭据只来自本机
  持久化登录（GUI 登录一次写 netrc，或 CLI `d3ltool login`）。
- **auth 边界**：检索匿名；下载与 `point_timeseries` 需本机已登录；GLASS 检索与下载全程免凭据，
  混合批次才要求先登录。
- **口径**：对外表述一律"data search & download"；时间序列只称"点位时间序列采样与绘图"；
  不替用户下大规模定量结论。
- 工具数、产品数是 live number（当前 13 个工具、GLASS 133 个变体 / 131 已核验）；引用前实测，
  别把本文数字写进对外文案。

## 高频陷阱 / Gotchas

- `search_granules` 默认 `limit=10` —— 要"全年/全部"时调大。
- GLASS **静态集**（AGB、Crops 等，描述含"无时间维（固定层）"）免日期；**全球 CMG 变体**
  （文件名无 `h##v##`，如 `*_0_05D`）`tiles`/`bbox` 无效——别传了参数还疑惑没过滤。
- GLASS 目录列表有礼貌策略：连续检索退避 8/24/72 秒 + 最终 15 秒冷却，列表间隔 ≥1.2 秒；
  同一 year 目录无论跨多大区间只列一次。别在循环里高频刷 GLASS 检索，合并成一次大区间。
- HiGLASS 20m 变体（`LAI_HR`/`FAPAR_HR`）按 ~1120 个 1° 带目录检索：MCP 侧传 `bbox` 或
  `tiles`（h##v##，自动换算地理范围求交）；**带名（`n045e120`）`tiles` 也直接接受**
  （v2.8.0 与 R 包对齐：显式带名与 bbox 求交取并集，只填带名时无需 bbox）。不带
  bbox/瓦片/带名会报错要求先定位。
- `plot_timeseries` 纯 MCP 返回 `not_in_gui`，不要重试。
- 下载默认进配置目录（`~/.d3ltool/config.json`，默认 `~/Downloads`）；用户指定路径就传 `dest`。

## MCP 不可用时 / Fallback

`mcp__d3ltool__*` 不在本会话时，同一核心走 CLI：

```bash
d3ltool login        # 交互式 Earthdata 登录，持久化 ~/.netrc（Windows _netrc）
d3ltool search MOD13Q1 --start 2020-01-01 --end 2020-12-31 --tile h25v03 --limit 200
d3ltool download MOD13Q1 --start 2020-01-01 --end 2020-12-31 --tile h25v03 --dest D:/data
d3ltool gui          # 桌面软件（search/download 参数两个子命令通用；--tile 可重复传）
```

MCP 服务器注册（Claude Desktop / Cursor / ZCode 等通用）：

```json
{ "mcpServers": { "d3ltool": { "command": "d3ltool-mcp" } } }
```

stdio 验证：

```bash
echo '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"t","version":"0"}}}' | d3ltool-mcp
# 应返回一行含 "serverInfo":{"name":"d3ltool"...} 的 JSON
```

## 深入阅读 / references

- 产品 short_name 速查、GLASS 变体规则 → `references/products.md`
- auth 排障、401 降级细节、限流参数、MCP 接入问题 → `references/troubleshooting.md`
