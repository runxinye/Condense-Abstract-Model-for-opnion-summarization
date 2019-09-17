# Condense-Abstract-Model-for-opnion-summarization
## Paper share: Informative and Controllable Opinion Summarization

### 摘要
自动评论总结任务（例如电影、产品的评论）一般是基于神经网络的，通常来说是分为两步：第一步、利用Extract Model从大量评论中预选出重要的突出评论；第二步、利用Abstract Model基于预选出来的重要评论产生一个评论总结。Extract-Abstract Model（简称EA模型）的缺点是损失了大量信息并且无法灵活地根据用户的偏好生成某些有意向性的评论总结。<br>
本文提出了一个新的两步神经网络模型Condense-Abstract Model（简称CA模型），评论总结任务的本质是多源文本信息的转化，第一步利用Condense Model将所有相关评论压缩成多个稠密的向量作为第二步的输入，第二步利用Abstract Model生成评论总结。<br>
CA模型不仅解决了EA模型损失大量信息的问题，还可以通过引入一个简单的定制化的零数据工具，从而生成有特定偏好的评论总结。

### 介绍
CA模型运用在从多个评论文档中提取一个评论总结的场景。<br>
常见的EA模型和本文提出的CA模型的示意图如下:<br>
![](image/1.jpg)
EA模型的第一步Extractive Model有两个缺点：信息损失和用户偏好无法纳入模型，而CA模型都有很好的改善这些问题。

### 方法
**Condense Model**
输入：N篇评论文档，一篇文档有M个词（每篇文档词的个数不一样）
输出：N篇文档的文档编码![](http://latex.codecogs.com/gif.latex?{d_i})，和词编码![](http://latex.codecogs.com/gif.latex?h_{i,1},h_{i,2},...,h_{i,M})，![](http://latex.codecogs.com/gif.latex?1<=i<=N)。<br>

第一步：通过一些预训练好的语言模型将这些词编码成词向量
记为![](http://latex.codecogs.com/gif.latex?X=(w_1,w_2,...,w_M))

第一步：Bi-LSTM编码得到d和h
![](image/2.jpg)
其中![](http://latex.codecogs.com/gif.latex?\overrightharpoon{h_i})和![](http://latex.codecogs.com/gif.latex?\overleftharpoon{h_i})分别是向前LSTM和向后LSTM的隐层态。<br>
第二步：LSTM解码
![](http://latex.codecogs.com/gif.latex?z_0=d)
![](image/3.jpg)
其中词![](http://latex.codecogs.com/gif.latex?{w_t}')是由softmax生成的。<br>
第三步：最大似然损失函数
![](image/4.jpg)
我们通过训练模型参数，要使得当![](http://latex.codecogs.com/gif.latex?{w_t}'=w_t)时，最大似然损失函数是最小的。<br>







