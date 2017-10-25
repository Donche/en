---
layout: post
title: 推荐系统实践概论
category: 算法实践
tags: 
    - 2017
    - 算法
    - 推荐系统
---
# 关于偷懒
不，我并没有偷懒   
即使已经有个把月没有搞博客   
I've being having a hard time living, just like everyone else.    
这段时间看了很多机器学习数据挖掘的东西，感觉是时候动手了    
就是现在，先写一个简单的推荐系统   
~~然后再写一个复杂的~~   
对，是推荐电影的系统   
为什么是电影，因为我并没有找到音乐的数据集   
后摇、自赏、dream pop、indie、后核、鞭挞、旋死、烤鸡翅膀，我的最爱   
仔细想想，电影其实也不错   
就这样了   

部分代码在之后的博客里

*本文参考《推荐系统实践》（1）*

# 推荐系统架构
推荐系统是联系用户与物品的媒介，连接的方式主要有这么几种：用户->物品->物品、用户->用户->物品、用户->特征->物品。
实际上，经过一层抽象之后，我们可以将用户或者物品也看做一种特征，这样我们就只有用户->特征->物品这一种方式了。   
于是接下来的任务便成为，1.如何为用户找到特征 和 2.如何依靠特征找到对应的物品。
1.用户特征可以简要分为如下：
* 人口统计学特征：年龄、性别、国籍和民族等
* 用户行为特征：浏览、评分、收藏等数据
* 用户话题特征：如用户对某一类型的电影感兴趣等     

2.推荐方式也有很多种，比如：
* 推荐最新物品
* 推荐需要商业推广的物品
* 推荐不同种类的物品
* 混合推荐
* 推荐不同新颖度的物品，如长尾中的物品
* 考虑用户访问的上下文

在同一个推荐算法里考虑这么多内容是很复杂的一件事。所以我们分开，写出不同的推荐引擎，然后利用加权得到最终结果即可。这样不仅更方便调试，而且更用户友好。比如我，只考虑访问上下文和特征就行（然而我依然不会点开网易云给我的每日推荐啊哈哈哈哈哈），比如很多其他用户就需要在一定时间段内流行趋势的推荐

# 推荐引擎的设计
引擎主要包括三部分：
* 提取用户特征向量
* 根据特征-物品生成推荐物品列表
* 对结果进行过滤、排名等处理   
于是，一个一个来吧：

## 提取用户特征向量
一般来说用户会有两种特征，一种是用户的人口统计学信息，一种是用户的行为信息。   
用户的人口统计学信息主要考虑：
* 好友
* 地理位置信息
* 年龄信息
* 性别

用户的行为信息需要主要考虑：
* 行为种类：点击、浏览、评分、留言、分享......
* 行为时间：近期的行为应有更高的权重
* 行为次数：次数越高权重越高
* 热门程度：很热门但并没有产生行为->很不感兴趣；很冷门但经常产生行为->很感兴趣

## 特征-物品生成推荐
得到用户的特征向量后，我们可以根据离线的相关表得到初始的物品推荐列表。对于一个推荐引擎，可以在
配置文件中配置很多相关表以及它们的权重，而在线服务在启动时会将这些相关表按照配置的权
重相加，然后将最终的相关表保存在内存中，而在给用户进行推荐时，用的已经是加权后的相关
表了。   
特征—物品相关推荐模块除了给用户返回物品推荐列表，还需要给推荐列表中的每个推荐结
果产生一个解释列表，表明这个物品是因为哪些特征推荐出来的。

## 过滤、排名
需要过滤的物品主要是以下几种：
* 用户已经产生过行为的
* 候选物品以外的（如果有候选物品的话）
* 质量很差的（如评分很低的）
一般来说排名需要不同的模块：
* 新颖性：尽量推荐用户不知道的、长尾中的产品。需要注意的是，用户没有产生行为并不代表用户不知道。具体可以通过对热门物品降权等方式。
* 多样性：可以通过每个类别各取其中排名最高的物品组成最终的推荐列表，或者尽量使用不同的推荐引擎得出的结果。
* 时间多样性：可以将给用户之前推荐过的物品降权。
* 用户反馈：通过反馈预测用户的兴趣。


# 算法指标
衡量一个推荐系统的优劣，可以从多个方面入手。
## 准确率和召回率
令R(u)是根据用户在训练集上的行为给用户作出的推荐列表，而T(u)是用户在测试集上的行
为列表。那么，推荐结果的召回率定义为：
$$Recall = \frac{\sum_{u\in U}|R(u)\bigcap T(u)|}{\sum_{u\in U}|T(u)|}$$
准确率定义为：
$$Recall = \frac{\sum_{u\in U}|R(u)\bigcap T(u)|}{\sum_{u\in U}|R(u)|}$$
## 覆盖率
覆盖率（coverage）描述一个推荐系统对物品长尾的发掘能力。覆盖率有不同的定义方法，最简单的定义为推荐系统能够推荐出来的物品占总物品集合的比例。一般来说，覆盖率越高，推荐系统越能达到推荐的目的，使各类物品都能平等的展示在用户面前
## 多样性
一般来说，用户的兴趣都是多样的。那么，为了满足用户广泛的兴趣，推荐列表需要能够覆盖用户不同的兴趣领域，即推荐结果需要具有多样性。举例来说，就是不能因为用户同时喜欢李志和黑麒，而李志的流行度远高于黑麒，推荐系统就给出万晓利麻油叶赵雷等等...


*参考资料*
1. 《推荐系统实践》 项亮