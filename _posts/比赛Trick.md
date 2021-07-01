# Trick

#### 对于日期数据可以将其转化为与某一时间点的差值

```python
def transform_day(date1):
    date2 = "2020-01-01"
    date1 = time.strptime(date1, "%Y-%m-%d")
    date2 = time.strptime(date2, "%Y-%m-%d")

    # 根据上面需要计算日期还是日期时间，来确定需要几个数组段。下标0表示年，小标1表示月，依次类推...
    # date1=datetime.datetime(date1[0],date1[1],date1[2],date1[3],date1[4],date1[5])
    # date2=datetime.datetime(date2[0],date2[1],date2[2],date2[3],date2[4],date2[5])
    date1 = datetime.datetime(date1[0], date1[1], date1[2])
    date2 = datetime.datetime(date2[0], date2[1], date2[2])
    # 返回两个变量相差的值，就是相差天数
    # print((date2 - date1).days)  # 将天数转成int型
    return (date2 - date1).days
```

#### 对于连续点击特征，行为顺序信息

使用word2vec对其进行embedding

word2vec   中skip-gram 使用中心词来预测相邻词

#### 改变数据分布

在经典的 YouTube 深度推荐模型中，我们就可以看到一些很有意思的处理方法。比如，在处理观看时间间隔（time since last watch）和视频曝光量（#previous impressions）这两个特征的时，YouTube 模型对它们进行归一化后，又将它们各自处理成了三个特征（图 6 中红框内的部分），分别是原特征值 x，特征值的平方x^2，以及特征值的开方

#### Embeddeding

node2vec  https://github.com/ChuanyuXue/KDDCUP-2020/blob/master/code/2_Similarity/deep_node_model.py

word2vec

one-hot

multi-hot