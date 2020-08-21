[toc]

# NFM模型详解

## 背景

> FM能够有效的建模特征之间的二阶组合关系，但问题在于，其捕获的二阶组合特征是线性组合的（其表达式就是线性组合），无法捕获非线性组合特征。
>
> 深度神经网络可以有效的建模特征的线性，例如谷歌的Wide&Deep，微软的DeepCross，但缺点是很难从头训练。
>
> 受FNN的启发，如果在进入多层网络之前能够用FM进行预训练，得到embedding向量，再进入NN中能够得到更好的效果，如图所示:

img

> 基于此，**本文提出NFM模型，其能将FM模型捕获的二阶线性组合特征以及神经网络捕获的高阶非线性组合特征组合起来，从而取得更好的效果。**

## 模型

img

> **NFM丢弃了直接把embedding vector拼接输入到神经网络的做法，而是在embedding层后增加了Bi-Interaction操作来对二阶组合特征进行建模。这使得low level的输入表达的信息更加的丰富，极大的提高了后面隐藏层学习高阶非线性组合特征的能力。**其中f(x)是NFM的核心，用来学习二阶组合特征和高阶的组合特征模式。前面两项为线性回归部分，与FM相似。

### Embedding Layer

img

> 输入向量x经过embedding层之后, 变成了embedding向量集合，<a href="https://www.codecogs.com/eqnedit.php?latex=\nu&space;_{x}&space;=&space;\{x_{i}v_{i}\},&space;x_{i}\neq&space;0" target="_blank"><img src="https://latex.codecogs.com/svg.latex?\nu&space;_{x}&space;=&space;\{x_{i}v_{i}\},&space;x_{i}\neq&space;0" title="\nu _{x} = \{x_{i}v_{i}\}, x_{i}\neq 0" /></a>。请注意，这里通过输入要素值重新调整了嵌入向量，而不是简单地嵌入表查找。

### Bi-Interaction Layer

img

> 该层定义了一种新的pooling操作: 将embedding向量集合<a href="https://www.codecogs.com/eqnedit.php?latex=\nu&space;_{x}" target="_blank"><img src="https://latex.codecogs.com/svg.latex?\nu&space;_{x}" title="\nu _{x}" /></a>转换为单一向量，具体公式如上，其中符号<a href="https://www.codecogs.com/eqnedit.php?latex=\odot" target="_blank"><img src="https://latex.codecogs.com/svg.latex?\odot" title="\odot" /></a>代表两个向量的元素相乘，上述公式的作用是将embedding向量两两元素相乘，之后将所有组合的结果求和。最终结果将是一个k维度向量，k就是embedding向量的长度。
>
> 这一层实现了embedding后特征的二阶交叉，没有引入新的模型参数, 而且可以在线性时间复杂度内就能计算完成。
>
> 上式可以参考FM算法得到优化，如下：

img



### Hidden Layer





### Prediction Layer



### 对比FM 

img

> FM是一种浅层的线性模型，**可以看做是一个神经网络架构，就是去掉隐藏层的NFM**，把去掉隐藏层的NFM称为NFM-0。**这是第一次把FM看做是神经网络来处理，这样的观点对于优化FM提供了一些新的思路。同时，像NN中常用的技巧也可以应用到这里面来，比如Dropout，实验发现在正则化FM的时候，使用Dropout比传统的L2正则化还要有效。**

## 实验

> 作者在Frape和MovieLens数据集上比较了NFM模型自身结构对模型效果的影响，并比较了LibFM、HOFM、Wide & Deep、DeepCross和NFM模型的效果，具体如下：

### Bi-Interaction Pooling

#### Dropout和L2正则化对避免模型overfitting的影响

img

> 在Bi-interaction层采用dropout的效果要优于l2正则，一个原因可能是强制L2只会在每次更新中以数字方式抑制参数值，而使用dropout可以看作是ensembling多个子模型，dropout方法不止能够用于nn中,对于带隐因子的线性模型也有很好的抑制过拟合的效果。

#### Batch Normalization对加速模型训练的影响

img

> BN可以加速训练,使得收敛更快, 模型的泛化能力能够提升

### Impact of Hidden Layers



> NFM中隐层个数不是越多越好，实验证明，一层的效果最好。原因如下：



#### 预训练对加速模型训练的影响

img

> 用FM embedding做预训练能够加速训练，但是对于NFM最终效果的影响不大，如下：

img

**这说明NFM模型的鲁棒性很好, 与wide&deep以及deepCross相比, NFM对参数初始化相对不那么敏感**

### Performance Comparison

img



## 代码

https://github.com/hexiangnan/neural_factorization_machine