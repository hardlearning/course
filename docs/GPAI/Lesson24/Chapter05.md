# 第5章 隐马尔科夫模型HMM

## 1 马尔科夫模型

天气变化种类：晴天、多云、雷雨，他们之间可以相互转换

![HMM01](https://gitlab.com/iknowledge/CourseImage/-/raw/main/GPAI/HMM/HMM01.png)

状态之间可以发生转换，昨天和今天转换的情况如下：

![HMM02](https://gitlab.com/iknowledge/CourseImage/-/raw/main/GPAI/HMM/HMM02.png)

一阶马尔科夫模型：今天能得到明天的情况，明天能得到后天的情况，假设今天的状态为T1，明天的状态为T2，后天的状态为T3，一阶就表示T3只和T2有关系，T2只和T1有关系，T3和T1没有关系。

这里我们定义一个一阶马尔科夫模型：
- 状态：晴天，多云，雷雨
- 状态转换概率：三种天气状态间的转换概率
- 初始状态：昨天是晴天

![HMM03](https://gitlab.com/iknowledge/CourseImage/-/raw/main/GPAI/HMM/HMM03.png)

计算今天(t=1)的天气状况：

今天为晴天的概率=初始晴天概率x晴天转晴天概率
             +初始多云概率x多云转晴天概率
             +初始雷雨概率x雷雨转晴天概率

![HMM04](https://gitlab.com/iknowledge/CourseImage/-/raw/main/GPAI/HMM/HMM04.png)

## 2 隐马尔科夫模型

现在我们漂到了一个岛上，这里没有天气预报，只有一片片海藻。这些海藻能观察到的状态为：Dry、Dryish、Damp、Soggy。这里我们就没有直接的天气信息了，有的是间接的信息，海藻的状态跟天气的变换有一定的关系。既然海藻是能看到的，那它就是**观察状态**，天气信息看不到就是**隐藏状态**。

观察状态和隐藏状态可能并不是一一对应的。

![HMM05](https://gitlab.com/iknowledge/CourseImage/-/raw/main/GPAI/HMM/HMM05.png)

当前的状态只和前一状态有关：

$$
P(z_t|z_{t-1},x_{t-1},z_{t-2},x_{t-2},\ldots,z_1,x_1)=P(z_t\mid z_{t-1})
$$

某个观测只和生成它的状态有关：

$$
P(x_{t}|z_{t},x_{t},z_{t-1},x_{t-1},z_{t-2},x_{t-2},\ldots,z_{1},x_{1})=P(x_{t}\mid z_{t})
$$

z表示隐藏状态，x表示观察状态。

![HMM06](https://gitlab.com/iknowledge/CourseImage/-/raw/main/GPAI/HMM/HMM06.png)

隐藏状态可以生成观察状态，也可以转换成另一个隐藏状态。

### 隐马尔科夫模型的组成

三个必备条件：初始概率($\pi$)，隐藏状态转移概率矩阵(A)，生成观测状态概率矩阵(B)

$$
HMM=(\pi,A,B)
$$

![HMM07](https://gitlab.com/iknowledge/CourseImage/-/raw/main/GPAI/HMM/HMM07.png)

### 要解决的问题

模型为$\lambda=(\pi,A,B)$，O表示观测状态序列，I表示隐藏状态序列。

1. 给定模型$(\pi,A,B)$及观测序列$O=\{o_{1},o_{2},...o_{T}\}$，计算其出现的概率$P(O|\lambda)$；
2. 给定观测序列$O=\{o_{1},o_{2},...o_{T}\}$，求解参数$(\pi,A,B)$使得$P(O|\lambda)$最大；
3. 已知模型$(\pi,A,B)$及观测序列$O=\{o_{1},o_{2},...o_{T}\}$，求状态序列，使得$P(I|O,\lambda)$最大。

### 求观测序列的概率

暴力求解：我们要求的是在给定模型下观测序列出现的概率，那如果我能把所有的隐藏序列都给列出来，也就可以知道联合概率分布$P(O,I|\lambda)$

$$
P(O|\lambda)=\sum_IP(O,I|\lambda) \\
P(O,I|\lambda)=P(I|\lambda)P(O|I,\lambda)
$$

$P(I|\lambda)$在给定模型下，一个隐藏序列出现的概率，是由初始状态一步步转换。
$I=\{i_1,i_2,\ldots,i_T\}$出现的概率为：$P(I|\lambda)=\pi_{i_1}a_{i_1i_2}a_{i_2i_3}\ldots a_{i_{T-1}i_T}$

推导：

$$
P(I|\lambda)=P(i_1,i_2,\ldots,i_T|\lambda) \\
=P(i_1|\lambda)P(i_2,\ldots,i_T|i_1) \\
=P(i_1|\lambda)P(i_2|i_1)P(i_3,\ldots,i_T|i_2) \\
=\pi_{i_1}a_{i_1i_2}a_{i_2i_3}\ldots a_{i_{T-1}i_T}
$$

对于固定的隐藏序列$I=\{i_1,i_2,\ldots,i_T\}$，得到观察序列$O=\{o_{1},o_{2},...o_{T}\}$的概率：$P(O|I,\lambda)=P(o_1|i_1)P(o_2|i_2)\ldots P(o_T|i_T)=b_{i_1}(o_1)b_{i_2}(o_2)\ldots b_{i_T}(o_T)$

联合概率：$P(O,I|\lambda)=P(I|\lambda)P(O|I,\lambda)=\pi_{i_1}b_{i_1}(o_1)a_{i_1i_2}b_{i_2}(o_2)\ldots a_{i_{T-1}i_T}b_{i_T}(o_T)$

观测序列概率：$P(O|\lambda)=\sum\limits_IP(O,I|\lambda)=\sum\limits_{i_1,i_2,...i_T}\pi_{i_1}b_{i_1}(o_1)a_{i_1i_2}b_{i_2}(o_2)\ldots a_{i_{T-1}i_T}b_{i_T}(o_T)$

复杂度：如果隐藏状态数有N个，$O(TN^T)$

### 前向算法

给定t时刻的隐藏状态为i，观测序列为$y_{1},y_{2},\ldots y_{t}$的概率叫做前向概率：

$$
\alpha_i(t)=p(y_1,y_2,\ldots y_t,q_t=i|\lambda)
$$

![HMM08](https://gitlab.com/iknowledge/CourseImage/-/raw/main/GPAI/HMM/HMM08.png)

当t=T时，$\alpha_i(T)=p(y_1,y_2,\ldots y_T,q_T=i|\lambda)$，这表示最后一个时刻，隐藏状态位于第i号状态上并且观测到$y_1,y_2,\ldots y_T$的概率。

如果能把所有可能的状态都拿过来，那就可以得到我们要求的目标了：$p(y_1,y_2,\ldots y_T|\lambda)=\alpha_1(T)+\alpha_2(T)+\ldots+\alpha_n(T)$，n表示所有状态的个数。

#### 动态规划问题：

- 第一个时刻：$\alpha_{i}(1)=P(y_{1},q_{1}=i|\lambda)$，表示第一时刻的隐藏状态为i，观测序列为y1，$\alpha_{i}(1)=\pi_i*b_{iy_1}$，$b_{iy_1}$表示由隐藏状态i生成观测$y_1$的概率。
- 现在到了第t时刻，状态为j，那t+1时刻状态为i表示为：$\alpha_{i}(t+1)=(\sum_{j=1}^n\alpha_{j}(t)a_{ji})b_{iy_{t+1}}$，$a_{ji}$表示隐藏状态j转换为隐藏状态i的概率
  - 其中t时状态为j的前向概率为：$\alpha_j(t)$转移到了状态i：$\alpha_j(t)a_{ji}$；
  - 但是这里要考虑t时刻所有的可能：$\sum_{j=1}^n\alpha_j(t)a_{ji}$；
  - 还要得到t+1的观测所以乘以$b_{iy_{t+1}}$。
- 最终结果：$P(Y|\lambda)=\sum\limits_{i}\alpha_{i}(T)$，时间复杂度为：$O(N^2T)$

#### 前向算法实例求解：

有3个盒子，每个盒子都有红色和白色两种球，分别为：
- 盒子1：5红5白
- 盒子2：4红6白
- 盒子3：7红3白

$$
\pi=\begin{pmatrix}0.2\\0.4\\0.4\end{pmatrix}\quad 
A=\begin{bmatrix}0.5&0.2&0.3\\0.3&0.5&0.2\\0.2&0.3&0.5\end{bmatrix}\quad 
B=\begin{bmatrix}0.5&0.5\\0.4&0.6\\0.7&0.3\end{bmatrix}
$$

$\pi$表示第一次分别选择1、2、3号盒子的概率，A表示盒子1、2、3分别转移到盒子1、2、3的概率，B表示盒子1、2、3生成红、白球的概率。

已知得到的观测序列为：O={红 白 红}

观测状态集合：V={红 白}，M=2

隐藏状态集合：Q={盒子1 盒子2 盒子3}，N=3

1. 时刻1：

- （红色球，盒子1）$\alpha_1(1)=\pi_1b_1(o_1)=0.2\times0.5=0.1$
- （红色球，盒子2）$\alpha_1(2)=\pi_2b_2(o_1)=0.4\times0.4=0.16$
- （红色球，盒子3）$\alpha_1(3)=\pi_3b_3(o_1)=0.4\times0.7=0.28$

2. 时刻2：

- （白色球，盒子1）$\alpha_2(1)=\Big[\sum_{i=1}^3\alpha_1(i)a_{i1}\Big]b_1(o_2)=[0.1*0.5+0.16*0.3+0.28*0.2]\times0.5=0.077$
- （白色球，盒子2）$\alpha_2(2)=\Big[\sum_{i=1}^3\alpha_1(i)a_{i2}\Big]b_2(o_2)=[0.1*0.2+0.16*0.5+0.28*0.3]\times0.6=0.1104$
- （白色球，盒子3）$\alpha_2(3)=\Big[\sum_{i=1}^3\alpha_1(i)a_{i3}\Big]b_3(o_2)=[0.1*0.3+0.16*0.2+0.28*0.5]\times0.3=0.0606$

3. 时刻3：

- （红色球，盒子1）$\alpha_3(1)=\Big[\sum_{i=1}^3\alpha_2(i)a_{i1}\Big]b_1(o_3)=[0.077*0.5+0.1104*0.3+0.0606*0.2]\times0.5=0.041$
- （红色球，盒子2）$\alpha_3(2)=\Big[\sum_{i=1}^3\alpha_2(i)a_{i2}\Big]b_2(o_3)=[0.077*0.2+0.1104*0.5+0.0606*0.3]\times0.5=0.035$
- （红色球，盒子3）$\alpha_3(3)=\Big[\sum_{i=1}^3\alpha_2(i)a_{i3}\Big]b_3(o_3)=[0.077*0.3+0.1104*0.2+0.0606*0.5]\times0.5=0.052$

4. 最后求得该观测序列出现的概率为：

$$
P(O|\lambda)=\sum_{i=1}^3\alpha_3(i)=0.130
$$

#### 求解参数

- 如果我们已知观测序列和其对应的状态序列：$\{(O_1,I_1),(O_2,I_2),\ldots,(O_S,I_S)\}$

- 计算状态转移概率：$A_{ij}$是状态i转移到状态j的频数

$$
\hat{a}_{ij}=\frac{A_{ij}}{\sum_{j=1}^NA_{ij}},i=1,2,\ldots,N,j=1,2,\ldots,N
$$

- 计算生成观测概率：$B_{jk}$是状态j生成观测k的频数

$$
\hat{b}_j(k)=\frac{B_{jk}}{\sum_{k=1}^MB_{jk}},j=1,2,\ldots,N;k=1,2,\ldots,M
$$

- 初始状态概率：查就得了

### Baum-Welch算法

问题：给定观测序列$O=\{o_{1},o_{2},...o_{T}\}$，求解参数$(\pi,A,B)$使得$P(O|\lambda)$最大。

这回我们就要估计模型参数，但由于状态序列未知，相当于是一个含有隐变量的参数估计问题，这就需要EM算法了。

Q函数是完全数据的对数似然函数关于给定模型参数和观测变量的前提下，对隐变量的条件概率分布的期望，接下来只需要对它进行极大化：

$$
Q(\lambda,\overline{\lambda})=\sum_IlogP(O,I|\lambda)P(I|O,\overline{\lambda})
$$

$\lambda$表示当前要求解的模型参数，$\overline{\lambda}$表示当前模型参数的估计值。

O,I表示完全数据。

#### 对Q函数进行化简：

其后部分：$P(I|O,\overline{\lambda})=\frac{P(O,I|\overline{\lambda})}{P(O|\overline{\lambda})}$，在这里对于估计参数来说分母就是常量了，所以可以用$P(O,I|\overline{\lambda})$来代替$P(I|O,\overline{\lambda})$。

得到新的Q函数：

$$
Q(\lambda,\overline{\lambda})=\sum_IlogP(O,I|\lambda)P(O,I|\overline{\lambda})
$$

其中，$P(O,I|\lambda)=\pi_{i_1}b_{i_1}(o_1)a_{i_1i_2}b_{i_2}(o_2)\ldots a_{i_{T-1}i_T}b_{i_T}(o_T)$

Q函数展开累乘的对数：

$$
Q(\lambda,\overline{\lambda})=\sum_{I}log\pi_{i_{1}}P(O,I|\overline{\lambda})
+\sum_{I}\left(\sum_{t=1}^{T-1}\log a_{i_{t}i_{t+1}}\right)P(O,I|\overline{\lambda})
+\sum_{I}\left(\sum_{t=1}^{T}\log b_{i_{t}}(o_{t})\right)P(O,I|\overline{\lambda})
$$

这三个项里的$\pi,a,b$就是我们想要求解的参数了，既然是加法，那只需要分别最大化就可以了！

以第一项举例：

$$
\sum_I\log\pi_{i_1}P(O,I|\hat{\lambda})=\sum_{i=1}^N\log\pi_iP(O,i_1=i|\hat{\lambda})\text{,}\sum_{i=1}^N\pi_i=1
$$

在条件下求极值，拉格朗日乘子法！

拉格朗日函数：

$$
\sum_{i=1}^N\log\pi_iP(O,i_1=i|\hat{\lambda})+\gamma(\sum_{i=1}^N\pi_i-1)
$$

令其偏导为0：

$$
\begin{aligned}
&\frac\partial{\partial\pi_i}[\sum_{i=1}^N\log\pi_iP(O,i_1=i|\hat{\lambda})+\gamma(\sum_{i=1}^N\pi_i-1)]=0 \\
&P(O,i_1=i|\hat{\lambda})+\gamma\pi_i=0
\end{aligned}
$$

对i进行求和：$\gamma=-P(O|\hat{\lambda})$

参数：

$$
\pi_i=\frac{P(O,i_1=i|\hat{\lambda})}{P(O|\hat{\lambda})} \\
a_{ij}=\frac{\sum_{t=1}^{T-1}P(O,i_t=i,i_{t+1}=j|\hat{\lambda})}{\sum_{t=1}^{T-1}P(O,i_t=i|\hat{\lambda})} \\
b_j(k)=\frac{\sum_{t=1}^TP(O,i_t=j|\hat{\lambda})I(o_t=v_k)}{\sum_{t=1}^TP(O,i_t=j|\hat{\lambda})}
$$

### 维特比算法

在给定模型$\lambda=(A,B,\pi)$和观测序列$O=\{o_{1},o_{2},...o_{T}\}$，求最有可能的状态序列$I^*=\{i_{1}^*,i_{2}^*,...i_{T}^*\}$，即使得$P(I^*|O)$最大。

对于t时刻，隐藏状态为i，要找到所有可能路径的最大值：

$$
\delta_t(i)=\max\limits_{i_1,i_2,\ldots i_{t-1}}P(i_t=i,i_1,i_2,\ldots i_{t-1},o_t,o_{t-1},\ldots o_1|\lambda),i=1,2,\ldots N
$$

递推公式：

$$
\begin{aligned}
\delta_{t+1}(i)& =\max_{i_1,i_2,\ldots i_t}P(i_{t+1}=i,i_1,i_2,\ldots i_t,o_{t+1},o_t,\ldots o_1|\lambda)  \\
&=\max_{1\leq j\leq N}\left[\delta_t(j)a_{ji}\right]b_i(o_{t+1})
\end{aligned}
$$

概率最大路径中t-1的状态：$\Psi_t\left(i\right)=\arg\max\limits_{1\leq j\leq N}\left[\delta_{t-1}(j)a_{ji}\right]$，即返回$\delta_{t-1}(j)a_{ji}$最大值的索引

观测序列：O={红 白 红}

$$
\pi=(0.2,0.4,0.4)^T\quad
A=\begin{pmatrix}0.5&0.2&0.3\\0.3&0.5&0.2\\0.2&0.3&0.5\end{pmatrix}\quad
B=\begin{pmatrix}0.5&0.5\\0.4&0.6\\0.7&0.3\end{pmatrix}
$$

t1时刻：

$$
\begin{gathered}
\delta_1(1)=\pi_1b_1(o_1)=0.2\times0.5=0.1 \\
\delta_1(2)=\pi_2b_2(o_1)=0.4\times0.4=0.16 \\
\delta_1(3)=\pi_3b_3(o_1)=0.4\times0.7=0.28 \\
\Psi_1(1)=\Psi_1(2)=\Psi_1(3)=0 
\end{gathered}
$$

t2时刻：

$$
\begin{gathered}
\delta_2(1)=\max\limits_{1\leq j\leq3}[\delta_1(j)a_{j1}]b_1(o_2)
=\max\limits_{1\leq j\leq3}[0.1\times0.5,0.16\times0.3,0.28\times0.2]\times0.5=0.028 \\
\Psi_2(1)=3 \\
\delta_2(2)=\max\limits_{1\leq j\leq3}[\delta_1(j)a_{j2}]b_2(o_2)
=\max\limits_{1\leq j\leq3}[0.1\times0.2,0.16\times0.5,0.28\times0.3]\times06=0.0504 \\
\Psi_2(2)=3 \\
\delta_2(3)=\max\limits_{1\leq j\leq3}[\delta_1(j)a_{j3}]b_3(o_2)
=\max\limits_{1\leq j\leq3}[0.1\times0.3,0.16\times0.2,0.28\times0.5]\times0.3=0.042 \\
\Psi_2(3)=3
\end{gathered}
$$

t3时刻：

$$
\begin{gathered}
\delta_3(1)=\max\limits_{1\leq j\leq3}[\delta_2(j)a_{j1}]b_1(o_3)
=\max\limits_{1\leq j\leq3}[0.028\times0.5,0.0504\times0.3,0.042\times0.2]\times0.5=0.00756 \\
\Psi_3(1)=2 \\
\delta_3(2)=\max\limits_{1\leq j\leq3}[\delta_2(j)a_{j2}]b_2(o_3)
=\max\limits_{1\leq j\leq3}[0.028\times0.2,0.0504\times0.5,0.042\times0.3]\times0.4=0.001008 \\
\Psi_3(2)=2 \\
\delta_3(3)=\max\limits_{1\leq j\leq3}[\delta_2(j)a_{j3}]b_3(o_3)
=\max\limits_{1\leq j\leq3}[0.028\times0.3,0.0504\times0.2,0.042\times0.5]\times0.7=0.0147 \\
\Psi_3(3)=3 \\
\end{gathered}
$$

隐藏序列结果：从后往前推得到最大值为$\delta_3(3),\delta_2(3),\delta_1(3)$，即隐藏盒子序列是(3,3,3)