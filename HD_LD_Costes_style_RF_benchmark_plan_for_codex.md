# HD/LD WGBS：参照 Costes et al. 2022 的模型构建与评价试验方案（供 Codex 执行）

## 0. 目的

本方案参照 Costes et al. 2022 在公牛 sperm methylome 中预测 fertility 的整体思路，针对当前 HD/LD 精子 WGBS 数据设计一个可执行的建模与评价流程。

目标不是完全照搬原文，而是抽取其核心策略：

```text
1. 先获得一批 fertility-related methylation features；
2. 使用 Random Forest 作为主要 benchmark；
3. 在主队列内做 repeated random split 评价；
4. 做 batch/source holdout 评价；
5. 最后在 independent final test set 上一次性评价；
6. 检查误分类样本是否在 methylation feature space 中呈现相反类别的模式。
```

当前我们的特殊限制：

```text
1. development set = 68 samples，HD=37, LD=31；
2. final test set = 20 samples，HD=11, LD=9；
3. DML 由全部 development set 发现，因此 development 内部 CV 存在 feature-discovery leakage；
4. 之前 exact CpG extraction 出现严重 NA，后续应优先使用 DML-centered window methylation；
5. 之前 elastic-net top100 independent test AUC 约 0.75，但 accuracy/balanced accuracy 较差，尤其 LD 识别弱；
6. 当前需要判断：Costes-style RF 是否能改善模型表现，尤其是 LD sensitivity。
```

---

## 1. 参考文章关键方法摘要

Costes et al. 2022 的关键建模策略：

```text
Main cohort:
  100 bulls
  fertile = 57
  subfertile = 43

Independent cohort:
  20 bulls
  fertile = 16
  subfertile = 4

Feature discovery:
  RRBS
  CpGs10 = CpGs covered by at least 10 reads
  去除影响 CpG 的 putative polymorphisms
  获得 fertility-related DMCs / DMRs

Predictive model:
  使用 no-missing DMC methylation matrix
  Random Forest
  ntree = 500
  mtry = sqrt(number_of_features)

Internal evaluation:
  main cohort 随机划分为 training = 2/3, testing = 1/3
  保持 fertile/subfertile 比例
  重复 50 次 random resampling
  报告平均 accuracy, AUC, sensitivity, specificity

Source/center validation:
  用一个 collection center 训练，在另一个 center 测试；
  再反向验证。

Independent validation:
  用 main cohort 训练最终模型；
  在 independent cohort 中提取相同 DMC 坐标 methylation；
  评价 independent cohort performance。
```

对我们的借鉴点：

```text
1. Random Forest 作为与 elastic net / SuperLearner 并列的主 benchmark；
2. 使用 repeated 2/3 training、1/3 internal testing；
3. 增加 batch/source holdout；
4. final test 只作为最终独立评价；
5. 分析 misclassified samples 是否表现为 opposite-class methylation pattern。
```

---

## 2. 本项目数据集定义

### 2.1 Development set

```text
n = 68
HD = 37
LD = 31
用途：
  DML discovery 已经完成
  feature ranking
  K / feature set selection
  repeated internal evaluation
  batch/source holdout
  final model training
```

### 2.2 Independent final test set

```text
n = 20
HD = 11
LD = 9
用途：
  只用于 locked pipeline 的最终评价
```

推荐 final test samples：

```text
HD:
HD5937, HD3442, HD4510, HD7699, HD7591,
HD2436, HD2749, HD1006, HD0689, HD0249, HD1588

LD:
LD1535, LD1458, LD2339, LD1771, LD2455,
LD2102, LD2070, LD3569, LD4727
```

### 2.3 排除样本

```text
HD5928
LD2450
HD3367
```

### 2.4 重要限制

final test 不能参与：

```text
DML discovery
feature ranking
K selection
window size selection
model selection
hyperparameter tuning
threshold selection
imputation / scaling parameter estimation
```

如果使用 final test 来选模型或 K，则该分析必须标记为 exploratory，不能作为严格 independent test。

---

## 3. Feature matrix 定义

### 3.1 优先使用 DML-centered window methylation

