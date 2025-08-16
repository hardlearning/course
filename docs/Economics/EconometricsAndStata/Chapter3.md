# 第3章 数学回顾

## 3.1 微积分

### 一元最优化

计量中常见的两种估计方法为最小二乘法与最大似然估计。二者都是最优化问题(optimization)，前者为最小化问题 (minimization)，后者为最大化问题(maximization)。

一元最大化和一元最小化的**一阶条件**(first order condition)是在最优值$x^*$处的切线斜率为0，即
$$
f^\prime(x^*)=0
$$

二者的区别仅在于**二阶条件**(second order condition)，即最大化要求二阶导数$f^{\prime\prime}(x^*)\leq 0$，而最小化要求$f^{\prime\prime}(x^*)\geq 0$。

### 多元最优化

考虑无约束的多元最大化问题，
$$
\max\limits_x f(\mathbf{x})=f(x_1,x_2,\cdots,x_n)
$$
其中，$\mathbf{x}=(x_1,x_2,\cdots,x_n)$。
一阶条件要求在最优值$x^*$处，所有偏导数均为0:
$$
\frac{\partial f(\mathbf{x}^*)}{\partial x_1}=\frac{\partial f(\mathbf{x}^*)}{\partial x_2}=\cdots=\frac{\partial f(\mathbf{x}^*)}{\partial x_n}=0
$$

多元最小化的一阶条件与此相同。

## 3.2 线性代数

### 方阵

如果方阵A中的元素满足$a_{ij}=a_{ji}$(任意$i,j=1,\cdots,n$)，则称矩阵A为对称矩阵(symmetric matrix)。

如果方阵A的非主对角线元素全部为0，则称为对角矩阵 (diagonal matrix)

如果一个n级对角矩阵的主对角线元素都为1，则称为n级单位矩阵(identity matrix)，记为$I$或$I_n$。

### 向量

向量$\mathbf{a}$与$\mathbf{b}$的内积(inner product)或点乘(dot product)可定义为
$$
\mathbf{a}^\prime\mathbf{b}=\sum_{i=1}^n a_ib_i
$$

如果$\mathbf{a}^\prime\mathbf{b}=0$，则称向量$\mathbf{a}$与$\mathbf{b}$**正交**(orthogonal)，意味着两个向量在n维向量空间中相互垂直。

### 矩阵的乘法

假设矩阵$A=(a_{ij})_{m\times n}$，矩阵$B=(b_{ij})_{n\times n}$，则矩阵乘积AB的(i, j)元素即为矩阵A第i行与矩阵B的第j列的内积:
$$
(AB)_{ij}=
\left(\begin{matrix}
a_{i1}& a_{i2}& \cdots& a_{in}
\end{matrix}\right)
\left(\begin{matrix}
b_{1j}\\ b_{2j}\\ \vdots\\ b_{nj}
\end{matrix}\right)
=\sum_{k=1}^n a_{ik}b_{kj}
$$

在做矩阵乘法时，需区分左乘(premultiplication)与右乘 (postmultiplication)。A左乘B为AB，而A右乘B为BA。

矩阵的乘法满足以下规则:
(1)$IA=A$，$AI=A$ (乘以单位矩阵不改变矩阵)
(2)$(AB)C=A(BC)$ (乘法结合律)
(3)$A(B+C)=AB+AC$ (乘法分配律)
(4)$(AB)^\prime=B^\prime A^\prime$，$(ABC)^\prime=C^\prime B^\prime A^\prime$ (转置与乘积的混合运算)

### 逆矩阵

对于n级方阵A，如果存在n级方阵B ，使得$AB=BA=I_n$，则 称A为可逆矩阵(invertible matrix)或非退化矩阵(nonsingular matrix)，而B为A的逆矩阵(inverse matrix)，记为$A^{-1}$。

方阵A可逆的充分必要条件为其行列式$|A|\neq 0$。

矩阵求逆满足以下规则: 
(1)$(A^{-1})^\prime=(A^\prime)^{-1}$ (求逆与转置可交换次序)
(2)$(AB)^{-1}=B^{-1}A^{-1}$，$(ABC)^{-1}=C^{-1}B^{-1}A^{-1}$ (求逆与乘积的混合运算)

