你的 idea 有潜力，而且两篇论文之间存在一条天然接口：

> 第一篇解释气候冲击如何改变旅游需求的生成机制；第二篇研究在复杂信息环境下如何预测旅游需求。
> 你的论文真正可以占据的位置，是把“冲击识别”转化成“可供预测模型使用的结构信息”。

目前方案的问题不在于内容少，反而在于东西很多，但尚未形成一条有方向的因果链。16种解释模型再叠加一排机器学习、深度学习模型，很容易变成 model zoo：每只动物都很珍贵，放在一起却未必构成生态系统。

------

# 一、先判断：原始 idea 的价值与缺口

## 1. 已经有价值的部分

你抓住了四个真正有意义的维度：

1. **线性与非线性**：气温、降水对旅游需求大概率存在舒适区间、阈值和边际效应变化。
2. **平稳与结构变化**：同样的天气条件，在不同历史阶段可能具有不同影响。
3. **单一与复合气候条件**：高温与强降水共同出现时，影响可能超出两者简单相加。
4. **解释与预测联动**：前半部分发现的气候机制，应当进入后半部分预测，而非论文中途换轨。

第一篇论文已经把 segmented regression 与 GAM 结合起来，用断点描述需求轨迹变化，用平滑函数解释非线性气候效应，并强调单一事件与复合事件、短期冲击与长期结构变化的差异。

第二篇论文则整合历史需求、天气、搜索指数、节假日和评论文本，比较 early、intermediate、late fusion，并通过改进 MIV 分析不同模态的预测贡献。

因此，你的研究不应只是：

> 把第一篇的分段 GAM 做一遍，再把第二篇的预测模型做一遍。

这样只能算 two papers stapled together。

你的核心命题应提升为：

> **气候冲击会不会改变旅游需求的 data-generating process；识别这种 regime change 后，能否构造更准确、更稳定、更可解释的 climate-aware forecasting model？**

这条主线一旦立住，解释部分和预测部分就会紧密衔接。

------

## 2. 当前方案最需要修正的几个地方

### 第一，16个模型的设计过于机械

你目前按照：

- 线性 / 非线性
- 分段 / 不分段
- 无极端天气 / 单一极端天气 / 复合天气 / 单一加复合

进行全排列。

这种 full factorial design 可以作为技术附录中的候选模型空间，却不宜成为论文的理论结构。因为其中一些维度并不真正正交：

- “非线性”通常已由 GAM 的 smooth term 表达；
- “复合效应”可以通过 tensor-product smooth 或 interaction term 表达；
- “分段”可能意味着时间断点、事件断点，也可能意味着参数随 regime 改变；
- “单一+复合变量”若处理不当，会产生严重共线性和重复编码。

论文会因此变成“比较16个模型谁的 AIC 小”，而没有回答更高层的问题。

更合适的是建立一个**嵌套式模型阶梯**，每一级对应一个明确的科学问题：

[
M_0:\text{baseline seasonality}
]

[
M_1:M_0+\text{linear climate effects}
]

[
M_2:M_0+\text{nonlinear climate effects}
]

[
M_3:M_2+\text{compound climate interaction}
]

[
M_4:M_3+\text{regime-varying climate effects}
]

然后检验每次增加复杂度究竟解决了什么。

这样模型比较具有逻辑方向：
**平均效应 → 非线性 → 复合性 → 状态依赖性。**

------

### 第二，断点不能轻易被叫作“气候导致的断点”

第一篇论文把断点与严重天气、旅游韧性联系起来，但你的样本如果覆盖 COVID-19、政策变化、景区闭园、交通中断、价格调整或统计口径变化，那么 demand breakpoint 并不自动等于 climate breakpoint。

这是整篇论文最大的 identification risk。

至少需要分三步：

1. **先在需求序列中识别候选结构断点**；
2. **再判断断点附近是否存在气候异常或极端气候累积**；
3. **控制其他重大事件，并进行 placebo 或 falsification test**。

因此表述上应区分：

- demand structural break；
- climate-associated break；
- climate-attributable break。

前两个较容易成立，第三个需要更强的识别设计。否则审稿人会非常自然地问：断点为什么不是疫情、票价或交通政策造成的？

------