由于 exact CpG 特征在 test set 中可能出现严重缺失，本方案优先使用：

```text
DML-centered ±250 bp window methylation
或
DML-centered ±500 bp window methylation
```

window beta 计算方式：

```text
window_beta = sum(methylated_reads_in_window) / sum(total_reads_in_window)
```

如果：

```text
window_total_coverage = 0
```

则该 sample × feature 记为 NA。

### 3.2 Development 与 test 必须一致

必须保证：

```text
development matrix 和 final test matrix 使用相同 DML list；
相同 window size；
相同 beta 计算方式；
相同 row order；
相同 feature_id。
```

禁止：

```text
development 使用 window beta；
final test 使用 exact CpG beta。
```

### 3.3 输入文件建议

Codex 需要在实际目录中定位或生成：

```text
costes_style_RF_benchmark/input/
├── DML_ranked.tsv
├── development_window250_beta.tsv.gz
├── development_window250_Cov.tsv.gz
├── test_window250_beta.tsv.gz
├── test_window250_Cov.tsv.gz
├── development_window500_beta.tsv.gz        # optional
├── test_window500_beta.tsv.gz               # optional
└── sample_metadata.tsv
```

---

## 4. Feature sets 设计

参照 Costes 使用 no-missing DMCs 的思想，同时结合我们的 K=40 经验，设置以下 feature sets。

### 4.1 主候选 feature sets

```text
FS_top40:
  top 40 ranked DML-window features

FS_top100:
  top 100 ranked DML-window features

FS_complete_low_missing:
  在 development set 中 missingness ≤ 5% 或 10% 的 ranked DML-window features
  如果数量过多，则取 rank 最靠前的前 100 或 200

FS_strict_DML:
  更严格 DML 阈值，例如 FDR < 0.01 且 |delta_beta| ≥ 0.10
  或 FDR < 0.05 且 |delta_beta| ≥ 0.15
  然后再取低 missingness 的 top features
```

### 4.2 初始建议

第一轮优先运行：

```text
FS_top40
FS_top100
FS_complete_low_missing_top100
```

如果时间允许，再加：

```text
FS_top200
FS_strict_DML_top100
```

### 4.3 feature QC

每个 feature set 需输出：

```text
feature_set_summary.tsv
```

字段：

```text
feature_set
n_features
mean_dev_missingness
mean_test_missingness
median_dev_coverage
median_test_coverage
n_features_sd0_in_dev
n_features_removed
n_features_final
```

---

## 5. Preprocessing

### 5.1 Random Forest 是否需要 z-score？

Random Forest 本身不要求 z-score，但为了和 elastic net / SVM / SuperLearner 比较，建议保存两个版本：

```text
RF 原始 beta 版本：
  imputation only, no z-score

通用 ML 版本：
  imputation + z-score
```

主 RF benchmark 使用原始 beta 即可。

### 5.2 缺失值填补

在每个训练集内学习 imputation 参数。

Repeated split 中：

```text
training subset:
  median_train(feature) = median(beta in training subset, na.rm=TRUE)

training NA:
  用 median_train 填补

internal testing NA:
  用 median_train 填补
```

Final test 中：

```text
all 68 development:
  median_dev(feature) = median(beta in all development samples, na.rm=TRUE)

development NA:
  用 median_dev 填补

final test NA:
  用 median_dev 填补
```

不能使用 test set 自身 median。

### 5.3 极端缺失处理

如果某 feature 在当前 training subset 中全部 NA：

```text
删除该 feature；
或使用 development 全局 median 作为 backup，但必须记录。
```

建议优先删除并在结果中记录。

---

## 6. Costes-style RF benchmark：repeated 2/3–1/3 split

### 6.1 设计

在 68 个 development samples 内进行：

```text
50 iterations
每次 stratified random split：
  training = 2/3 development samples，约 45 samples
  internal testing = 1/3 development samples，约 23 samples
保持 HD/LD 比例
```

如果样本数导致比例无法完全一致，允许 ±1 个样本偏差。

### 6.2 每次 split 内流程

对每个 feature set：

