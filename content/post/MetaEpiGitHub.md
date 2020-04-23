---
title: "Meta流行病学"
author: '章仲恒; 浙江大学医学院附属邵逸夫医院急诊医学科. Email: zh_zhang1984@zju.edu.cn'
csl: elsevier-vancouver.csl
output: 
  html_document:
    keep_md: true
    df_print: paged
    toc: true
    toc_float: true
    number_sections: true
    theme: united 
    highlight: tango
  github_document:
    toc: true
    toc_depth: 2
  word_document:
    reference_docx: word-refernece-style.docx
bibliography: ref.bib
---



# Meta流行病学概述
“Meta流行病学”一词最早由Naylor在1997年提出[@Naylor:1997ij]，随后Sterne等通过系统的研究，正式提出了Meta流行病学的定义：**即采用统计学的方法来评估研究水平的某些特征对疗效的影响[@Sterne:2002iq]。**一般来说，假如把传统的流行病学看做是对病人个体水平的危险因素的研究，那么Meta流行病学则是对一项原创性研究某些特征因素的研究。这些研究水平的因素就包括样本量[@Zhang:2013jv]、发表的语言、研究资助类型等[@Janiaud:2018fb]。如果是针对随机对照研究（RCT）的特征进行分析，很多Meta流行病学研究会集中在研究质量上。评估RCT质量的几个要素主要包括：盲法、随机数的产生、分配隐藏、失访等。另外，Meta流行病学也可认为是Meta分析加上流行病学研究。传统的Meta分析是指对多项研究的结果通过统计学的方法进行整合，从而得出一个更加确切的结论。而Meta流行病学中我们需要对各项研究的特征进行提取，最后进行整合，符合传统Meta分析的定义。  

为什么需要进行Meta流行病学分析呢？那是因为在进行系统评价和Meta分析过程中研究者发现，各项研究之间会存在很大的异质性，而解释这些异质性就需要对各项原始研究的特征进行分析[@Bae:2014by]。比如有研究发现样本量较小的研究更倾向于报道阳性的较好的疗效[@Zhang:2013jv]，经过这样的实证性分析，我们就可以做出推测：如果一个小样本的研究得出阴性结论，其发表的可能性较小。这就是发表偏倚的可能成因。

目前文献中采用Meta流行病学方法来做的研究很多，笔者用“Meta&nbsp;epidemiological”作为关键词在PubMed进行了粗略检索，结果发现，在2000年的时候文献数量为300篇左右，而2018年达到了5000余篇。这个增长趋势也表明Meta流行病学在循证医学中扮演者重要的角色，研究者对该领域的关注也是与日俱增。因此我们编写了Meta流行病学这一章节，期待能给相关领域的研究者提供一些参考。 

# Meta流行病学统计方法及R语言的实现
本章节中Meta流行病学统计方法与R语言联合讲解，通过R代码来诠释统计学算法，这样能达到即学即用的效果，避免读者面对枯燥的统计学公式而无从下手的尴尬局面。

## 模拟数据
本章节采用模拟数据的方式介绍如何进行Meta流行病学分析，本文的R代码主要来自于笔者已发表的一篇文献[@Zhang:2016ga]，但有针对性地做了一些修改和更新。首先按照如下代码建立一个研究数据集。该数据集包含5个Meta分析，每个Meta分析包含5-8个研究，所有Meta分析所纳入的原始研究中，每项特征至少存在于1项原始研究中。如研究发表的语言包含中文、英文、日文，那么每项Meta分析所包含的原始研究中，这三种语言任意一种至少出现在一项原始研究中。研究的观察终点用二分类变量来表示。

```r
> set.seed(888)  #设定种子保证结果可重复
> event <- rep(0:1, 60)
> Freq <- abs(round(rnorm(120, 50, 20)))
> treat <- rep(rep(0:1, 30), each = 2)
> study <- rep(1:30, each = 4)
> char <- rep(rbinom(30, 1, 0.5), each = 4)
> meta <- c(rep(1, 16), rep(2, 20), rep(3, 
+     24), rep(4, 28), rep(5, 32))
> data <- data.frame(meta, study, char, treat, 
+     event, Freq)
> str(data)
```

```
## 'data.frame':	120 obs. of  6 variables:
##  $ meta : num  1 1 1 1 1 1 1 1 1 1 ...
##  $ study: int  1 1 1 1 2 2 2 2 3 3 ...
##  $ char : int  0 0 0 0 1 1 1 1 1 1 ...
##  $ treat: int  0 0 1 1 0 0 1 1 0 0 ...
##  $ event: int  0 1 0 1 0 1 0 1 0 1 ...
##  $ Freq : num  11 19 65 44 17 45 7 62 63 72 ...
```

