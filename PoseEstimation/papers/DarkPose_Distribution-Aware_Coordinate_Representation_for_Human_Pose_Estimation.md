# Distribution-Aware Coordinate Representation for Human Pose Estimation

------

原文链接：[点这里](https://arxiv.org/abs/1910.06278)

## 目录

- [1. 摘要](#1)
- [2. 介绍](#2)
- [3. 相关工作](#3)
- [4. 方法论](#4)
  - [4.1 坐标解码](#4.1)
  - [4.2 坐标编码](#4.2)
  - [4.3 集成到SOTA模型](#4.3)
- [5. 实验](#5)
  - [5.1 评估坐标表示](#5.1)
  - [5.2 与SOTA模型比较](#5.2)
- [6. 结论](#6)

<a name="1"></a>

## 1. 摘要

虽然heatmap是人体姿态估计中现实意义上的标准坐标表示，但文献中从未对其进行过系统研究。本文集中关注heatmap，通过研究坐标表示填补这一空白。本文发现，将预测的heatmap解码为原始图像关节坐标的过程对人体姿态估计性能意义重大，这在以前是不被认可的。本文进一步讨论现有方法广泛使用的标准坐标解码方法设计的局限性，提出一种更具原则性的分布感知解码方法。与此同时，本文为无偏模型训练生成标准的heatmap分布，以此提升标准坐标编码过程（即将gt坐标转换为heatmap）。将两者结合起来，本文提出一种新的分布感知关键点坐标表示方法(Distribution-Aware coordinate Representation of Keypoint, DARK)。作为一个模型无感知插件，DARK显著提高了各种SOTA人体姿态估计模型的性能。大量实验表明，在MPII和COCO这两个数据集上，DARK取得了最好的结果，一致验证了本文的坐标表示思想。

<a name="2"></a>

## 2. 介绍

人体姿态估计是一个基础的计算机视觉问题，旨在无约束图像中检测人体关节的空间位置(即坐标)。由于服装风格、遮挡和背景环境不可控， 身体关节的外观有巨大差异，同时需要识别细粒度关节坐标。CNN擅长解决这样的问题，现有工作通常侧重于为人体姿态推理设计CNN框架。

与图像分类中用作表示目标类别标签的通用one-hot向量类似，人体姿态CNN模型也需要一个标签表示，来编码人体关节坐标标签，以便在训练期间量化和计算监督学习损失，并正确推断关节坐标。现实意义的坐标标签表示为坐标heatmap，以每个关节标签坐标为中心生成二维高斯分布/核。它由坐标编码过程获得，由坐标转换到heatmap。heatmap的特点是在gt位置周围给定空间支持，同时考虑上下文线索和固有目标位置的模糊性。与类标签平滑正则化思想类似，这种做法可以有效降低模型过拟合风险。SOTA姿态模型大多是基于heatmap坐标表示的。

heatmap标签表示的一个主要难点在于计算成本，由于涉及到输入图像的二次函数，会妨碍CNN模型处理典型高分辨率原始图像数据。为了负担得起计算量，一种标准策略是，在输入到人体姿态估计模型之前，通过数据预处理将所有大分辨率人体边界框图像下采样为小分辨率。为了在原始图像坐标空间中预测关节位置，在heatmap预测之后，需要将分辨率转换回原始坐标空间。最终将激活值最大的位置视为预测位置。这个过程称为从heatmap到坐标的坐标解码过程。值得注意的是，分辨率降低的过程中可能会引入量化误差。 为了缓解这个问题，在现有的坐标解码过程中，通常会根据从最大激活值到第二大激活值方向执行手动偏移操作。

坐标编码和解码（即坐标表示）问题很少受到关注，尽管在模型推理中是不可或缺的。当前的研究重点是设计更有效的CNN结构，与此不同，本文揭示了坐标表示对模型性能的重要作用，比预期的重要得多。例如，上述坐标编码偏移操作使HRNet-W32在COCO验证集上的AP提高了5.7%。这种操作使收益显著提高，但在文献中从未得到充分注意和仔细研究。

与现有人体姿态估计研究相反，本文专门研究了包括编码和解码在内的关节坐标表示问题。此外，heatmap分辨率是一个主要屏障，不能单纯的使用较小的输入分辨率来提高模型的推理速度。当输入分辨率从256×192降低到128×96时，HRNet-W32在COCO验证集上的性能从74.4%下降到66.9%，推理开销仅从7.1×10e9FLOPs下降到1.8×10e9FLOPs。

鉴于坐标表示的发现意义，本文进行了深入调查，发现一个关键限制在于坐标解码过程。虽然现有的标准偏移操作已被证明是有效的，但本文提出一种原则性分布感知表示方法，以便在亚像素精度下进行更精确的关节定位。具体来说， 它旨在通过基于泰勒展开式的分布近似，综合考虑heatmap激活的分布信息，此外，本文发现，生成gt heatmap的标准方法存在量化错误，这会导致监督信息不精确、训练出的模型性能较差。为了解决这个问题，本文提出生成无偏heatmap，允许高斯核集中在亚像素位置。

本文贡献在于，发现了坐标表示在人体姿态估计中未被发现的现实意义，并提出一种新的分布感知关键点坐标表示(DARK)方法，该方法包括两个关键部分：（1）基于泰勒展开的高效坐标解码；（2）以亚像素为中心的无偏坐标编码。重要的是，现有的人体姿态估计方法可以无缝使用DARK，无需修改算法。在MPII和COCO上的大量实验表明，本文方法显著提升了SOTA人体姿态估计模型的性能。DARK支持使用更小的输入图像分辨率，模型性能下降很小，同时大大提高模型推理效率，从而促进嵌入式人工智能场景对低分辨率和低能耗的要求。

<a name="3"></a>

## 3. 相关工作

人体姿态估计有两种常见的坐标表示设计：直接坐标和heatmap。两者都被用作模型训练的回归目标。

**坐标回归**    直接将坐标作为输出目标，简单直观。现有方法中只有少数采用这种设计。一个原因是这种表示缺乏空间和上下文信息。由于关节位置存在固有的视觉模糊性，采用这种表示使得模型学习具有挑战性。

**heatmap回归**    heatmap表示有效解决了上述缺陷。它最早由Tompson等人引入，并迅速称为最常用的坐标表示法。一般来说，主流研究重点是设计网络结构，以便有效地回归heatmap。代表性设计有顺序建模、感受野扩展、位置投票、中间监督、成对关系建模、树形结构建模、金字塔残差学习、级联金字塔学习、知识引导学习、主动学习、对抗学习、反卷积上采样、多尺度监督、注意机制和高分辨率表示保持。

与之前所有工作相比，本文研究了人体姿态估计中的heatmap表示问题，这在之前的文献中是基本上被忽略的。本文不仅揭示了在使用heatmap时分辨率降低对模型效果存在巨大影响，还提出一种原则性的坐标表示方法，以显著提高现有模型的性能。重要的是，本文方法即插即用。

<a name="4"></a>

## 4. 方法论

本文考虑在人体姿态估计中包括编码和解码在内的坐标表示问题。目标是在给定的输入图像中预测关节坐标。为此，需要学习一个从输入图像到输出坐标的回归模型，heatmap通常用作模型训练和测试期间的坐标表示。假设现有一组训练图像，为了便于模型学习，本文将关节gt坐标编码成heatmap，作为监督学习目标。在测试中，需要将预测的heatmap解码为原始图像坐标空间中的坐标。

随后，本文首先描述解码过程，重点分析现有标准方法的局限性，并提出一种新的解决方案。然后进一步讨论并解决编码过程中的局限性。最后描述如何将本方法集成到现有人体姿态估计模型中。

<a name="4.1"></a>

### 4.1 坐标解码

尽管坐标解码被认为是模型测试中一个不重要的组成部分，但本文研究发现，坐标解码对人体姿态估计的性能有重要贡献。具体来说，这是一个将关节预测heatmap转换为原始图像空间中的坐标的过程。假设heatmap与原始图像具有相同的空间尺寸，只需要找到最大激活位置作为关节坐标预测，这种方法简单直接。但这并不是通用的方法。相反，通常需要使用一个针对特定样本的无约束因子λ ∈ R+，将heatmap上采样至原始图像一样的分辨率。这涉及到亚像素定位问题。在介绍本文方法之前，先回顾现有姿态估计模型中使用的标准坐标解码方法。

**标准坐标解码方法**    是根据模型性能经验设计的。具体来说，给定一个训练模型预测的heatmap h，首先确定最大(m)和第二最大(s)激活值的坐标。然后将关节位置预测为：

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/func1.png" width="404" height="64"/></div>

|| . ||2定义为向量的数量级。这个公式表示在heatmap空间中最大激活向第二大激活偏移0.25个像素后获得的结果。原始图像中的最终坐标预测计算如下：

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/func2.png" width="321" height="52"/></div>

λ表示分辨率下降率。

*备注*    等式(1)中亚像素偏移的目的是补偿图像分辨率下采样的量化效应。也就是说，预测heatmap中的最大激活并不对应于关节在原始坐标空间中的准确位置，仅对应一个粗略位置。这种转换显著提升了性能，令人惊讶。这解释了此操作经常被用作模型测试标准操作的原因。目前还没有具体的工作深入研究这种操作对人体姿态估计性能的影响。这种标准方法在设计上缺乏直觉和可解释性，但一直没有专门去改进。本文针对偏移估计提出一种原则性方法，获得了更加精确的人体姿态估计结果，从而填补了这一空白。

**所提坐标解码方法**    探索了二预测heatmap的分布结构，用于推理潜在的最大激活。这与上述依靠手工设计偏移预测的标准方法大有不同，上述方法几乎没有设计依据和理论基础。

具体来说，为了获得亚像素级的准确位置，本文假设预测heatmap遵循2D高斯分布，与gt heatmap相同。因此，预测heatmap表示为：

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/func3.png" width="542" height="66"/></div>

x表示预测heatmap中的像素位置，µ是对应待估计关节位置的高斯平均（中心）。协方差∑是一个对角线矩阵，与坐标编码使用的相同：

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/func4.png" width="360" height="82"/></div>

σ表示两个方向有相同的标准差。

在对数似然优化原则中，使用对数转换G，便于推理，同时保持最大激活的原始位置为：

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/func5.png" width="509" height="121"/></div>

本方法的目标是估算μ。作为分布中的一个极值点，μ点的一阶导数满足如下条件：

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/func6.png" width="521" height="100"/></div>

为了探索这个条件，这里采用泰勒定理。形式上，在预测heatmap的最大激活m处得出的泰勒级数（直到二次项）来近似激活P(μ)，表示为：

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/func7.png" width="545" height="75"/></div>

D''(m)表示在m处得出的P的二阶导数（即Hessian），形式上定义为：

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/func8.png" width="433" height="87"/></div>

选择m来近似μ的直觉是，它能代表一个接近μ的良好粗略关节预测。

将公式(6)(7)(8)组合，最终获得：

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/func9.png" width="431" height="63"/></div>

D''((m) 和 D'(m)从heatmap中可以有效估计。一旦获得μ，也可以用公式(2)来预测原始图像空间的坐标。

*备注*    与只考虑heatmap中第二大激活的标准方法相比，所提坐标解码充分探索了热图分布统计信息，以便更准确地揭示潜在的最大激活。理论上，在训练监督一致性假设下，本文方法基于一个原则性的分布近似，即假设heatmap是高斯分布。关键是计算非常高效，只需要计算每个heatmap上一个位置的一阶和二阶导数。因此，现有人体姿态估计方法可以在没有任何计算成本妨碍的情况下提升效果。

**heatmap分布调制**    由于所提坐标解码方法是基于高斯分布假设，因此有必要检查该条件的满足程度。本文发现，与训练heatmap数据相比，由人体姿态估计模型预测的heatmap通常不具有良好的高斯结构。如下图(a)所示，heatmap通常在最大激活周围呈现多个峰值。这可能会对解码方法的性能造成消极影响。为了解决这个问题，本文建议预先调整heatmap分布。

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/fig3.png" width="658" height="312"/></div>

具体来说，为了满足本文方法的要求，提出利用高斯核K来平滑heatmap h中多个峰值的影响，高斯核的变量与训练数据采用的变量相同，平滑形式如下：

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/func10.png" width="340" height="49"/></div>

*指定为卷积运算。

为了保持原始heatmap的大小，本文最终通过以下转换缩放h'，使其最大激活等于h的最大激活：

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/func11.png" width="453" height="74"/></div>

其中max()和min()分别返回输入矩阵的最大值和最小值。实验分析验证了这种分布调制进一步提高了本文坐标解码方法（表3）的性能，产生的视觉效果和定性评估如上图(b)所示。

**总结**    本文在图2中总结了所提坐标解码方法。具体而言，总共涉及三个步骤：a) heatmap分布调制（等式(10)，(11)）；b) 通过亚像素精度泰勒展开获得分布感知关节定位（等式(3) - (9)）；c) 分辨率恢复到原始坐标空间（等式(2)）。这些步骤都不会增加过多的计算成本，因此能够作为现有模型的有效插件。

<a name="4.2"></a>

### 4.2 坐标编码

上节讨论了坐标解码问题，其根源是分辨率降低了。坐标编码也有同样的局限性。具体来说，标准坐标编码方法从下采样原始人体图像到网络输入尺寸开始。在生成heatmap之前， gt关节坐标需要进行相应的转换。

形式上，用 g=(u, v) 表示关节的gt坐标。降低分辨率的过程定义为：

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/func12.png" width="406" height="58"/></div>

λ是下采样率。

通常情况，为了便于生成kernel，我们通常量化g'：

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/func13.png" width="521" height="60"/></div>


quantise()表示量化函数，常用选项包括floor、ceil和round。

随后，通过以下方式合成heatmap，heatmap以量化坐标g''为中心：

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/func14.png" width="520" height="68"/></div>


其中(x, y)表示heatmap的像素位置，σ表示固定的空间方差。

显然，由于量化误差，以上述方式生成的heatmap并不准确且有偏差（如下图）。这可能会引入次优监督信号，并导致模型性能下降，尤其是在本文所提精确坐标编码情况下。

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/fig4.png" width="184" height="172"/></div>

为了解决这个问题，只需将heatmap中心点设置为无偏量化位置g'，该位置代表了准确的gt坐标。本文仍然使用公式(14)，但是将g''替换为g'。本文将论证这种无偏heatmap生成方法的好处（表3）。

<a name="4.3"></a>

### 4.3 集成到SOTA模型

DARK方法与模型无关，可以与任何现有基于heatmap的姿态模型无缝集成，并且这些算法不需要修改。在训练过程中，唯一的变化是根据精确的关节坐标生成gt heatmap数据。测试阶段，将任意模型预测的heatmap作为输入，在原始图像空间中输出更精确的关节坐标。在整个生命周期中，本文方法保持现有模型与原始设计无异。这使得本文方法具有最大的通用性和可扩展性。

<a name="5"></a>

## 5. 实验

**数据**    使用COCO和MPII两种常用人体姿态估计数据。COCO keypoint数据集包含各种人体姿态、无约束环境、不同身体比例和遮挡模式。每个目标包含人体实例和人体关节点。数据集包含20万张图像共25万个人体样本。每个人体实例都标注有17个关节。训练集和验证集的标注是公开的benchmark。验证阶段，本文遵循常用拆分方法，将数据集拆分为train2017/val2017/test-dev2017。MPII人体姿态数据集包含4万个人体样本，每个样本标注有16个关节。本文遵循标准拆分方法，拆分为train/val/test。

**评价指标**    使用目标关键点相似度(Object Keypoint Similarity, OKS)评估COCO性能，使用准确关键点百分比(Percentage of Correct Keypoints, PCK)评估MPII性能。

**实施细节**    训练阶段使用Adam优化器。对于HRNet和SimpleBaseline，采用原始方法设置的学习率调整策略和epoch。对于Hourglass，基础学习率微调为2.5e-4，在第90和120个epoch分别衰减到2.5e-5和2.5e-6。epoch总共设置成140。本文实验使用三种不同的输入尺寸（128×96、256×192、384×288）。数据预处理采用Sun等人提出的方法。

<a name="5.1"></a>

### 5.1 评估坐标表示

本文首先研究了坐标表示对模型性能的影响，以及与输入图像分辨率（尺寸）的关系。在测试中，默认使用HRNet-W32作为主干模型，28×96作为输入尺寸，并在COCO验证集上展示准确性结果。

**(i) 坐标解码**    本文评估了坐标解码的效果，尤其是偏移操作和分布调制。使用传统的偏置heatmap。测试中，比较了所提无偏分布感知偏移（即直接使用最大激活位置）方法和标准偏移方法。主要观察两项：1）标准偏移可提高5.7%的AP精度，效果惊人。这揭示了坐标解码对人体姿态估计的重要性。2）所提方法将AP分数进一步提高了1.5%，分布调制给出了0.3%的提升。如表2所示。这验证了本文解码方法的优越性。

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/tab2.png" width="525" height="158"/></div>

**(ii) 坐标编码**    本文验证了坐标编码的有效性。对比所提无偏编码和标准有偏编码，以及标准解码和所提解码方法。表3表示，无论采用哪种坐标解码方法，所提无偏编码都会提升性能。特别的，无偏编码在这两种情况下能持续提供超过1%的AP增益。这表明坐标编码的重要性。

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/tab3.png" width="527" height="214"/></div>

**(iii) 输入分辨率**    考虑到输入图像的分辨率/尺寸是与模型推理效率相关的一个重要因素，本文通过测试许多不同尺寸来验证其影响程度。将DARK模型（HRNet-W32为主干）与原始HRNet-W32进行比较，使用有偏heatmap监督进行训练，并使用标准偏移进行测试。观察结果如下：1）随着输入图像尺寸的减小，模型性能持续下降，而推理成本下降明显。2）加入DARK后，模型性能损失得到有效缓解，尤其是在输入分辨率非常小的情况下。这有助于在低资源设备上部署人体姿态估计模型。

**(iv) 普适性**    除了HRNet，还在SimpleBaseline和Hourglass上进行了测试。表5结果表明，在大多数情况下，DARK能为现有模型提供显著的性能增益。这表明所提方法具有普适性。图5中展示了定性评估。

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/tab5.png" width="908" height="372"/></div>

**(v) 复杂性**    在HRNet-W32上测试了所提方法在128×96输入尺寸下对推理效率的影响。在Titan V GPU上，低效python环境中，运行速度从360fps下降到320fps，即下降了11%。这种额外性能损失是可以接受的。

<a name="5.2"></a>

### 5.2 与SOTA模型比较

**(i) 在COCO上的评估**    比较DARK和SOTA方法，包括G-RMI、积分姿态回归、CPN、RMPE、SimpleBaseline和HRNet。表6显示了SOTA方法和DARK在COCO test-dev上的准确性结果。在本测试中，借用了Sun等人的人体检测结果。观察结果如下：在输入尺寸为384×288的情况下，HRNet-W48 + DARK可以获得最佳精度，无需额外的模型参数，且成本增加很小。具体地，与具有相同输入尺寸的HRNet-W48相比，DARK将AP提高了0.7%（76.2-75.5）。与积分姿态回归相比，DARK（HRNet-W32）的AP增益为2.2%（70.0-67.8），而运行成本仅为16.4%（1.8/11.0 GFLOPs）。这些都表明DARK在准确性和效率方面优于现有模型。

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/tab6.png" width="855" height="347"/></div>

**(ii)  在MPII上的评估**    比较DARK与HRNet-W32。表7表明，本文方法具有性能优势。在更严格的精度测量标准PCKh@0.1之下，DARK的性能优势更为显著。MPII提供的训练数据比COCO小得多，这表明本文方法适用于不同大小的训练数据。

<div align=center><img src="../images/DarkPose_Distribution-Aware_Coordinate_Representation_for_Human_Pose_Estimation/tab7.png" width="520" height="281"/></div>

<a name="6"></a>

## 6. 结论

本文工作首次系统地研究了在无约束图像中用于人体姿态估计的坐标表示（包括编码和解码）这一基本上被忽略但却很重要的问题。不仅揭示了这个问题的真正意义，而且提出了一种新的分布感知关键点坐标表示方法（DARK），用于更具辨别力的模型训练和推理。作为一个随取随用的插件组件，现有SOTA模型可以无缝地从DARK方法中受益，无需任何算法调整，增加的计算成本可以忽略不计。除了从经验上证明坐标表示的重要性外，还在两个具有挑战性的数据集上进行了广泛的模型对比实验，验证了DARK的性能优势。提供了一系列深入的组件分析，以深入了解模型公式的设计原理。
