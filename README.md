# fuzhanggui · 富掌柜演示收银（看板 + 前台点餐 + 手机 APP）

富掌柜风格的餐饮交易演示：**电脑端看板**、**前台收银点餐**、**手机端 APP** 三块页面，加上一个本地收银服务，让三端共享同一份订单数据。

## 页面

| 页面 | 地址 | 用途 |
| --- | --- | --- |
| 交易看板 | `index.html`（站点根路径） | 登录 → 销售总览 / 订单列表 / 销售账单 / 商品分析 / 数据说明 |
| 前台收银点餐 | `order-pad.html` | 桌面端收银台，选菜、开单、撤销、导出；开单即写收银服务 |
| 手机端 APP | `app.html` | 手机上"添加到主屏幕"后可全屏运行（PWA），含 点餐 / 订单 / 汇总 三页 |

历史数据口径：4,768 笔订单、32,946 行菜品明细、114 个 SKU，区间 2026-03-18 ~ 2026-11-24，已固化进 `index.html`（图表库 ECharts 走 jsDelivr CDN）。

## 三端数据怎么流通

页面按 **收银服务 → 云端 `data/orders.json` → 本机 localStorage** 的顺序自动选数据源，顶部/侧栏会显示当前来源（收银服务 / 云端 / 本机）。

```
前台收银(order-pad.html) ─┐
                          ├─► 本机收银服务 sync_server.py ──► GitHub: data/orders.json ──► Pages 手机/其他设备
手机 APP(app.html) ───────┘            (实时，局域网)              (推送，需 token)
```

- **同一 WiFi 演示（推荐）**：手机直接访问 `http://<电脑内网IP>:18080/app.html`，与电脑共用收银服务，开单即时双向可见，不需要 token。
- **纯 GitHub Pages 演示**：手机打开 Pages 的 `app.html`，在右上角 ⚙ 里填 GitHub 用户名、仓库、分支、路径和 Personal Access Token，点「同步」把订单写进 `data/orders.json`；电脑端刷新看板即可看到（Pages 构建约 30~60 秒）。

## 本机收银服务

需要 Python 3（纯标准库，不装任何依赖）：

```bash
python sync_server.py
```

启动后会打印三个地址：

```
PC 看板  : http://127.0.0.1:18080/
前台点餐 : http://127.0.0.1:18080/order-pad.html
手机 APP : http://192.168.x.x:18080/app.html
```

配置文件（与 `sync_server.py` 同目录）`sync_config.json`：

```json
{
  "owner": "heiyelidebaiyang",
  "repo": "fuzhanggui",
  "branch": "main",
  "path": "data/orders.json",
  "port": 18080,
  "token": "ghp_xxx（可选，填了就会自动推送 GitHub）"
}
```

也可以用环境变量 `GITHUB_TOKEN`。仓库、分支、路径、端口都能改；`port` 想换的话同步改配置即可。

### 接口

| 接口 | 方法 | 说明 |
| --- | --- | --- |
| `/api/orders` | GET | 取当前全部订单 |
| `/api/orders` | POST | 新增/覆盖一笔订单，body `{"order":{...}}`；有 token 时自动推送 GitHub |
| `/api/orders?no=xxx` | DELETE | 撤销某笔订单，并同步云端 |
| `/api/sync` | POST | 手动推送当前全部订单到 GitHub |
| `/api/health` | GET | 服务状态、订单数、token 是否配置、最后同步时间 |

## 演示流程

1. 电脑启动 `sync_server.py`，打开看板登录（演示账号预填，密码任意 6 位，验证码按图片输入）。
2. 电脑打开"点餐开单"或手机打开 APP，选菜 → 开单（桌台、人数）。
3. 手机 APP 的「订单」「汇总」立即出现这笔；电脑看板点右侧「刷新数据」或在登录时自动拉取，数据截止时间与营业额同步更新。
4. 点右上「同步云端」把订单推到 GitHub，Pages 上的 `app.html` 也能读到（离线演示除外）。

## 声明

纯演示与分析用途，页面中的商户号、终端号、门店名仅用于界面还原，与真实交易系统无关；订单数据为本地模拟库。