### 第三，“没有极端天气数据就用气温降水构造”完全可行，但不能随意二值化

这一点反而可能成为你的亮点。

你不必等待现成的“极端天气事件数据库”，可以基于气温和降水构造**相对极端天气指标**。相对阈值往往比跨地区统一的绝对阈值更适合旅游研究，因为 32℃ 在新加坡可能普通，在九寨沟却可能异常。

建议构造三层变量：

#### 层一：连续气候暴露

- 日最高温、最低温、平均温；
- 日降水量；
- 温度日较差；
- 连续降水天数；
- 温度或降水相对于当地季节均值的 anomaly。

连续变量用于 GAM，避免阈值设定损失信息。

#### 层二：单一极端指标

按地区、月份或季节计算历史分位数：

[
Hot_{it}=I(T_{it}>Q_{0.90}^{r,m})
]

[
Cold_{it}=I(T_{it}<Q_{0.10}^{r,m})
]

[
HeavyRain_{it}=I(P_{it}>Q_{0.90}^{r,m})
]

也可以使用 exceedance intensity：

# [ HeatIntensity_{it}

\max\left(
0,
\frac{T_{it}-Q_{0.90}^{r,m}}{\sigma_{T,r,m}}
\right)
]

这样可以区分“刚超过阈值”和“极端超过阈值”。

#### 层三：复合气候压力

不要只设一个简单的 `hot × rain` dummy。可以同时构造：

1. compound-event dummy；
2. joint exceedance intensity；
3. rolling cumulative stress；
4. 事件持续时间。

例如：

# [ CCS_{it}

HeatIntensity_{it}
+
RainIntensity_{it}
+
\lambda HeatIntensity_{it}RainIntensity_{it}
]

或者用二维经验分布构造 joint rarity：

# [ JRI_{it}

-\log
\Pr(T\geq T_{it},P\geq P_{it})
]

这个 Joint Rarity Index 会比“是否同时高温暴雨”更细腻，也更容易形成方法创新。

------

# 二、我建议你把论文主线改成什么

可以概括为一句话：

> **从气候冲击的非线性识别，走向需求状态识别，再走向状态感知预测。**

整篇论文围绕三个连续问题展开。

## RQ1：气候条件怎样影响旅游需求？

重点回答：

- 线性还是非线性；
- 是否存在舒适区和风险阈值；
- 单一气候因素与复合气候压力是否具有不同效应；
- 效应是否具有滞后性和持续性。

## RQ2：气候冲击会不会改变需求机制本身？

重点回答：

- 是否存在需求结构断点；
- 断点之后，气候敏感度是否改变；
- 冲击表现为 temporary disruption、persistent loss，还是 adaptive transformation；
- 不同目的地为何呈现不同恢复路径。

这里的关键不再只是“需求水平是否下降”，而是：

[
f_{\text{before}}(Climate)
\neq
f_{\text{after}}(Climate)
]

即断点前后，气候—需求响应曲线本身是否改变。

这比原第一篇论文单纯将 segmented regression 与 GAM 并置更进一步。第一篇的思路是分别用断点和 GAM 捕捉离散变化与连续非线性。 你的改进应当是让二者真正耦合：

# [ Y_{it}

\alpha_r
+
f_{k}(T_{it},P_{it})
+
s_{k}(time)
+
controls
+
\varepsilon_{it},
\quad t\in Regime_k
]

每个 regime 拥有不同的气候响应函数。

## RQ3：结构识别能否改善预测？

重点回答：

- 气候变量是否只改善平均预测精度；
- 在极端天气期、结构转折期是否改善得更多；
- regime information 是否能增强模型在 distribution shift 下的稳定性；
- 模型能否预测需求下降方向、幅度和恢复时间。

此时预测研究就不再是“又比较了一些模型”，而是检验一个明确命题：

> **当旅游需求机制发生状态变化时，显式输入气候压力和 regime 信息，比依赖模型自己从历史数据中隐式学习更有效。**

------

# 三、最值得做的创新组合

单个创新点通常不够。你的亮点应是一个彼此咬合的 innovation bundle。

## 创新一：从 event dummy 转向 continuous climate stress representation

第一篇主要采用严重天气事件及组合事件，第二篇将天气当作预测模态之一。你的改进是建立一个包含：