### 矩阵的秩

考虑两个n维列向量$a_1$与$a_2$，如果$a_1$正好是$a_2$的固定倍数，则在向量组$\{a_1,a_2\}$中，真正含有信息的只是其中的一个向量。

更一般地，考虑由K个n维向量构成的向量组$\{a_1,a_2,\cdots,a_K\}$，如果存在$c_1,c_2,\cdots, c_K$不全为零，使得
$$
c_1a_1+c_2a_2+\cdots+c_Ka_K=0 \quad (3.24)
$$
则称向量组$\{a_1,a_2,\cdots,a_K\}$**线性相关**(linearly dependent)。

如果$\{a_1,a_2,\cdots,a_K\}$线性相关，则其中至少有一个向量可写为其他向量的**线性组合**(linear combination),也称**线性表出**。

反之，如果方程(3.24)必然意味着$c_1=c_2=\cdots=c_K=0$，则称$\{a_1,a_2,\cdots,a_K\}$**线性无关**(linearly independent)。

如果$\{a_1,a_2,\cdots,a_K\}$线性相关，但从中去掉一个向量后，就变得线性无关，则$\{a_1,a_2,\cdots,a_K\}$中正好有(K-1)个向量真正含有信息，称(K-1)为此向量组的秩。

向量组$\{a_1,a_2,\cdots,a_K\}$的极大线性无关部分组所包含的向量个数，称为该向量组的**秩**(rank)。

对于$m\times n$级矩阵A，可将其n个列向量看成一个向量组，称此列向量组的秩为矩阵A的**列秩**(column rank)。

如果矩阵$A_{m\times n}$的列秩正好等于n，则称矩阵A**满列秩**(full column rank)。

将矩阵$A_{m\times n}$的m个行向量看成一个向量组，称此行向量组的秩为矩阵A的**行秩**(row rank)。

任何矩阵的行秩与列秩一定相等，称为**矩阵的秩**(matrix rank)。

### 二次型
   
可使用任意对称矩阵A，构成如下**二次型**(quadratic form):

$$
f(x_1,x_2,\cdots,x_n)=
\left(\begin{matrix}
x_1 & x_2 & \cdots & x_n
\end{matrix}\right)
\left(\begin{matrix}
a_{11} & a_{12} & \cdots & a_{1n} \\
a_{11} & a_{22} & \cdots & a_{2n} \\
\vdots & \vdots & \vdots & \vdots \\
a_{n1} & a_{n2} & \cdots & a_{nn}
\end{matrix}\right)
\left(\begin{matrix}
x_1 \\ x_2 \\ \vdots \\ x_n
\end{matrix}\right)
=\mathbf{x}^\prime\mathbf{A}\mathbf{x}
=\sum_{i=1}^n\sum_{j=1}^na_{ij}x_ix_j
$$
对称矩阵A称为此二次型的矩阵。

所谓二次型，就是$x_1,x_2,\cdots,x_n$的二次齐次多项式函数:
$$
\begin{aligned}
f(x_1,x_2,\cdots,x_n)=a_{11}x_1^2+2a_{12}x_1x_2+\cdots+2a_{1n}x_1x_n & \\
+a_{22}x_2^2+\cdots+2a_{2n}x_2x_n & \\
+\cdots\quad\cdots\quad\cdots & \\
+a_{nn}x_n^2 &
\end{aligned}
$$

在一般的n维情况下，给定对称矩阵A，针对二次型$\mathbf{x}^\prime\mathbf{A}\mathbf{x}$的取
值确定性，可引入以下定义：
(1) 对于任意非零列向量x，都有$\mathbf{x}^\prime\mathbf{A}\mathbf{x}>0$，则对称矩阵A为**正
定矩阵**(positive definite)。
(2) 对于任意非零列向量x，都有$\mathbf{x}^\prime\mathbf{A}\mathbf{x}\geq0$，则对称矩阵A为**半正定矩阵**(positive semidefinite)。
(3) 对于任意非零列向量x，都有$\mathbf{x}^\prime\mathbf{A}\mathbf{x}<0$，则对称矩阵A为**负定矩阵**(negative definite)。
(4) 对于任意非零列向量x，都有$\mathbf{x}^\prime\mathbf{A}\mathbf{x}\leq0$，则对称矩阵A为**半负定矩阵**(negative semidefinite)。

