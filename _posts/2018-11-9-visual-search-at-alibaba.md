---
layout: post
author: memory.sh
title:  "阿里-拍立淘架构分析"
background: '/img/visualsearch/back.png'
---
分析了阿里在视觉搜索方面的应用-拍立淘。参考文献：[Visual Search At Alibaba](/article/visualsearch.pdf)

## 面对的挑战
* Heterogeneous images matching
* Billions of data with fine-grained categories
* Huge expense for maintaining training data
* Improving the user engagement

## 视觉搜索应用到的技术
* CNN应用到实例检索
	1. 将CNN作为NeuralCode
	2. 将全连接层的输出作为实例检索的特征
	3. 编码CNN特征和卷积特征并映射到VLAD
	4. max-pooling作用到所有卷积特征上，产生有效的视觉符号
	5. 问题：阿里视觉搜索重点关注图像中目标对象，而非整个图像
* 深度度量嵌入：在图片的相似性度量上有很好的性能
	1. 在线硬采样挖掘，计算triplet loss+用户点击习惯
* 弱监督对象定位
	1. 自学习对象定位-掩模(masking)
	2. 结合多个实例的CNN特征定位

## 主要工作
* 介绍一种高效的类别预测方法
该方法结合了模型和基于搜索的
* 提出一种深度CNN模型，用于联合检测和特征学习
* 在应用部署阶段，利用二进制索引和重排序完成检索

## 可视化搜索架构
* 架构概览
> 该架构主要分为在线处理和离线处理两个部分。离线处理：主要工作是建立索引文档。
> 文档主要包括属性选择、离线特征提取、索引建立。在线处理：主要包括用户上传图片
> 的种类预测、在线检测、特征提取，最后返回索引和重拍后的结果。
![架构概览](/img/visualsearch/overview.png)
* 种类预测
	1. 过滤图片
	2. 聚合模型和基于搜索的方法
> 在模型方面，采用GoogleNet V1。训练集为多种类带标签的库存子集。输入的图像
> 大小为256X256，并随机裁剪为227x227。使用训练过程使用标准的softmax-loss。
> 在基于搜索方面，利用深度网络输出的特征，在200亿参考图片中，索引Top30的结果。
> 并给与每个结果一个权重去预测类别。
* 联合检测和特征学习
> 为了匹配买家和卖家之间的图片，提出了一中基于深度度量学习的带分支的深度CNN模型。
> 该模型能够同时对象检测和特征学习。
	1. PVLOG triplet 挖掘
	![Triplets](/img/visualsearch/triplets.png)
	2. 深度排序框架
	![Joint](/img/visualsearch/joint.png)
> 
	![Rank](/img/visualsearch/rank.png)
* 图像索引和检索
> 为了适应实时和稳定的效果采用了**多重复**和**多分片**的搜索引擎架构。
> 多分片：使用多个机器去存储完整的数据集，每个分片保存完整数据集的子集。
> 对于每一个请求，每个分片节点搜索自己的子集并返回K个结果，之后，
> 将这每个分片的结果排序合并产生最终的K个结果。
> 多重度：多个重复的机器。假设我们同时有Q请求访问系统，我们将这Q个请求分成R
> 份，每一部分有Q/R个请求。使用这种方法，将一个索引集群需要处理Q个请求降低到了Q/R。
	![Search](/img/visualsearch/search.png)
## 参考文献
[Visual Search At Alibaba](/article/visualsearch.pdf)