- anomaly；
- intensity；
- duration；
- compoundness；
- cumulative exposure；

的 Climate Stress Representation。

这解决了两个问题：

1. 没有现成极端天气事件数据；
2. 简单事件变量无法反映程度和持续性。

可以把核心气候表征写成：

# [ \mathbf{C}_{it}

(
A^T_{it},
A^P_{it},
D^T_{it},
D^P_{it},
JRI_{it},
CumStress_{it}
)
]

这不是为了制造六个变量，而是把“极端天气”从一个标签改造成一个多维过程。

------

## 创新二：从 demand breakpoint 转向 climate-sensitivity breakpoint

普通断点回答：

> 旅游需求什么时候变了？

你的模型进一步回答：

> 旅游需求对气候的敏感度什么时候变了？

例如断点前，高温超过30℃后需求略降；断点后，相同温度产生更大下降。这意味着目的地可能出现适应能力下降、游客风险感知增强或市场结构变化。

可设：

# [ Y_{it}

\alpha_k+
f_k(T_{it})+
g_k(P_{it})+
h_k(T_{it},P_{it})
+\varepsilon_{it}
]

并检验：

[
H_0:
f_1=f_2=\cdots=f_K
]

这比仅比较分段和不分段模型更有理论含义。

------

## 创新三：构造“韧性画像”，但不要做任意加权的综合得分

你提到定义旅游韧性指标，这个方向对，但不建议先拍脑袋设权重，再做 entropy weight、TOPSIS 一类综合评价。那会把丰富的动态过程压成一个看似精确的数字。

建议先定义四个可解释维度：

### 冲击幅度

# [ A_e

\frac{Y_{expected}-Y_{observed}}{Y_{expected}}
]

### 恢复时间

# [ \tau_e

\min
{h:Y_{t+h}\geq(1-\delta)Y_{expected,t+h}}
]

### 恢复完整度

# [ R_e

\frac{\bar Y_{post}}{\bar Y_{counterfactual}}
]

### 气候敏感度漂移

# [ S_e

\int
|
f_{post}(c)-f_{pre}(c)
|
,dc
]

这四个量对应：

- 抗冲击能力；
- 恢复速度；
- 恢复程度；
- 机制稳定性。

然后对目的地进行 clustering，形成：

- resistant；
- rapid-recovery；
- adapted-but-transformed；
- persistently vulnerable。

这比硬凑一个 resilience score 更有解释力。如果确实需要单一指标，可在最后通过 PCA 或 latent factor 提取，而不是主观加权。

你甚至可以提出：

> 韧性并非“需求回到原位”的单维性质，而是由冲击吸收、恢复速度、恢复完整度和气候敏感度稳定性共同构成的动态画像。

这会比简单讨论 bounce back 更亮眼。第一篇本身就强调 bounce back 与永久改变状态的区别。

------

## 创新四：Regime-aware forecasting

这是整篇论文最有辨识度的部分。

预测模型除了输入：

- 历史旅游需求；
- 气温；
- 降水；
- 节假日；
- 搜索指数；

还输入前文识别出的：

- regime label；
- break probability；
- climate stress score；
- resilience state；
- time since shock。

模型可以写成 mixture-of-experts：

# [ \hat Y_{t+h}

\sum_{k=1}^{K}
\pi_{k,t}
F_k(X_t)
]

其中：

- (\pi_{k,t}) 是当前处于第 (k) 个需求状态的概率；
- (F_k) 是对应状态下的预测器。

这意味着模型不会强迫平稳时期与冲击时期共享完全相同的参数。

更简单的实现则是：

# [ \hat Y_{t+h}

F(
Y_{t-L:t},
Climate_{t-L:t},
Regime_t,
Stress_t,
Controls_t
)
]

这一创新相对第二篇论文很清晰。第二篇主要比较不同 multimodal fusion 策略，并发现 early fusion 整体表现较好。 你的研究关注另一条边界：

> 当数据生成机制发生变化时，哪种预测结构能够利用显式的冲击状态信息维持准确性？

也就是说，从 multimodal fusion 向 **mechanism-informed fusion** 推进。

------

## 创新五：预测评价从平均误差扩展到“冲击条件下的可靠性”

