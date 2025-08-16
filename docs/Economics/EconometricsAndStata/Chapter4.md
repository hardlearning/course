# 第4章  一元线性回归

## 4.1 一元线性回归模型 

工资对数与教育年限的线性关系：
$$
\ln w=\alpha+\beta s+\epsilon
$$
lnw为工资对数，s为教育年限(schooling)，$\alpha$为截距项，表示当教育年限为0时的工资对数水平，$\beta$为斜率，表示教育年限对工资对数的边际效应，即每增加一年教育，将使工资增加百分之几，$\epsilon$为其他因素。

数据集grilic.dta包括758位美国年轻男子的教育投资回报率数据，查看此数据集的变量s与lnw的前10个观测值 ：
```
use grilic.dta,clear 
list s lnw in 1/10 
```