```text
1. 在 training subset 上计算 imputation median；
2. 填补 training 和 internal testing；
3. 用 training subset 训练 Random Forest；
4. 在 internal testing subset 预测 P(HD)；
5. threshold=0.5 得到预测类别；
6. 记录 accuracy, balanced accuracy, ROC-AUC, HD sensitivity, LD sensitivity。
```

### 6.3 RF 参数

参照文章：

```text
ntree = 500
mtry = floor(sqrt(p))
```

其中：

```text
p = feature number
```

可使用 R 包：

```text
caret::train(method="rf")
randomForest
ranger
```

推荐优先使用 `ranger`，因为速度快；但为了贴近原文，也可以用 `randomForest` 或 `caret::train(method="rf")`。

推荐参数：

```text
num.trees = 500
mtry = floor(sqrt(p))
probability = TRUE
seed fixed per iteration
```

### 6.4 输出

```text
costes_style_RF_benchmark/internal_resampling/RF_50resampling_predictions.tsv
```

字段：

```text
iteration
feature_set
sample_id
role
true_group
P_HD
predicted_group_threshold_0.5
```

```text
costes_style_RF_benchmark/internal_resampling/RF_50resampling_metrics.tsv
```

字段：

```text
iteration
feature_set
n_train
n_test
n_features
ROC_AUC
PR_AUC
accuracy
balanced_accuracy
HD_sensitivity
LD_sensitivity
HD_specificity
LD_specificity
PPV_HD
NPV_HD
Brier_score
log_loss
```

```text
costes_style_RF_benchmark/internal_resampling/RF_50resampling_summary.tsv
```

字段：

```text
feature_set
mean_ROC_AUC
SE_ROC_AUC
mean_accuracy
SE_accuracy
mean_balanced_accuracy
SE_balanced_accuracy
mean_HD_sensitivity
mean_LD_sensitivity
mean_HD_specificity
mean_LD_specificity
mean_Brier_score
mean_log_loss
```

### 6.5 注意解释

这个 repeated split 评价是 Costes-style benchmark，但由于 feature discovery 使用了全部 development set，结果可能偏乐观。报告中必须写明。

---

## 7. Batch/source holdout validation

参照 Costes 的 center-based validation，在我们的数据中做 batch/source holdout。

### 7.1 可用分组变量

Codex 从 `sample_metadata.tsv` 中检查以下字段：

```text
sequencing_batch
DNA_extraction_batch
library_batch
sample_collection_wave
purification
remove_somatic
operator
```

### 7.2 Holdout 设计

优先设计：

```text
Batch holdout 1:
  train on Seq1 development samples
  test on Seq2 development samples

Batch holdout 2:
  train on Seq2 development samples
  test on Seq1 development samples
```

如果某个 batch 中 HD/LD 极不平衡，记录并跳过或只做描述性分析。

也可以增加：

```text
Purification holdout:
  train on one purification/preparation group
  test on another group

Collection wave holdout:
  train on earlier samples
  test on later samples
```

### 7.3 输出

```text
costes_style_RF_benchmark/batch_holdout/RF_batch_holdout_metrics.tsv
costes_style_RF_benchmark/batch_holdout/RF_batch_holdout_predictions.tsv
```

字段同 repeated split。

### 7.4 解释

如果 repeated split 表现好，但 batch/source holdout 表现差，说明模型可能依赖 batch/source 结构，泛化风险高。

---

## 8. Independent final test evaluation

### 8.1 何时执行

只有在以下内容固定后才执行：

```text
feature set
window size
RF 参数
threshold
preprocessing 规则
```

### 8.2 最终模型训练

对选定 feature set：

```text
1. 用全部 68 development samples 计算 feature median；
2. 填补 development NA；
3. 训练 RF final model；
4. 保存 model object；
5. 保存 feature importance。
```

RF 参数：

```text
ntree = 500
mtry = floor(sqrt(p))
```

### 8.3 final test 预测

```text
1. 取相同 features；
2. 使用 development median 填补 final test NA；
3. 使用 final RF model 预测 P(HD)；
4. threshold=0.5 得到类别；
5. 输出 metrics。
```

