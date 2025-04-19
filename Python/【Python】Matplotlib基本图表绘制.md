@[TOC](目录)
# Matplotlib基本图表绘制

## 折线图

```python
from matplotlib import pyplot as plt  
import matplotlib  

#字典类型的字体预设，键值对依次为字体名、粗细、大小
font = {'family': 'MicroSoft YaHei',  
        'weight': 'bold',  
        'size': 10}

#设置线条宽度
matplotlib.rc('lines', lw=2)
#设置字体样式
matplotlib.rc('font', **font)
#通过figure实例设置窗口大小figsize和清晰度dpi
fig = plt.figure(figsize=(20, 8), dpi=80)  

#准备数据
x = range(2, 26, 2)  
y_1 = [15, 13, 14.5, 17, 20, 25, 26, 26, 24, 22, 18, 15]  
y_2 = [0, 3, 7, 4, -1, 5, 8, 2, 3, 2, 5, 0]

#在同一折线图上分别绘制两组数据，label参数为图例，外观参数见下文
plt.plot(x, y_1, label="苏州", color="pink", linestyle="-.")  
plt.plot(x, y_2, label="哈尔滨", color='c', linestyle=':')

#准备x轴刻度
_xtick_labels = ["10月{}日".format(i) for i in range(2, 26, 2)]  
#设置x轴刻度，参数一为x轴上打刻度标签的值，参数二为x轴上的刻度标签，二者均为列表，一一对应，rotation关键字参数设置标签旋转角度
plt.xticks(x, _xtick_labels, rotation=45)
#设置y轴刻度，只给出y轴上打刻度标签的值，标签即数值
plt.yticks(range(min(y_2), max(y_1)+1))  

#绘制图表网格，alpha参数为透明度
plt.grid(alpha=1, linestyle="--")  
#绘制图例在左上角，loc参数为绘制位置
plt.legend(loc="upper left")  
#绘制轴标签
plt.xlabel("日期")  
plt.ylabel("温度/℃")  
#绘制图题
plt.title("10月气温变化情况")

#将图表保存为相对路径下的文件
plt.savefig("./sig_size.png")  
#显示图表
plt.show()
```