如果对称矩阵A为正定矩阵，则该矩阵可通过线性变换转换为一个主对角线元素全为正数的对角矩阵；而这些主对角线元素正好是矩阵A的特征值，故正定矩阵A一定可逆，即逆矩阵$A^{-1}$存在。

线性变换后的正定二次型可写为
$$
f(x_1,x_2,\cdots,x_n)=a_{11}x_1^2+a_{22}x_2^2+\cdots+a_{nn}x_n^2
$$
其中，$a_{11},a_{22},\cdots,a_{nn}$全部为正数，故当$x_1,x_2,\cdots,x_n$不全为0时，$f(x_1,x_2,\cdots,x_n)$必然大于0。

如果$a_{11},a_{22},\cdots,a_{nn}$全部为正数或0，则 A为半正定矩阵。
如果$a_{11},a_{22},\cdots,a_{nn}$全部为负数，则A 为负定矩阵。
如果$a_{11},a_{22},\cdots,a_{nn}$全部为负数或0，则 A为半负定矩阵。
反之，如果$a_{11},a_{22},\cdots,a_{nn}$有正有负，则A为不定的(indefinite)，其二次型$\mathbf{x}^\prime\mathbf{A}\mathbf{x}$的取值可正可负。

在计量中，常使用形如$x^\prime[Var(x)]^{-1}x$的二次型。其中，x为n维随机向量，而$[Var(x)]^{-1}$为其协方差矩阵Var(x)的逆矩阵。
二次型$x^\prime[Var(x)]^{-1}x$的直观含义是，以$[Var(x)]^{-1}$为权重，将x到零向量0(原点)的距离标准化(避免受到x度量单位的影响)。

## 3.3 概率与条件概率

### 条件概率

在事件B发生的前提条件下，事件A发生的条件概率(conditional probability)为：
$$
P(A|B)=\frac{P(AB)}{P(B)}
$$

### 独立事件

如果条件概率等于无条件概率，$P(A|B)=P(A)$，即B是否发生不影响A的发生，则称A, B为相互独立的随机事件。

### 全概率公式

如果事件组$\{B_1,B_2,\cdots,B_n\}(n\geq2)$两两互不相容，但必有一件事发生，且每件事的发生概率均为正数，则对任何事件A(无论A与$\{B_1,B_2,\cdots,B_n\}$是否有任何关系)，都有
$$
P(A)=\sum_{i=1}^nP(B_i)P(A|B_i)
$$

## 3.4  分布与条件分布

### 离散型概率分布

假设随机变量X的可能取值为$\{x_1,x_2,\cdots,x_k,\cdots\}$，其对应概率为$\{p_1,p_2,\cdots,p_k,\cdots\}$，即$p_k=P(X=x_k)$，则称X为离散型随机变量。其中$p_k\geq0,\sum_kp_k=1$。

### 连续型概率分布

连续型随机变量X可以取任意实数，其概率密度函数(probability density function，简记pdf)$f(x)$满足，
1. $f(x)\geq0,\forall x$；
2. $\int_{-\infty}^{+\infty}f(x)dx=1$；
3. X落入区间[a, b]的概率为$P(a\leq X\leq b)=\int_{a}^{b}f(x)dx$。

定义累积分布函数(cumulative distribution function，简记 cdf)：
$$
F(x)=P(-\infty\leq X\leq x)=\int_{-\infty}^{x}f(t)dt
$$
其中，t 为积分变量。 F(x)度量的是，从$-\infty$至x为止，概率密度函数f(t)曲线下的面积。

### 多维随机向量的概率分布