### 8.4 输出

```text
costes_style_RF_benchmark/final_test/RF_final_test_predictions.tsv
```

字段：

```text
sample_id
true_group
feature_set
P_HD
predicted_group_threshold_0.5
```

```text
costes_style_RF_benchmark/final_test/RF_final_test_metrics.tsv
```

字段：

```text
feature_set
ROC_AUC
PR_AUC
accuracy
balanced_accuracy
HD_sensitivity
LD_sensitivity
HD_specificity
LD_specificity
PPV_HD
NPV_HD
Brier_score
log_loss
```

```text
costes_style_RF_benchmark/final_test/RF_confusion_matrix.tsv
```

---

## 9. Variable importance 与 marker 稳定性

### 9.1 RF variable importance

对 final RF model 输出：

```text
costes_style_RF_benchmark/final_model/RF_variable_importance.tsv
```

字段：

```text
feature_id
chr
pos
rank_final
adjusted_p
delta_beta
importance
importance_rank
dev_missingness
test_missingness
dev_median_coverage
test_median_coverage
```

### 9.2 Repeated split importance stability

如果计算可行，在每次 RF split 中保存 variable importance，统计：

```text
feature_id
selected_feature_set
mean_importance
SE_importance
importance_rank_median
top10_frequency
top20_frequency
```

输出：

```text
costes_style_RF_benchmark/internal_resampling/RF_importance_stability.tsv
```

---

## 10. PCA / clustering 与 misclassification 分析

参照文章 Fig. 8 / Fig. 9，对 feature matrix 做 PCA 并标注预测是否正确。

### 10.1 Development PCA

对选定 feature set，在全部 development samples 上做 PCA：

```text
rows = samples
columns = selected features
values = imputed beta 或 z-score beta
```

输出：

```text
costes_style_RF_benchmark/diagnostics/development_PCA_selected_features.png
costes_style_RF_benchmark/diagnostics/development_PCA_scores.tsv
```

点颜色：

```text
HD correctly predicted
LD correctly predicted
HD misclassified as LD
LD misclassified as HD
```

如果使用 repeated split，每个样本可能被多次预测。可定义：

```text
majority_prediction
mean_P_HD
misclassification_frequency
```

### 10.2 Final test PCA projection

推荐将 final test 投影到 development PCA space：

```text
1. PCA 只在 development samples 上 fit；
2. final test 使用 development PCA rotation 投影；
3. 标注 final test 预测正确/错误。
```

输出：

```text
costes_style_RF_benchmark/diagnostics/final_test_projected_PCA.png
costes_style_RF_benchmark/diagnostics/final_test_projected_PCA_scores.tsv
```

### 10.3 错分样本解释

输出：

```text
costes_style_RF_benchmark/diagnostics/misclassified_sample_analysis.tsv
```

字段：

```text
sample_id
true_group
predicted_group
P_HD
PC1
PC2
nearest_development_group
mean_feature_beta
mean_z_score
coverage_summary
notes
```

判断：

```text
如果错分 LD 在 PCA 中靠近 HD，则说明该 LD 样本的 methylation pattern 更像 HD；
如果错分分布与 batch/coverage 有关，则提示技术或批次影响。
```

---

## 11. 与已有 elastic net / SuperLearner 结果比较

为了判断算法是否是主要瓶颈，需要比较 RF 与已有模型是否错分同一批样本。

输入已有文件：

```text
elastic_net final test predictions
SuperLearner K40 final test predictions
RF final test predictions
```

输出：

```text
costes_style_RF_benchmark/comparison/model_error_overlap.tsv
```

字段：

```text
sample_id
true_group
elastic_net_P_HD
elastic_net_pred
superlearner_P_HD
superlearner_pred
RF_P_HD
RF_pred
n_models_predict_HD
n_models_predict_LD
misclassified_by_models
```

解释：

