# 9特征模型报告：5个DMR + 4个孤立DML

## 模型概况

- **特征来源**：从DSS 88样本分析得到的3102个DML中，经top 1.7%（K=53）弹性网络筛选出**31个非零特征**
- **DMR合并**：按"≥3个DMC且相邻间距≤100bp"定义，31个特征合并为**5个DMR** + **4个孤立DML** = **9个特征**
- **模型**：弹性网络（α=0.5, λ=λ.1se, weighted loss），训练集87样本，测试集20样本

## 性能对比

| 指标 | 31-site EN | **9-feature EN** |
|------|:----------:|:----------------:|
| **AUC** | 0.7879 | **0.7778** |
| **Accuracy** | 0.8500 | **0.8500** |
| **HD灵敏度** | 1.0000 | **1.0000** |
| **LD灵敏度** | 0.6667 | **0.6667** |
| 特征数 | 31 | **9** |
| 非零系数 | 31 | **9** |

**结论**：从31个特征压缩到9个，预测性能几乎无损失。

---

## 一、4个孤立DML（直接作为特征）

| 特征 | 染色体 | 位置(bp) | 系数 | HD vs LD甲基化 |
|------|:-----:|:--------:|:----:|:--------------:|
| **chrX:102779218** | chrX | 102,779,218 | **+0.740** | 高甲基化（HD > LD） |
| **chr2:132118** | chr2 | 132,118 | **+0.687** | 高甲基化（HD > LD） |
| **chr22:33296140** | chr22 | 33,296,140 | **-0.561** | 低甲基化（HD < LD） |
| **chr4:36624424** | chr4 | 36,624,424 | **+0.435** | 高甲基化（HD > LD） |

> 孤立DML的系数绝对值最大，是模型中最强的预测信号。

---

## 二、5个DMR

### DMR1 — chr14:88890170-88890175

| 参数 | 值 |
|------|------|
| DMC数 | 4 |
| 跨度 | 5 bp |
| 整体方向 | **全部低甲基化（HD < LD）** ✓ |

| DMC | 位置(bp) | 间距 | 系数 |
|-----|:--------:|:----:|:----:|
| chr14:88890170 | 88,890,170 | — | -0.099 |
| chr14:88890171 | 88,890,171 | 1bp | -0.100 |
| chr14:88890174 | 88,890,174 | 3bp | -0.098 |
| chr14:88890175 | 88,890,175 | 1bp | -0.095 |

### DMR2 — chr15:39267261-39267281

| 参数 | 值 |
|------|------|
| DMC数 | 3 |
| 跨度 | 20 bp |
| 整体方向 | **全部低甲基化（HD < LD）** ✓ |

| DMC | 位置(bp) | 间距 | 系数 |
|-----|:--------:|:----:|:----:|
| chr15:39267261 | 39,267,261 | — | -0.073 |
| chr15:39267270 | 39,267,270 | 9bp | -0.073 |
| chr15:39267281 | 39,267,281 | 11bp | -0.070 |

### DMR3 — chr18:63533257-63533262

| 参数 | 值 |
|------|------|
| DMC数 | 4 |
| 跨度 | 5 bp |
| 整体方向 | **全部低甲基化（HD < LD）** ✓ |

| DMC | 位置(bp) | 间距 | 系数 |
|-----|:--------:|:----:|:----:|
| chr18:63533257 | 63,533,257 | — | -0.097 |
| chr18:63533258 | 63,533,258 | 1bp | -0.098 |
| chr18:63533261 | 63,533,261 | 3bp | -0.094 |
| chr18:63533262 | 63,533,262 | 1bp | -0.090 |

### DMR4 — chr6:54864309-54864388（最大DMR）

| 参数 | 值 |
|------|------|
| DMC数 | **9**（最多） |
| 跨度 | **79 bp**（最长） |
| 整体方向 | **全部低甲基化（HD < LD）** ✓ |

| DMC | 位置(bp) | 间距 | 系数 |
|-----|:--------:|:----:|:----:|
| chr6:54864309 | 54,864,309 | — | **-0.126** |
| chr6:54864310 | 54,864,310 | 1bp | **-0.127** |
| chr6:54864336 | 54,864,336 | 26bp | -0.051 |
| chr6:54864337 | 54,864,337 | 1bp | -0.051 |
| chr6:54864382 | 54,864,382 | 45bp | -0.031 |
| chr6:54864383 | 54,864,383 | 1bp | -0.031 |
| chr6:54864386 | 54,864,386 | 3bp | -0.030 |
| chr6:54864387 | 54,864,387 | 1bp | -0.030 |
| chr6:54864388 | 54,864,388 | 1bp | -0.029 |