![](https://i-blog.csdnimg.cn/direct/da44aa0cda8a4fd480236de3b807d630.png#pic_center)

### 更多外观

#### 外观类型

| 关键字参数                | 含义     |
| -------------------- | ------ |
| lw(lineweight)       | 线条宽度   |
| ls(linestyle)        | 线条样式   |
| c(color)             | 线条颜色   |
| fc(facecolor)        | 图表填充颜色 |
| ec(edgecolor)        | 图表边缘颜色 |
| mew(markeredgewidth) | 标记边缘宽度 |
| aa(antialiased)      | 抗锯齿    |

#### 颜色

| 类别   | 举例           |
| ---- | ------------ |
| 字符   | r（红色）        |
| 字符串  | black（黑色）    |
| 16进制 | #000000 （白色） |

#### 线条样式

| 字符  | 类型  |
| --- | --- |
| -   | 实线  |
| --  | 破折线 |
| -.  | 点划线 |
| :   | 点虚线 |
| ''  | 无线条 |

#### 图例位置

图例中loc关键字参数的值可以是字符串、数字或坐标元组。

| 字符串          | 数字  |
| ------------ | --- |
| best         | 0   |
| upper right  | 1   |
| upper left   | 2   |
| lower left   | 3   |
| lower right  | 4   |
| right        | 5   |
| center left  | 6   |
| center right | 7   |
| lower center | 8   |
| upper center | 9   |
| center       | 10  |

#### 使用本地字体

```python
from matplotlib import font_manager

#fname关键字参数从字体文件路径导入字体，可选size关键字参数
my_font = font_manager.FontProperties(fname="/System/Library/Fonts/PingFang.ttc")

#在添加文本时使用fontporperties关键字参数设置对应字体样式，图例中使用prop关键字参数
plt.xticks(x, _xtick_labels, rotation=45, fontproperties=my_font)
```

## 散点图

```python
#用scatter方法绘制散点图，除了不需要linestyle参数其余代码与折线图相通
plt.scatter(x, y_1, label="苏州", color="pink")  
plt.scatter(x, y_2, label="哈尔滨", color='c')
```

![](https://i-blog.csdnimg.cn/direct/92295fde084642a582774a19923526d2.png#pic_center)


## 条形图

```python
from matplotlib import pyplot as plt  
font = {'family': 'MicroSoft YaHei',  
        'weight': 'bold',  
        'size': 15}  
matplotlib.rc("font", **font)  
fig = plt.figure(figsize=(20, 8), dpi=80)  

#准备x轴数据名称
a = ["黑神话：悟空", "最终幻想7：重生", "艾尔登法环：黄金树幽影", "小丑牌", "暗喻幻想", "宇宙机器人"]  
#准备数据名称对应的数据
b_1 = [81, 92, 94, 90, 94, 94]  
b_2 = [83, 90, 81, 88, 88, 92] 

#设置条宽 
bar_width = 0.3
#在间隔单位刻度的基础上，将两组数据的横坐标分别向两侧偏移半个条宽，否则数据条会重叠  
x = list(range(len(a)))  
x_1 = [i-bar_width*0.5 for i in x]  
x_2 = [i+bar_width*0.5 for i in x] 

#绘制条形图，依次给出条形横坐标、条高、条宽、条色、图例名
plt.bar(x_1, b_1, width=0.3, color="orange", label="媒体")  
plt.bar(x_2, b_2, width=0.3, color="red", label="用户")  
#绘制x轴刻度
plt.xticks(range(len(a)), a)
#绘制网格并设置透明度  
plt.grid(alpha=0.3)  
#绘制图例在左上角
plt.legend(loc="upper left")  
#绘制图题
plt.title("TGA年度游戏提名M站评分")  
#显示图表
plt.show()
```

![](https://i-blog.csdnimg.cn/direct/3ab82bb2f18a45869a5a7df8ac3cc1c5.png#pic_center)


### 横向条形图

在竖向条形图代码的基础上作如下修改：

```python
#绘制横向条形图，宽度width参数改为高度height参数
plt.barh(x_1, b_1, height=0.3, color="orange", label="媒体")  
plt.barh(x_2, b_2, height=0.3, color="red", label="用户")
#将数据名改到y轴刻度上
plt.yticks(range(len(a)), a)  
#将图例调整到右下角的空闲区域
plt.legend(loc="lower right")
```

![](https://i-blog.csdnimg.cn/direct/10e7548d119948739e8a45f71fb8fabf.png#pic_center)


## 直方图

```python
from matplotlib import pyplot as plt  
import matplotlib  
font = {'family': 'MicroSoft YaHei',  
        'weight': 'bold',  
        'size': 15}  
matplotlib.rc("font", **font)  
fig = plt.figure(figsize=(15, 8), dpi=80)  
#准备数据
a = [15, 20, 15, 20, 25, 25, 30, 15, 30, 25,  
     15, 30, 25, 35, 30, 35, 30, 25, 20, 30,  
     20, 25, 35, 30, 25, 20, 30, 25, 35, 25,  
     15, 25, 35, 25, 25, 30, 35, 25, 35, 20,  
     30, 30, 15, 30, 40, 30, 40, 15, 25, 40,  
     20, 25, 20, 15, 20, 25, 25, 40, 25, 25,  
     40, 35, 25, 30, 20, 35, 20, 15, 35, 25,  
     25, 30, 25, 30, 25, 30, 43, 25, 43, 22,  
     20, 23, 20, 25, 15, 25, 20, 25, 30, 43,  
     35, 45, 30, 45, 30, 45, 45, 35]  
#设置组距
d = 5  
#计算组数，需检验能否整除
num_bins = (max(a)-min(a))//d  
#绘制直方图，第一个参数为数据，第二个参数为组数
plt.hist(a, num_bins)  
#绘制x轴刻度标签
plt.xticks(range(min(a), max(a)+d, d))  
#绘制网格
plt.grid(alpha=0.3)  
#显示图像
plt.show()
```

![](https://i-blog.csdnimg.cn/direct/8602acd588d14c6ca8972e7d655e41d0.png#pic_center)


### 频率分布直方图

```python
#绘制频率分布直方图添加关键字参数density并设为True
plt.hist(a, num_bins, density=True)
```

![](https://i-blog.csdnimg.cn/direct/0733cadaa6084432bf3739b54057579a.png#pic_center)