只比较 MAE、RMSE、MAPE 太常规。你需要按情境评估。

建议至少分成：

- normal days；
- single-extreme days；
- compound-extreme days；
- pre-break transition window；
- post-break recovery window。

然后回答：

# [ Gain_{extreme}

## Error_{baseline,extreme}

Error_{climate-aware,extreme}
]

真正亮眼的问题是：

> 模型的改进来自普通时期的微小优化，还是来自在最难预测的冲击时期避免失灵？

还可以加入：

- Directional Accuracy：是否预测对上升/下降方向；
- Peak Error：是否低估需求骤降；
- Recovery Timing Error：恢复时间预测偏差；
- prediction interval coverage：不确定性区间是否可信；
- worst-group error：各地区最差情境下的表现；
- stability across regimes：跨状态误差波动。

第二篇已经研究稳定期和动荡期以及模态贡献变化。 你的推进应是把“动荡期”从一个宽泛历史阶段，细化为由气候压力和结构状态定义的状态空间。

------

# 四、建议的论文结构

# 拟定题目

**气候冲击、需求状态转移与旅游需求预测：基于多目的地的分段非线性识别与状态感知预测**

英文可考虑：

**Climate Shocks, Demand Regime Shifts, and Tourism Forecasting: A Multi-destination Framework Integrating Segmented Nonlinear Modeling and Regime-aware Prediction**

## 1. Introduction

### 1.1 研究背景

旅游需求同时受到长期气候变化、短期天气异常以及游客风险感知的影响。现有研究分别关注气候冲击的解释和多源信息的预测，但对“气候冲击如何改变需求生成机制，以及这一变化如何进入预测模型”讨论不足。

### 1.2 核心问题

本文研究气候条件是否仅造成暂时需求波动，还是会改变气候—旅游需求关系本身；进一步考察显式识别需求状态变化能否提高极端气候期与恢复期的预测能力。

### 1.3 研究问题

RQ1：气温、降水及其复合压力对旅游需求是否存在非线性、阈值与滞后效应？

RQ2：旅游需求及其气候敏感度是否存在结构性状态变化？

RQ3：不同目的地的冲击幅度、恢复速度、恢复完整度和气候敏感度漂移有何差异？

RQ4：将气候压力和需求状态信息嵌入预测模型，能否提高普通时期、冲击时期和恢复时期的预测精度与稳定性？

### 1.4 研究贡献

第一，构建基于异常程度、持续时间、联合稀有度和累积暴露的多维气候压力指标，在缺乏现成极端天气事件数据时仍能刻画极端与复合气候条件。

第二，将需求水平断点扩展为气候敏感度断点，识别结构变化前后气候—需求响应函数的变化。

第三，以冲击幅度、恢复时间、恢复完整度和敏感度漂移构建目的地韧性画像，区分恢复、适应性转型与持续脆弱状态。

第四，提出状态感知预测框架，将气候压力、断点概率及需求 regime 输入预测模型，并检验其在 distribution shift 下的预测价值。

## 2. Literature Review and Conceptual Framework

### 2.1 气候条件与旅游需求

讨论平均气候、天气异常、极端天气和复合事件对旅游需求的不同影响。

### 2.2 旅游韧性与结构变化

区分 resistance、recovery、adaptation 和 transformation，说明韧性是动态路径而非单一恢复率。

### 2.3 旅游需求预测与外部信息

回顾天气、搜索指数、节假日、评论文本等变量在预测中的作用，指出现有模型通常将天气视为普通输入，较少显式处理气候冲击引起的 regime shift。

### 2.4 理论机制

气候压力通过舒适度下降、出行受阻、风险感知增强和计划可控性下降影响旅游需求；重复冲击进一步改变目的地形象、游客构成和旅游供给，由此导致气候敏感度漂移与需求状态转移。

形成逻辑链：

Climate exposure
→ perceived/physical travel constraint
→ demand response
→ repeated stress and adaptation
→ regime shift
→ forecasting distribution shift

## 3. Data and Variable Construction

### 3.1 样本地区

根据气候类型、目的地类型、数据可得性和旅游市场结构选取若干代表性目的地。避免只以“能获取数据”为唯一标准。