> DMR4有两个子簇：上游2个DMC系数较大（-0.13），下游7个DMC系数较小（-0.03~-0.05）。

### DMR5 — chr7:103463278-103463338

| 参数 | 值 |
|------|------|
| DMC数 | 7 |
| 跨度 | 60 bp |
| 整体方向 | **全部低甲基化（HD < LD）** ✓ |

| DMC | 位置(bp) | 间距 | 系数 |
|-----|:--------:|:----:|:----:|
| chr7:103463278 | 103,463,278 | — | -0.066 |
| chr7:103463286 | 103,463,286 | 8bp | -0.069 |
| chr7:103463293 | 103,463,293 | 7bp | -0.069 |
| chr7:103463310 | 103,463,310 | 17bp | -0.067 |
| chr7:103463318 | 103,463,318 | 8bp | -0.065 |
| chr7:103463331 | 103,463,331 | 13bp | -0.063 |
| chr7:103463338 | 103,463,338 | 7bp | -0.109 |

> DMR5的最后一个DMC（chr7:103463338）系数最大（-0.109），可能是一个关键位点。

---

## 三、缺失值与填补

| 数据集 | 31个特征缺失率 |
|--------|:-------------:|
| 开发集（87样本） | **0.00%** |
| 测试集（20样本） | **0.00%** |

填补方法：开发集每个特征的中位数填补（测试集使用相同中位数），然后z-score归一化。

---

## 四、DMR内DMC方向一致性

**5个DMR全部方向一致**：

| DMR | DMC数 | 方向 | 一致性 |
|-----|:-----:|:----:|:------:|
| DMR1 chr14 | 4 | 全部负向（低甲基化） | ✓ |
| DMR2 chr15 | 3 | 全部负向（低甲基化） | ✓ |
| DMR3 chr18 | 4 | 全部负向（低甲基化） | ✓ |
| DMR4 chr6 | 9 | 全部负向（低甲基化） | ✓ |
| DMR5 chr7 | 7 | 全部负向（低甲基化） | ✓ |

4个孤立DML中3个高甲基化、1个低甲基化。孤立DML系数绝对值显著大于DMR内DMC，是模型主要预测信号。

---

## 五、输出文件

| 文件 | 路径 |
|------|------|
| ROC曲线（测试集） | `C:\Users\liran\Documents\Codex\2026-06-30\yan\outputs\independent20_DMR9_features_EN_ROC.png` |
| 混淆矩阵（测试集） | `C:\Users\liran\Documents\Codex\2026-06-30\yan\outputs\independent20_DMR9_features_EN_confusion.png` |
| ROC曲线（训练集） | `C:\Users\liran\Documents\Codex\2026-06-30\yan\outputs\training87_DMR9_features_EN_ROC.png` |
| 混淆矩阵（训练集） | `C:\Users\liran\Documents\Codex\2026-06-30\yan\outputs\training87_DMR9_features_EN_confusion.png` |
| PCA（训练集87样本） | `C:\Users\liran\Documents\Codex\2026-06-30\yan\outputs\training87_9features_PCA.png` |
| PCA（测试集20样本） | `C:\Users\liran\Documents\Codex\2026-06-30\yan\outputs\independent20_9features_PCA.png` |

---

## 六、基因注释（hg19/GRCh37）

### 5个DMR

| DMR | 区域 | DMC数 | 基因 | 链 | 关系 | 距TSS | 基因功能 |
|-----|------|:-----:|:----:|:--:|:----:|:-----:|----------|
| **DMR1** | chr14:88890170-88890175 | 4 | **SPATA7** | + | **重叠** | +38.4kb | 精子发生相关蛋白7 |
| **DMR2** | chr15:39267261-39267281 | 3 | — | — | >50kb内无基因 | — | 基因沙漠 |
| **DMR3** | chr18:63533257-63533262 | 4 | **CDH7** | + | **重叠** | +115.8kb | 钙黏蛋白7（细胞黏附） |
| **DMR4** | chr6:54864309-54864388 | 9 | — | — | >50kb内无基因 | — | 基因沙漠 |
| **DMR5** | chr7:103463278-103463338 | 7 | **RELN** | - | **重叠** | -166.7kb | reelin（神经元迁移） |