为研究变量间关系，常同时考虑两个或多个随机变量，即随机向量(random vector)。二维连续型随机向量(X,Y)的联合密度函数 (joint pdf) f(x, y)满足：
1. $f(x,y)\geq0,\forall x,y$；
2. $\int_{-\infty}^{+\infty}\int_{-\infty}^{+\infty}f(x,y)dxdy=1$；
3. (X,Y)落入平面某区域D 的概率为为$P[(X,Y)\in D]=\int\int\limits_{D}f(x,y)dxdy$。

二维随机向量的联合密度函数就像倒扣的草帽。落入平面某区域D的概率就是此草帽下在区域D之上的体积。

从二维联合密度f(x,y)，可计算X的(一维)边缘密度函数(marginal pdf)：
$$
f_x(x)=\int_{-\infty}^{+\infty}f(x,y)dy
$$
即给定X=x，把所有Y取值的可能性都“加总”起来。

类似地，可以计算Y的(一维)边缘密度函数： 
$$
f_x(x)=\int_{-\infty}^{+\infty}f(x,y)dy
$$
即给定Y=y，把所有X取值的可能性都“加总”起来。

定义二维随机向量(X,Y)的累积分布函数为： 
$$
F(x,y)=P(-\infty\leq X\leq x,-\infty\leq Y\leq y)=\int_{-\infty}^x\int_{-\infty}^y f(t,s)dtds
$$

### 条件分布

计算在$X\in[x-\epsilon,x+\epsilon]$条件下Y的累积分布函数，即$P\{Y\leq y|X\in[x-\epsilon,x+\epsilon]\}$，然后让$\epsilon\to 0^+$，则可证明条件密度函数为：
$$
f(y|x)=\frac{f(x,y)}{f_x(x)}
$$

## 3.5 随机变量的数字特征
 
**定义**：对于分布律为$p_k\equiv P(X=x_k)$的离散型随机变量X，其期望(expectation)为
$$
E(X)\equiv\mu\equiv\sum_{k=1}^\infty x_kp_k
$$
期望的直观含义就是对$x_k$进行加权平均，而权重为概率$p_k$。

**定义**；对于概率密度函数为$f(x)$的连续型随机变量X，其期望为
$$
E(X)\equiv\mu\equiv\sum_{-\infty}^{+\infty}xf(x)dx
$$
上式也是对x进行加权平均，权重为概率密度$f(x)$。

有时称求期望这种运算为**期望算子**(expectation operator)。期望算子满足线性性(linearity)，即对于任意常数k都有$E(X+Y)=E(X)+E(Y),E(kX)=kE(X)$。

**定义**：随机变量X的方差(variance)为
$$
Var(X)\equiv\sigma^2\equiv E[X-E(X)]^2
$$
称方差的平方根为标准差(standard deviation)，记为$\sigma$。

在计算方差时，常利用以下简便公式：
$$
Var(X)=E(X^2)-[E(X)]^2
$$

**定义**：随机变量X与Y的协方差(covariance)为
$$
Cov(X,Y)\equiv\sigma_{XY}\equiv E[(X-E(X))(Y-E(Y))]
$$

如果$Cov(X,Y)=0$，则说明二者**线性不相关**(uncorrelated)，但不一定**相互独立**(independent)，因为还可能存在非线性的相关关系。

在计算协方差时，常使用以下简便公式：
$$
Cov(X,Y)=E(XY)-E(X)E(Y)
$$

协方差的运算也满足线性性，可以证明：
$$
Cov(X,Y+Z)=Cov(X,Y)+Cov(X,Z)
$$

**定义**：随机变量X与Y的相关系数(correlation)为
$$
\rho\equiv Corr(X,Y)\equiv\frac{Cov(X,Y)}{\sqrt{Var(X)Var(Y)}}=\frac{\sigma_{XY}}{\sigma_X\sigma_Y}
$$
相关系数一定介于-1与1之间，即$-1\leq\rho\leq1$。

**定义**：一阶原点矩为$E(X)$(即期望)，二阶原点矩为$E(X^2)$，三阶原点矩为$E(X^3)$，四阶原点矩为$E(X^4)$，等等。

**定义**：二阶中心矩为$E[X-E(X)]^2$(即方差)，三阶中心矩为$E[X-E(X)]^3$，四阶中心矩为$E[X-E(X)]^4$，等等。

