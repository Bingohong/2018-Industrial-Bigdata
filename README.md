# 2018工业大数据创新竞赛-钢卷仓储吞吐量趋势预测
- 比赛介绍：[2018工业大数据创新竞赛](http://www.industrial-bigdata.com/datacompetition)
- 时间：2018.11-2019.01
- 赛题：[钢卷仓储吞吐量趋势预测](http://www.industrial-bigdata.com/competition/competitionAction!showDetail34.action?competition.competitionId=4)
- 成绩：25/329（84.7118）

# 赛题思路
## 1.数据清洗整理
- 去除入库为0,出库>0的数据
- 丢弃，数量=0而重量>0 重量=0而数量>0的数据
- 转换为储户id-产品id-日期格式三层索引,重量为训练值
- 无记录时间记出入库重量为0
- 补齐中断的缺失时间

## 2.特征工程
- 提取每[3,5,7,14]周期内得统计量，包括均值、方差、中位数、加权均值方差等
- 近14天的原始数据
- 滑窗选取时间窗，提取时间窗范围的统计量作为特征
- 有大量稀疏值，可以选择合并，可有效降低误差
- 数据存在异方差性，取log1p减少异方差，可降低误差
- 滑动平均平滑原始数据，可降低误差
- 周数据
  - 截取时间段->/1000->ewm->merge->np.log1p->resample
  - 周数据预测使用累计方式计算，然后逐次相减
  - resample代表按1周、2周、3周、4周重采样
- 天数据
  - 截取时间段->/1000->ewm->merge->np.log1p

## 3.模型
- xgboost 分出库和入库两类模型，每天/每周一个模型
  - 损失函数：rmse
  - booster:"gblinear"
  - 学习率策略：step decay
  
## 4.深度学习尝试
- CNN+LSTM
  - causal dilated convolution
  - 直接使用原始数据就能跑出84+的成绩
  - 原始数据需要缩放，不用提取滑窗的统计量特征
  - cnn感受野需要覆盖窗口时间段
- 调参难度大，时间不够，我是比赛结束前半个月开始搞得，精力时间有限

## 5.资源资料
- 花了些时间慢慢总结的时间序列预测方法，全在知乎
  - [有什么好的模型可以做高精度的时间序列预测呢？](https://www.zhihu.com/question/21229371/answer/533770345)
  - 包括：ARIMA + xgboost + CNN + seq2seq(attention) + LSTM + Prophet
  
## 6.数据
- 数据太大，需要的给我发邮件
- 包括：原始训练数据，测试数据

# 其余解法
- prophet（89.3213）
- xgboost/LSTM/模型融合
