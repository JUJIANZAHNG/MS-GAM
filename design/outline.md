

# 1. Introduction

正文按以下逻辑连续展开：

* 大背景
* 小背景->切入点->大致研究现状：

- climate–tourism relationship 通常被假定为稳定函数；
- 这种稳定性假设可能被 latent regime changes 破坏；
- 现有研究分别讨论 nonlinear response、regime switching 和 forecasting，但没有把三者连接起来；
- 提出研究问题、MS-GAM 框架和主要贡献；
- 简要说明论文结构。

------

# 2. Literature Review

## 2.1 From nonlinear climate effects to regime-dependent responses

这一节把以下文献放在同一条逻辑链中：

- temperature、rainfall 和 extreme weather 对旅游需求的影响；
- linear specification 的局限；
- GAM、threshold model 和 nonlinear interaction；
- 现有研究虽然允许非线性，却通常仍假定 response function 在整个样本期内固定。

这一节最后自然导向：问题不只是函数是否 nonlinear，还包括函数是否随 latent state 改变。

## 2.2 Structural change, latent regimes and tourism demand

这一节讨论：

- structural break；
- breakpoint segmentation；
- hidden Markov model 和 Markov-switching model；
- deterministic break 与 probabilistic regime 的区别；
- 为什么 tourism demand 可能表现出持续但不可直接观测的状态变化。

重点不是全面综述所有 regime-switching 文献，而是说明为什么本研究选择 latent regime，而不是预先切割时间段。

## 2.3 Destination heterogeneity, functional comparison and forecasting

这一节连接三个问题：

- 不同目的地的 climate-response function 为什么不能直接用单一参数比较；
- functional geometry 如何比较 response shape；
- regime information 是否具有 real-time forecasting value。

最后集中提出研究缺口：

> Existing studies have not jointly examined destination-specific latent regimes, regime-dependent nonlinear climate responses, cross-destination functional heterogeneity, and the out-of-sample predictive value of real-time regime information.

这样 Literature Review 只有三个二级标题，但每节都完成一段完整推导。

------

# 3. Data and Methods

## 3.1 Empirical setting and data construction

这一节统一处理：

- study destinations；
- sample period 和 temporal frequency；
- tourism-demand outcome；
- temperature、rainfall 和 extreme indicators；
- holidays、COVID、closures、transport disruptions 等 controls；
- missing values、aggregation 和 forecast-time availability。

不需要给每类变量单独开标题。可以在正文中使用几个段落，例如：

1. destinations and tourism demand；
2. climate and extreme exposure；
3. controls and data treatment。

这一节的结尾应明确数据最终向模型提供什么：

[
{Y_{r,t},T_{r,t},R_{r,t},E_{r,t},W_{r,t}}.
]

## 3.2 A destination-level regime-dependent functional model

这是方法部分的核心，统一介绍：

- baseline single-regime GAM；
- destination-level MS-GAM；
- latent state process；
- regime-specific climate-response functions；
- homogeneous transition matrix；
- regime number selection；
- estimation、initialization 和 model diagnostics。

正文可以按照“benchmark → full model → estimation”连续展开，而不必把每个公式都变成一个标题。

该节需要明确区分：

# [ \mu_{r,k,t}

\text{temporal structure}
+
\mathcal C_{r,k}(T,R,E)
+
\text{controls},
]

其中 (\mathcal C_{r,k}) 才是后续跨目的地比较的对象。

## 3.3 Functional comparison, forecasting and validation

这一节统一说明三个 downstream tasks。

首先，解释如何比较 regime-specific climate-response functions：

- common support；
- regime alignment；
- curvature、sensitivity、interaction strength 和 extreme salience；
- functional distance。

其次，说明预测设计：

[
F_1:\text{demand history},
]

[
F_2:\text{history + climate},
]

[
F_3:\text{history + climate + extremes},
]

[
F_4:\text{previous information + filtered regime probabilities}.
]

最后说明 validation：

- rolling or expanding estimation；
- forecast metrics；
- regime robustness；
- functional robustness；
- predictive-gain robustness。

这里不必分别设置“Functional comparison”“Forecast evaluation”“Robustness analysis”三个大标题，因为它们都属于模型输出如何被使用和验证。

------

# 4. Empirical Results and Analysis

这一部分最需要避免碎片化。结果章节不应按照“温度结果—降雨结果—interaction 结果—transition matrix—forecast metric”逐项排开，而应按照论文的核心判断组织。

## 4.1 Evidence of regime-dependent climate responses

这一节回答第一个核心问题：

> 是否确实存在具有实质意义的 regime-dependent climate response？

内容包括：

- descriptive patterns；
- baseline GAM；
- single-regime 与 switching models 的比较；
- regime number selection；
- transition persistence；
- regime-specific temperature、rainfall、interaction 和 extreme effects。

这些结果应放在同一节，因为它们共同支持或否定“climate response 随状态改变”这一判断。

小结不应只是复述图表，而应形成一句结论，例如：

> The results indicate that regime switching is not limited to changes in demand levels or volatility; the shape and magnitude of climate responses also vary across latent states.

若证据不足，则应写得更保守。

## 4.2 Cross-destination structure of climate sensitivity

这一节回答第二个问题：

> 不同目的地的 regime-specific functions 有哪些共同结构和差异？

内容包括：

- regime alignment；
- common-support comparison；
- functional features；
- distance matrix 和 clustering；
- destination-specific patterns；
- high-sensitivity 或 high-risk states 的异同。

标题应强调 analytical conclusion，而不是写成“Functional comparison results”。

该节的小结可以围绕：

> Cross-destination heterogeneity arises from both the shape of climate-response functions and the configuration of latent regimes.

这样可以把 function heterogeneity 与 regime heterogeneity 区分开。

## 4.3 Predictive value of real-time regime information

这一节回答第三个问题：

> filtered regime information 是否提供了超出 climate 和 extreme variables 的增量预测价值？

内容包括：

- (F_1)–(F_4) 的预测比较；
- 不同 destination 和 horizon 的差异；
- extreme periods 下的表现；
- regime feature 是否稳定改善预测；
- 哪些目的地没有改善。

这一节不能只报告“MS-GAM 预测更准”，而要集中比较：

[
F_4-F_3.
]

因为真正的研究问题是 regime representation 的增量信息，而不是某个复杂模型是否击败简单模型。

## 4.4 Robustness and interpretation of the findings

这里统一处理：

- regime stability；
- function stability；
- extreme-definition sensitivity；
- alternative (K)；
- alternative estimation choices；
- forecast robustness；
- failure cases。

随后给出 integrated interpretation：

- 哪些结果可以视为可靠的 structural association；
- 哪些只能视为 latent segmentation；
- 哪些只证明 predictive usefulness；
- 哪些结论不能上升为 causal mechanism。

这一节实际上承担传统 Discussion 的功能，因此不必再额外设置大量 discussion 子标题。

------

# 5. Conclusions

Conclusion 可以不分二级标题，按四段组织：

第一段：概括研究问题和方法。

第二段：集中陈述三个主要发现：

1. 是否存在 regime-dependent climate responses；
2. 不同目的地如何表现出 functional 和 regime heterogeneity；
3. filtered regime information 是否改善预测。

第三段：说明理论和实践含义。

第四段：说明限制与未来研究，例如 climate-dependent transitions、distributed-lag effects 和 hierarchical MS-GAM。
