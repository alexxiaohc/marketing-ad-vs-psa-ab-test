# Marketing Ad vs PSA A/B Test — Conversion Analysis

**中文名称：数字广告投放 A/B 测试：购买转化效果分析**

## 1. 项目简介

本项目基于 Kaggle 的匿名数字广告实验数据，对 Ad 广告组与 PSA（Public Service Announcement）对照组的购买转化率进行比较，并使用双样本比例 Z 检验评估差异是否具有统计显著性。

数据集未披露真实企业名称，因此本项目不将其描述为任何企业的官方实验。项目重点是复现已经完成的数据质量检查、购买转化分析、显著性检验与广告曝光频次探索。

## 2. 数据来源与字段说明

- 数据来源：[Marketing A/B Testing（Kaggle）](https://www.kaggle.com/datasets/faviovaz/marketing-ab-testing)
- Kaggle 页面列出的许可：CC0 1.0 / Public Domain
- 原始文件：`marketing_AB.csv`，共 588,101 行
- 原始 CSV 不纳入本仓库，请从上述 Kaggle 页面下载后放入 `data/`

| 字段 | 含义 |
| --- | --- |
| `user id` | 用户唯一标识 |
| `test group` | 实验组：`ad` 或 `psa` |
| `converted` | 是否购买（True/False） |
| `total ads` | 用户看到的广告/PSA 总次数 |
| `most ads day` | 曝光最多的星期 |
| `most ads hour` | 曝光最多的小时 |
| `Unnamed: 0` | 原始导出索引，无业务含义；分析时删除，原始 CSV 不修改 |

## 3. 分析目标及思路

1. 检查数据完整性、唯一性、字段类型和取值范围。
2. 分组汇总用户数、购买人数和购买转化率。
3. 使用双侧双样本比例 Z 检验比较 Ad 与 PSA 的转化率。
4. 对 Ad 组曝光次数进行描述性统计和分区间探索。
5. 在明确前提和数据局限的基础上给出业务结论。

## 4. 数据质量检查

本地原始数据核对结果：

- 588,101 行、7 列；
- 无缺失值；
- 无重复 `user id`；
- `test group` 仅包含 `ad` 和 `psa`；
- `converted` 仅包含 True 和 False；
- `total ads` 均为正整数；
- 星期与小时字段取值合法。

分析副本中删除无业务意义的 `Unnamed: 0`，不改动原始 CSV。

## 5. 购买转化率与统计显著性

| 指标 | Ad 广告组 | PSA 对照组 |
| --- | ---: | ---: |
| 用户数 | 564,577 | 23,524 |
| 购买人数 | 14,423 | 420 |
| 购买转化率 | 2.5547% | 1.7854% |

- Absolute Lift：+0.7692 个百分点；
- Relative Lift：+43.09%；
- 双侧双样本比例 Z 检验：Z = 7.3701，p-value = 1.7053 × 10⁻¹³；
- 比例差（Ad − PSA）的 95% CI：`[0.5951, 0.9434]` 个百分点。

检验统计量在零假设下使用 pooled standard error；比例差置信区间使用两组独立比例的 unpooled Wald interval。区间完全大于 0，观察到的转化率差异具有统计显著性。

检验结果也使用我开发的 [A/B Testing Decision Toolkit](https://ab-testing-decision-toolkit.onrender.com/) 进行过核对；工具源码见 [GitHub](https://github.com/alexxiaohc/ab-testing-decision-toolkit)。原始 MDE 与预期分流比例未知，因此本项目不把练习参数当作原始实验设计，也不宣称 SRM 已通过验证。

## 6. 广告曝光频次探索

Ad 组曝光次数的均值为 24.82，中位数为 13，95% 分位数为 87，最大值为 2,065。

| 曝光区间 | 用户数 | 购买人数 | 购买转化率 |
| --- | ---: | ---: | ---: |
| 1–5 | 169,962 | 427 | 0.25% |
| 6–10 | 79,537 | 386 | 0.49% |
| 11–20 | 123,334 | 1,036 | 0.84% |
| 21–50 | 125,541 | 3,660 | 2.92% |
| 51–100 | 44,149 | 5,135 | 11.63% |
| 101+ | 22,054 | 3,779 | 17.14% |

高曝光用户的购买转化率更高，但曝光次数没有独立随机分配。这一结果仅说明相关性，不能证明增加曝光次数会导致购买转化率提高。

## 7. 核心业务结论

1. Ad 组购买转化率显著高于 PSA 对照组。
2. 在随机分组有效且两组可比的前提下，按 Ad 组规模估算的增量购买约为 4,343 人。
3. 原始随机分流方案未知，因果解释必须以上述实验前提成立为条件。
4. 数据不含广告费用、订单金额或贡献利润，无法计算真实 ROI，不能据此直接建议扩大预算。
5. 后续应补充成本及贡献利润数据，并评估边际 ROI。

## 8. 数据局限

- 未提供原始实验设计文档、MDE、预期分流比例和随机化实现细节；
- 分组样本量高度不均衡，但因预期分流比例未知，不能据此完成 SRM 判断；
- 曝光次数是实验后的观测变量，不适合直接作因果解释；
- 缺少成本、收入和利润字段，无法回答投放是否盈利。

## 9. 项目运行方法

```bash
git clone https://github.com/alexxiaohc/marketing-ad-vs-psa-ab-test.git
cd marketing-ad-vs-psa-ab-test
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

从 [Kaggle 数据页](https://www.kaggle.com/datasets/faviovaz/marketing-ab-testing) 下载 `marketing_AB.csv` 并放到 `data/marketing_AB.csv`，然后运行：

```bash
jupyter notebook notebooks/marketing_ad_vs_psa_analysis.ipynb
```