一阶原点矩(期望)表示随机变量的平均值。
二阶中心矩(方差)表示随机变量的波动程度。
三阶中心矩表示随机变量密度函数的不对称性(偏度)。
四阶中心矩表示随机变量密度函数的最高处(山峰)有多“尖”及尾部有多“厚”(峰度)。

**定义**：随机变量X的偏度(skewness)为$E[(X-\mu)/\sigma]^3$。

如果随机变量为对称分布，则其偏度为0；因为根据微积分，奇函数在关于原点对称的区间上积分为0。

**定义**：随机变量X的峰度(kurtosis)为$E[(X-\mu)/\sigma]^4$。

对于正态分布，其峰度为3。如果随机变量X的峰度大于3，则其密度函数的最高处(山峰)比正态分布更“尖”，而两侧尾部则更“厚”，称为“厚尾”(fat tails)。存在厚尾的概率分布更容易在尾部取值，称为极端值(outlier)。

**定义**：随机变量X的超额峰度(excess kurtosis)为$E[(X-\mu)/\sigma]^4-3$。

更一般地，对于随机变量X与任意函数$g(\cdot)$，称随机变量函数$g(X)$的期望$E[g(X)]=\int_{-\infty}^{+\infty}g(x)f(x)dx$为**矩**(moment)。

**定义**：条件期望(conditional expectation)就是条件分布$Y|x$的期望，即
$$
E(Y|X=x)\equiv E(Y|x)=\int_{-\infty}^{+\infty}yf(y|x)dy
$$
由于y已被积分积掉，故$E(Y|x)$只是x的函数。

**定义**：条件方差(conditional variance)就是条件分布$Y|x$的方差
$$
Var(Y|X=x)\equiv Var(Y|x)=\int_{-\infty}^{+\infty}[y-E(Y|x)]^2f(y|x)dy
$$
由于y已被积分积掉，故$Var(Y|x)$也只是x的函数。

**定义**：设$\mathbf{X}=(X_1,X_2,\cdots,X_n)$为n维随机向量，则其协方差矩阵(covariance matrix)为$n\times n$的对称矩阵：
$$
\begin{aligned}
Var(\mathbf{X})&\equiv E[(\mathbf{X}-E(\mathbf{X}))(\mathbf{X}-E(\mathbf{X}))^\prime] \\
&=E\left[\begin{pmatrix}
X_1-E(X_1) \\ \vdots \\ X_n-E(X_n)
\end{pmatrix}
\begin{pmatrix}
X_1-E(X_1) & \cdots & X_n-E(X_n)
\end{pmatrix}\right] \\
&=E\begin{pmatrix}
[X_1-E(X_1)]^2 & \cdots & [X_1-E(X_1)][X_n-E(X_n)] \\
\vdots & \vdots & \vdots \\
[X_1-E(X_1)][X_n-E(X_n)] & \cdots & [X_n-E(X_n)]^2
\end{pmatrix} \\
&=\begin{pmatrix}
\sigma_{11} & \cdots & \sigma_{1n} \\
\vdots & \vdots & \vdots \\
\sigma_{n1} & \cdots & \sigma_{nn}
\end{pmatrix}
\end{aligned}
$$
主对角线元素$\sigma_{ii}\equiv Var(X_i)$，非主对角线元素$\sigma_{ij}\equiv Cov(X_i,X_j)$。

协方差矩阵必然为半正定矩阵(positive semidefinite)。在一维 情况下，这意味着随机变量的方差必然为非负。

对于随机向量X的期望与协方差矩阵的运算，有如下法则。假设A为$m\times n$常数矩阵(不含随机变量)，可以证明：
1. $E(AX)=A\cdot E(X)$ (期望算子的线性性)
2. $Var(X)=E(XX^\prime)-E(X)[E(X)]^\prime$ (一维公式的推广)
3. $Var(AX)=A\cdot Var(X)A^\prime$ (夹心估计量)

如果A为对称矩阵，则$Var(AX)=AVar(X)A$，称为**夹心估计量**(sandwich estimator)。

