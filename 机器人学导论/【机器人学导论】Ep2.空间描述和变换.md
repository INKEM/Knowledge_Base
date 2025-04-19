> 主要参考学习资料：
>
> 《机器人学导论（第4版）》John J.Craig著
>
> 台大机器人学之运动学——林沛群（本文插图来自该课程课件）
>
> 本章前置知识：矢量和矩阵的四则运算-单位矩阵-转置矩阵-逆矩阵-正交矩阵
>
> 码字不易，求点赞收藏(´•ω•̥`)
>
> 有问题欢迎评论区讨论~

@[TOC](目录)
# 空间描述和变换
___
## 描述

我们用$\{A\}$来表示一个**坐标系(Frame)**

讨论问题之前，我们需要建立一个**世界坐标系(World Frame)**，之后定义的位置和姿态都是参照世界坐标系定义的笛卡尔坐标系

一个平面中的物体拥有沿$x$轴、$y$轴两个方向移动的自由度和绕垂直于平面的轴转动的**自由度（Degree Of Freedom，可简记为DOF）**，即两个移动自由度和一个转动自由度，共三个自由度

同理，一个空间中的物体拥有沿$x$轴、$y$轴、$z$轴三个方向移动的自由度和绕这三个轴转动的自由度，即三个移动自由度和三个转动自由度，共六个自由度
___
### 位置描述

位置描述涵盖刚体在三个方向上的移动自由度

一旦建立了坐标系，我们可以用一个$3\times 1$的**位置矢量**给世界坐标系中的任何点定位

矢量用一个左上标来表明其定义在哪个坐标系，例如$^AP$的元素数值是沿坐标系$\{A\}$三个主轴方向上的投影，矢量的各个元素用下标$x$、$y$和$z$来标明：

$^AP=^A\begin{bmatrix}p_x\\p_y\\p_z\end{bmatrix}$

（其中矩阵左上标表示矩阵中坐标都是相对于该坐标系而言，由于格式输入限制位置偏下）

![位置矢量应用举例](https://i-blog.csdnimg.cn/direct/46418f46be1b451c886139e246dc83a4.png)

___
### 姿态描述

姿态描述涵盖刚体三个方向上的转动自由度

我们可以在刚体上固定一个坐标系$\{B\}$，用$\{B\}$相对于$\{A\}$的描述来表示物体的姿态，而$\{B\}$可以用该坐标系三个主轴方向的单位矢量来确定

我们用$\hat{X}_B$、$\hat{Y}_B$和$\hat{Z}_B$来表示$\{B\}$主轴方向的单位矢量，当$\{A\}$作为参考坐标系时，它们被写为$^A\hat{X}_B$、$^A\hat{Y}_B$和$^A\hat{Z}_B$

将这三个单位矢量按照$^A\hat{X}_B$、$^A\hat{Y}_B$和$^A\hat{Z}_B$的顺序排列组成一个$3\times3$的矩阵，称为**旋转矩阵(Rotation Matrix)**。表达$\{B\}$相对于$\{A\}$的旋转矩阵记为$_B^AR$

$_B^AR$=$\begin{pmatrix}^A\hat{X}_B&^A\hat{Y}_B&^A\hat{Z}_B\end{pmatrix}$

又$^A\hat{X}_B$、$^A\hat{Y}_B$和$^A\hat{Z}_B$是以$\{A\}$为参考，所以矢量中各个元素的数值为$\hat{X}_B$、$\hat{Y}_B$和$\hat{Z}_B$分别在$\{A\}$三个主轴方向上的投影长度，因而上式可以进一步写为

$_B^AR=\begin{pmatrix}^A\hat{X}_B&^A\hat{Y}_B&^A\hat{Z}_B\end{pmatrix}=\begin{bmatrix}\hat{X}_B\cdot\hat{X}_A&\hat{Y}_B\cdot\hat{X}_A&\hat{Z}_B\cdot\hat{X}_A\\\hat{X}_B\cdot\hat{Y}_A&\hat{Y}_B\cdot\hat{Y}_A&\hat{Z}_B\cdot\hat{Y}_A\\\hat{X}_B\cdot\hat{Z}_A&\hat{Y}_B\cdot\hat{Z}_A&\hat{Z}_B\cdot\hat{Z}_A\end{bmatrix}$

又矩阵中各个矢量均为单位矢量，点积之值也是$\hat{X}_B$、$\hat{Y}_B$和$\hat{Z}_B$对于$\{A\}$的方向余弦，该矩阵又称为**方向余弦矩阵(Direction Cosine Matrix)**

综上，我们用旋转矩阵描述了刚体的姿态
___
#### 旋转矩阵

接下来重点介绍旋转矩阵的性质与作用

##### 性质

将旋转矩阵中各个点积前后矢量互换，我们有如下推导（其中右上标T为转置符号）：

$_B^AR=\begin{pmatrix}^A\hat{X}_B&^A\hat{Y}_B&^A\hat{Z}_B\end{pmatrix}=\begin{bmatrix}\hat{X}_B\cdot\hat{X}_A&\hat{Y}_B\cdot\hat{X}_A&\hat{Z}_B\cdot\hat{X}_A\\\hat{X}_B\cdot\hat{Y}_A&\hat{Y}_B\cdot\hat{Y}_A&\hat{Z}_B\cdot\hat{Y}_A\\\hat{X}_B\cdot\hat{Z}_A&\hat{Y}_B\cdot\hat{Z}_A&\hat{Z}_B\cdot\hat{Z}_A\end{bmatrix}$
$=\begin{bmatrix}\hat{X}_A\cdot\hat{X}_B&\hat{X}_A\cdot\hat{Y}_B&\hat{X}_A\cdot\hat{Z}_B\\\hat{Y}_A\cdot\hat{X}_B&\hat{Y}_A\cdot\hat{Y}_B&\hat{Y}_A\cdot\hat{Z}_B\\\hat{Z}_A\cdot\hat{X}_B&\hat{Z}_A\cdot\hat{Y}_B&\hat{Z}_A\cdot\hat{Z}_B\end{bmatrix}=\begin{bmatrix}\hat{X}_A{^T}\\\hat{Y}_A{^T}\\\hat{Z}_A{^T}\end{bmatrix}$$=\begin{pmatrix}^A\hat{X}_B&^A\hat{Y}_B&^A\hat{Z}_B\end{pmatrix}^T=^B_AR^T$

由此$^A_BR=^B_AR^T$

又由：

$^A_BR{^T}^A_BR=\begin{bmatrix}\hat{X}_A{^T}\\\hat{Y}_A{^T}\\\hat{Z}_A{^T}\end{bmatrix}\begin{pmatrix}^A\hat{X}_B&^A\hat{Y}_B&^A\hat{Z}_B\end{pmatrix}=I_3$

其中$I_3$是$3\times3$的单位矩阵，因此：

$^A_BR=^B_AR^{-1}=^B_AR{^T}$

即$^B_AR$的转置矩阵等于它的逆矩阵，又同时等于$^A_BR$。由线性代数可知，前者是正交矩阵的性质

旋转矩阵有九个数字，但只有三个矢量两两垂直、均为单位矢量总共六个条件约束，因此只有三个数字是独立变量，与空间中转动具有三个自由度相符
___
##### 作用

除了前文所讲的描述一个刚体的姿态外，旋转矩阵还有两个作用

**1.转换矢量的坐标**

矢量$P$相对于$\{B\}$的坐标表示：$^BP=^Bp_x\hat{X}_B+^Bp_y\hat{Y}_B+^Bp_z\hat{Z}_B$

矢量$P$相对于$\{A\}$的坐标表示：$^AP=^Ap_x\hat{X}_A+^Ap_y\hat{Y}_A+^Ap_z\hat{Z}_A$

$^BP$和$^AP$只是坐标表示不同，本质上是同一个矢量，由此矢量$P$相对于$\{A\}$的坐标也可以用$^BP$在$\{A\}$三个主轴上的投影计算：

$^Ap_x=^BP\hat{X}_A=^Bp_x\hat{X}_B\hat{X}_A+^Bp_y\hat{Y}_B\hat{X}_A+^Bp_z\hat{Z}_B\hat{X}_A$

$^Ap_y=^BP\hat{Y}_A=^Bp_x\hat{X}_B\hat{Y}_A+^Bp_y\hat{Y}_B\hat{Y}_A+^Bp_z\hat{Z}_B\hat{Y}_A$

$^Ap_z=^BP\hat{X}_A=^Bp_x\hat{X}_B\hat{Z}_A+^Bp_y\hat{Y}_B\hat{Z}_A+^Bp_z\hat{Z}_B\hat{Z}_A$

用矩阵表示则为：

$^AP=^A\begin{bmatrix}p_x\\p_y\\p_z\end{bmatrix}=\begin{bmatrix}\hat{X}_B\cdot\hat{X}_A&\hat{Y}_B\cdot\hat{X}_A&\hat{Z}_B\cdot\hat{X}_A\\\hat{X}_B\cdot\hat{Y}_A&\hat{Y}_B\cdot\hat{Y}_A&\hat{Z}_B\cdot\hat{Y}_A\\\hat{X}_B\cdot\hat{Z}_A&\hat{Y}_B\cdot\hat{Z}_A&\hat{Z}_B\cdot\hat{Z}_A\end{bmatrix}^B\begin{bmatrix}p_x\\p_y\\p_z\end{bmatrix}=^A_BR^BP$

由此得到$\{B\}$相对于$\{A\}$的旋转矩阵可以将$P$的表示由相对于$\{B\}$转换为相对于$\{A\}$

**2.描述刚体转动的状态**

分别研究坐标系绕三个主轴旋转的状态

以$\{A\}$绕$\hat{Z}_A$逆时针旋转$\theta$（以后不标明旋转方向默认为逆时针旋转）得到$\{B\}$为例，计算此时$\{B\}$相对于$\{A\}$的旋转矩阵得：

$^A_BR=\begin{bmatrix}\cos\theta&-\sin\theta&0\\\sin\theta&\cos\theta&0\\0&0&1\end{bmatrix}=\begin{bmatrix}c\theta&-s\theta&0\\s\theta&c\theta&0\\0&0&1\end{bmatrix}$

（由于我们将在计算中用到许多三角函数，因此三角函数用对应首字母简记）

由此，我们将绕$\hat{Z}_A$旋转$\theta$以这种方式计算出来的旋转矩阵记为$R_{\hat{Z}_A}(\theta)$，则：

$R_{\hat{Z}_A}(\theta)=\begin{bmatrix}c\theta&-s\theta&0\\s\theta&c\theta&0\\0&0&1\end{bmatrix}$

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/c96e03ffd7fa46b4b024894164ddd1c4.png)


同理得到：

$R_{\hat{X}_A}(\theta)=\begin{bmatrix}1&0&0\\0&c\theta&-s\theta\\0&s\theta&c\theta\end{bmatrix}$

$R_{\hat{Y}_A}(\theta)=\begin{bmatrix}c\theta&0&s\theta\\0&1&0\\-s\theta&0&c\theta\end{bmatrix}$

通过接下来的例子我们来理解这类旋转矩阵如何描述刚体转动的状态

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/2bff05995efc446487cd24f002e29ed2.png)


$^AP=^A(p_x\;p_y\;p_z)^T$对$\hat{X}_A$旋转$\theta$得到$^AP'$，$^AP'=?$

不妨让$\{A\}$跟着$^AP$同步旋转，得到$\{A'\}$，那么$^AP'$相对于$\{A'\}$的坐标跟$^AP$相对于$\{A\}$的坐标相等，而后者已知

又通过题目条件和前文定义我们知道$^A_{A'}R=R_{\hat{X}_A}(\theta)=\begin{bmatrix}1&0&0\\0&c\theta&-s\theta\\0&s\theta&c\theta\end{bmatrix}$

利用旋转矩阵转换矢量坐标的作用，我们可以把$^AP'$相对于$\{A'\}$的坐标转换为相对于原坐标系$\{A\}$的坐标，即：

$^AP'=^A_{A'}R^AP=R_{\hat{X}_A}(\theta)^AP$

所以旋转矩阵$R(\theta)$可以计算矢量在同一坐标系下旋转后的坐标
___
### 位姿描述

位置和姿态的组合称为**位姿**

我们同样用$\{A\}$来表示一个位姿

以$\{A\}$为参考描述位姿$\{B\}$，结合前文的位置描述和姿态描述，我们用$^A_BR$表示$\{B\}$的姿态，用$^AP_{BORG}$表示$\{B\}$的原点的位置矢量（下标ORG意为原点origin），于是位姿描述记为：

$\{B\}=\{^A_BR,{\,}^AP_{BORG}\}$
___
## 复杂姿态描述

上一节我们用旋转矩阵来描述姿态，但仅限于绕其中一个主轴的单自由度旋转，且我们得知姿态描述中包含三个独立变量，接下来我们讨论如何用三个参数描述空间中的任意姿态

任何三自由度的旋转都可以拆解成三个单自由度的旋转，由于旋转不满足交换律，所以多次旋转的先后顺序需要明确定义。而旋转转轴也需要明确定义，围绕方向固定不动的参考坐标系主轴旋转的拆解方式称为**固定角（Fixed Angles）**，围绕转动坐标系当下的主轴旋转的拆解方式称为**欧拉角（Euler Angles）**

___

### X-Y-Z固定角

![](https://i-blog.csdnimg.cn/direct/0ca1db0c03ad4fef9840ed5b793649a1.png)

给定参考坐标系$\{A\}$和我们要描述姿态的坐标系$\{B\}$，从固定角的视角来看，任意姿态的$\{B\}$都可以由以下过程表示：将$\{B\}$与$\{A\}$重合，先绕$\hat{X}_A$旋转$\gamma$角，再绕$\hat{Y}_A$旋转$\beta$角，最后绕$\hat{Z}_A$旋转$\alpha$角得到最终的$\{B\}$，因而可以推导$\{B\}$到$\{A\}$的等价旋转矩阵：$^A_BR_{XYZ}(\gamma,\beta,\alpha)=R_Z(\alpha)R_Y(\beta)R_X(\gamma)=\begin{bmatrix}c\alpha&-s\alpha&0\\s\alpha&c\alpha&0\\0&0&1\end{bmatrix}\begin{bmatrix}c\beta&0&s\beta\\0&1&0\\-s\beta&0&c\beta\end{bmatrix}\begin{bmatrix}1&0&0\\0&c\gamma&-s\gamma\\0&s\gamma&c\gamma\end{bmatrix}$

（矩阵书写顺序与实际旋转操作顺序相反，从旋转一个矢量的角度来看，是因为旋转矩阵乘在矢量的左边，所以右侧的矩阵会先与矢量相乘）

(等价旋转矩阵下标表示绕主轴旋转的操作顺序，可以根据实际情况调换)

最终得到：

$^A_BR_{XYZ}(\gamma,\beta,\alpha)=\begin{bmatrix}c\alpha c\beta&c\alpha c\beta s\gamma-s\alpha c\gamma&c\alpha s\beta c\gamma+s\alpha s\gamma\\s\alpha c\beta&s\alpha s\beta s\gamma+c\alpha c\gamma&s\alpha s\beta c\gamma-c\alpha s\gamma\\-s\beta&c\beta s\gamma&c\beta c\gamma\end{bmatrix}$①

反过来，如果已知旋转矩阵各个元素的数值，逆求三个固定角，令

$^A_BR_{XYZ}(\gamma,\beta,\alpha)=\begin{bmatrix}r_{11}&r_{12}&r_{13}\\r_{21}&r_{22}&r_{23}\\r_{31}&r_{32}&r_{33}\end{bmatrix}$

则

$\left\{\begin{matrix}\beta=\mathrm{Atan2}(-r_{31},\sqrt{r_{11}^2+r_{21}^2})\\ \alpha=\mathrm{Atan2}(\frac{r_{21}}{c\beta},\frac{r_{11}}{c\beta})\\ \gamma=\mathrm{Atan2}(\frac{r_{31}}{c\beta},\frac{r_{33}}{c\beta})\end{matrix}\right.$

$(\beta\neq 90^{\circ})$

（式中$\mathrm{Atan2}(y,x)=arctan\frac{y}{x}$，即反正切函数的双参量表示法，可以根据$x$和$y$的符号判别角的象限，而单输入值的反正切函数无法区分）

上式中$\cos\beta$只考虑正根以得到单解来建立各种姿态表示法之间一一对应的映射函数，但在某些情况下有必要求出所有的解，会在第四章说明

如果$\beta=90^{\circ}$，则只能求出$\alpha$和$\gamma$的和或差，在这种情况下一般取$\alpha=0$，$\gamma=\left\{\begin{matrix}\mathrm{Atan2}(r_{12},r_{22})(\beta=90^{\circ})\\-\mathrm{Atan2}(r_{12},r_{22})(\beta=-90^{\circ})\end{matrix}\right.$

结合式①即可理解求解公式的推导

___

### Z-Y-X欧拉角

![](https://i-blog.csdnimg.cn/direct/5767d00747f540689ea594b52d9cabd3.png)

给定参考坐标系$\{A\}$和我们要描述姿态的坐标系$\{B\}$，从欧拉角的视角来看，任意姿态的$\{B\}$都可以由以下过程表示：将$\{B\}$与$\{A\}$重合，先绕$\hat{Z}_B$转$\alpha$角得到$\{B'\}$，再绕$\hat{Y}_{B'}$转$\beta$角得到$\{B''\}$，最后绕$\hat{X}_{B''}$转$\gamma$角得到最终的$\{B\}$（欧拉角选取转轴的顺序与固定角相反，原因会在下面解释）

借由$\{B\}$的中间状态$\{B'\}$和$\{B''\}$和欧拉角的定义表达$^A_BR$，我们可以写出：

$^A_BR=^A_{B'}R^{B'}_{B''}R^{B''}_{B}R$

即：

$^A_BR_{X'Y'Z'}(\gamma,\beta,\alpha)=R_{Z'}(\alpha)R_{Y'}(\beta)R_{X'}(\gamma)=\begin{bmatrix}c\alpha&-s\alpha&0\\s\alpha&c\alpha&0\\0&0&1\end{bmatrix}\begin{bmatrix}c\beta&0&s\beta\\0&1&0\\-s\beta&0&c\beta\end{bmatrix}\begin{bmatrix}1&0&0\\0&c\gamma&-s\gamma\\0&s\gamma&c\gamma\end{bmatrix}$

相乘后得：

$^A_BR_{X'Y'Z'}(\gamma,\beta,\alpha)=\begin{bmatrix}c\alpha c\beta&c\alpha c\beta s\gamma-s\alpha c\gamma&c\alpha s\beta c\gamma+s\alpha s\gamma\\s\alpha c\beta&s\alpha s\beta s\gamma+c\alpha c\gamma&s\alpha s\beta c\gamma-c\alpha s\gamma\\-s\beta&c\beta s\gamma&c\beta c\gamma\end{bmatrix}$

我们发现，这个结果与以相反顺序绕固定轴旋转三次得到的结果完全相同，因而逆求解公式也相同

可以这样更深地理解固定角与欧拉角之间的关系：两种姿态描述方法不同的解释在于运用了旋转矩阵两种不同的作用，一是在同一个坐标系下旋转矢量，二是将矢量的坐标在不同坐标系之间变换。固定角是用前者描述$\{A\}$的三个主轴矢量旋转到$\{B\}$的过程，而欧拉角是用后者将在$\{B\}$下的主轴矢量坐标一步步借由中间坐标系变换到$\{A\}$

___

### 其他转角组合

在实际情况我们会遇到更多按不同顺序进行绕主轴的三个旋转，所有排列总共24种，固定角12种，欧拉角12种，都被称为**转交排列设定法**（需要注意第一次转动和最后一次转动绕同一个主轴也是一种排列，只要保证相邻两次切换围绕的主轴即可）

除此之外，姿态描述还有**轴角表示法**和**四元数表示法**，完全解释篇幅较长，若有余力我会写拓展专题

___

## 映射（Mapping）

**映射**是将矢量从一个坐标系投影到另一个坐标系的变换

___

### 坐标平移

![](https://i-blog.csdnimg.cn/direct/ad8f2eb67a4b4878b7f5777b639393d2.png)


我们用矢量$^AP_{BORG}$表示$\{B\}$的原点相对于$\{A\}$的位置

当$\{A\}$与$\{B\}$姿态相同，位置不同时，由矢量加法的几何意义：

$^AP=^BP+^AP_{BORG}$

___

### 坐标旋转

当$\{A\}$与$\{B\}$位置相同，姿态不同时，由上一篇所介绍的旋转矩阵转换矢量坐标的作用：

$^AP=^A_BR^BP$

___

### 一般变换

![](https://i-blog.csdnimg.cn/direct/4d58fdd926444253aba24ef0cdf9b396.png)


当$\{A\}$与$\{B\}$位置和姿态都不同时，可以先将$\{B\}$旋转到和$\{A\}$相同的姿态得到$\{B'\}$，以$\{B'\}$为中间坐标系对$^BP$进行旋转变换，再作$\{B'\}$到$\{A\}$的平移变换：

$^AP=^A_BR^BP+^AP_{BORG}$②

为了简化算式，我们将平移和旋转整合起来，定义**齐次变换矩阵**：

$^A_BT=\begin{bmatrix}^A_BR&^AP_{BORG}\\\mathbf{0}^T&1\end{bmatrix}$

其中$\mathbf{0}^T$表示三维零向量的转置

这样式②可写为：

$\begin{bmatrix}^AP\\1\end{bmatrix}=\begin{bmatrix}^A_BR&^AP_{BORG}\\\mathbf{0}^T&1\end{bmatrix}\begin{bmatrix}^BP\\1\end{bmatrix}$

这样我们就以一个简单的矩阵形式表示了一般变换的旋转和平移

___

### 复合变换

复合变换借用中间坐标系求解两坐标系的相对关系

例如已知$\{C\}$相对于$\{B\}$，并且已知$\{B\}$相对于$\{A\}$，已知$^CP$求$^AP$，可先将$^CP$变换成$^BP$再变换成$^AP$：

$^BP=^B_CT^CP$

$^AP=^A_BT^BP=^A_BT^B_CT^CP$

由此定义$^A_CP=^A_BT^B_CT$

___

### 逆变换

已知$^A_BT$求$^B_AT$可以直接对$^A_BT$求逆矩阵，但还可以利用变换的性质

关于$^B_AT$的旋转变换，由旋转矩阵的性质可知$^B_AR=^A_BR^T$

关于$^B_AT$的平移变换，将$^AP_{BORG}$变换成$\{B\}$中的描述：$^B(^AP_{BORG})=^B_AT^AP_{BORG}=^B_AR^AP_{BORG}+^BP_{AORG}$

上式左侧为零，可得：$^BP_{AORG}=-^B_AR^AP_{BORG}=-^A_BR^T{}^AP_{BORG}$

由此$^B_AT=\begin{bmatrix}^B_AR^T&-^A_BR^T{}^AP_{BORG}\\\mathbf{0}^T&1\end{bmatrix}=^A_BT^{-1}$

___

### 变换方程

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/1b6ffa67d63b4eb8b02683c55cf5924f.png)

图中$\{D\}$可以用两种不同的方式表达成变换相乘的方式：

$^U_DT=^U_AT^A_DT$

$^U_DT=^U_BT^B_CT^C_DT$

两个表达式构造成一个变换方程：

$^U_AT^A_DT=^U_BT^B_CT^C_DT$

如有$n$个未知变换和$n$个变换方程，变换可由变换方程解出。设上式中所有变换除了$^B_CT$外均已知，可解出：

$^B_CT=^U_BT^{-1}{}^U_AT^A_DT^C_DT^{-1}$

___

## 算子（Operator）

**算子**是对矢量进行平移和旋转操作的变换

$^A_BT$除了变换以外，也可以充当算子对矢量或点进行平移或转动

___

### 平移算子

![](https://i-blog.csdnimg.cn/direct/c0dec060b884489481a7fbbf65d1082a.png)


将$^AP_1$指向的点沿$^AQ$进行平移得到一个新的矢量$^AP_2$，计算如下：

$^AP_2=^AP_1+^AQ$

用矩阵算子写出平移变换，有：

$^AP_2=D(Q)^AP_1$

算子$D_Q$可以被看成是一个特殊形式的齐次变换：

$D(Q)=\begin{bmatrix}I&^AQ\\\mathbf{0}^T&1\end{bmatrix}$

___

### 旋转算子

由上一篇所介绍的旋转矩阵描述刚体转动状态的作用，可以得到旋转算子的定义：将矢量$^AP_1$用旋转$R$变换成一个新的矢量$^AP_2$

当一个旋转矩阵作为算子时无需写出上下标，因为不涉及两个坐标系，但我们将用另一个符号说明是绕哪个轴旋转的，写成：

$^AP_2=R_K(\theta)^AP_1$

符号$R_K(\theta)$是一个旋转算子，表示绕$\hat{K}$轴旋转$\theta$角度，例如绕$\hat{Z}$轴旋转$\theta$的算子：

$R_z(\theta)=\begin{bmatrix}c\theta&-s\theta&0&0\\s\theta&c\theta&0&0\\0&0&1&0\\0&0&0&1\end{bmatrix}$

___

### 变换算子

与一般变换一样，将平移和旋转整合起来我们得到通用的变换算子。因为只涉及一个坐标系，所以没有上下标。算子$T$将矢量$^AP_1$平移并旋转得到一个新矢量：$^AP_2=T^AP_1$

___

## 齐次变换矩阵

源于旋转矩阵在上一篇所讲的三个作用，在这一篇我们也探索出了齐次变换矩阵的三个作用：

1.描述位姿：$^A_BT$表示$\{B\}$相对于$\{A\}$，其中$^A_BR$的各列是定义$\{B\}$主轴方向的单位矢量，$^AP_{BORG}$确定了$\{B\}$的原点

2.变换映射：$^A_BT$是映射$^BP\rightarrow^AP$

3.变换算子：$^A_BT$将$^AP_1$变换为$^AP_2$

___

本章我们只考虑了位置矢量的变换，而速度矢量和力矢量由于类型不同，它们的变换形式也不同，将在第五章讨论

___

**本章完**