### 3.2 旅游需求数据

优先使用日度游客量、酒店入住率、客房需求或景区客流。不同地区的因变量口径应尽量一致；若无法一致，需要分组分析或标准化处理。

### 3.3 气候变量

包括气温、降水、温差、连续降水日数、季节异常值及滞后变量。

### 3.4 极端与复合气候指标

根据地区—月份历史分位数构建高温、低温和强降水指标；进一步构建 exceedance intensity、duration、joint rarity 和 cumulative stress。

### 3.5 控制变量

节假日、星期效应、月份和季节、长期趋势、疫情及重大政策事件、景区关闭、交通可达性、搜索指数等。

## 4. Climate-impact and Regime-identification Framework

### 4.1 基准模型

建立包含趋势、周期、节假日和自回归项的 baseline model。

### 4.2 非线性气候效应

使用 GAM 或 distributed lag nonlinear model 估计气温和降水的非线性及滞后效应。

### 4.3 复合气候效应

采用 tensor-product smooth 或二维响应面识别高温—降水等复合效应，检验其是否具有 non-additivity。

### 4.4 状态与断点识别

识别旅游需求序列中的结构变化，并对断点与气候压力的时序对应关系进行检验。通过控制重大非气候事件、placebo dates 和替代阈值进行稳健性分析。

### 4.5 气候敏感度漂移

比较不同 regime 下的气候响应函数，检验结构变化是否仅发生于需求水平，或同时发生于气候敏感度。

## 5. Destination Resilience Profiles

### 5.1 冲击幅度

测量实际需求相对于反事实需求的最大损失。

### 5.2 恢复速度

测量需求回到反事实轨迹一定比例所需的时间。

### 5.3 恢复完整度

判断需求是否回到原轨迹、停留在较低水平，或进入新的增长路径。

### 5.4 气候敏感度稳定性

计算断点前后气候响应函数的距离。

### 5.5 目的地分类

基于上述维度识别抗冲击型、快速恢复型、适应性转型型和持续脆弱型目的地。

## 6. Regime-aware Tourism Demand Forecasting

### 6.1 预测实验设计

比较以下信息集：

A：仅历史旅游需求；

B：历史需求 + 常规气候变量；

C：历史需求 + 气候压力指标；

D：历史需求 + 气候压力 + regime information；

E：进一步加入搜索指数、节假日或其他模态。

### 6.2 候选模型

选择少量具有代表性的模型，而非穷举算法：

统计基准：Seasonal Naive、ARIMAX；

机器学习：LightGBM 或 XGBoost；

序列模型：LSTM 或 TCN；

attention-based model：TFT、PatchTST 或 Transformer 中选择一种；

状态感知模型：regime-gated model 或 mixture-of-experts。

### 6.3 评价方式

同时报告平均误差和情境误差：

普通时期；
单一极端天气期；
复合极端天气期；
断点邻近时期；
冲击后恢复时期。

评价 MAE、RMSE、sMAPE、方向准确率、恢复时间误差、prediction interval coverage 及跨状态稳定性。

### 6.4 统计比较

使用 Diebold–Mariano test 比较两模型预测误差；使用 Model Confidence Set 识别统计上无法被淘汰的优胜模型集合。

## 7. Results

### 7.1 气候响应曲线

呈现各类目的地的非线性阈值、舒适区和复合效应。

### 7.2 状态变化与敏感度漂移

展示断点是否存在，以及断点前后气候响应函数如何变化。

### 7.3 韧性画像

比较不同目的地的冲击、恢复和转型路径。

### 7.4 预测结果

说明气候压力和 regime information 在哪些情境、哪些目的地和哪些预测期限下最有价值。

### 7.5 机制一致性检验

检验解释模型中识别出的关键变量和状态，是否也在预测模型中表现出较高边际贡献；若二者不一致，讨论解释效应与预测价值之间的差别。

## 8. Discussion

### 8.1 理论意义

将旅游韧性从需求恢复轨迹扩展为需求水平与气候敏感度共同演化的动态过程。

### 8.2 方法意义

连接结构变化识别与状态感知预测，避免解释模型和预测模型彼此割裂。

### 8.3 管理意义

根据目的地的阈值、复合气候脆弱性和恢复类型，提出差异化预警、客流管理和产品调整方案。

