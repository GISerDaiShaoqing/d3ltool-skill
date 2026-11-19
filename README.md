# d3ltool-skill

[d3ltool](https://github.com/GISerDaiShaoqing/D3LTool) 的 Agent Skill：让 AI agent
（ZCode / Claude / Cursor 等支持 MCP 与 skill 的客户端）在用户提到 NASA 数据需求时，
自动按正确的方式编排 d3ltool MCP 工具——检索与下载 MODIS / VIIRS / MERRA-2 / GLASS 数据、
采样点位时间序列，而不是把时间序列请求拿去下载整景文件。

An agent skill that teaches AI clients to orchestrate the d3ltool MCP tools the
right way: routing (file download vs point time series vs browser handoff),
Earthdata auth boundaries, rate-limit politeness, and CLI fallbacks.

## 安装 / Install

前提：已注册 d3ltool MCP 服务器（`python -m pip install "d3ltool[mcp]"` 后，在客户端
`mcpServers` 里加 `{"d3ltool": {"command": "d3ltool-mcp"}}`，详见主仓库 `docs/mcp.md`）。

```bash
# 用户级（所有项目可用）
git clone https://github.com/GISerDaiShaoqing/d3ltool-skill.git ~/.agents/skills/d3ltool

# 或项目级（仅当前工作区可用）
git clone https://github.com/GISerDaiShaoqing/d3ltool-skill.git <your-project>/.agents/skills/d3ltool
```

## 结构 / Layout

```
SKILL.md                  路由决策树 + 硬约束 + 高频陷阱（触发时加载）
references/products.md    产品目录速查：常用 short_name、GLASS 变体规则
references/troubleshooting.md  auth 排障、401 降级、限流、MCP 接入
```

## 兼容性 / Compatibility

对齐 d3ltool v2.8.0 工具集（13 个 MCP 工具）。工具与产品数量会随版本增长，skill 正文与
目录速查均标注"以实测为准"——agent 引用数字前会用 `list_products` / MCP 工具列表实测。

## 许可 / License

MIT，作者 Dai Shaoqing（戴劭勍）。主项目：<https://github.com/GISerDaiShaoqing/D3LTool>
· 主页：<https://gisersqdai.top/D3LTool/>
