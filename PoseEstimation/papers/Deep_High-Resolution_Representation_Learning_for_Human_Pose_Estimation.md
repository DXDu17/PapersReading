# Deep High-Resolution Representation Learning for Human Pose Estimation

------

原文链接：[点这里](https://arxiv.org/abs/1902.09212)

## 目录

- [1. 摘要](#1)
- [2. 介绍](#2)
- [3. 相关工作](#3)
- [4. 方法](#4)
- [5. 实验](#5)
  - [5.1 COCO关键点检测](#5.1)
  - [5.2 MPII人体姿态估计](#5.2)
  - [5.3 姿态跟踪应用](#5.3)
  - [5.4 消融实验](#5.4)
- [6. 结论及未来工作](#6)
- [7. 附录](#7)

<a name="1"></a>

## 1. 摘要

本文对人体姿态估计问题感兴趣，重点学习可靠的高分辨率表示。大多数现有方法是从high-to-low分辨率网络产生的低分辨率表示中恢复高分辨率表示。相反，本文提出的网络在整个过程中始终保持了高分辨率表示。

第一阶段是一个高分辨率的子网络，接着逐个添加high-to-low分辨率子网络，形成多个阶段，并将多分辨率子网络并行连接。本文进行重复的多尺度融合，使得每个high-to-low分辨率表示可以反复接收到其他并行网络的表示信息，从而得到丰富的高分辨率表示。因此，预测的关键点heatmap可能会更精确。通过在两个基准数据集（COCO关键点检测数据集和MPII人体姿态数据集）上评估模型性能，证明了本文所提网络的有效性。此外还展示了本文网络在PoseTrack数据集上的优越性。

<a name="2"></a>

## 2. 介绍

2D人体姿态估计一直是计算机视觉领域中富有挑战的问题。其目标是定位人体解剖关键点（如肘部、手腕等）或部位。它有很多应用，包括人体动作识别、人机交互、动画等。本文研究的是单人姿态估计，它是其他相关问题的基础，如多人姿态估计、视频姿态估计和跟踪等。

近期发展表明，深度卷积神经网络已经取得了SOAT性能。大多数现有方法将输入送到网络中，这些网络通常由串联的high-to-low分辨率子网络及提高分辨率的网络组成。例如，Hourglass通过low-to-high对称过程恢复高分辨率。SimpleBaseline采用了几个反卷积层来生成高分辨率表示。此外，膨胀卷积也应用在了high-to-low分辨率网络（例如VGGNet或ResNet）之后。

本文提出了一种新的网络结构，即High-Resolution Net（HRNet），它能够在整个过程中保持高分辨率表示。本文将一个高分辨率子网络作为第一阶段，接着逐个添加high-to-low分辨率子网络，形成多个阶段，并将多分辨率子网络并行连接。在整个过程中通过并行的多分辨率子网络反复交换信息来实现重复的多尺度融合。最终由网络输出的高分辨率表示来估计关键点。
HRNet网络结构如下所示：

<div align=center><img src="../images/Deep_High-Resolution_Representation_Learning_for_Human_Pose_Estimation/hrnet.png" width="626" height="305"/></div>

与目前广泛使用的姿态估计网络相比，本文网络有两个优点：(i) 所提方法是并行连接high-to-low分辨率子网络，不是串连。因此能够保持高分辨率，而不是通过从低到高的过程恢复分辨率，因此预测的heatmap在空间上可能会更精确；(ii) 大多数现有融合方案聚合了低层和高层表示。相反，本文使用重复的多尺度融合，并在相同深度和相似阶段低分辨率表示的帮助下，提高高分辨率表示，从而使高分辨率表示也可以用于姿态估计。因此，本文预测的heatmap可能更准确。

实验证明，关键点检测在COCO关键点检测数据集和MPII人体姿态数据集上表现出优越的性能。此外，在PoseTrack数据集上展示了本文网络在视频姿态跟踪方面的优势。

<a name="3"></a>

## 3. 相关工作

大多数传统单人姿态估计解决方案采用概率图模型或图结构模型，近期工作利用深度学习对这种方案进行了改进，能够更好的建模一元和成对能量，或是模拟迭代过程。目前，深度卷积神经网络提供了主要的解决方案，分为两种主流方法：1) 回归关键点的位置；2) 估计关键点heatmap，然后选择热值最高的位置作为关键点。

大多数用于关键点heatmap估计的卷积神经网络由一个类似于分类网络的stem子网络组成，该子网络降低了分辨率，网络主体生成与其输入具有相同分辨率的表示，然后接一个回归器估计heatmap及关键点位置，最后转换至全分辨率。主体主要采用high-to-low和low-to-high框架，可能会增加多尺度融合和中间（深度）监督。

**High-to-low和low-to-high**    high-to-low过程旨在生成低分辨率和高层表示，low-to-high过程旨在生成高分辨率表示。为了提高性能，这两个过程可能会重复多次。

典型的网络设计模式包括：(i) 对称的high-to-low和low-to-high过程。Hourglass及其扩展将low-to-high过程设计为high-to-low过程的镜像。(ii) Heavy high-to-low和light low-to-high。high-to-low过程基于ImageNet分类网络，例如采用ResNet，low-to-high过程只是几个双线性上采样或转置卷积层。(iii) 合并膨胀卷积。文27，51，35中，在ResNet或VGGNet的最后两个阶段采用膨胀卷积，以消除空间分辨率损失，然后进行light low-to-high处理，以进一步提高分辨率，避免仅使用膨胀卷积导致昂贵的计算成本。下图描述了四个具有代表性的姿态估计网络。

<div align=center><img src="../images/Deep_High-Resolution_Representation_Learning_for_Human_Pose_Estimation/pose_estimation_networks.png" width="727" height="293"/></div>

**多尺度融合**    简单的方法是将多分辨率图像分别送入多个网络，再聚合输出响应图。 Hourglass及其扩展通过跳连接，将high-to-low过程中的低层特征逐渐组合到low-to-high过程中相同分辨率的高层特征中。在级联金字塔网络中，globalnet将high-to-low过程中的low-to-high层特征逐步组合到low-to-high过程，然后refinenet将卷积处理后的low-to-high层特征组合起来。本文方法重复了多尺度融合，部分灵感来自深度融合及其扩展方法。

**中间监督**    早期为图像分类而开发的中间监督或深度监督也被用于帮助深度网络训练和提高heatmap估计质量。Hourglass方法和卷积姿态机方法将中间heatmap作为剩余子网络的输入或部分输入进行处理。

**本文方法**    本文网络并行连接high-to-low子网络。网络在整个过程中保持高分辨率表示，用于精确的heatmap估计。通过反复融合high-to-low子网络生成的表示，生成可靠的高分辨率表示。本文方法不同于大多数现有的工作，它们需要一个单独的low-to-high上采样过程，并聚合低层和高层表示。本文方法在不使用中间heatmap监督的情况下，在关键点检测精度和计算复杂度及参数方面具有优越性。

分类和分割有相关的多尺度网络。本文部分工作受到了其中一些因素的启发，但两者之间存在着明显的差异，所以这些网络不适用于本文的问题。由于每个子网络（depth, batch normalization）和多尺度融合缺乏适当的设计，卷积神经结构以及相互关联的CNN无法产生高质量的分割结果。网格网络（由多个权值共享的U-Net组成）由两个独立的多分辨率表示融合过程组成：第一阶段，信息仅从高分辨率送到低分辨率；第二阶段，信息只能从低分辨率送到高分辨率，因此缺乏竞争力。多尺度densenet不针对目标，也无法生成可靠的高分辨率表示。

<a name="4"></a>

## 4. 方法

人体姿态估计，也称为关键点检测，旨在从大小为W×H×3的图像I中检测k个关键点或部位（例如肘部、手腕等）的位置。SOTA方法将此问题转化为估计出大小为W′×H′, {H1，H2，…，HK}的K个heatmap，heatmap k表示第K个关键点的位置置信度。

本文遵循广泛采用的pipeline，使用卷积网络预测人体关键点，该网络由一个stem组成，stem由两个降分辨率卷积组成，主体输出的feature map分辨率与其输入的feature map分辨率相同，回归器用于估计heatmap，并将关键点位置并转换至全分辨率。本文关注主体设计，并介绍HRNet。

**序列多分辨率子网络**    现有姿态估计网络是通过串连high-to-low子网络来构建的，每个子网络为一个阶段（由一系列卷积组成），相邻子网络之间有一个下采样层，用以将分辨率减半。

设Nsr为第s阶段的子网络，r为分辨率指数。high-to-low网络具有S（例如S取4）个阶段：

<div align=center><img src="../images/Deep_High-Resolution_Representation_Learning_for_Human_Pose_Estimation/func1.png" width="348" height="39"/></div>

**并行多分辨率子网络**    本文以一个高分辨率子网络作为第一阶段，逐个添加high-to-low分辨率子网络，形成新的阶段，并将多分辨率子网络并行连接。因此，后一阶段并行子网络的分辨率由前一阶段的分辨率和一个较低的分辨率组成。

一个包含4个并行子网络的网络结构如下所示：

<div align=center><img src="../images/Deep_High-Resolution_Representation_Learning_for_Human_Pose_Estimation/func2.png" width="339" height="94"/></div>

**重复多尺度融合**    本文在并行子网络中引入交换单元，使得每个子网络可以重复的从其他并行子网络接收信息。下面是一个信息交换方案示例。本文将第三阶段划分为几个（如3个）交换block，每个block由3个并行卷积单元组成，每个并行单元之间有一个交换单元，如下图所示：

<div align=center><img src="../images/Deep_High-Resolution_Representation_Learning_for_Human_Pose_Estimation/func3.png" width="484" height="109"/></div>


其中，Cbsr表示第s阶段中第b个block的第r分辨率的卷积单元，Ebs是对应的交换单元。

本文在下图中说明了交换单元，并在下面给出公式。为了便于讨论，去掉了下标s和上标b。输入是s个响应图：{X1，X2，…，Xs}。输出是s个响应图：{Y1，Y2，…，Ys}，其分辨率及宽度与输入相同。每个输出都是输入映射的聚合。跨阶段的交换单元有一个额外的输出映射。

<div align=center><img src="../images/Deep_High-Resolution_Representation_Learning_for_Human_Pose_Estimation/exchange_unit.png" width="798" height="245"/></div>

函数a(Xi, k)由从分辨率i到分辨率k的上采样或下采样Xi组成。采用步长为3×3的卷积进行下采样。例如，一个步长为3×3的卷积和一个步长为2的2×下采样，两个连续的步长为3×3的卷积和一个步长为2的4×下采样。对于下采样，为了对齐通道数，本文在1×1卷积后采用简单的最近邻采样。

**Heatmap估计**    本文从最后一个交换单元输出的高分辨率表示回归heatmap。损失函数定义为均方误差，用于比较预测和gt heatmap。gt heatmap由2D高斯分布生成，高斯分布以每个关键点的gt位置为中心，标准偏差为1个像素。

**网络实例化**    本文遵循ResNet的设计规则，将深度分配到每个阶段，并将通道数分配到每个分辨率，从而实例化关键点heatmap估计网络。

主体（即HRNet）包含四个阶段，每个阶段是一个平行子网络，分辨率每个阶段降低一半，宽度（通道数）增加为原先的两倍。第一阶段包含4个残差单元，每个单元（与ResNet-50相同）是宽度为64的bottleneck模块，后接一个3×3卷积，将特征图的宽度减小到C。第二、第三、第四阶段分别包含1、4、3个交换block。一个交换block包含4个残差单元，每个单元在同一分辨率中包含两个3×3卷积，两个分辨率之间包含一个交换单元。综上所述，共有8个交换单元，即进行了8次多尺度融合。

在本文实验中，研究了一个小网络和一个大网络：HRNet-W32和HRNet-W48，其中32和48分别代表最后三个阶段的高分辨率子网络的宽度（C）。HRNet-W32最后三个并行子网络的宽度分别为64、128、256，HRNet-W48的分别为96、192、384。

<a name="5"></a>

## 5. 实验

<a name="5.1"></a>

### 5.1 COCO关键点检测

**数据**    COCO数据集包含超过20万张图像和25万标记有17个关键点的人体实例。本文在COCO train2017数据集上训练模型，包括57K张图片和150K个人体实例。在val2017和test-dev2017上评估本文方法，两个数据集分别包含5000张图像和20K张图像。

**评价指标**    标准评价指标使用目标关键点相似度（OKS）。di表示检测到的关键点和相应的gt之间的欧氏距离，vi表示gt是否可见，s表示目标比例，ki是控制衰减的关键点常数。本文给出了标准平均精确度和召回分数。

**训练**    本文将人体检测框的高度或宽度扩展到固定的纵横比：高:宽=4:3，然后从图像中裁剪出该框，并将其调整为固定大小，256×192或384×288。数据增强包括随机旋转([−45°, 45°])，随即尺度变换([0.65,1.35])和翻转。随后是半身数据增强。

本文使用Adam优化器，学习策略采用文72方法。初始学习率设置为1e-3，在第170个epoch衰减至1e-4，第200个epoch衰减至1e-5，总的epoch是210。

**测试**    二阶段top-down方法：使用人体检测算法检测出人体实例，然后预测关键点。

本文在验证集和测试集上使用相同的人体检测器。按照惯例，通过计算原始图像和反转图像的heatmap均值获取最终的heatmap。使用从最高响应到第二高响应方向偏移四分之一的方法调整最高热值位置，来预测每个关键点的位置。

**在验证集上的结果**    表1展示了本文方法与其他SOTA方法的结果。小网络HRNet-W32从零开始训练，输入大小为256×192，AP分数为73.4，优于相同输入大小的其他方法。(i) 与Hourglass相比，本文小网络将AP提高了6.5，GFLOPs降低至不到Hourglass的一半，但本文的参数略大。(ii) 与CPN w/o 和 w/OHKM 相比，本文网络模型尺寸稍大，复杂性稍高，分别获得4.8和4.0的增益。(iii) 与SimpleBaseline相比，HRNet-W32获得了显著的提高：当使用相同的主干网络ResNet-50，并且具有相似的模型尺寸和GFLOPs时，本文增益提高3.0；使用相同的主干网络ResNet-152，而具有两倍于本文模型尺寸和GLOPs时，本文增益提高1.4。

<div align=center><img src="../images/Deep_High-Resolution_Representation_Learning_for_Human_Pose_Estimation/table1.png" width="742" height="281"/></div>

本文网络受益于：(i) 使用了针对ImageNet分类问题的预训练模型做训练：HRNet-W32的增益为1.0；(ii) 通过增加宽度来增加容量：对于256×192和384×288的输入尺寸，HRNet-W48分别得到了0.7和0.5的改进。

对于输入大小384×288，HRNet-W32和HRNet-W48的AP分别为75.8和76.3，与输入大小256×192相比，分别有1.4和1.2的改进。与使用ResNet-152作为主干的SimpleBaseline相比，HRNet-W32和HRNet-W48分别以45%和92.4%的计算成本实现了1.5和2.0的AP增益。

**在测试集上的结果**    表2展示了本文方法与其他SOTA方法的姿态估计性能。本文方法明显优于bottom-up方法。HRNet-W32的AP为74.9，它优于所有其他top-down方法，并且在模型大小（Params）和计算复杂性（GFLOPs）方面更高效。HRNet-W48的AP为75.5。与输入大小相同的SimpleBaseline相比，本文小型和大型网络分别得到1.2和1.8的提高。通过AI Challenger提供的额外数据进行训练，本文大网络的AP为77.0。

<div align=center><img src="../images/Deep_High-Resolution_Representation_Learning_for_Human_Pose_Estimation/table2.png" width="731" height="386"/></div>

<a name="5.2"></a>

### 5.2 MPII人体姿态估计

**数据集**    MPII人体姿态数据集由从各种真实活动中拍摄的图像组成，带有全身姿态标注。有约25K张图像共40K个目标，其中12K个用于测试，其余用于训练。数据增强和训练策略与MS COCO相同，只是输入大小被裁剪为256×256，以便与其他方法进行公平比较。

**测试**    测试过程与COCO中的几乎相同，只是采用了标准测试策略，使用提供的人体box而不是检测到的box。

**评价指标**    使用标准度量PCKh分数。如果关节位于gt位置的αl像素范围内，则该关节是正确的，其中α是一个常数，l是头部大小，对应于gt头部边界框对角线长度的60%。（PCKh@0.5（α=0.5））

**在测试集上的结果**   表3和表42展示了PCKh@0.5结果，模型大小和GFLOP是最常用的方法。本文使用ResNet-152作为主干网络，输入大小为256×256，重新实现了SimpleBaseline。HRNet-W32的PKCh@0.5分数为92.3，优于堆叠hourglass方法及其扩展。还测试了HRNet-W48，得到了相同的结果92.3，原因可能是在此数据集上的性能趋于饱和。

<div align=center><img src="../images/Deep_High-Resolution_Representation_Learning_for_Human_Pose_Estimation/table3.png" width="418" height="395"/></div>

<div align=center><img src="../images/Deep_High-Resolution_Representation_Learning_for_Human_Pose_Estimation/table4.png" width="418" height="214"/></div>

<a name="5.3"></a>

### 5.3 姿态跟踪应用

**数据集**    PoseTrack是人体姿态估计和关节跟踪的大规模基准视频。该数据集基于MPII人体姿态数据集提供的原始视频，包含550个视频序列，共66374帧。视频序列分为292、50、208个视频，分别用于训练、验证和测试。训练视频的长度在41-151帧之间，视频中间的30帧被密集标注。验证/测试视频的长度在65-298帧之间。MPII姿态数据集中关键帧周围的30帧被密集标注，之后每四帧被标注一次。总的来说，大约包括23000个带标签的帧和153615个姿态注释。

**评价指标**    本文从两个方面评估结果：基于帧的多人姿态估计和多人姿态跟踪。姿态估计通过文51和28中的平均精度（mAP）进行评估。多人姿态跟踪由多目标跟踪精度（MOTA）评估。

**训练**    本文在PoseTrack2017训练集上训练HRNet-W48进行单人姿态估计，网络由COCO数据集上预先训练的模型初始化。通过将所有关键点（一个人）的边界框扩展15%的长度，用以从训练帧带注释的关键点中提取人体框，作为网络的输入。训练设置（包括数据增强）与COCO几乎相同，只是学习策略不同（微调）：学习率从1e−4开始，在第10个epoch下降到1e−5，第15个epoch下降到1e-6，20个epoch迭代结束。

**测试**    本文按照文72来跟踪帧间姿态。包括三个步骤：人体框检测及传播，人体姿态估计，跨相邻帧的姿态关联。使用SimpleBaseline中的人体框检测器；根据FlowNet 2.0计算的光流传播预测的关键点，将检测到的框传播到附近的帧中；然后使用非最大值抑制进行框移除。姿态关联方案基于当前帧关键点与根据光流从附近帧传播的关键点之间的目标关键点相似度(OKS)。然后使用贪婪匹配算法计算相邻帧关键点之间的对应关系。

**在PoseTrack2017测试集上的结果**    表5展示了结果。HRNet-W48取得了优异的成绩，mAP分数为74.9，MOTA分数为57.9。与SimpleBaseline相比，本文方法在mAP和MOTA方面分别获得0.3和0.的增益。与FlowTrack相比，同样具有优势。这进一步说明本文姿态估计网络的有效性。

<div align=center><img src="../images/Deep_High-Resolution_Representation_Learning_for_Human_Pose_Estimation/table5.png" width="492" height="362"/></div>

<a name="5.4"></a>

### 5.4 消融实验

本文研究了每个部分在COCO关键点检测数据集的影响。除了关于输入大小影响的研究外，所有结果都是在256×192的输入大小上获得的。

**重复多尺度融合**    本文分析了重复多尺度融合的效果。研究三个网络变体：(a) W/o 中间交换单元（1个融合）：除最后一个交换单元外，多分辨率子网络之间没有交换；(b) W/ 跨阶段交换单元（3个融合）：每个阶段内的并行子网络之间没有交换；(c) 跨阶段和阶段内交换单元（共8次融合）：这是本文提出的方法。所有网络都是从头开始训练的。表6给出了在COCO验证集上的结果，说明多尺度融合是有帮助的，更多融合会带来更好的性能。

<div align=center><img src="../images/Deep_High-Resolution_Representation_Learning_for_Human_Pose_Estimation/table6.png" width="502" height="200"/></div>

**保持分辨率**    本文研究了HRNet一种变体的性能：所有四个high-to-low分辨率子网络都添加在最开始，并且深度相同；融合方案都一样。HRNet-W32和变体都是从头开始训练并在COCO验证集上测试的。该变体的AP为72.5，HRNet-W32的AP为73.4。本文认为出现这种情况的原因是从低分辨率子网络早期阶段提取的低层别特征没有那么有用。此外，在没有低分辨率并行子网络的情况下，参数和计算复杂度相似的简单高分辨率网络的性能要低得多。

**分辨率表示**    本文从两个方面研究了分辨率表示对姿态估计性能的影响：检查从高到低分辨率特征图估计的heatmap质量，研究输入大小对质量的影响。

本文使用在ImageNet上的预训练模型初始化小网络和大网络。网络从高到低输出四个响应图。最低分辨率响应图上的heatmap预测质量最低，AP分数低于10。比较表明，分辨率确实会影响关键点预测质量。

下图显示了与SimpleBaseline（ResNet-50）相比，输入图像大小如何影响性能。本文发现，较小输入的改善比较大输入的改善更显著，例如256×192改善4.0，128×96改善6.3，原因是在整个过程中都保持了高分辨率。另一方面，我本文输入为256×192的方法优于输入为384×288的SimpleBaseline。

<div align=center><img src="../images/Deep_High-Resolution_Representation_Learning_for_Human_Pose_Estimation/fig6.png" width="442" height="322"/></div>

<a name="6"></a>

## 6. 结论及未来工作

本文提出一种用于人体姿态估计的高分辨率网络，产生精确的关键点heatmap。其成功来自两个方面：(i) 整个过程保持高分辨率，不需要恢复高分辨率；(ii) 反复融合多分辨率表示，以提供可靠的高分辨率表示。

未来的工作包括在其他密集预测任务中的应用，例如语义分割、目标检测、人脸对齐、图像翻译，以及聚合多分辨率表示的研究。

<a name="7"></a>

## 7. 附录

#### 在MPII验证集上的结果

本文提供了在MPII验证集的上结果。模型在MPII训练集的子集上训练，在包含2975张图像的heldout验证集上评估。训练过程与完整MPII训练集的训练过程相同。heatmap由原始图像和翻转图像的平均heatmap计算获得，用于测试。本文还进行了六尺度金字塔测试过程（多尺度测试）。

<div align=center><img src="../images/Deep_High-Resolution_Representation_Learning_for_Human_Pose_Estimation/table7.png" width="423" height="361"/></div>

#### 在PoseTrack数据集的更多结果

本文提供了在PoseTrack数据集上所有关键点的结果。HRNet-W48在验证集和测试集上的mAP分别为77.3和74.9，比之前的SOTA方法分别高出0.6和0.3。还提供了在PoseTrack2017测试集上多人姿态跟踪性能的详细结果，作为补充。

<div align=center><img src="../images/Deep_High-Resolution_Representation_Learning_for_Human_Pose_Estimation/table8.png" width="408" height="326"/></div>

#### 在ImageNet验证集上的结果

将本文网络应用于图像分类任务。模型在ImageNet 2013分类数据集上训练和评估。训练了100个epoch，batch size为256。初始学习率设置为0.1，在第30、60和90个epoch时降低10倍。本文模型可以实现与专门为图像分类设计的网络（如ResNet）相当的性能。HRNet-W32单模型top-5验证误差为6.5%，top-1验证误差为22.7%。HRNet-W48获得了更好的性能：top-5误差为6.1%，top-1误差为22.1%。本文使用在ImageNet数据集上的预训练模型初始化姿态估计网络。

<div align=center><img src="../images/Deep_High-Resolution_Representation_Learning_for_Human_Pose_Estimation/table9.png" width="413" height="182"/></div>