### 8.4 局限性

包括气候冲击归因限制、跨地区数据口径差异、搜索指数可得性差异及极端事件样本稀疏问题。

------

# 五、模型究竟怎样选，才不会显得失控

你原来列出 Transformer、FEDformer、Informer、MLP、RNN、LSTM、CNN、XGBoost、LightGBM、Random Forest 等。这个范围太宽，容易让论文的计算量和叙事复杂度同时爆炸。

建议遵循“每个模型代表一种归纳偏置”：

| 类型         | 模型                                | 存在理由                         |
| ------------ | ----------------------------------- | -------------------------------- |
| 朴素基准     | Seasonal Naive                      | 判断复杂模型是否真的有价值       |
| 统计模型     | ARIMAX / dynamic regression         | 可解释的线性外生变量基准         |
| 树模型       | LightGBM                            | 捕捉非线性与交互，适合表格型变量 |
| 局部序列模型 | TCN 或 LSTM                         | 捕捉局部动态和滞后               |
| 长依赖模型   | TFT / PatchTST / Transformer 三选一 | 捕捉长时间依赖及多变量关系       |
| 提出模型     | Regime-gated model                  | 承载你的核心创新                 |

六类已经足够。模型数量更多，论文不会自动更科学，只会导致表格横向滚动到天边。

------

# 六、MCS 检验能不能用

你记得的是 **Model Confidence Set**，可以用，但位置要放对。

MCS 更适合预测部分：根据 loss differential，从候选预测模型中逐步剔除表现显著较差的模型，最后得到一个在给定置信水平下的 superior set。

它不适合作为前半部分解释模型的唯一筛选手段。解释模型需要关注：

- 理论设定；
- smooth term significance；
- AIC/BIC；
- out-of-sample fit；
- residual diagnostics；
- concurvity；
- breakpoint stability；
- nested-model comparison。

预测部分可以采用：

- MCS：识别优胜模型集合；
- Diebold–Mariano：进行成对预测误差比较；
- Giacomini–White：比较条件预测能力；
- block bootstrap：处理时间序列相关性。

值得注意的是：不要简单地“每个地区选一个最佳模型”然后结束。更有意义的是分析：

> 哪一种模型属性，在何种目的地、何种气候状态、何种预测期限下占优？

------

# 七、结果部分按什么逻辑组织

你的直觉是对的：按照“效应”组织比逐地区流水账更好。

建议主体结构为：

1. 气候关系是否非线性；
2. 复合压力是否产生超加性影响；
3. 是否存在结构变化和敏感度漂移；
4. 不同目的地形成哪些韧性类型；
5. 哪些结构信息提高了预测；
6. 预测增益主要出现在哪些困难状态。

地区作为异质性维度嵌入每一节。

例如，不要写成：

- 香港结果；
- 澳门结果；
- 九寨沟结果；
- 新加坡结果。

而应写成：

- 温度阈值的共同规律及地区差异；
- 降水效应的目的地类型差异；
- compound stress 的非加性；
- regime shift 的空间异质性；
- resilience profile；
- forecast gains under stress。

这样读者看到的是一个论证过程，而非九份景区体检报告。

------

# 八、这篇论文最可能形成的核心结论

理想情况下，论文最终会得到这样一种比“气候变量提高预测精度”更深入的结论：

> 气候信息的价值具有 state dependence。正常时期，历史需求和季节性已经解释大部分波动；单一异常天气时期，连续气候压力指标提供边际信息；复合极端和结构转折时期，需求生成机制发生变化，显式建模 regime 与气候敏感度漂移才能显著减少预测失误。不同目的地的预测模型差异，本质上对应其气候暴露、游客构成与适应能力差异。

这里有三个层级：

- 气候影响 demand level；
- 气候影响 demand mechanism；
- demand mechanism 的变化影响 forecastability。

这才是两篇参考论文真正被消化后的第三篇论文，而不是两份方法的拼盘。

最后给一个优先级判断：**最应该保留的是“非线性—复合压力—状态变化—状态感知预测”这条链；最应该删减的是16个解释模型与十余个预测算法的穷举；最应该补强的是断点归因、相对极端指标和韧性动态测量。**