```r
> head(data)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["meta"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["study"],"name":[2],"type":["int"],"align":["right"]},{"label":["char"],"name":[3],"type":["int"],"align":["right"]},{"label":["treat"],"name":[4],"type":["int"],"align":["right"]},{"label":["event"],"name":[5],"type":["int"],"align":["right"]},{"label":["Freq"],"name":[6],"type":["dbl"],"align":["right"]}],"data":[{"1":"1","2":"1","3":"0","4":"0","5":"0","6":"11","_rn_":"1"},{"1":"1","2":"1","3":"0","4":"0","5":"1","6":"19","_rn_":"2"},{"1":"1","2":"1","3":"0","4":"1","5":"0","6":"65","_rn_":"3"},{"1":"1","2":"1","3":"0","4":"1","5":"1","6":"44","_rn_":"4"},{"1":"1","2":"2","3":"1","4":"0","5":"0","6":"17","_rn_":"5"},{"1":"1","2":"2","3":"1","4":"0","5":"1","6":"45","_rn_":"6"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>
上述代码首先设定一个种子，保证数据结果可重复性。接着我们用*rep()*函数构建了一个取值为0或1的二分类变量*event*来表示感兴趣的事件（在实际研究中一般表示结局变量如死亡、患病与否等）是否发生，一共重复60次。*Freq*代表每种事件出现的频次。数据集构建完成后我们可以用*head()*函数查看，该输出结果的第一行表示第一项Meta分析中的第一项研究，该研究不存在某特征（char=0），该研究中对照组总共$11+19=30$人，其中事件发生数为$19$，而干预组总共$65+44=109$人，其中事件发生数为$44$人。这样的一个表格的数据通常可以从发表的原始文献中进行提取。本章节为了展示方便，采用数据模拟方式产生数据，同时也方便读者练习理解。

## Logistic回归法
Meta流行病研究探究研究水平的因素对效应大小的影响可以通过标准的回归模型进行[@Sterne:2002iq]。二分类Logistic回归模型可以写成：$$Logit(\pi)=\beta_0+\beta_1 I_t+\beta_2I_{tc}+\sum_{i=2}^{m}\gamma_iI_{tm_i}+\sum_{j=2}^{s}\delta_jI_{s_j}，
$$其中$\pi$表示事件发生的可能性，这里做了logit变换；$\beta_0$为截距，$I_t$代表治疗组别（治疗组$I_t=1$，对照组$I_t=0$），其相应的$\beta_1$为治疗效应大小。$I_{tc}$为治疗和特征交互作用（对于特征存在的治疗组患者$I_{tc}=1$，其余患者$I_{tc}=0$），$I_{tm_i}$为治疗和Meta分析的交互项（第i项Meta分析中的治疗组患者$I_{tm_i}=1$，其余患者为0），该交互项的存在意味着治疗效果在不同的Meta分析中可以不同。$I_{s_j}$为研究的编号（j项研究中的患者$I_{s_j}=1$，其余为0）。从上式可以看出我们感兴趣的参数是$\beta_2$，其大小估计了研究特征对效应的影响：$ROR=e^{\beta_2}$。其中ROR全称为Ratio&nbsp;of&nbsp;Odds&nbsp;Ratio，其中Odds&nbsp;Ratio（OR）值可以反应疗效的大小（干预组和对照组Odds的比值），那么含有某种特征的研究中报道的疗效（$OR_{present}$）跟不含某种特征的研究中的疗效（$OR_{abscent}$）的比值（ROR）则可以反应该特征的影响。其正式的数学表达式如下：当某种特征存在时（$I_c=1$），干预组事件发生可能性为：$$\pi_{treated}=\frac{e^{\beta_0+(\beta_1+\beta_2)I_t+\sum_{i=2}^{m}\gamma_iI_{tm_i}+\sum_{j=2}^{s}\delta_jI_{s_j}}}{1+e^{\beta_0+(\beta_1+\beta_2)I_{t}+\sum_{i=2}^{m}\gamma_iI_{tm_i}+\sum_{j=2}^{s}\delta_jI_{s_j}}},$$干预组事件不发生的可能性为：$$1-\pi_{treated}=\frac{1}{1+e^{\beta_0+(\beta_1+\beta_2)I_{t}+\sum_{i=2}^{m}\gamma_iI_{tm_i}+\sum_{j=2}^{s}\delta_jI_{s_j}}},$$则治疗组的$$odds_{treated}=\frac{\pi_{treated}}{1-\pi_{treated}}=e^{\beta_0+(\beta_1+\beta_2)I_{t}+\sum_{i=2}^{m}\gamma_iI_{tm_i}+\sum_{j=2}^{s}\delta_jI_{s_j}}$$同理可得对照组$$odds_{ctrl}=e^{\beta_0+\sum_{j=2}^{s}\delta_jI_{s_j}}$$干预手段的效应大小为：$OR_{present}=e^{\beta_1+\beta_2+\sum_{i=2}^{m}\gamma_iI_{m_i}}$；当该特征不存在时，其效应大小为：$OR_{abscent}=e^{\beta_1+\sum_{i=2}^{m}\gamma_iI_{m_i}}$，因此$ROR= \frac{OR_{present}}{OR_{abscent}}=e^{\beta_2}$。当$ROR>1$时，某特征的存在使效应值OR高估，相反，某特征可使效应值低估。另外由该公式也可以看出，该模型需要估计$m+s+1$个参数。这种方法假设特征对效应大小的影响在所有的Meta分析中是一致的（ROR值不随Meta分析的变化而变化，如果假设ROR在Meta分析中不一致，则需要增加交互项$I_{tcm_i}$），如果这个假设不能成立，则估算获得ROR的标准误会偏小。

上述公式也可进行扩展同时纳入多个研究特征，从而进行混杂因素的矫正[@Siersma:2007ci]。$$Logit(\pi)=\sum_{k=1}^{p}\beta_kI_{tc_k}+\sum_{i=2}^{m}\gamma_iI_{tm_i}+\sum_{j=2}^{s}\delta_jI_{s_j},$$
假设总共有p个特征，该公式中$I_{tc_k}$代表了干预和特征k的交互（治疗组$I_t=1$中，特征$I_c_k=1$的患者$I_{tc_k}=1$，否则为0），进行回归分析后每个特征都是独立的因素，每个特征效应大小用$\beta_k$来表示。这样的算法一般可用于混杂因素的矫正去研究某特征对效应值的影响是否稳健。

上面的公式需要患者水平的数据，我们需要对*data*进行变换从而使每行代表一个患者。

```r
> dataExpanded <- data[rep(rownames(data), 
+     data$Freq), 1:(ncol(data) - 1)]
> str(dataExpanded)
```

```
## 'data.frame':	5900 obs. of  5 variables:
##  $ meta : num  1 1 1 1 1 1 1 1 1 1 ...
##  $ study: int  1 1 1 1 1 1 1 1 1 1 ...
##  $ char : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ treat: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ event: int  0 0 0 0 0 0 0 0 0 0 ...
```
上面结果表明，*dataExpanded*将原来的*data*展开了，每行代表一个患者，一共有5900例患者，而该数据框包含了5个变量。
接着我们将*meta*和*study*两个变量转化为哑变量。

```r
> library(dummies)
```

```
## dummies-1.5.6 provided by Decision Patterns
```

```r
> data.dummy <- dummy.data.frame(dataExpanded, 
+     names = c("meta", "study"))
```

```
## Warning in model.matrix.default(~x - 1, model.frame(~x - 1), contrasts = FALSE):
## non-list contrasts argument ignored

