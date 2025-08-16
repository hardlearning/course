# 第2章 Stata入门

## 2.3 Stata 操作实例

### 导入数据

使用命令use打开dta数据文件，需输入此文件的路径：
```
use E:\grilic_small.dta,clear
```
逗号“,”之后的“clear”为“选择项”(option)，表示可替代内 存中的已有数据。

如要关闭一个数据集，以便使用另外一个数据集，可输入命令
```
clear
```

### 审视数据

如想看数据集中的变量名称、标签等，可输入命令
```
describe
```

如想看变量s与lnw的具体数据，可使用命令
```
list s lnw
```

如想连续滚屏显示命令运行结果，可输入命令
```
set more off
```

如又想恢复分页显示运行结果，可输入命令
```
set more on
```

如只想对数据集的一部分子集执行命令，比如只看s与lnw的
前5个数据，可使用命令
```
list s lnw in 1/5
```

如要罗列从第11-15个观测值，可输入命令
```
list s lnw in 11/15
```

也可通过逻辑关系来定义数据集的子集。比如，要列出教育年限为16年及以上的数据，可使用命令
```
list s lnw if s>=16
```

如要删除满足“s>=16”条件的观测值，可输入命令
```
drop if s>=16
```
反之，如只想保留满足“s>=16”条件的观测值，可使用命令 
```
keep if s>=16
```

如想将数据按照变量 s 的升序排列，可输入命令
```
sort s
list
```

如想按降序排列，可使用命令gsort:
```
gsort -s
list
```

### 画图

想看变量 s 的分布情况，可输入以下命令画直方图：
```
histogram s, width(1) frequency 
```

- 选择项“width(1)”表示将组宽设为1(否则将使用Stata根据样本容量计算的默认分组数)，
- 选择项“frequency”表示将纵坐标定为频数(默认使用密度)。

画s与lnw之间的散点图，可输入命令：
```
scatter lnw s
```

如想在散点图上标注出每个点对应于哪个观测值，可先定义变量n，表示第n个观测值：
```
gen n=_n
scatter lnw s,mlabel(n)
```

- “_n”表示第n个观测值。然后以变量n作为每个点的标签来画散点图。
- 选择项“mlabel(n)”表示，以变量n作为标签(mark label)。

### 统计分析

如想看变量s的统计特征（样本容量、平均值、标准差、最小值与最
大值），可输入命令
```
summarize s
```

如不指明变量，则显示所有变量的统计指标。
```
sum
```

如要显示变量s的经验累积分布函数(empirical cumulative distribution function)，可使用命令
```
tabulate s
```
Freq”表示频数，“Percent”表示百分比，而“Cum.”表示累积百分比。

如要显示工资对数、教育年限、工龄之间的相关系数，可输入命令
```
pwcorr lnw s expr,sig star(.05)
```

- “pwcorr”表示“pairwise correlation”(两两相关)，“sig”表示显示相关系数的显著性水平(即p值，列在相关系数的下方)。
- “star(.05)”表示给所有显著性水平小于或等于5%的相关系数打上星号。

### 生成新变量

在Stata中定义新变量，可通过命令generate来实现。
```
// 定义教育年限的对数
generate lns=log(s)
// 定义s的平方项
gen s2=s^2
// 生成s与expr的互动项(interaction term)
gen exprs=s*expr
// 根据工资对数lnw计算工资水平w
gen w=exp(lnw)
```

在计量经济学中，常使用“虚拟变量”(dummy variable，也称“哑变量”)，即取值只能为0或1的变量，比如性别。
```
gen colleg=(s>=16)
```
括弧“()”表示对括弧中的表达式“s>=16”进行逻辑评估：如果此式为真，则取值为 1；如果为假，则取值为 0。

在上面命令中，不慎把college打成colleg了。可使用如下命令
将变量colleg重新命名为college：
```
rename colleg college
```

如想将“受过高等教育”的定义改为“s>=15”，但仍用college作为变量名。
- 方法之一，去掉现有变量college，再重新定义一次：
```
drop college
gen college=(s>=15)
```
- 方法之二，只需一个命令：
```
replace college=(s>=15)
```

对于较长的变量名，输入变量名较麻烦。有如下三个简便方法：
- 方法一，直接在变量窗口双击需要的变量，该变量名就会出现在命令窗口。
- 方法二，如有以下变量s1, s2, s3, s4, s5，可用s1-s5来表示这5个变量。
- 方法三，用“*”号来简化变量名的书写。假设想将内存中所有以“s”开头的变量都去掉，可输入命令
```
drop s*
```

### Stata的计算器功能

命令格式为：display expression

```
// 计算ln2
display log(2)
// 计算根号2
dis 2^0.5
```

### Stata的日志

在命令窗口输入如下命令，在当前路径就会生成一个名为“today.smcl”的日志文件：
```
log using today
```

如要暂时关闭日志(不再记录输出结果)，可输入命令
```
log off
```

如要恢复使用日志，可输入命令
```
log on
```

如要彻底退出日志，则可输入命令
```
log close
```

## 2.4 Stata命令库的更新

更新Stata命令库(Stata“ado”文件及其他可执行文件)
```
update all
```

从[SSC(Statistical Software Components)](http://ideas.repec.org/s/boc/bocode.html)下载Stata程序的命令为：
```
ssc install newcommand
```

如非官方命令不是来自SSC，一般需手工安装，将所有相关文件下载到指定的Stata文件夹中即可(通常为ado/plus/)。如不清楚应把文件复制到哪个文件夹，可输入以下命令，显示Stata的系统路径(system directories)：
```
sysdir
```

搜索Stata帮助文件、Stata 常见问题、Stata案例、Stata Journal、Stata Technical Bulletin等。
```
search keyword
```

命令findit的搜索范围比命令search更广，还包括Stata的网络资源。事实上，“findit”等价于“search,all”。
```
findit keyword
```

## 2.5 进一步学习Stata的资源

Stata英文参考书包括Baum(2006)，Cameron and Trivedi(2010)

加州大学洛杉矶分校(UCLA)网站(http://www.ats.ucla.edu/stat/stata/)有大量Stata的资源及实例(搜索“Stata UCLA”即可找到此网站)。

中文参考书包括陈传波《Stata 十八讲》