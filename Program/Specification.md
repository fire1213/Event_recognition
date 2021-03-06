## 实验说明

#### 1.数据描述

1.总体数据概况

原始微博总数为：181950条（54.7M），删除缺失值后数据180669条，人工标注数据有3600条，删除分词后空微博后剩下3490条，将其分成训练集和测试集，测试集占（1/5）。

---
2.包含的企业有：

['中兴通讯', '华为中国区', 'vivo智能手机', 'coolpad官方微博', 'OPPO', '联想', '小米公司', '金立智能手机', "小辣椒","360手机", 'HTC官方微博', '魅族科技', '天语手机', '超级手机官微', '一加手机', '朵唯女性手机', '锤子科技',  'TCL通讯中国','nubia智能手机']

---
3.标签映射关系为： {"招募": 1, "合作": 2, "研发": 3, "推广": 4, "销售": 5, "无": 6}




#### 2.实验思路

##### Dataprocess类：
---
1.读取原始语料数据（raw_data）, 删除微博内容为空的实例(corpora_clean_data), 对语料库(corpora_clean_data)进行分词,并删除分词后为词序列为空的微博实例最终得到(corpora_split_content),将分词后的语料库进行word2vec训练, 分词后对语料库中所有数据进行表征得到(sentence_represent)

2.同时读取带有标签的微博数据，形成target_data，去除分词后空的实例，然后把数据分为训练集和测试集

---
##### Trigger 类：
1. 从训练集中形成针对事件的前top个高频种子动词(top1默认是400)，根据词频对种子动词进行加权表征形成种子向量(seed_vec)，依据种子向量对测试集中剩余前高频（top2默认2000，相似度阈值设置为threshold=0.8）进行触发器识别，形成加权企业事件向量（event_vec）.

---
##### Trigger_classifier类：
2. 基于加权的企业事件向量对测试集中进行cosine相似度比较(相似度阈值设置为threshold=0.7）,选择相似度最高的企业事件，如果相似度低于阈值，那么这一微博不包含企业事件。

#####  other_method 类
对比方法：使用了sklearn上的接口， SVM+BOW， SVM+Word2vec，Bayes+BOW，NN(简单的多层感知器)。

##### 指标
PRF（with weight）

#### 实现结果
见Results目录


-----------------------------------------------
#### Version_2:
原始微博总数为：181950条（54.7M）,只保留对应企业的微博共计181442条（47.8M）.
中兴通讯 512
华为中国区 599
vivo智能手机 262
coolpad官方微博 333
OPPO 283
联想 176
小米公司 480
金立智能手机 55
小辣椒 20
360手机 108
HTC官方微博 131
魅族科技 76
天语手机 46
超级手机官微 34
朵唯女性手机 50
锤子科技 138
TCL通讯中国 155
nubia智能手机 142
从语料库中随机抽取了600个数据实例用于测试（test_data.csv）

#### 实验过程：
##### 预处理过程：Preprocess_2

1.statistics.py 从Data_2原始数据中筛选出上述企业的微博实例共计（181442条）（../Data_2/raw/clean_company_corpora.csv）。
人工根据关键词标注数据3600条（../Data_2/raw/target.csv）
2.datapreprocess.py 读取新语料库数据以及目标数据，对其进行分词、训练词向量以及句子表征，并从新语料库中随机抽取600条数据实例用于测试
（test_data.csv）
3.对测试集数据进行人工标注（test_data_label.csv）

---
##### 触发器识别：trigger_2

参数介绍：
weight_1, weight_2, threshold, top_1, top_2 enrich, source_from, new = True, True, 0.8, 500, 2000,True, 1, False
weight_1: default=True 表示种子向量形成是通过词频加权
weight_2: default=True 表示事件向量形成是通过词频加权
threshold：default=0.8 种子向量识别剩余动词时所用的相似度阈值
top_1: 种子向量的高频动词选择，选择前top_1个动词
top_2： 剩余动词中前top个高频动词用于触发器的识别
source_from: default=1 表示剩余高频动词来自于整个语料库，=2表示来自于测试集
new：为True 不用种子动词，事件表征用触发器识别后的动词

1. 基于target_data,形成事件动词即：{e1:[],e2:[],e3:[],e4:[],e5:[],e6:[]}
事件：1, 总计种子数973
事件：2, 总计种子数956
事件：3, 总计种子数1177
事件：4, 总计种子数1243
事件：5, 总计种子数1064
事件：6, 总计种子数1039
2. 选择前top_1=500个高频动词 并基于词频计算得到种子向量seed_vec
3. 对剩余动词进行触发器识别，选择相似度最大的一类事件，如果其大于阈值，则将其加入到触发器中。
例如：
增加词：买到,频率:296,相似度:0.8212696313858032,对应事件5
增加词：加送,频率:214,相似度:0.8349044322967529,对应事件5
增加词：降临,频率:197,相似度:0.8200686573982239,对应事件6
增加词：无比,频率:189,相似度:0.8043558597564697,对应事件6
增加词：猛击,频率:178,相似度:0.8499212265014648,对应事件1
增加词：坐等,频率:172,相似度:0.8169016242027283,对应事件4
增加词：下手,频率:161,相似度:0.8773057460784912,对应事件5
增加词：精选,频率:128,相似度:0.8113262057304382,对应事件4
增加词：留在,频率:128,相似度:0.8176668882369995,对应事件6
增加词：献给,频率:123,相似度:0.808208167552948,对应事件6
:增加词：迎锋,频率:118,相似度:0.818031907081604,对应事件6
增加词：控股,频率:118,相似度:0.8256145405213025,对应事件2
增加词：换上,频率:106,相似度:0.8190345168113708,对应事件6
4. 对最终形成的触发器词，根据词频进行事件加权表征得到event_vec

##### 分类器：trigger_classifer
1.利用event_vec 对微博实例进行分类， 选择相似度最大且大于阈值的那个一类事件。

#### 实验结果
（1）参数实验
1. word2vec 模型参数:选择skip-gram, 窗口window=4, 维度size=150
2. 对种子动词前top个词进行参数实验,差距不大，选择种子动词的top_1=400
3. 对剩余高频动词前top_2进行触发器识别,选择top_2=1000
4. 对触发器相似度阈值threshold_1进行相似度识别, 选择0.7
5. 对分类器，相似度阈值threshold_2, 选择0.8
---
（2）对比实验
我们方法的特点：word2vec+trigger, 通过word2vec表征词向量, 再通过trigger丰富触发事件的动词, 构造分类器后通过阈值识别微博中的企业事件.

对比结果见Results目录.


