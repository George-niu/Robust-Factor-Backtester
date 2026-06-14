# Robust-Factor-Backtester
**基于动态大盘止损与真实摩擦建模的端到端量化回测引擎**

![Python Version](https://img.shields.io/badge/Python-3.8%2B-blue)
![Pandas](https://img.shields.io/badge/Pandas-Data%20Engineering-green)
![Status](https://img.shields.io/badge/Status-Completed-success)

## 📌 项目背景与核心痛点解决
在传统的量化因子研究中，“纸上富贵（Paper Profit）”是导致实盘破产的最大诱因。本项目旨在解决传统动量策略（Momentum Factor）在回测中常见的三个致命缺陷：
1. **未来函数作弊（Look-ahead Bias）：** 信号生成与交易执行时间错位。
2. **高换手率摩擦（Friction Costs）：** 忽略滑点与佣金导致净值虚高。
3. **系统性崩盘（Systemic Risk）：** 裸奔策略在泥沙俱下的熊市中回撤过深。

本项目从零搭建了一个纯 Python (Pandas-based) 的向量化回测框架，严格处理了上述痛点，并创新性地引入了“大盘绝对动量红绿灯”机制，成功将策略最大回撤控制在 30% 以内。

---

## 🚀 核心特性 (Key Features)

* **严格的时间序列对齐（防未来函数）：** 基于 Pandas `.shift(1)` 机制重构底层交易逻辑。确保 $T$ 日收盘后的计算信号，仅能在 $T+1$ 日产生真实仓位，从工程底层杜绝了偷看未来数据的可能。
* **真实交易成本建模（Turnover Cost Modeling）：** 通过一阶差分 `.diff().abs()` 精准追踪每日仓位变动（Turnover），并基于双边 2‰ 的费率模型强制扣除摩擦成本，剥离“伪 Alpha”收益。
* **大盘择时防抱死系统（Dynamic Market Hedging）：** 引入指数级绝对动量过滤（20-day Market Trend）。当系统性风险来临时（大盘跌破动量均线），强制覆写个股信号，执行无条件清仓（Cash 仓位），彻底切断极值亏损。
* **专业级绩效归因（Performance Attribution）：** 内置夏普比率（Sharpe Ratio）、最大回撤（Maximum Drawdown）、等权基准对比（Equal-weight Benchmark）等机构级风控指标计算模块。

---

## 📊 核心数据表现 (Backtest Results)
*测试区间：2020-01-01 至 2023-12-31 | 标的池：美股科技/核心龙头资产*

| 指标名称 (Metric) | 等权大盘基准 (Benchmark) | 裸奔动量策略 (Gross Strategy) | **本引擎优化策略 (Net + Hedged)** |
| :--- | :--- | :--- | :--- |
| **累计净值 (Total Return)** | 2.72x | 4.23x (虚假繁荣) | **3.68x** |
| **最大回撤 (Max Drawdown)** | -35.21% | -35.57% | **-27.86%** (极高安全性) |
| **年化夏普比率 (Sharpe)** | 1.10 | 1.25 | **1.27** (最优风险收益比) |

> **💡 数据洞察：** > 如图表所示（见项目中生成的资金曲线图），在 2022 年全市场大暴跌期间，本引擎的“防抱死系统”成功触发。净值曲线呈现出完美的水平防守形态（平仓保本），成功规避了长达数月的系统性大跌。

---

## 🛠️ 技术栈与架构 (Tech Stack)

* **数据工程：** `yfinance`, `Pandas`, `NumPy` (缺失值 ffill 清洗, 向量化滚动计算)
* **策略回测：** 基于 Pandas DataFrame 的纯向量化截面分析（Cross-sectional ranking）
* **数据可视化：** `Matplotlib` (多图层叠加, 资金曲线对比渲染)

**核心代码架构工作流：**
`Data Ingestion` ➡️ `Momentum Scoring (.pct_change)` ➡️ `Cross-sectional Ranking` ➡️ `Market Trend Filter` ➡️ `Signal Shift (T+1)` ➡️ `Friction Deduction` ➡️ `Cumprod Evaluation`

---

## 🔮 未来迭代路线图 (Roadmap)

本项目目前作为 Alpha 研究的原型引擎，已具备完整闭环逻辑。未来计划向全市场工业级标准进行升维：
- [ ] **全市场扩容：** 接入全市场 4000+ 股票池，测试资金容量上限与流动性约束。
- [ ] **多因子融合与机器学习：** 引入 XGBoost / LightGBM，将单一价格动量扩展为高维非线性特征组合预测。
- [ ] **接入事件驱动框架：** 将核心逻辑重构并迁移至 `Backtrader` 或 `Qlib`，支持更精细的盘口挂单模拟与订单撮合。

---
*声明：本项目的回测结果仅作学术与工程展示，不构成任何真实世界的投资建议。*