### 4个孤立DML

| DML | 系数 | 基因 | 链 | 关系 | 距TSS | 基因功能 |
|-----|:----:|:----:|:--:|:----:|:-----:|----------|
| **chr2:132118** | **+0.687** | — | — | >50kb内无基因 | — | 基因沙漠（2p末端） |
| **chr22:33296140** | **-0.561** | **SYN3** | - | **重叠** | -158.2kb | synapsin III（突触功能） |
| **chr4:36624424** | **+0.435** | — | — | >50kb内无基因 | — | 基因沙漠 |
| **chrX:102779218** | **+0.740** | **RAB40A** | - | 相距4.8kb | +4.8kb | Ras家族GTP酶 |

### 小结

- **重叠基因**：DMR1→SPATA7, DMR3→CDH7, DMR5→RELN, chr22孤立DML→SYN3
- **邻近基因**：chrX孤立DML距RAB40A仅4.8kb（最强预测信号）
- **基因沙漠**：DMR2, DMR4, chr2, chr4孤立DML 50kb内无注释基因，可能在**基因间调控区域**或**未注释的调控元件**
- 所有DMR内DMC方向一致（全部低甲基化），提示这些区域的差异甲基化具有协同性

---

*报告生成日期：2026-06-30*

---

## 七、表观基因组注释（H1hesc ESC数据 + hg19公共注释）

### 7.1 完整注释矩阵

| 特征 | 基因 | 系数 | CpG | ChromHMM | H3K4me3 | H3K4me1 | H3K27me3 | 重复元件 | 重复类别 | SegDup |
|------|:----:|:----:|:----:|:--------:|:-------:|:-------:|:---------:|:---------:|:---------:|:------:|
| DMR1 chr14 | SPATA7 | -0.098 | Open sea(38kb) | Low | 36kb | 37kb | 86kb | **AluYc** | SINE | 470kb |
| **DMR2 chr15** | — | -0.072 | Open sea(**410kb**) | **Quies** | 120kb | 33kb | 219kb | **AluYa5** | SINE | 128kb |
| DMR3 chr18 | CDH7 | -0.095 | Open sea(115kb) | **Quies** | 113kb | 14kb | 34kb | **AluYa5** | SINE | 85kb |
| **DMR4 chr6** | — | -0.056 | Open sea(152kb) | **Quies** | 151kb | 47kb | 96kb | **AluYa5** | SINE | 224kb |
| DMR5 chr7 | RELN | -0.073 | Open sea(**166kb**) | **Quies** | 165kb | 12kb | 164kb | **AluYb8** | SINE | 534kb |
| **chr2:132118** | — | **+0.687** | Open sea(65kb) | Low | 83kb | **✅ OVERLAP** | 84kb | AluSp(281bp) | SINE | 29kb |
| chr22:33296140 | SYN3 | **-0.561** | Open sea(98kb) | Low | 97kb | 11kb | 62kb | **AluYb8+MIR3** | SINE | 205kb |
| **chr4:36624424** | — | **+0.435** | Open sea(**380kb**) | **Quies** | **379kb** | 13kb | 134kb | **MLT1K** | **LTR/MaLR** | **1.2Mb** |
| chrX:102779218 | RAB40A(4.8kb) | **+0.740** | Open sea(61kb) | **Quies** | **9.0kb** | 29kb | 47kb | **MIRb** | SINE | 21kb |

> *距离值均为距最近peak/feature的bp数*

### 7.2 按特征分类

#### 基因内部（4个）
| 特征 | 基因 | 位置 | 启动子/增强子信号 |
|------|:----:|:----:|:-----------------:|
| DMR1 chr14 | **SPATA7** (内含子) | 14q31.3 | 无（H1hesc中为Low染色质） |
| DMR3 chr18 | **CDH7** (内含子) | 18q22.1 | 无（Quies，距H3K4me1 14kb） |
| DMR5 chr7 | **RELN** (内含子) | 7q22.1 | 无（Quies，距H3K4me1 12kb） |
| chr22:33296140 | **SYN3** (内含子) | 22q12.3 | 无（Low，距H3K4me1 11kb） |