以上随机变量的数字特征都可视为“总体矩”(population
moments)。在抽取随机样本后，可用样本数据计算相应的“样本矩”(sample moments)，作为相应总体矩的估计值。

以数据集grilic.dta为例，打开此数据集，查看所包含的变量：
```
use grilic.dta,clear
describe
```

看各变量的基本统计指标。
```
sum
```

如想看lnw的更多统计指标，可加选择项 detail
```
sum lnw,detail
```

通过画lnw的直方图来看其(无条件)分布
```
hist lnw,width(0.1)
```

如想得到概率密度函数的连续估计，可输入命令：
```
kdensity lnw,normal normop(lpattern(dash)) 
```
- “kdensity”表示核密度估计(kernel density estimation)。
- 选择项“normal”表示画正态分布的密度函数作为对比。
- 选择项“normop(lpattern(dash))”则指示将正态密度用虚线(dash)来画(其中，normop表示normal options;而 lpattern表示line pattern)。

作为对比，考察工资水平本身的分布：
```
gen wage=exp(lnw)
kdensity wage
```

给定教育年限为16年(大学毕业)，工资对数的条件密度：
```
kdensity lnw if s==16
```

将lnw的无条件密度与条件密度画在一起：
```
twoway kdensity lnw || kdensity lnw if s==16,lpattern(dash)
```

分隔符“||”的作用，也可以通过两个括号“( ) ( )”来等价
地实现，比如：
```
twoway (kdensity lnw) (kdensity lnw if s==16,lpattern(dash))
```

比较lnw的无条件期望、方差与条件期望、条件方差：
```
sum lnw
sum lnw if s==16
```

比较在s=12(中学毕业)与s=16(大学毕业)情况下，lnw的条件 密度：
```
twoway (kdensity lnw if s==12) (kdensity lnw if s==16,lpattern(dash))
```

## 3.6 迭代期望定律

**定理**：对于条件期望的运算，有以下重要的迭代期望定律(Law of iterated expectation)
$$
E(Y)=E_X[E(Y|x)]
$$
无条件期望$E(Y)$等于，给定$X=x$情况下Y的条件期望$E(Y|x)$(仍为x的函数)，再对X求期望。

如果X为离散随机变量，则根据期望定义，上式可写为:
$$
E(Y)=\sum_iP(X=x_i)E(Y|x_i)
$$
无条件期望等于条件期望之加权平均，而权重为条件X=x的概率。

## 3.7 随机变量无关的三个层次概念

**定义**：对于连续型随机变量X与Y，如果其联合密度等于边 缘密度的乘积，即$f(x,y)=f_x(x)f_y(y)$，则称X与Y相互独立(independent)。

“相互独立”是有关随机变量“无关”的最强概念。线性不相关的概念则更弱，仅要求协方差为0，即$Cov(X,Y)=0$。在二者之间还有一个中间层次的无关概念，即“均值独立”(mean-independence)。

**定义**：假设条件期望$E(Y|x)$存在。如果$E(Y|x)$不依赖于X，则称Y均值独立于X(Y is mean-independent of X)。

**命题**：Y均值独立于X，当且仅当$E(Y|x)=E(Y)$(条件期望等于无条件期望)。

**命题**：如果X与Y相互独立，则Y均值独立于X，且X均值独立于Y。

**定理**：如果Y均值独立于X，或X均值独立于Y，则$Cov(X,Y)=0$。

“相互独立”$\to$“均值独立”$\to$“线性不相关”；反之不然。

## 3.8 常用连续型统计分布

1. 正态分布

如果随机变量X的概率密度函数为：
$$
f(x)=\frac{1}{\sqrt{2\pi\sigma^2}}\exp\{\frac{-(x-\mu)^2}{2\sigma^2}\}
$$
则称X服从正态分布(normal distribution)，记为$X\sim N(\mu,\sigma^2)$，其中$\mu$为期望，而$\sigma^2$为方差。

将X进行标准化，定义$Z\equiv\frac{X-\mu}{\sigma}$，则Z服从标准正态分布(standard normal distribution)，记为$Z\sim N(0,1)$，其概率密度函数为
$$
\phi(x)=\frac{1}{\sqrt{2\pi}}\exp\{-x^2/2\}
$$
标准正态分布的概率密度以原点为对称，呈钟形，通常记为$\phi(x)$；其累积分布函数记为$\Phi(x)$。

