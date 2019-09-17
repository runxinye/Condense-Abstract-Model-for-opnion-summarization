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
#### Condense Model
输入：N篇评论文档，一篇文档有M个词（每篇文档词的个数不一样）
输出：N篇文档的文档编码![](http://latex.codecogs.com/gif.latex?{d_i})，和词编码![](http://latex.codecogs.com/gif.latex?h_{i,1},h_{i,2},...,h_{i,M})，![](http://latex.codecogs.com/gif.latex?1<=i<=N)。<br>

**1.word2vec**<br>
通过一些预训练好的语言模型将这些词编码成词向量<br>
记为![](http://latex.codecogs.com/gif.latex?X=(w_1,w_2,...,w_M))<br>

**2.Bi-LSTM Encoding**<br>
Bi-LSTM编码得到d和h<br>
![](image/2.jpg)<br>
其中![](http://latex.codecogs.com/gif.latex?\overrightarrow{h_i})和![](http://latex.codecogs.com/gif.latex?\overleftarrow{h_i})分别是向前LSTM和向后LSTM的隐层态。<br>

**3.LSTM解码**<br>
令初始隐层态![](http://latex.codecogs.com/gif.latex?z_0=d)<br>
![](image/3.jpg)<br>
其中词![](http://latex.codecogs.com/gif.latex?{w_t}')是由softmax生成的。<br>

**4.Training**<br>
训练使用最大似然损失函数<br>
![](image/4.jpg)<br>
我们通过训练模型参数，要使得当![](http://latex.codecogs.com/gif.latex?{w_t}'=w_t)时，最大似然损失函数是最小的。<br>

#### Abstract Model
**1.Multi-source Fusion** <br>
本质：使用attentive pooling methodN对文档编码![](http://latex.codecogs.com/gif.latex?{d_i})和词编码![](http://latex.codecogs.com/gif.latex?h_{i,1},h_{i,2},...,h_{i,M})融合在一起，得到融合后的文档编码![](http://latex.codecogs.com/gif.latex?d')和词编码![](http://latex.codecogs.com/gif.latex?{h_1}',{h_2}',...,{h_M}')。<br>
记![](http://latex.codecogs.com/gif.latex?D_d)为![](http://latex.codecogs.com/gif.latex?{d_i})的维数，权重向量参数![](http://latex.codecogs.com/gif.latex?a_i) ![](http://latex.codecogs.com/gif.latex?\in) ![](http://latex.codecogs.com/gif.latex?R^{D_d})<br>
文档编码的融合如下：<br>
![](image/5.jpg)<br>
其中![](http://latex.codecogs.com/gif.latex?\bar{d})是查询向量，![](http://latex.codecogs.com/gif.latex?W_p)![](http://latex.codecogs.com/gif.latex?\in)![](http://latex.codecogs.com/gif.latex?R^{D_d*D_d*D_d})是模型参数。<br>
词编码的融合如下：<br>
记V为N篇文档中所有词（排重后）的个数，![](http://latex.codecogs.com/gif.latex?V_{w_j})为词![](http://latex.codecogs.com/gif.latex?w_j)在所有文档中出现的次数。<br>
![](image/6.jpg)<br>

**2.Decoder**<br>
使用LSTM decoder + attentive mechanism + copy mechanism模型<br>
*(1)LSTM*<br>
令初始隐层态![](http://latex.codecogs.com/gif.latex?s_0=d')<br>
![](image/7.jpg)<br>
*(2)attentive mechanism*<br>
在第t时间，引入attentive mechanism输出attention weight向量![](http://latex.codecogs.com/gif.latex?a_t)和上下文向量![](http://latex.codecogs.com/gif.latex?c_t)<br>
![](image/8.jpg)<br>
*(3)copy mechanism*<br>
通过分别得到输出词的生成概率（generation probability）![](http://latex.codecogs.com/gif.latex?p_g({y_t}'))和复制概率（copy probability）![](http://latex.codecogs.com/gif.latex?p_c({y_t}'))，然后结合attention weight向量![](http://latex.codecogs.com/gif.latex?a_t)得到最终的词向量概率![](http://latex.codecogs.com/gif.latex?p({y_t}'))<br>
![](image/9.jpg)<br>

**3.Salience-biased Extracts**<br>
模型现阶段为止还是把所有的文档一视同仁，但是有的文档更加重要，所以为了鼓励模型更重视重要的那些文档，我们直接加入了额外信息。<br>
*(1)预选k篇重要文档*<br>
通过 SUMMARUNNER方法（一种extractive model）预选出k篇重要文档，拼接成一个长序列<br>
*(2)Bi-LSTM encoder + LSTM decoder*<br>
对长序列进行BiLSTM encoding，输出的结果作为输入进行LSTM decoding，得到隐层态序列{![](http://latex.codecogs.com/gif.latex?r_t)}<br>
*(3)在Abstract Model - decoder - LSTM步骤中更新*<br>
令![](http://latex.codecogs.com/gif.latex?s_t=[s_t;r_t])<br>
<br>
注意的是模型仍旧是把所有文档信息都纳入了模型，只不过更加重视那些重要的文档了。<br>

**4.Training**<br>
这个是一个有监督的模型训练过程，本身最优的评论总结为![](http://latex.codecogs.com/gif.latex?Y={y_1,y_2,...,y_L})，但是通过代入概率![](http://latex.codecogs.com/gif.latex?p({y_t}'))最小化最大似然损失函数：<br>
![](image/10.jpg)<br>
除此之外，在Multi-source Fusion阶段，文章融合得到的文档编码![](http://latex.codecogs.com/gif.latex?d')需要同最优的评论总结y的condense模型结果z=Condense(y)尽量相等，因此这一步引入hinge损失函数，需要![](http://latex.codecogs.com/gif.latex?d')与z的距离近，而离其他任意的随机评论总结的condense模型结果向量（记为![](http://latex.codecogs.com/gif.latex?n_i)）远，在这里选了5个任意随机评论总结。<br>
![](image/11.jpg)<br>
![](image/12.jpg)<br>

#### Zero-shot Customization
这个模型还有一个优点就是可以根据用户的偏好来生成评论总结，这里只需要修改一小步。<br>
在Multi-source Fusion 步骤中首先要得到一个查询向量![](http://latex.codecogs.com/gif.latex?\bar{d})，原本模型是利用所有文档编码求平均得到的，现在只需要引入一些有背景信息的评论文档（例如，想要生成一篇有正向情绪的评论总结，就增加一些有正向情绪的评论）。<br>
记![](http://latex.codecogs.com/gif.latex?\C_x)是这些有背景信息的评论文档的集合，则![](http://latex.codecogs.com/gif.latex?\hat{d}=\sum_{c=1}^{|C_x|}d_c/|C_x|)，![](http://latex.codecogs.com/gif.latex?\hat{d})就代替了![](http://latex.codecogs.com/gif.latex?\bar{d})成为新的查询向量，从而实现了用户偏好的实现。

### 总结
从效果上看，CA效果比EA模型好，信息量更多，zero-shot customization technique 在测试中也能够很好的产生情绪相关或者概况相关的评论总结。<br>
在未来上，CA模型需要尝试在不同的多文档总结的任务，并且尝试开发半监督或者无监督的方法。







