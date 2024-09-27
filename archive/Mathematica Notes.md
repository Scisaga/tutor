## 读取文件
```java
ReadList[FileNameJoin[{NotebookDirectory[], "\\list.txt"}]][[1]];
```
其中文本文件中是一个List描述。

## 定义函数
```java
FuncName[A_]:=(对A的操作);
```

参考：
* http://reference.wolfram.com/language/tutorial/MakingDefinitionsForFunctions.html
* http://reference.wolfram.com/language/tutorial/DefiningFunctions.html
* http://reference.wolfram.com/language/guide/VariablesAndFunctions.html
* https://www.wolfram.com/language/elementary-introduction/2nd-ed/40-defining-your-own-functions.html

## 数组操作
二位矩阵，获取第一维，生成List
```java
A[[All, 1]]
```
遍历，生成列表
```java
Table[A[[i]],{i,2，Length[A]-1]
```

## 拟合
```
line = Fit[VA, {1, x}, x]
```
其中VA是一个二位数组，进行线性拟合

## 聚类
```java
CX = FindClusters[FX, 12];
```
其中FX是一个数组表示的二维矩阵

## 生成 GammaDistribution
```java
// 计算均值
mean = N[Mean[Select[VA, 110 > #[[1]] > 90 &][[All, 2]]]];
// 计算方差
var = N[Variance[Select[VA, 110 > #[[1]] > 90 &][[All, 2]]]]/4;
// 生成参数
\[Beta] = var/mean;
\[Alpha] = mean/\[Beta];
// 生成线性趋势
G = (1051.6654427872547` + 3.6202846419271673` #) &;
// 对方差进行修正
V = (#/100)^0.6*var &;
// 基于Gamma分布生成随机数的方法
F = (RandomReal[GammaDistribution[G[#]*G[#]/V[#], V[#]/G[#]]]) &;
// 展示
Show[{ListPlot[{#, F[#]} & /@ Table[Last[TX[[i]]][[2]] - First[TX[[i]]][[2]], {i, 1, Length[TX]}], AxesOrigin -> {0, 0}, PlotRange -> All], Plot[{line}, {x, 0, 200}]}]
```

## 可视化 
- 柱状图`Histogram`
- 将散点图和拟合曲线画在一起
   ```java
   Show[ListPlot[VA, AxesOrigin -> {0, 0}, PlotRange -> All], Plot[{line}, {x, 0, 200}]]
   ```