```text
如果所有模型都错分同一批 LD 或 HD：
  主要问题可能是 feature 泛化、生物学重叠、batch/coverage shift，而不是算法类型。

如果 RF 能纠正 elastic net 的错误：
  说明非线性/树模型可能捕捉到有用模式。

如果 SuperLearner 与最佳单模型接近：
  不必过度依赖复杂集成。
```

---

## 12. 可选：更严格的 training-fold feature discovery 版本

如果计算资源允许，可增加更严格版本：

```text
每次 2/3 training split 中：
  只用 training split 进行 coverage filtering / DSS / DML discovery / ranking；
  在 testing split 中提取 training DML-window features；
  训练 RF 并评价。
```

这个版本最接近完整 pipeline 泛化能力，但 DSS 计算量较高。

建议作为后续阶段，不作为第一轮 Costes-style benchmark 的必须步骤。

---

## 13. 输出目录结构

```text
costes_style_RF_benchmark/
├── input/
│   ├── DML_ranked.tsv
│   ├── feature_set_summary.tsv
│   └── selected_feature_sets/
├── internal_resampling/
│   ├── RF_50resampling_predictions.tsv
│   ├── RF_50resampling_metrics.tsv
│   ├── RF_50resampling_summary.tsv
│   └── RF_importance_stability.tsv
├── batch_holdout/
│   ├── RF_batch_holdout_predictions.tsv
│   └── RF_batch_holdout_metrics.tsv
├── final_model/
│   ├── RF_final_model.rds
│   ├── RF_preprocessing_parameters.tsv
│   └── RF_variable_importance.tsv
├── final_test/
│   ├── RF_final_test_predictions.tsv
│   ├── RF_final_test_metrics.tsv
│   └── RF_confusion_matrix.tsv
├── diagnostics/
│   ├── development_PCA_selected_features.png
│   ├── development_PCA_scores.tsv
│   ├── final_test_projected_PCA.png
│   ├── final_test_projected_PCA_scores.tsv
│   └── misclassified_sample_analysis.tsv
├── comparison/
│   └── model_error_overlap.tsv
└── report/
    └── costes_style_RF_benchmark_report.md
```

---

## 14. 最终报告必须回答的问题

报告：

```text
costes_style_RF_benchmark/report/costes_style_RF_benchmark_report.md
```

必须回答：

```text
1. 使用 Costes-style RF repeated split，top40/top100/low-missing feature set 的平均性能如何？
2. RF 是否比 elastic net / SuperLearner 更好，尤其是否改善 LD sensitivity？
3. repeated split 表现与 independent final test 表现是否一致？
4. batch/source holdout 是否明显低于 random split？
5. RF variable importance 是否集中在少数稳定 window features？
6. final test 中 RF 错分样本是否与 elastic net / SuperLearner 错分样本重叠？
7. 错分样本在 PCA 中是否靠近相反类别？
8. 结果是否支持继续优化算法，还是提示应回到 feature discovery / DML filtering / DMR-window 层面？
```

---

## 15. 推荐结论模板

如果 RF 明显改善 LD：

```text
The Costes-style Random Forest benchmark improved LD recognition compared with the previous elastic-net model, suggesting that nonlinear feature interactions may contribute to HD/LD classification. However, because development-set DML discovery was performed before internal resampling, independent final test performance remains the key evidence for generalization.
```

如果 RF 与 elastic net / SuperLearner 错同一批样本：

```text
Random Forest, elastic-net, and SuperLearner misclassified largely overlapping test samples. This suggests that the main limitation is not classifier family but the generalizability of the selected methylation features, sample heterogeneity, batch/coverage shift, or biological overlap between HD and LD.
```

如果 random split 好但 batch-holdout/final-test 差：

```text
The discrepancy between random internal resampling and batch/final-test validation indicates that the model may capture development-set or batch-specific methylation patterns rather than robust HD/LD biomarkers.
```

---

## 16. 一句话总结

本方案参照 Costes et al. 2022，使用 Random Forest 作为 fertility methylome prediction benchmark。核心试验包括：development set 内 50 次 stratified 2/3–1/3 random split、batch/source holdout、independent final test、RF variable importance、PCA/misclassification 分析，以及与 elastic net / SuperLearner 的错分样本重叠比较。
