# 信用评分卡建模项目

基于 Kaggle [Give Me Some Credit](https://www.kaggle.com/c/GiveMeSomeCredit) 数据集，完成从数据清洗到评分卡生成的全流程信用评分卡开发。

## 项目概述

信用评分卡是银行和消费金融公司用于评估借款人违约风险的核心工具。本项目使用逻辑回归 + WOE 编码的经典方法，构建了一套可落地的信用评分卡模型。

## 数据说明

- **样本量：** 150,000 条借款人记录
- **目标变量：** `SeriousDlqin2yrs`（未来两年内是否发生 90 天以上逾期）
- **特征变量：** 包含信用卡利用率、年龄、逾期次数、负债比率、月收入等 10 个维度

## 建模流程

```
数据清洗 → EDA → 特征工程(WOE/IV) → 逻辑回归 → 模型评估(AUC/KS) → 评分卡转换
```

### 1. 数据清洗

- 异常值处理：对逾期次数、信用卡利用率等字段进行截断处理
- 缺失值处理：`MonthlyIncome` 添加缺失指示变量后中位数填充，`NumberOfDependents` 中位数填充
- 年龄异常值（<18）置为缺失后填充

### 2. 特征工程

- 使用 WOE（证据权重）编码将连续变量转换为对逻辑回归友好的特征
- 通过 IV（信息价值）筛选出区分度最高的 5 个变量：
  - `RevolvingUtilizationOfUnsecuredLines`（信用卡利用率）
  - `NumberOfTimes90DaysLate`（90 天以上逾期次数）
  - `NumberOfTime30-59DaysPastDueNotWorse`（30-59 天逾期次数）
  - `NumberOfTime60-89DaysPastDueNotWorse`（60-89 天逾期次数）
  - `age`（年龄）

### 3. 模型训练与评估

- 模型：逻辑回归（L2 正则化）
- 先 train_test_split 再计算 WOE，避免数据泄露
- 测试集使用训练集的分箱边界和 WOE 映射进行转换
- 评估指标：AUC、KS

### 4. 评分卡转换

- 基础分：600 分
- PDO（比率翻倍分数）：20
- 基础 Odds：1:20
- 输出每个变量每个分箱对应的评分映射表

## 技术栈

- Python 3.11
- pandas / numpy / matplotlib
- scikit-learn（LogisticRegression、roc_auc_score、roc_curve）

## 文件结构

```
├── README.md
├── GiveMeSomeCredit.ipynb      # 完整建模流程           
└── cs-training.csv             # 测试数据/训练数据
```

## 运行方式

```bash
# 安装依赖
pip install pandas numpy matplotlib scikit-learn jupyterlab

# 启动 JupyterLab
jupyter lab
```

打开 `GiveMeSomeCredit.ipynb`，按顺序运行所有 Cell 即可。