#### 基因沙漠/基因间（5个）
| 特征 | 最近基因 | 染色质 | 调控信号 |
|:----:|:--------:|:------:|:---------:|
| **chr2:132118** | >50kb | **Low** | **H3K4me1重叠 → 可能增强子** |
| **chrX:102779218** | RAB40A(4.8kb) | **Quies** | **H3K4me3 9kb → 近启动子** |
| DMR2 chr15 | >50kb | Quies | 无（Alu SINE内部） |
| DMR4 chr6 | >50kb | Quies | 无（Alu SINE内部） |
| chr4:36624424 | >50kb | Quies | 无（LTR重复内部） |

### 7.3 重复元件与表观调控的关系

**Alu SINE（6/9特征）**：DMR1/2/3/4/5 + chr22均重叠或紧邻Alu家族SINE。Alu元件在精子发生过程中受到DNA甲基化调控，HD-LD间的差异甲基化可能反映了Alu元件甲基化状态的群体差异。

**LTR/MLT1K（1/9特征）**：chr4:36624424位于MLT1K LTR重复内。LTR元件在早期胚胎发育中具有增强子活性，其甲基化差异可能影响邻近基因表达。

**非重复唯一特征（1/9）**：chr2:132118是唯一不直接重叠重复元件的特征，且是**唯一具有H3K4me1信号**的特征，提示其可能位于**bona fide增强子**区域。

### 7.4 总结

- **CpG环境**：全部9特征位于Open sea，距最近CpG岛>4kb
- **染色质状态**：全部处于Quies/Low，无活跃启动子/增强子信号（H1hesc中）
- **组蛋白修饰**：仅chr2:132118重叠H3K4me1（增强子标记）；chrX:102779218距H3K4me3仅9kb（近启动子）
- **重复元件**：8/9重叠SINE/LTR重复；仅chr2:132118不与重复直接重叠
- **预测最强的两个特征**（chrX:+0.74, chr2:+0.69）均位于基因沙漠但具有独特的表观特征（近启动子/H3K4me1增强子）

> **注**：以上组蛋白数据来自H1hesc（人胚胎干细胞），精子/生殖细胞特异性的组蛋白数据（如GSE156108）需要额外下载。

---

*报告生成日期：2026-06-30 | 最后更新：2026-07-01（添加表观基因组注释）*

---

## 八、补充注释：ENCODE cCRE / H3K27ac / DNase / Repeat QC / GSE156108 检查（2026-07-01）

### 8.1 本次补充数据与口径

- 坐标版本：沿用本报告的 **hg19/GRCh37**。DMR 用真实 DMR span；孤立 DML 用 1bp CpG core，同时另查 **±250bp** 窗口。
- cCRE：ENCODE SCREEN Registry V4 `cCRE.hg19.bed`。
- H3K27ac / DNase：Roadmap consolidated peaks，补充 H1 ESC、iPSC、placenta、fetal brain、ovary proxy。Roadmap metadata 中未找到 testis / fetal gonad reference epigenome，因此这两类不能用 Roadmap 直接判定。
- Repeat/QC：UCSC RepeatMasker exact copy、ENCODE hg19 blacklist v2、UCSC/ENCODE CRG 100mer mappability。
- GSE156108：GEO supplementary `filelist.txt` 显示该数据集仅提供 H3K4me3/DNA methylation **bigWig** 文件和 2.0GB `GSE156108_RAW.tar`，没有可直接 overlap 的 sperm H3K4me3 peak BED 或 H3K4me3+DNA methylation overlap BED。因此本次不能把“sperm H3K4me3 peak overlap / sperm H3K4me3 + DNA methylation overlap / SINE-associated sperm H3K4me3 region”判定为阳性或阴性；需要后续从 bigWig/原始数据统一 call peaks，或找到作者补充 peak/overlap 表后再交集。

### 8.2 9 个特征的新增 overlap 结果

