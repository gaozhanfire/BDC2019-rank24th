# BDC2019-rank24th
中国高校计算机大赛-大数据挑战赛，Rank24 解决方案
---
# 赛题描述：  
https://www.kesci.com/home/competition/5cc51043f71088002c5b8840/content/1  
我对业务的简单理解：  
实际上就是搜索引擎，你搜一个  “我是蔡”，即一个query  
对应可能的搜索结果（即title）：①我是蔡虚鲲 ②我是菜徐坤 ③我是共产党的接班人  
这时候你很可能就会去点 我是蔡徐坤  
这就是这个比赛的任务，给你一个query-title对，让你预测这个query-title对 被点击的概率，即可看做一个二分类问题，标签为0或1.  

---
# 赛题解决方案：
通过分析可知，  
①query和title的相关度（也就是相似度）越高，这个query-title 被点击的概率也就越大，  
②query-title的内容越吸引人，被点击的概率也就越大  
③可以根据一个query或一个title曾被浏览过/点击过 的次数，来判断这个query/title 的点击概率是不是很高

于是可制定以下方案：  
①手动/通过nn，来提取query和title相似度特征（诸如query和title公共词数量，非公共词数量等等）  
②通过nn，来提取query-title的语义信息  
③对query和title的历史点击率（即如 title出现过的次数 title被点击的次数   title出现过的次数/title被点击的次数）  

---
所用特征：  
所用的相似度特征与其他队无太大差别。  
主要还是fuzzy和莱文斯顿这两个库里的相似度特征。  

**本队的trick特征：**
- ctr特征（三个特征即可达到0.578的baesline）：
    * 拿复赛举例，复赛10亿训练数据，我们选取了最后5千万条数据进行训练。  
      我们使用了没用来训练的那9.5亿条数据进行groupby('title')['label'].agg({'count':'title_count','sum','title_sum','mean','title_ctr'})
      来构造出了点击率特征，count的含义，就是该条title的出现量（投放量），而sum，就是title出现了这么多次，这其中有多少次用户点击了该条title。
      *我们对ctr特征进行了贝叶斯平滑，以此取得了两个千分位的提升，具体代码已包含在文件中*  
          - 我们没有对query进行统计，这是因为query在测试集和训练集的重复率并不高，所以没法利用query的历史信息。
        
- tsvd降维特征（3个千的提升）：
    * 我们对query的tfidf矩阵进行tsvd降维。这个特征很多队伍没有用到，原因是tfidf维度过高。
      *我们采用的方法是使用了gensim库里的lsi主题模型*，lsi的本质即对矩阵进行tsvd降维。
      我们曾尝试降到50维 30维 20维，最终结果为20维效果最佳。
 
      

