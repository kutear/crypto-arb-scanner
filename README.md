# Crypto Arbitrage Scanner (Zero-Token Programmatic Inspection)

全市场跨所加密货币量化套利机会监控系统。纯 Python 独立运行，不产生任何 LLM API Token 消耗。使用 Astral **uv** 进行包管理与极速依赖解析。

支持全市场空间价差套利（Track A）与资金费率对冲套利（Track B - 24h归一化）双轨监控，内置严苛的六道硬核风控网（Gate 1~6），发现高质量机会时自动向飞书群推送单张高信息密度的交互式汇总表格卡片。

---

## ⚡ 包管理器：Astral `uv`

本项目基于现代 Python 工具链 **`uv`**（pyproject.toml + uv.lock），实现极速依赖解析与开箱即用运行环境。

```bash
# 1. 安装与同步依赖
uv sync

# 2. 极速运行扫描器
uv run arbitrage_scanner.py --once
```

---

## 🚀 GitHub Actions 自动化配置 (每小时/次)

本项目已预配置为 **Public 仓库模式**，在 GitHub Actions 上享受 **完全免费、无分钟数配额限制** 的定时巡检服务。

工作流定义位于 `.github/workflows/arbitrage_scan.yml`：
- 采用官方 `astral-sh/setup-uv@v5`，启动与依赖准备仅需 **1 秒**；
- 调度配置：`cron: '15 * * * *'`（每小时第 15 分钟错峰自动巡检，规避整点拥堵）；
- 支持在 Actions 界面随时单次手动触发（`workflow_dispatch`）。

### 必填 Secrets 配置

由于本项目为公开（Public）仓库，代码内已**彻底剔除所有敏感凭据与硬编码 Token**。请在 GitHub 仓库主页配置以下 Secrets：

前往 GitHub 仓库：**Settings** $\to$ **Secrets and variables** $\to$ **Actions** $\to$ 点击 **New repository secret**，添加以下三个变量：

| Secret 名称 | 必须 | 说明 | 示例值 |
| :--- | :---: | :--- | :--- |
| `FEISHU_WEBHOOK_URL` | **是** | 飞书自定义机器人 Webhook 地址 | `https://open.feishu.cn/open-apis/bot/v2/hook/...` |
| `CF_CLIENT_ID` | **是** | Cloudflare Access Client ID | `<your-cf-access-client-id>` |
| `CF_CLIENT_SECRET` | **是** | Cloudflare Access Client Secret | `<your-cf-access-client-secret>` |
| `MCP_SERVER_URL` | 否 | MCP 服务端接入点（默认已内置） | `https://arb-mcp.kutear.com/mcp` |

---

## 🛠️ 本地运行与调试

### 1. 设置环境变量
```bash
export FEISHU_WEBHOOK_URL="https://open.feishu.cn/open-apis/bot/v2/hook/..."
export CF_CLIENT_ID="<your-cf-access-client-id>"
export CF_CLIENT_SECRET="<your-cf-access-client-secret>"
```

### 2. 使用 uv 运行
```bash
# 单次执行并推送汇总表格
uv run arbitrage_scanner.py --once

# Dry-run 模式（只扫描测算，不推飞书）
uv run arbitrage_scanner.py --once --dry-run
```

---

## 🛡️ 核心风控网 (Gate 1~6)

1. **Gate 1（标的一致性）**：严格校验指数成分构成，过滤跨币合成比价（带 `_`）与乘数偏差。
2. **Gate 2（退市停用过滤）**：排除即将下架、停止充提或受限标的。
3. **Gate 3（撮合价真实性）**：买卖端真实成交价（last）与实时盘口点差交叉核验，排除虚假挂单。
4. **Gate 4（历史均值回归建模）**：拉取 48h~72h 历史序列计算中位数 $P_{50}$ 与波动率 $\sigma$，排除常态化死锁高溢价。
5. **Gate 5（硬核净利红线）**：扣除双边买卖点差损耗（BidAskLoss）和 0.20% 手续费后，平仓净利 $\ge 0.50\%$；资金费率回本周期 $< 3$ 天且 9 期历史胜率 $\ge 70\%$。
6. **Gate 6（深度滑点核验）**：买卖双方前 5 档深度充足，建仓滑点 $< 0.05\%$。