在Stata中，使用函数normalden(x)与normal(x)分别表示标准正态的密度函数$\phi(x)$与累积分布函数$\Phi(x)$。

比如，计算标准正态变量小于1.96的概率：
```
dis normal(1.96)
```

如要画标准正态的密度函数，可输入命令：
```
twoway function y=normalden(x),range(-5 5) xline(0) ytitle(概率密度)
```

- 选择项“range(-5 5)”表示在横轴区间(-5, 5)上画此图；默认为“range(0 1)”，即在(0, 1)区间画图。
- “xline(0)”表示在横轴x=0处画一条直线。
- “ytitle(概率密度)”表示将纵轴标签设为“概率密度”。

正态分布$N(m,s^2)$的密度函数可用normalden(x,m,s)来表示，其中m与s分别为期望与标准差。

将N(0,1)与N(1,4)的密度函数画在一起：
```
twoway function y=normalden(x),range(-5 10) || function z=normalden(x,1,2),range(-5 10) lpattern(dash) ytitle(概率密度)
```

- 选择项“lpattern(dash)”表示使用虚线画图。

**多维正态分布**：如果n维随机向量$\mathbf X=(X_1,X_2\cdots X_n)^\prime$的联合密度函数为
$$
f(x_1,\cdots,x_n)=\frac{1}{(2\pi^{n/2}|\mathbf\Sigma|^{1/2})}\exp\{-\frac{1}{2}(\mathbf X-\mathbf\mu)^\prime\mathbf\Sigma(\mathbf X-\mathbf\mu)\}
$$
则称$\mathbf X$服从期望为$\mathbf\mu$、协方差矩阵为$\mathbf\Sigma$的n维正态分布，记为$\mathbf X\sim N(\mathbf\mu,\mathbf\Sigma)$。其中，$(\mathbf X-\mathbf\mu)^\prime\mathbf\Sigma(\mathbf X-\mathbf\mu)$为$(\mathbf X-\mathbf\mu)$的二次型，其二次型矩阵为协方差矩阵的逆矩阵$\mathbf\Sigma^{-1}$；$|\mathbf\Sigma|$为协方差矩阵$\mathbf\Sigma$的行列式。

多维正态分布具有良好的性质。比如，多维正态的每个分量 都是正态，其分量之任意线性组合仍然是正态。反之，每个分量 均为一维正态并不足以保证其联合分布也是多维正态的。

如果$(X_1,X_2,\cdots,X_n)$服从n维正态分布，则“$X_1, X_2,\cdots,X_n$相互独立”与“$X_1,X_2,\cdots,X_n$两两不相关”是等价的。利用此性质，有时容易证明正态变量的独立性。

2. $\chi^2$分布(卡方分布，Chi-square)

如果$Z\sim N(0,1)$，则$Z^2\sim\chi^2(1)$，即自由度为1的$\chi^2$分布。

如果$\{Z_1,\cdots,Z_k\}$为独立同分布的标准正态，则其平方和服从自由度为k的卡方分布，记为
$$
\sum_{i=1}^k Z_i^2\sim\chi^2(k)
$$
参数k为自由度(degree of freedom)，因为$\sum_{i=1}^k Z_i^2$个相互独立(自由)的随机变量所构成。

$\chi^2$分布来自标准正态的平方和，故取值为正。可以证明，$\chi^2(k)$分布的期望为k，而方差为2k。

在Stata中，使用函数chi2den(k,x)与chi2(k,x)分别
表示自由度为k的卡方分布的概率密度与累积分布函数。比如，输入命令将$\chi^2(3)$与$\chi^2(5)$的密度函数画在一起：
```
twoway function chi3=chi2den(3,x),range(0 20) || function chi5=chi2den(5,x),range(0 20)
lpattern(dash) ytitle(概率密度)
```

3. t分布