## Warning in model.matrix.default(~x - 1, model.frame(~x - 1), contrasts = FALSE):
## non-list contrasts argument ignored
```

```r
> str(data.dummy)
```

```
## 'data.frame':	5900 obs. of  38 variables:
##  $ meta1  : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ meta2  : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ meta3  : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ meta4  : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ meta5  : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study1 : int  1 1 1 1 1 1 1 1 1 1 ...
##  $ study2 : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study3 : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study4 : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study5 : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study6 : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study7 : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study8 : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study9 : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study10: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study11: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study12: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study13: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study14: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study15: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study16: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study17: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study18: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study19: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study20: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study21: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study22: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study23: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study24: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study25: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study26: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study27: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study28: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study29: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ study30: int  0 0 0 0 0 0 0 0 0 0 ...
##  $ char   : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ treat  : int  0 0 0 0 0 0 0 0 0 0 ...
##  $ event  : int  0 0 0 0 0 0 0 0 0 0 ...
##  - attr(*, "dummies")=List of 2
##   ..$ meta : int  1 2 3 4 5
##   ..$ study: int  6 7 8 9 10 11 12 13 14 15 ...
```
上面的*library()*函数将*dummies*这个程辑包加载到了工作台，如果读者之前没有下载安装过这个包，还需要运行*install.packages("dummies")*这个命令。我们查看一下*data.dummy*的结构发现，其行数保持5900行不变，但列数增加了很多，这里每个研究都用0或1来标识，例如*study7*变量下如果取值是1则标识这些患者属于第7项研究。这种哑变量化的方法又可以称之为独热编码（One-Hot&nbsp;Encoding）。此处为便于理解，笔者将哑变量转化的过程展示出来，实际上，在R语言中我们也可将以数值表示的分类变量用*as.factor()*函数进行因子化，后续的建模中就可以自动算出因子变量每个水平的OR值。

行文至此，我们已经完成了数据框的准备，接着我们就可以进行模型的拟合。

```r
> model.logit <- glm(event ~ . + treat:char + 
+     treat:meta2 + treat:meta3 + treat:meta4 + 
+     treat:meta5 - char - study1 - meta1 - 
+     meta2 - meta3 - meta4 - meta5, data = data.dummy, 
+     family = "binomial")
```
在R语言中，我们可以用*glm()*函数来拟合Logistic回归模型，这里我们将Logistic回归理解为广义线性模型（generalized&nbsp;linear&nbsp;model）大家族中的一种，其中连接函数就是Logit函数$Logit(\pi)=log(\frac{\pi}{1-\pi})$。*glm()*函数的第一个赋值就是一个公式，“~”符号的左边为应变量，右边为预测因子。其中“.”代表将*data.dummy*数据框中的除应变量外的所有因素都纳入作为自变量，随后可以用$+$或$-$添加或删除一些因子。
模型拟合后我们可以查看结果。

```r
> library(tableone)
> ShowRegTable(model.logit)
```

```
##             exp(coef) [confint] p     
## (Intercept) 1.16 [0.74, 1.82]    0.523
## study2      3.64 [2.03, 6.68]   <0.001
## study3      1.03 [0.64, 1.63]    0.916
## study4      0.65 [0.39, 1.07]    0.088
## study5      0.88 [0.51, 1.54]    0.660
## study6      0.97 [0.56, 1.70]    0.922
## study7      1.21 [0.69, 2.11]    0.502
## study8      0.61 [0.37, 1.03]    0.064
## study9      0.67 [0.39, 1.16]    0.158
## study10     1.25 [0.73, 2.14]    0.412
## study11     0.68 [0.38, 1.19]    0.175
## study12     1.47 [0.83, 2.59]    0.184
## study13     0.74 [0.43, 1.27]    0.277
## study14     0.71 [0.41, 1.24]    0.231
## study15     0.79 [0.47, 1.32]    0.363
## study16     0.93 [0.54, 1.60]    0.792
## study17     0.60 [0.35, 1.02]    0.059
## study18     0.36 [0.18, 0.71]    0.003
## study19     1.59 [0.91, 2.76]    0.102
## study20     1.36 [0.80, 2.32]    0.256
## study21     0.47 [0.25, 0.89]    0.020
## study22     1.53 [0.90, 2.62]    0.119
## study23     0.93 [0.56, 1.55]    0.790
## study24     0.51 [0.30, 0.87]    0.014
## study25     0.30 [0.16, 0.56]   <0.001
## study26     0.77 [0.44, 1.36]    0.373
## study27     1.45 [0.82, 2.58]    0.203
## study28     0.94 [0.56, 1.55]    0.798
## study29     0.51 [0.28, 0.93]    0.029
## study30     0.97 [0.55, 1.72]    0.927
## treat       0.65 [0.44, 0.96]    0.030
## char:treat  1.71 [1.32, 2.21]   <0.001
## meta2:treat 1.20 [0.80, 1.81]    0.375
## meta3:treat 0.93 [0.63, 1.38]    0.713
## meta4:treat 1.45 [0.94, 2.23]    0.094
## meta5:treat 1.83 [1.24, 2.72]    0.002
```
从上表可以看出，该模型一共拟合了$m+s+1=5+30+1=36$个参数，其中交互项*char:treat*的估计系数经过以e为底的指数函数转化后$ROR=1.71$，且$p<0.001$，表明该特征的存在能使研究高估治疗效果（OR值偏高）。
因为ROR在Meta分析内一致的假设有可能不对，而这种假设直接后果是低估ROR的标准误，因此可采用信息三明治（information&nbsp;sandwich）法去估计稳健性（robust）标准误[@32225]。

```r
> library(sandwich)
> sandwich_se <- diag(vcovHC(model.logit, type = "HC"))^0.5
> sandwich_se["char:treat"]
```

```
## char:treat 
##  0.1315834
```
上面的代码首先加载*sandwich*程辑包，接着我们用*vcovHC()*函数计算模型系数的方差-协方差矩阵。因为模型的系数非常多，所以这里不做展示。该矩阵的对角线上的值表示系数的方差，取平方根则可得到标准误。上面的结果表明，*char:treat*这个项的标准误为0.13。

```r
> # 传统方法计算标准误
> summary(model.logit)$coefficients["char:treat", 
+     "Std. Error"]
```

```
## [1] 0.1325405
```
上面计算结果表明，稳健性标准误（0.1315834）与之前传统方法计算的值（0.1325405）非常接近。因此可以认为ROR在各项Meta分析中是一致的。
有了标准误后我们可进一步估算95%置信区间及假设检验的p值。

```r
> # 95%置信区间下限
> coef(model.logit)["char:treat"] - 1.96 * 
+     sandwich_se["char:treat"]
```

```
## char:treat 
##   0.276612
```

```r
> # 95%置信区间上限
> coef(model.logit)["char:treat"] + 1.96 * 
+     sandwich_se["char:treat"]
```

```
## char:treat 
##  0.7924189
```

```r
> Z <- coef(model.logit)["char:treat"]/sandwich_se["char:treat"]
> Pval <- pchisq(Z^2, 1, lower.tail = FALSE)
> Pval
```

```
##   char:treat 
## 4.861654e-05
```

## Meta分析法
在实际研究工作中纳入Meta流行病学研究的每项Meta分析一般探讨不同的主题，如我们之前发表的一篇Meta流行病学研究探讨了小样本效应在危重症医学中的作用[@Zhang:2013jv]。从Meta流行病学研究的水平来看我们以危重症作为研究主题，里面纳入的27项Meta分析探讨不同亚专科问题，包括激素的使用、镇静镇痛、撤机策略及降钙素原导向的抗生素治疗等。这些不同的主题很可能导致Meta分析之间的异质性，因此多数情况下我们会假设**研究特征对效应大小的影响在不同主题的Meta分析中是不同的。**此时我们可以在每项Meta分析内先求出ROR值，然后用Meta分析方法将数据进行合并。根据是否存在异质性，我们可以选择固定效应模型或者随机效应模型[@DerSimonian:1986wm]。这样的方法又可称之为“Meta-meta分析”。这种方法是在每项Meta分析下求ROR，所以就要去掉之前的模型公式里的Meta分析项。$$Logit(\pi)=\beta_0+\beta_1 I_t+\beta_2I_{tc}+\sum_{j=2}^{s}\delta_jI_{s_j}
$$上面公式各项代表的意义与前面相同，只是这里的研究编号$s_j$表示在某个Meta分析内每项研究的编号，假设每项Meta分析内的原始研究编号均从1开始，我们可用如下代码完成原始研究的重新编号：

```r
> data1 <- dataExpanded[dataExpanded$meta == 
+     1, -1]
> data2 <- dataExpanded[dataExpanded$meta == 
+     2, -1]
> data2$study <- data2$study - length(unique(dataExpanded[dataExpanded$meta == 
+     1, "study"]))
> data3 <- dataExpanded[dataExpanded$meta == 
+     3, -1]
> data3$study <- data3$study - length(unique(dataExpanded[dataExpanded$meta <= 
+     2, "study"]))
> data4 <- dataExpanded[dataExpanded$meta == 
+     4, -1]
> data4$study <- data4$study - length(unique(dataExpanded[dataExpanded$meta <= 
+     3, "study"]))
> data5 <- dataExpanded[dataExpanded$meta == 
+     5, -1]
> data5$study <- data5$study - length(unique(dataExpanded[dataExpanded$meta <= 
+     4, "study"]))
```
上述代码容易理解，但较为繁琐，我们也可以用更简洁的代码完成上述数据格式的重整。

```r
> library(plyr)
> DataList <- dlply(dataExpanded, .(meta), 
+     function(xx) {
+         xx$study <- xx$study - length(unique(dataExpanded[dataExpanded$meta <= 
+             xx$meta[1] - 1, "study"]))
+         xx$study <- as.factor(xx$study)  #因子化
+         xx$meta <- NULL  #去掉meta这个变量
+         return(xx)
+     })
```
上述代码将拆分后的5套数据框存放在一个*DataList*这个list对象下面。这样的方法适用于含有任意多个Meta分析的情况。以下代码就基于*DataList*继续进行模型的拟合。

```r
> Mods <- lapply(DataList, function(xx) {
+     mod <- glm(event ~ . + treat:char - char, 
+         data = xx, family = "binomial")
+ })
> # 查看任意一项Meta分析内的结果
> ShowRegTable(Mods[[2]])
```

```
##             exp(coef) [confint] p     
## (Intercept) 0.96 [0.70, 1.31]    0.790
## study2      1.05 [0.68, 1.60]    0.838
## study3      1.27 [0.84, 1.92]    0.253
## study4      1.08 [0.66, 1.76]    0.751
## study5      0.75 [0.49, 1.14]    0.177
## treat       0.35 [0.20, 0.61]   <0.001
## char:treat  4.84 [2.59, 9.16]   <0.001
```
接下来我们可以将模型的结果提出，进行Meta分析数据合并。

```r
> Results <- lapply(Mods, function(xx) {
+     c(coef(xx)["char:treat"], summary(xx)$coefficients["char:treat", 
+         2])
+ })
> DTlin <- t(as.data.frame(Results))
> colnames(DTlin) <- c("log(ROR)", "SE")
> rownames(DTlin) <- 1:5
> DTlin
```

```
##     log(ROR)        SE
## 1  1.1334081 0.4611690
## 2  1.5775890 0.3216732
## 3  0.6229919 0.3194935
## 4  1.5948729 0.3660628
## 5 -0.4993242 0.2102525
```
上面语句中我们用了*lapply()*函数对list对象进行操作，该函数优点是能将某个函数作用于list对象内的每个元素。在该例子中list对象*Mods*的子元素是拟合的回归模型，我们提取每个模型的*char:treat*项的系数和标准误，接着进行一些列数据框的变化操作，最终生成了一个包含有*log(ROR)*和*SE*两列的数据框，可用于后续操作。

```r
> library(meta)
```

```
## Loading 'meta' package (version 4.11-0).
## Type 'help(meta)' for a brief overview.
```

```r
> meta <- metagen(DTlin[, 1], DTlin[, 2], studlab = paste("Meta", 
+     1:5, sep = ""), sm = "log(ROR)")
> forest(meta)
```

![](MetaEpiGitHub1_files/figure-html/MetaAnalysis-1.png)<!-- -->

Meta分析的森林图（图1）结果表明，ROR值在各项Meta分析中分布存在一定异质性，且异质性检验的$p<0.01$。图中显示了固定效应模型（0.50，95%CI：0.24-0.77）和随机效应模型（0.86，95%CI：-0.07-1.80）的合并值。注意图中的结果为log(ROR)值，可通过$ROR_{fixed}=e^{0.50}=1.65$或$ROR_{rand}=e^{0.86}=2.36$转化成ROR值。由该结果可知，固定效应模型的置信区间较窄，因为固定效应模型假设ROR在各项Meta分析中没有差异，都是对同一个总体的估计。另外需要注意，*metagen()*函数采用逆方差权重法（Inverse&nbsp;variance&nbsp;weighing）合并数据，即$$ES_{pooled}=\sum_{i=1}^{k}\frac{w_iES_i}{w_i},$$其中$w_i=\frac{1}{var_i},$其中$ES_i$代表每项研究的效应值，$w_i$代表每项研究的权重，而权重是方差$var_i$的倒数。需要用log(ROR)及其标准误进行合并，然后必要时再转化成ROR值。ROR的方差不是简单地把log(ROR)的方差取以e为底的指数函数：$ROR=exp(log(ROR));但Var(ROR)\neq exp(Var_{log(ROR)})$。

上面的方法在每项Meta分析内建立Logistic回归模型来估算*char:treat*项的系数，这假定了Meta分析内原始研究之间不存在异质性（Between-trial-heterogeneity），即每项原始研究估算同一个效应值。当这个假设不成立时，我们可采用随机效应meta回归模型进行分析。本书的其它章节曾经提到Meta回归主要研究原始研究水平的特征对效应值大小的影响，而这个特征可以是连续性变量（比如平均年龄、样本量、研究人群所在的地球纬度）。因此我们可以用Meta回归的方法获得ROR来比较不同特征研究之间的差异。

首先我们将数据框进行整理使得每行代表一项研究。

```r
> library(reshape)
```

```
## 
## Attaching package: 'reshape'
```

```
## The following objects are masked from 'package:plyr':
## 
##     rename, round_any
```

```r
> dtTrial <- cast(data, meta + study + char ~ 
+     treat + event, value = "Freq")
> colnames(dtTrial)[4:7] <- c("Cnonevent", 
+     "Cevent", "Tnonevent", "Tevent")
> dtTrial$Ntreat <- dtTrial$Tnonevent + dtTrial$Tevent
> dtTrial$Ncontrl <- dtTrial$Cnonevent + dtTrial$Cevent
> head(dtTrial)
```

<div data-pagedtable="false">
  <script data-pagedtable-source type="application/json">
{"columns":[{"label":[""],"name":["_rn_"],"type":[""],"align":["left"]},{"label":["meta"],"name":[1],"type":["dbl"],"align":["right"]},{"label":["study"],"name":[2],"type":["int"],"align":["right"]},{"label":["char"],"name":[3],"type":["int"],"align":["right"]},{"label":["Cnonevent"],"name":[4],"type":["dbl"],"align":["right"]},{"label":["Cevent"],"name":[5],"type":["dbl"],"align":["right"]},{"label":["Tnonevent"],"name":[6],"type":["dbl"],"align":["right"]},{"label":["Tevent"],"name":[7],"type":["dbl"],"align":["right"]},{"label":["Ntreat"],"name":[8],"type":["dbl"],"align":["right"]},{"label":["Ncontrl"],"name":[9],"type":["dbl"],"align":["right"]}],"data":[{"1":"1","2":"1","3":"0","4":"11","5":"19","6":"65","7":"44","8":"109","9":"30","_rn_":"1"},{"1":"1","2":"2","3":"1","4":"17","5":"45","6":"7","7":"62","8":"69","9":"62","_rn_":"2"},{"1":"1","2":"3","3":"1","4":"63","5":"72","6":"56","7":"77","8":"133","9":"135","_rn_":"3"},{"1":"1","2":"4","3":"1","4":"69","5":"58","6":"40","7":"27","8":"67","9":"127","_rn_":"4"},{"1":"2","2":"5","3":"1","4":"74","5":"46","6":"6","7":"43","8":"49","9":"120","_rn_":"5"},{"1":"2","2":"6","3":"1","4":"49","5":"44","6":"35","7":"66","8":"101","9":"93","_rn_":"6"}],"options":{"columns":{"min":{},"max":[10]},"rows":{"min":[10],"max":[10]},"pages":{}}}
  </script>
</div>
上面代码中用*cast()*函数将*data*转化成横表的格式，即每行代表一项研究，因此最后的数据框出现了对照组、干预组的例数。接着我们就可以在Meta分析水平进行Meta回归。在进行Meta回归之前先用*metabin()*函数针对二分类结局进行合并值的估计。

```r
> dtTrialList <- split(dtTrial, dtTrial$meta)
> Metas <- lapply(dtTrialList, function(xx) {
+     metabin(Tevent, Ntreat, Cevent, Ncontrl, 
+         data = xx, sm = "OR", byvar = xx$char)
+ })
> forest(Metas[[5]])
```

![](MetaEpiGitHub1_files/figure-html/MetaReg-1.png)<!-- -->

```r
> bubble(metareg(Metas[[5]], ~char))
```

![](MetaEpiGitHub1_files/figure-html/MetaReg-2.png)<!-- -->
上述的代码首先将*dtTrial*以Meta分析为单位拆分成n个元素，并存放在*dtTrialList*这个list中，接着我们可以用*lapply()*函数在每个meta分析内部做数据合并，同时设定亚组的分组变量为*char*。完成Meta分析内的数据合并后，我们可以进一步对合并结果进行可视化，例如我们可以对第5项Meta分析结果展示森林图（图2）和气泡图（图3）。

```r
> MetaRegs <- lapply(Metas, function(xx) {
+     metareg(xx)
+ })
> MetaRegCoef <- lapply(Metas, function(xx) {
+     c(metareg(xx)$b[2], metareg(xx)$se[2])
+ })
> MetaRegCoef <- do.call(rbind, MetaRegCoef)
> colnames(MetaRegCoef) <- c("Beta", "SE")
> MetaRegCoef
```

```
##         Beta        SE
## 1  1.2269612 0.7188293
## 2  1.7007555 1.4757099
## 3  0.6133007 1.0034691
## 4  1.7354172 0.8353027
## 5 -0.6859661 0.7650031
```
上面的语句中我们用*lapply()*函数对每项Meta分析做Meta回归，并将回归的结果存放在*MetaRegs*这个list对象下面。另外我们还提取了所有Meta回归中的回归系数和标准误，这些结果存放在*MetaRegCoef*下面。后面的*do.call()*函数调用了*rbind()*函数将list对象*MetaRegCoef*转化成数据框形式。我们可以查看一下*MetaRegCoef*的具体内容。

```r
> MetaPool <- metagen(MetaRegCoef[, "Beta"], 
+     MetaRegCoef[, "SE"], studlab = paste("Meta", 
+         1:5))
> forest(MetaPool)
```

![](MetaEpiGitHub1_files/figure-html/MetsPool-1.png)<!-- -->
有了*char*的回归系数和标准差后，我们就可以用meta分析的方法用*metagen()*函数将数据进行合并，同时绘制森林图（图4）。

# Meta流行病学报告规范
为了使各种临床研究报告规范化，学术界针对不同的研究类型相继提出了一系列的报告规范。例如，对于针对系统评价及Meta分析我们有PRISMA（Preferred Reporting Items for Systematic Reviews and&nbsp;Meta-analyses）标准，对于个案报道有CARE标准，对于RCT有CONSORT标准。这些标准以核查表的形式要求作者在论文写作过程当中严格按照一定的规范对研究结果进行报道。目前国际各种主流期刊均要求作者在投稿的时候将核查表以附件的形式和临床研究报告一起递交。这些报告规范的执行和实施使得研究报告更加规范、透明。同时也方便读者、审稿人和编辑对文章质量的把控。对于Meta流行病研究的报告，目前也有相应的报告规范[@Murad:2017ew]。下面的内容就该报告规范进行一系列的解读，同时结合一些案例进行分析，让读者在写文章的时候有规范可遵循。

## 标题和摘要
标题一般需要明确该文章是个Meta流行病学研究，并且一个好的标题应该包含研究特征、研究范围和主要研究结论。比如对于“Small studies may overestimate the effect sizes in critical care meta-analyses: a meta-epidemiological&nbsp;study”这样一个标题[@Zhang:2013jv]，读者一眼就能知道这是个Meta流行病学研究，研究的特征是样本量的大小，研究集中的范围是重症医学领域，而研究主要结论是小样本研究容易高估效应值。从中可以看出，一个好的标题应该用最少的字传达最丰富的信息。

摘要一般为结构式摘要，包含研究背景、研究假设及目标、研究方法、数据来源、结果、局限性以及结论。里面包含的内容基本上就是文章后面所要展开阐释的东西。从某种意义上来说，摘要是论文的窗口，透过这个窗口审稿人、编辑或读者就能一览全文概况，然后再决定是否需要（有兴趣）看看具体细节。如果一个摘要不吸引人，或者没有把主要内容传达出来，编辑也就懒得再往下看，这就是为什么很多论文被直接拒稿的原因。各部分内容因为下面还会谈及，在此就不一一展开。

## 背景简介
此部分内容起到搭建舞台背景的作用，内容主要包括研究背景，既往研究的不足、该研究能如何弥补这样的不足。如果是投稿综合性期刊，需要对专业知识进行简单的介绍，使得一些大同行也能顺利阅读你的论文。如下面一段文字[@Panagiotou:2013ir]：

> The predominant share of the global burden of disease is concentrated in less developed countries. However, until recently relative few trials were being conducted in these nations. Evidence on the management of many diseases affecting less developed countries often had to be tentatively extrapolated from studies performed in more developed countries with a longer standing tradition of conducting clinical research. This situation is currently changing. As participation rates in clinical trials decrease and cost increases in Western countries, many organisations providing contracts for research are focusing on eastern Europe, Asia, and South America, where the cost of recruiting participants is low. 

这篇文章探讨了一项研究在不同发展程度的国家（发达国家vs发展中国家）开展对效应值的影响。文章开篇首先提出发展中国家疾病负担很重，但其诊疗的主要依据却来自发达国家，但由于发展中国家患者入组较为便宜，来自发展中国家的研究不断增多。

>Trials carried out in less developed countries may differ in important aspects from those done in countries with stronger traditions in clinical research. Firstly, publication dynamics and biases may differ. ...

文章第二段开始分析研究来自不同发达程度的国家可能会导致结果的不同。但这些理论上的推测需要实证性数据的支持，文章接着就自然而然地过渡到了本文要做的研究：

>We performed a large scale assessment of meta-analyses on topics with randomised evidence from more developed and less developed countries. We assessed how often randomised trials performed in these countries with different levels of development and different traditions in clinical research give different results, whether treatment effects are systematically larger in one setting than the other, and whether discordant effects are the result of bias or genuine differences.

背景的最后一段提出了本研究要做的内容，上面一段例子中作者就非常明确提出本文将要研究的内容：1、随机对照研究在不同发达程度地区开展的普遍程度；2、在不同地区开展的研究其效应值是否有系统性差异；3、这种差异是偏倚引起的还是确实存在的真实差异。一般来说在研究简介的最后还会加上一些研究假设，如下面的例子[@Abraha:2015by]：

> Given the preponderance of significant results among the trials that have deviated from the intention to treat approach, we hypothesise that such trials overestimate the treatment effect compared with those trials reporting standard intention to treat analysis independent from the occurrence of exclusions.

## 方法
### 研究方案注册及发表
一般而言，一项系统评价需要进行数据分析前的注册，以避免拿到数据并分析后对结果选择性报道。一项Meta流行病学研究是否需要事先注册目前尚未形成统一的建议[@Murad:2017ew]。笔者认为，虽然Meta流行病学研究所探讨的特征一般是固定的，但仍可以选择性报道某些效应值，从而导致偏倚发生，例如经过分析可能发现某特征针对死亡率的ROR值大于1，而针对再入院率的ROR小于1，这时如果事先没有注册研究方案，研究者很可能只报道符合自己事先假设的结果。因此，事先注册或发表研究方案能有效避免此类报道偏倚的发生。跟传统的系统评价一样，我们可以通过专门的网站进行注册：https://www.crd.york.ac.uk/PROSPERO/ 。也可以将研究方案发表在一些杂志上如BMJ open、Medicine以及Systematic Reviews等。

### 纳排标准
接下来是确定研究的入选标准。该部分需要限定研究主题、研究类型、研究特征，并且说明这样做的理由。

> We analysed only systematic reviews of therapeutic or preventive interventions, with at least one meta-analysis with categorical data, each containing at least two randomised trials of which at least one had to report a deviation from the intention to treat approach. Reviews with meta-analyses that included only trials using a deviation were excluded. 
Diagnostic, prognostic, epidemiological, and cost-effectiveness studies; animal studies; and reviews with meta-analyses of cluster or crossover trials were excluded. We excluded non-English language reviews. Pairs of investigators independently assessed relevant reviews for eligibility.

上面这段话就设定了纳入标准[@Abraha:2015by]，纳入研究的系统评价至少纳入两项RCT，其中至少有一项RCT报道了偏离意向性分析。排除标准包括诊断实验、预后研究、流行病项研究等等。这些纳排标准的设定需要概念清晰，有较强的可执行性。

### 数据库及检索策略 
接下来是设定数据库及检索策略。其标准案例如下[@Zhang:2013jv]：

> Medline and Embase databases were searched from inception to August 2012. There was no language restriction. The core search terms consisted of critical care, mortality and meta-analysis (detailed search strategy is shown in Additional file 1).

其中检索策略可能较为复杂，故可以用附件的形式加以详细说明。

### 数据提取
此部分需要描述如何从原始文献中提取数据，比如制作标准表格、双人互相独立提取数据以保证其准确性、联系原文作者获取原始文献中没有报道的信息等。另外还要明确规定需要从原始文献中提取那些变量，以及对变量做何种转换。比如提取研究是否在发展中国家开展这个特征，那么就需要界定发展中国家的定义，对包含发达国家和发展中国家参与的多中心研究该如何界定等实操性问题。

### 采用何种统计学方法
此部分主要描述如何选择统计方法（Logistic回归法、meta分析法），以及这样选择的前提假设，比如是否存在Meta分析间的异质性、是否存在原始研究之间的异质性等。不同的假设可导致很不相同的结果，甚至相反的结论。另外，因为Meta流行病学研究一般报道ROR来衡量一种特征的影响大小及方向，而一般读者可能并不熟悉ROR，故作者还需对ROR进行解释说明。比如下面的描述方式[@Abraha:2015by]：

> Results are expressed as the average ratio of odds ratios (ROR)—that is, the ROR between mITT trials and ITT trials, and the ROR between mITT trials and no ITT trials. An ROR less than 1 indicated a larger effect estimate in mITT trials compared with ITT trials or no ITT trials. The heterogeneity across meta-analyses was measured with $\tau^2$.

### 保证结果稳定性采取其它一些分析方法
为保证结果的稳定性，文献中也可采用一定的方法，比如混杂因素矫正、亚组分析和敏感性分析等。例如下面一段文字就是对混杂因素矫正的描述[@Abraha:2015by]：

> To assess the robustness of our primary analysis, we performed additional adjusted analyses to take into account several potential confounding factors that can influence the estimate of the treatment effect. The confounders were identified from the medical literature for which a meta-epidemiological assessment was made: type of centre (single centre v multicentre) in which the trials were conducted, sample size (quarters within each meta-analysis and small v big sample size), presence of post-randomisation exclusions...

## 结果

### 研究纳入
研究纳入一般可用流程图的方式进行描述，注意需要交代文献被排除的理由[@Zhang:2013jv]。

> Our initial search identified 371 citations. Of them, 329 were excluded by reviewing the title and abstract because they were duplicate meta-analyses, included non-randomized trials, did not report data on mortality, and other irrelevant articles. Full text of the remaining 42 citations was reviewed, of which 15 citations were excluded. In these excluded 15 citations, eight did not include large trials, study end point was not mortality in four meta-analyses, and three were duplicated meta-analyses. A total of 27 meta-analyses involving randomized controlled trials were finally included in our analysis (Figure 1).

### 各个纳入研究的基本特征
这部分需要描述各个纳入研究的特征，包括描述性特征和量化的特征，同时可以根据感兴趣特征将演技进行分组进行组间比较，并报告相应统计量和推断的p值。下面的例子就展示了ITT和mITT研究之间的比较[@Abraha:2015by]。

> Compared with no ITT trials, mITT trials were more likely to be multicentre studies (cluster weighted χ2, P<0.001), report post-randomisation exclusions (P=0.03), report sample size calculation (P<0.001), and receive funding from private enterprises (P<0.001). Year of publication was more recent for mITT trials than for no ITT trials: 79 (67%) mITT trials were published between 2001 and 2009 compared with 34 (31%) no ITT trials (for trend, P<0.001). Studies classified as no ITT trials had a smaller sample size (median=83) than those classified as mITT trials (252) and ITT trials (272). The table shows the characteristics of included trials.

### 研究特征对效应值的影响
此部分一般报道ROR值及其95%置信区间。如果涉及较多的Meta分析也可用森林图方法展示结果。同时估算Meta分析间的异质性。以下就是标准的描述方法[@Abraha:2015by]：

> The 43 reviews provided 50 meta-analyses. Twelve trials contributed twice to the analyses; therefore, there were 322 comparisons in total. The treatment effect of mITT trials with respect to ITT trials was inflated by 17% (unadjusted ROR 0.83, 95% confidence interval 0.71 to 0.97; P=0.01; fig 2),), with moderate variance between meta-analyses (τ2=0.13). After adjusting the comparison for use of placebo, sample size, type of centre, items of risk of bias, post-randomisation exclusions, funding, and publication bias, the ROR was 0.80 (0.69 to 0.94; P=0.005; $\tau^2=0.08$). The comparison between mITT trials and no ITT trials provided an ROR of 1.00 (0.75 to 1.33; P=0.99; $\tau^2=0.57$; fig 2).

### 其它各种补充分析
这部分内容包括亚组分析、混杂因素矫正及敏感性分析，其主要目的在于查看结论的稳健性。如下就是一个敏感性分析的写作案例[@Panagiotou:2013ir]：

> With comparisons of fixed thresholds of sample size, treatment effect estimates were also significantly larger in smaller trials, regardless of the threshold level. The heterogeneity across meta-analyses was low for all analyses (fig 2). Results were consistent after adjustment on the following trial characteristics: domains of risk of bias, overall risk of bias, centre status, and time of publication since the first trial (web appendix 4). 

上文描述了根据不同的定义方法去定义样本量大小，结果发现小样本研究的疗效始终高于大样本研究，这是个典型的敏感性分析，表明结果稳健。另外这里也对混杂因素进行了矫正，结果发现小样本对效应值的影响无显著变化。

## 讨论
### 研究的总结
讨论部分的第一段先对本研究发现的证据进行陈述。

> In this meta-epidemiological analysis, we compared the estimated treatment effect of 50 meta-analyses based on the description (reporting) of the intention to treat approach of 310 trials, comprising 322 comparisons. We found that in meta-analyses of trials that deviated from the intention to treat reporting, the treatment effect increased significantly. Our results remained consistent, after adjustment for risk of bias items, post-randomisation exclusions, and other potential sources of bias (such as sample size, type of centre, funding, and publication bias).

上面的文字主要有三个意思：1、本研究纳入的样本量；2、主要分析结果；3、矫正混杂因素后结果稳健。这些基本上囊括了本研究工作的主要结果。

### 研究的局限性
这里主要讨论本研究一些局限，通常这些局限性可以是某些特征定义存在不确定性，例如在意向性分析研究中，一项研究是否采用ITT通常根据文献的报道，而不是实际原始资料的查看，这里就可能存在报道偏差。还有是检索数据库的局限，比如有文章仅仅检索了PubMed，这样就有可能遗漏一些文献。另外，某研究可能只分析了二分类变量的RCT，这样本研究的结论就未必适合连续变量作为结局的研究。有的文章在谈论局限性的同时也会谈到研究的优势，比如下面的例子就同时探讨了该研究的优势和缺陷所在[@Panagiotou:2013ir]：

> Our results were based on a large meta-epidemiological study of 93 meta-analyses, representing various medical areas published in the leading journals of each medical speciality or in the Cochrane Database of Systematic Reviews. Cochrane reviews have generally been shown to be of higher methodological quality, are better reported, and have fewer conflicts of interest than do non-Cochrane reviews. To explore the influence of sample size on treatment effect, we used several complementary approaches, which all showed consistent results. However, because our results were based on meta-analyses of trials assessing binary outcomes, they cannot be extrapolated to trials assessing continuous outcomes because such trials usually differ in medical condition, risk of bias, sample size, and statistical analysis.

### 研究结果的解读
这部分主要是对研究结果如何解读，其中就包括跟其它一些研究的比较，如某项研究是做危重症人群中小样本效应研究，其结果就可以跟其它人群相比较。另外该部分也可探讨目前研究结果的一些可能机制，比如小样本效应可能机制就是一些阴性结果的小样本研究很难发表，这就是发表偏倚所导致的。最后可以讨论一下这样的研究结果对未来做系统评价或设计临床研究时的价值和指导意义。

### 总结
该部分内容可以总结一下本文的发现，但重点要指出本文结论对于实际的指导意义，以及未来研究的展望。下面的这段文字就对研究内容进行了总结，并且对未来的研究提出了切实可行的方案[@Leyrat:2019if]。

> For binary outcomes, CRTs [cluster randomized trials] and IRTs [individually randomized trials] produced the same intervention  effect  estimates,  but  intervention  effect  estimates  were  marginally  more  favourable  (i.e.  either  morebeneficial  or  less  detrimental)  for  IRTs  with  continuousoutcomes. However, this result was not observed for trials assessing a pharmacological intervention or with an objec-tive outcome. More work is needed, in particular to under-stand  how  the  type  of  intervention,  outcome,  setting  ortrial sample size affects the results.

# 图例说明
-图1、Meta分析方法合并log(ROR)值。
-图2、亚组分析森林图。
-图3、Meta回归气泡图。
-图4、Meta回归得出的log(ROR)进行数据合并

# 参考文献
