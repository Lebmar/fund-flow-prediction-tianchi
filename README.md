# Fund Flow Forecasting (Tianchi)

货币基金每日申购 / 赎回金额预测 · DSAA1001 数据科学与分析导论 · 成绩 A+（班级唯一）

> **一句话（简历用）**：面向货币基金每日申购/赎回的时序预测任务，用 SQLite 完成四张原始表的清洗入库、索引优化与查询耗时对比，通过 SQL JOIN 与聚合构建日粒度训练集；采用直接多步（1–30 天）集成预测避免递归误差累积，并用多起点滚动前向验证调参。线上评分 162.0652。

---

## 任务

天池「资金流入流出预测 - 挑战 Baseline」：基于余额宝用户的资金流水数据，预测 2014 年 9 月每一天全站的**申购总额**与**赎回总额**（共 30 天 × 2 个目标）。

## 数据

| 表 | 内容 |
|---|---|
| `user_balance_table` | 用户每日申购/赎回余额流水 |
| `user_profile_table` | 用户画像（城市、星座、性别等） |
| `mfd_day_share_interest` | 货币基金每日万份收益与利率 |
| `mfd_bank_shibor` | 银行间同业拆放利率 |
| `comp_predict_table` | 竞品收益 |

## 三个阶段

### Phase 1 · 环境与数据校验

核对所需文件是否齐全、读取各表结构与行数，确认数据可用后再进入后续阶段。

### Phase 2 · 清洗入库 + SQL 性能实证

1. **数据清洗**：字符串 `"unknown"`（忽略大小写与空格）与空串统一视为缺失；`category1`–`category4` 等分类字段按规则处理
2. **入库**：四张清洗后的表存入 SQLite 数据库 `fund.db`
3. **SQL 查询**：按日期查询、按用户查询等多条代表性查询
4. **索引优化对比**：在 `user_balance_table(report_date)`、`user_balance_table(user_id)` 等列上建索引，**对每条查询重复执行并统计最小/平均耗时，量化建索引前后的性能差异**
5. **SQL JOIN 生成训练集**：把 `user_balance_table`、`mfd_day_share_interest`、`mfd_bank_shibor` 连接成日粒度训练/测试 CSV
6. **SQL 聚合分析**：验证两个业务结论——① 不同城市的用户行为存在差异；② 申购与赎回金额随时间波动

### Phase 3 · 特征工程与预测

**特征组**（每组都说明了动机与缺失/泄露处理）：

| 特征组 | 示例 | 动机 |
|---|---|---|
| 日历周期 | weekday 的 sine/cosine 编码、月内位置、周末标记 | 申购赎回存在明显的周内与月内节律 |

**建模策略**：

- **直接多步预测（direct multi-horizon）**，一次性学习未来 1–30 天，而不是递归地用前一天的预测喂给下一天——避免 30 天horizon 上的误差累积
- 对 purchase 与 redeem **分别集成**（两者的动态特征不同）
- **滚动前向验证**：以 2014-05-01、2014-06-01、2014-07-01 等多个 origin 分别验证，而不是只做一次切分
- 最终预测与一个已验证的部署锚点做融合，按模型一致性、离散度和受保护日期调整权重
- **提交文件做字节级校验**：日期范围、形状、取值区间、分支逻辑逐一检查后再提交

**线上结果**：评分 **162.0652**（Round80）。

> 注：该赛事为长期榜，名次会随参与情况波动（notebook 中记录为 86）。**简历上建议写评分与课程成绩，不要以名次为主要卖点。**

## 项目结构

```
fund-flow-prediction-tianchi/
├── README.md
├── LICENSE
└── src/
    ├── phase1_setup.ipynb      # 环境与数据校验
    ├── phase2_project.ipynb    # 清洗 + SQLite + 索引对比 + SQL JOIN/聚合
    └── Phase3_project.ipynb    # 特征工程 + 建模 + 验证 + 提交
```

## 如何运行

```bash
pip install pandas numpy scipy matplotlib scikit-learn
```

1. 把天池原始文件夹命名为 `Purchase Redemption Data`，放在与 notebook 同级目录
2. 依次运行 `src/phase1_setup.ipynb` → `src/phase2_project.ipynb` → `src/Phase3_project.ipynb`
3. Phase 2 会生成 `fund.db`、`example_train.csv`、`example_test.csv`；Phase 3 读取它们并输出最终预测

## 技术栈

Python · pandas / numpy / matplotlib / scipy · **SQLite（建库、索引、查询优化、JOIN、聚合）** · 时序特征工程 · 直接多步集成预测

---

## English Summary

Time-series forecasting of daily purchase and redemption amounts for a money-market fund (Tianchi "Fund Flow Prediction" challenge). The pipeline covers: cleaning four raw tables and loading them into SQLite; benchmarking query times before and after index creation; building daily-level train/test sets with SQL JOINs and validating business hypotheses with SQL aggregations; calendar-cycle feature engineering (weekday sine/cosine, month position, weekend flags); and a direct multi-horizon (1–30 day) ensemble trained separately for purchase and redeem to avoid recursive error propagation, validated with rolling-forward origins and byte-verified before submission. Online score: 162.0652.

---

**Author**: Chengxuan Yuan (Lebmar) · Hong Kong University of Science and Technology (Guangzhou)
**Course**: DSAA1001 Introduction to Data Science and Analytics · A+ (only A+ in class), top 10%