假设$Z\sim N(0,1),Y\sim\chi^2(k)$，且Z与Y相互独立，则$\frac{Z}{\sqrt{Y/k}}$服从自由度为k的t分布，记为
$$
\frac{Z}{\sqrt{Y/k}}\sim t(k)
$$
k为自由度。

t分布也以原点为对称，但与标准正态分布相比，中间的“山峰”
更低(但更尖)，而两侧有“厚尾”(fat tails)。

在Stata中，使用函数tden(k,t)与t(k,t)分别表示自由度为 k的t分布的概率密度与累积分布函数。比如，使用命令将t(1)与t(5)的密度函数画在一起：
```
twoway function t1=tden(1,x),range(-5 5) ||
function t5=tden(5,x),range(-5 5) lpattern(dash) ytitle(概率密度)
```

Stata以函数ttail(k,t)表示自由度为k的t分布的右侧尾部概率，即P(T>t)，正好是反向的累积分布函数。

4. F分布

假设$Y_1\sim\chi^2(k_1),Y_2\sim\chi^2(k_2)$，且$Y_1,Y_2$相互独立，则$\frac{Y_1/k_1}{Y_2/k_2}$服从 自由度为$k1,k2$的F分布，记为
$$
\frac{Y_1/k_1}{Y_2/k_2}\sim F(k_1,k_2)
$$
其中，$k_1,k_2$为自由度。

F分布的取值也只能为正数，其概率密度形状与$\chi^2$分布相似。

Stata使用函数Fden(k1,k2,x)与F(k1,k2,x)分别表示自由度为(k1,k2)的F分布的概率密度与累积分布函数。比如，输入命令将F(10,20)与F(10,5)的密度函数画在一起；
```
twoway function F20=Fden(10,20,x),range(0 5) || function F5=Fden(10,5,x),range(0 5)
lpattern(dash) ytitle(概率密度)
```

F分布与t分布存在着密切关系，因为t分布的平方就是F分布。

**命题**：如果$X\sim t(k)$，则$X^2\sim F(1,k)$。

## 3.9 统计推断的思想 

我们感兴趣的全体研究对象被称为总体(population)，其中的每个研究对象称为个体(individual)。 

由于总体包含的个体可能很多，普查成本较高，故常从总体抽取部分个体，称为样本(sample)。 样本所包含的个体数目称为样本容量(sample size)。 

通常希望样本为随机样本(random sample)，即总体中的每位个体都有相同的概率被抽中，且被抽中的概率相互独立，称为独立同分布(independently identically distributed，简记 iid)。 

统计推断就是根据样本数据，对总体性质进行推断的科学。 统计推断的主要形式有参数估计(点估计、区间估计)、假设检验及预测等，其中点估计为统计推断的基础。

假设随机变量X的概率密度为 $f(x;\theta)$，其中$\theta$为待估参数。为估计总体参数$\theta$，从总体中抽取了样本容量为n的样本数据$\{x_1,x_2,\cdots,x_n\}$。我们希望根据此样本数据来设计一个性质良好的统计量$\hat\theta(x_1,x_2,\cdots,x_n)$，以此估计$\theta$。由于使用$\hat\theta$来估计$\theta$，故称$\hat\theta$为$\theta$的估计量(estimator)。 相应地，给定$\{x_1,x_2,\cdots,x_n\}$后，可得到估计量$\hat\theta(x_1,x_2,\cdots,x_n)$的具体取值，称为估计值(estimate)。

**定义 **：以估计量$\hat\theta$来估计参数$\theta$，则其偏差为 $Bias(\hat\theta)\equiv E(\hat\theta)-\theta$。

**定义**：如果偏差$Bias(\hat\theta)=0$，则称$\hat\theta$为无偏估计量(unbiased estimator)；反之，则称为有偏估计(biased estimator)。

**定义 **：以估计量$\hat\theta$来估计参数$\theta$，则其均方误差(Mean Squared Error，简记MSE)为$MSE(\hat\theta)\equiv E[(\hat\theta-\theta)^2]$。

**命题**：均方误差可以分解为方差与偏差平方之和，即 
$$
MSE(\hat\theta)=Var(\hat\theta)+[Bias(\hat\theta)]^2
$$