| 特征 | cCRE / SCREEN | H3K27ac | DNase/ATAC | ENCODE blacklist | RepeatMasker exact / ±250bp | CRG 100mer mappability |
|------|:-------------:|:-------:|:----------:|:----------------:|------------------------------|:----------------------:|
| DMR1 chr14:88890170-88890175 | 无；最近 dELS 299bp | 无 | 无 | 否 | core 100% overlap AluYc, SINE/Alu, strand -, div 1.2% | core 1.000；±250 0.852 |
| DMR2 chr15:39267261-39267281 | ±250bp overlap **dELS** | 无 | 无 | 否 | core 100% overlap AluYa5, SINE/Alu, strand -, div 0.7% | core 0.477；±250 0.542 |
| DMR3 chr18:63533257-63533262 | **core overlap pELS** | 无 | 无 | 否 | core 100% overlap AluYa5, SINE/Alu, strand +, div 0.7% | core 0.0003；±250 0.536 |
| DMR4 chr6:54864309-54864388 | ±250bp overlap CA-CTCF | 无 | 无 | 否 | core partial overlap AluYa5, SINE/Alu, strand +, div 0.6%, feature frac 0.12 | core 0.768；±250 0.600 |
| DMR5 chr7:103463278-103463338 | 无；最近 dELS 1034bp | 无 | 无 | 否 | core 100% overlap AluYb8, SINE/Alu, strand +, div 0.9% | core 0.595；±250 0.583 |
| chrX:102779218 | 无；最近 dELS 316bp | 无 | 无 | 否 | ±250bp overlap MIRb, SINE/MIR, strand -, div 25.6%, window frac 0.24 | core 1.000；±250 1.000 |
| chr2:132118 | 无；最近 dELS 1637bp | 无 | 无 | 否 | core/±250bp 内无 RepeatMasker overlap；AluSp 是邻近距离，不是 overlap | core 1.000；±250 1.000 |
| chr22:33296140 | 无；最近 dELS 693bp | 无 | 无 | 否 | ±250bp overlap AluYb8 + MIR3, SINE, strand + | core 1.000；±250 0.804 |
| chr4:36624424 | ±250bp overlap CA-H3K4me3 | 无 | ±250bp overlap fetal brain DNase | 否 | core/±250bp 内无 RepeatMasker overlap（注意上一版报告称 MLT1K 需按修正坐标复核） | core 1.000；±250 1.000 |

### 8.3 解释更新

- **active enhancer 证据不足**：9 个特征均未 overlap 本次查询到的 H3K27ac peak；因此不能把 H3K4me1-only 信号解释为 active enhancer。若只有 H3K4me1 而无 H3K27ac，更偏 primed/poised enhancer 或弱/组织不匹配信号。
- **cCRE 证据最强的是 DMR3**：DMR3 core 直接落在 SCREEN **pELS**；DMR2 只在 ±250bp 窗口碰到 **dELS**。其余多数特征没有 ELS overlap。
- **H1hESC 不能代表 sperm/germ cell**：本次 Roadmap 补充确认，H1 ESC / iPSC / placenta / fetal brain / ovary proxy 中没有给出 sperm、spermatogonia、spermatocyte、round spermatid 或 mature sperm 的等价 chromatin 注释。sperm/germ-cell 专属结论仍需 GSE156108 peak calling 或生殖细胞 ATAC/DNase 数据。
- **repeat/mappability QC 风险集中在 Alu features**：DMR2、DMR3、DMR5 core 都 100% 落在低 divergence AluYa5/AluYb8，DMR3 core mappability 极低（0.0003），DMR2 core mappability 也偏低（0.477）。这些位点的甲基化差异更需要结合 local read pileup、MAPQ 分布、多重比对比例和外部 sperm methylation dataset 方向一致性复核。
- **blacklist 未命中**：9 个特征均未 overlap ENCODE hg19 blacklist v2。

### 8.4 尚未完成、但建议作为下一步

1. 从 GSE156108 bigWig/原始 ChIP-seq 统一 call sperm H3K4me3 peaks，再与 9 个特征 core/±250bp overlap。
2. 用 sperm DNA methylation bigWig 定义高甲基化区间，进一步得到 sperm H3K4me3 + DNA methylation overlap region。
3. 单独寻找 embryonic enhancer 与 PGC reprogramming escape region 的 hg19 BED，再做同样 overlap。
4. 对 DMR2/DMR3/DMR5/chr22 等 repeat-associated features，补 BAM 级别 QC：read pileup、MAPQ、soft-clipping、多重比对比例、strand balance。

本次生成的明细表：

- `C:\Users\liran\Documents\Codex\2026-07-01\bu\outputs\9features_regulatory_qc_summary.tsv`
- `C:\Users\liran\Documents\Codex\2026-07-01\bu\outputs\9features_all_overlap_hits.tsv`
- `C:\Users\liran\Documents\Codex\2026-07-01\bu\outputs\9features_repeatmasker_exact_pm250.tsv`
