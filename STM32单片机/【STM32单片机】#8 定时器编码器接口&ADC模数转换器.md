> 主要参考学习资料：
> 
> B站@江协科技
> 
> STM32入门教程-2023版 细致讲解 中文字幕
>
>开发资料下载链接：https://pan.baidu.com/s/1h_UjuQKDX9IpP-U1Effbsw?pwd=dspb
>
> 单片机套装：STM32F103C8T6开发板单片机C6T6核心板 实验板最小系统板套件科协

实验：
- 编码器接口测速
- ADC单通道
- ADC多通道

新函数：
- 定时器库函数（编码器接口部分）
- ADC库函数（部分）

@[TOC](目录)

# 编码器接口

## 编码器接口简介

- Encoder Interface 编码器接口
- 编码器接口可接受增量（正交）编码器的信号，根据编码器旋转产生的正交信号脉冲，自动控制CNT自增或自减，从而指示编码器的位置、旋转方向和旋转速度。
- 每个高级定时器和通用定时器都拥有一个编码器接口。
- 两个输入引脚借用了输入捕获的通道1和通道2。
## 正交编码器

正交编码器一般可以测量位置或带有方向的速度值，有A相和B相两个信号输出引脚，在正转和反转情况下输出电平波形如下：

![](https://i-blog.csdnimg.cn/direct/9ee9c4da31d4439da2f796eb160ca29a.png)


编码器旋转的速度决定方波的频率，方向决定A相和B相方波的相位差。当编码器正转时，A相提前B相90°；当编码器反转时，A相滞后B相90°，因此该对信号被称为正交信号。旋转的方向可以自行定义极性。正交信号的优点是精度更高，A相和B相都可以计次，且可以以两路信号交替跳变为依据判别噪声。

## 编码器接口框图

![](https://i-blog.csdnimg.cn/direct/65c8caf6b09f43f4b060858e9172c06b.png)


编码器接口的两个输入端TI1FP1和TI2FP2借用了输入捕获单元的两个通道，分别接A相和B相，并和输入捕获单元共用输入滤波器和边沿检测器。编码器接口的输出部分通过从模式控制器控制CNT的计数时钟和计数方向，如果出现了边沿信号，且对应另一相的状态为正转，则控制CNT自增，反之CNT自减。此时CNT不会使用内部时钟和时基单元配置的计数方向。

## 编码器接口基本结构

![](https://i-blog.csdnimg.cn/direct/cf513891817e479a9b9a1f309c8912ab.png)


在编码器接口中，ARR仍然有效，一般设置为65535，以便利用补码的定义得到负数。当CNT从0自减时，下一个数为65535，将其从16位无符号数转换为16位有符号数即可。

## 工作模式

![](https://i-blog.csdnimg.cn/direct/b285b3b0302a42ef9f1d8c540168fa4d.png)


有效边沿决定了CNT在哪一相的上升沿和下降沿时根据另一相的电平状态执行自增或自减，仅在某一相计数时则忽略另一相的边沿。右侧对应关系与正交编码器介绍中展示的表格等效，正转向上计数，反转向下计数。一般使用双相计数，因为其计数精度更高。

## 实例

![](https://i-blog.csdnimg.cn/direct/d4ddd02c8f22434982d29b4aa5f983e7.png)


图中信号产生毛刺时，CNT只会自增自减来回摆动，实现抗噪声效果。

![](https://i-blog.csdnimg.cn/direct/68f93d76bc664515a6dcf9ff61f8aa0a.png)


由于高低电平对编码器接口都有效，因此输入通道的极性选择不影响其有效性，选上升沿则不取反，选下降沿则取反。通过改变其中一相的极性可以交换正反转的方向。

## 函数详解

### TIM_EncoderInterfaceConfig函数

**简介**：定时器编码器接口配置。

**参数一**：定时器名称

**参数二**：工作模式

	TIM_EncoderMode_TI1, TIM_EncoderMode_TI2, TIM_EncoderMode_TI12

**参数三**：通道1电平极性

	TIM_ICPolarity_Falling, TIM_ICPolarity_Rising

**参数四**：通道2电平极性

## 实验15 编码器接口测速

### 接线图

![](https://i-blog.csdnimg.cn/direct/c4fd596079bc471880e4b000bfda821c.jpeg)


### 编码器接口驱动

由于博主可能存在硬件问题，即使下了官方代码也未能复现。以下程序全程跟随视频手敲，但效果没有得到验证。

**Encoder.h**

```c、
#ifndef __ENCODER_H
#define __ENCODER_H

void Encoder_Init(void);
int16_t Encoder_Get(void);

#endif
```

**Encoder.c**

```c
#include "stm32f10x.h"

void Encoder_Init(void)
{
	//初始化代码沿用实验13
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	GPIO_InitTypeDef GPIO_InitStructure;
	//补充说明：上拉/下拉输入的选择原则是外部模块的默认状态输出，二者保持一致
	//默认高电平比较普遍，因此一般选上拉输入
	//若不确定默认状态或外部信号输出功率非常小则尽量选择浮空输入
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	//配置PA6和PA7
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6 | GPIO_Pin_7;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_TIM3, ENABLE);
	//编码器接口托管时钟，删除TIM_InternalClockConfig(TIM3);
	
	TIM_TimeBaseInitTypeDef TIM_TimeBaseInitStructure;
	TIM_TimeBaseInitStructure.TIM_ClockDivision = TIM_CKD_DIV1;
	//编码器接口托管计数器模式，配置无用
	TIM_TimeBaseInitStructure.TIM_CounterMode = TIM_CounterMode_Up;
	//ARR满量程计数
	TIM_TimeBaseInitStructure.TIM_Period = 65536 - 1;
	//编码器时钟驱动计数器，不分频
	TIM_TimeBaseInitStructure.TIM_Prescaler = 1 - 1;
	TIM_TimeBaseInitStructure.TIM_RepetitionCounter = 0;
	TIM_TimeBaseInit(TIM3, &TIM_TimeBaseInitStructure);
	
	TIM_ICInitTypeDef TIM_ICInitStructure;
	//初始化结构体保证配置完整
	TIM_ICStructInit(&TIM_ICInitStructure);
	TIM_ICInitStructure.TIM_Channel = TIM_Channel_1;
	TIM_ICInitStructure.TIM_ICFilter = 0x0F;
	/*输入捕获单元只需配置滤波器即可，极性使用定时器接口函数配置，避免重复配置
	TIM_ICInitStructure.TIM_ICPolarity = TIM_ICPolarity_Rising;
	TIM_ICInitStructure.TIM_ICPrescaler = TIM_ICPSC_DIV1;
	TIM_ICInitStructure.TIM_ICSelection = TIM_ICSelection_DirectTI;
	*/
	TIM_ICInit(TIM3, &TIM_ICInitStructure);
	//配置通道2
	TIM_ICInitStructure.TIM_Channel = TIM_Channel_2;
	TIM_ICInit(TIM3, &TIM_ICInitStructure);
	
	//配置定时器接口
	TIM_EncoderInterfaceConfig(TIM3, TIM_EncoderMode_TI12, TIM_ICPolarity_Rising, TIM_ICPolarity_Rising);
	
	TIM_Cmd(TIM3, ENABLE);
}

//使用16位有符号数实现负数表示
int16_t Encoder_Get(void)
{
	//读取单位时间内的计数变化量后清零
	int16_t Temp;
	Temp = TIM_GetCounter(TIM3);
	TIM_SetCounter(TIM3, 0);
	return Temp;
}
```

### 主程序

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"
#include "Encoder.h"

int16_t Speed;

int main(void)
{
	OLED_Init();
	Encoder_Init();
	OLED_ShowString(1, 1, "Speed:");
	while(1)
	{
		OLED_ShowSignedNum(1, 7, Encoder_Get(), 5);
	}
}

//通过TIM2的定时中断实现每隔1s刷新速度
void TIM2_IRQHandler(void)
{
	if (TIM_GetITStatus(TIM2, TIM_IT_Update))
	{
		Speed = Encoder_Get();
		
	}
}
```

# ADC数模转换器

## ADC简介

- ADC（Analog-Digital Converter）模拟-数字转换器
- ADC可以将引脚上连续变化的模拟电压转换为内存中存储的数字变量，建立模拟电路到数字电路的桥梁。与之相对的是使用加权电阻网络将数字量转换为模拟量的DAC数模转换器。
- STM32的ADC为12位逐次逼近型ADC，转换时间为1μs，对应1MHz转换频率。12位是分辨率，位数越高，量化结果越精细。
- 输入电压范围：0\~3.3V，转换结果范围：0\~4095，两者之间为一一对应的线性关系。
- 18个输入通道，最多可测量16个外部（GPIO）和2个内部（内部温度传感器和参考电压）信号源。
- 规则组和注入组两个转换单元。普通的AD转换流程是启动一次转换读一次值，STM32可一次性启动一个组以连续转换多个值，规则组用于常规使用，注入组用于突发事件。
- 模拟看门狗自动监测输入电压范围，当AD值高于设定的上阈值或低于下阈值时会申请中断。
- STM32F103C8T6 ADC资源：ADC1、ADC2，10个外部输入通道。

### 逐次逼近型ADC

![](https://i-blog.csdnimg.cn/direct/f77add0fe4644e91aa333f174f7bfe3c.png)


ADC0809是一个独立的8位逐次逼近型ADC芯片。IN0\~IN7为8路输入通道，通过通道选择开关选择一路到比较器进行转换。地址锁存和译码接收3位通道号ADDA\~ADDC，并在锁存信号ALE传入时拨好对应通路的开关。比较器的两个输入端一端为待测电压，另一端为已知编码的DAC输出。经比较后，如果DAC输出电压较小则增大DAC输入数据，如果DAC输出电压较大则减小DAC输入数据，直到DAC输出电压和外部通道输入电压近似相等，此时DAC输入数据即外部电压的编码数据。电压调节的过程由SAR完成，寻找未知电压的方法为二分法，对二进制来说就是从高位到低位依次判断是1还是0的过程，即“逐次逼近”。转换结束后，DAC逼近得到的输入数据通过缓冲器输出。除此之外，CLOCK为推动ADC逐次逼近的时钟，START为开始转换信号，EOC为转换结束信号。$V_{REF(+)}$和$V_{REF(-)}$为DAC参考电压，用于决定8位最大值所表示的电压，它也决定了ADC输入范围，因此也是ADC参考电压。通常参考电压的正极和$V_{CC}$、负极和GND相同。

### ADC框图

![](https://i-blog.csdnimg.cn/direct/bbda30b921b84113ad6edc90b32d1fe1.png)


ADC左侧有ADCx_IN0\~ADCx_IN15共16个GPIO输入通道和温度传感器与$V_{REFINT}$两个内部通道。18个通道通过模拟多路开关指定想要选择的通道，并输出到模数转换器执行逐次逼近过程，转换结果存入通道数据寄存器以供读取。STM32可以一次性同时选中多个通道依次转换，注入组最多可选中4个，规则组最多可选中16个。但是规则组只有一个数据寄存器，因此需要配合DMA在每转换一个通道后将数据转移以便转换下一个通道，防止覆盖。

外围线路中，左下角为触发转换部分，相当于开始转换信号。STM32的ADC触发信号又两种，一种为手动调用代码软件触发，另一种为触发转换部分接收的硬件触发源，主要来自定时器和外部中断引脚，通常使用TRGO实现自动触发转换。左上角$V_{REF+}$、$V_{REF-}$为ADC参考电压，$V_{DDA}$、$V_{SSA}$为ADC供电引脚，参考电压在内部与供电引脚接在一起。右侧ADDCLK为ADC用于驱动内部逐次逼近的时钟，来自于RCC时钟树的ADC预分频器，分频系数可以是2/4/6/8，但ADCCLK最大只支持14MHz，因此只能选六分频12MHz和八分频9MHz。DMA请求用于触发DMA进行数据转运。模拟看门狗可以存储一个阈值高限和阈值低限，如果启动了模拟看门狗并指定了看门通道，一旦看门通道电压超出阈值，看门狗就会申请模拟看门狗中断通往NVIC，而规则组和注入组在转换完成后也会产生转换结束信号将对应标志位置一，可选择触发中断。

### ADC基本结构图

![](https://i-blog.csdnimg.cn/direct/b51c46f9a8944804a5e027bb784db1e3.png)


### 输入通道

![](https://i-blog.csdnimg.cn/direct/ccf0a606231e44a4b2a5cf82e46345cb.png)


其中STM32F103C8T6没有ADC3和PC0\~PC5引脚。

### 转换模式

ADC初始化结构体包含两个参数分别选择单次/连续转换和扫描/非扫描模式，两者排列组合构成以下四种模式。

#### 单次转换-非扫描模式

![](https://i-blog.csdnimg.cn/direct/45f5868075f24c9a94e76e0b56fbfae5.png)


序列1\~16写入需要转换的通道，在单次转换模式下只有序列1有效，一次只能选择一个通道。将通道写入序列1，触发转换，经过一个转换时间后转换完成，转换结果存储在数据寄存器，同时EOC标志位置一，转换过程结束。如果想再启动一次转换需要再触发一次，如果想更换通道需要在触发之前更改。

#### 连续转换-非扫描模式

![](https://i-blog.csdnimg.cn/direct/49f2e37c641045c7836faae1ba0f10de.png)


连续转换在一次转换之后不会停止，而是立刻开始下一轮的转换，一直持续下去。好处是无需继续手动开始转换，后续读取也无需判断是否转换结束。

#### 单次转换-扫描模式

![](https://i-blog.csdnimg.cn/direct/2d4ec6c2b77f45b0bb24962eeafad113.png)


扫描模式可以指定通道数目参数，向序列写入多个通道，允许重复。每次触发之后，ADC依次对序列中前通道数目个位置进行转换，在所有通道转换完成之后产生EOC信号，转换结束。在扫描模式的情况下还有间断模式，其作用是在扫描的过程中每隔几个转换暂停一次，需要再次触发才能继续，仅供了解。

#### 连续转换-扫描模式

![](https://i-blog.csdnimg.cn/direct/7e6a9e4532d1426893ecdc90eaff242a.png)


### 触发控制

![](https://i-blog.csdnimg.cn/direct/88f74b442ba042f2bad381a1c1202107.png)


外部触发信号的选择通过配置EXTSEL寄存器来完成。

### 数据对齐

ADC转换结果为12位，但数据寄存器有16位，因此存在数据对齐的问题。一般使用右对齐，这样读取寄存器直接得到转换结果，而左对齐的结果还需除以$2^4$。而当不需要12位精度时，使用左对齐可以方便地提取转换结果的高位，舍弃低位。

![](https://i-blog.csdnimg.cn/direct/a323590b272544f48b903d544aa7de6e.png)


### 转换时间

AD转换步骤为采样-保持-量化-编码。其中量化编码为逐次逼近过程，位数越多耗时越长。采样保持将采样的电压存储进电容后断开采样开关再进行AD转换，以确保量化编码期间电压保持不变，精确定位未知电压位置，采样时间手动配置，时间越长越能避免毛刺信号干扰。由此STM32ADC总转换时间：

$T_{CONV}$=采样时间+12.5个ADC周期

如果不需要极高的转换频率，可忽略转换时间。

### 校准

ADC有内置自校准模式。校准可大幅减小因内部电容器组的变化而造成的准精度误差。校准期间，在每个电容器上都会计算出一个误差修正码，用于消除在随后的转换中每隔电容器上产生的误差。建议在每次上电后执行一次校准，启动校准前ADC必须处于关电状态超过两个ADC时钟周期。

## 硬件电路

![](https://i-blog.csdnimg.cn/direct/a9e3e37848fb4facbbead3e346814cf9.png)


上图为ADC实验外围电路设计。左侧通过电位器产生可调电压，滑动端往上滑电压增大，往下滑电压减小。中间将传感器电阻（等效为一个可变电阻）与一个固定电阻串联分压，传感器阻值变小，下拉作用变强，输出端（在传感器模块中为AO引脚）电压下降，反之升高。右侧为电压转换电路，将电压值超过ADC转换范围的电压通过分压转换为范围内的电压，但能力有限，过高的电压建议使用隔离放大器等专用采集芯片。

## 函数详解

### RCC_ADCCLKConfig函数

**简介**：配置ADCCLK分频器。

**参数**：分频系数

	RCC_PCLK2_Div2, RCC_PCLK2_Div4, RCC_PCLK2_Div6, RCC_PCLK2_Div8

### ADC_DeInit函数

**简介**：恢复缺省配置。

**参数**：ADC名称

	ADC1, ADC2, ADC3

### ADC_Init函数

**简介**：配置ADC模块。

**参数一**：ADC名称

**参数二**：ADC_InitTypeDef结构体指针

#### ADC_InitTypeDef结构体

**成员ADC_Mode**：ADC工作模式

	ADC_Mode_Independent（独立模式）
	其余为双ADC模式，较为复杂，暂时不用

**成员ADC_DataAlign**：数据对齐

	ADC_DataAlign_Right, ADC_DataAlign_Left

**成员ADC_ExternalTrigConv**：启动规则组转换的外部触发源

	ADC_ExternalTrigConv_x（x对应ADC框图的触发源名称，其中CH改为CC）
	ADC_ExternalTrigConv_None（不使用外部触发源）

**成员ADC_ContinuousConvMode**：是否启用连续转换

	ENABLE, DISABLE

**成员ADC_ScanConvMode**：是否启用扫描模式

	ENABLE, DISABLE

**成员ADC_NbrOfChannel**：扫描模式下的通道数目

	1~16

### ADC_StructInit函数

**简介**：初始化ADC_InitTypeDef结构体。

**参数**：ADC_InitTypeDef结构体指针

### ADC_Cmd函数

**简介**：给ADC上电。

**参数一**：ADC名称

**参数二**：使能/失能

### ADC_DMACmd函数

**简介**：开启DMA输出信号。

**参数一**：ADC名称

**参数二**：使能/失能

### ADC_ITConfig函数

**简介**：中断输出控制。

**参数一**：ADC名称

**参数二**：ADC中断源

	ADC_IT_EOC（转换结束）, ADC_IT_AWD（模拟看门狗事件）, ADC_IT_JEOC（注入转换结束）

**参数三**：使能/失能

### 校准函数

**ADC_ResetCalibration函数**（复位校准）

**ADC_GetResetCalibrationStatus函数**（获取复位校准状态）

**ADC_StartCalibration函数**（开始校准）

**ADC_GetCalibrationStatus函数**（获取开始校准状态）

**参数**：ADC名称。

在ADC初始化完成后依次调用即可。

## ADC_SoftwareStartConvCmd函数

**简介**：启用软件触发转换。

**参数一**：ADC名称

**参数二**：使能/失能

### ADC_GetSoftwareStartConvStatus函数（不常用）

**简介**：读取SWSTART位的转换开始信号状态。软件设置该位以启动转换，硬件在转换开始后马上清除此位。

**参数**：ADC名称

### ADC_DiscModeChannelCountConfig函数

**简介**：配置间断模式每隔几个通道间断一次。

**参数一**：ADC名称

**参数二**：通道数

### ADC_DiscModeCmd函数

**简介**：启用间断模式。

**参数一**：ADC名称

**参数二**：使能/失能

### ADC_RegularChannelConfig函数

**简介**：配置ADC规则组通道。

**参数一**：ADC名称

**参数二**：指定通道

	ADC_Channel_0, ..., ADC_Channel_17

**参数三**：序列位置（1\~16）

**参数四**：采样时间（x取1/7/13/28/41/55/71/239，采样时间为x+0.5个ADC周期）

	ADC_SampleTime_xCycles5

### ADC_ExternalTrigConvCmd函数

**简介**：允许外部触发转换。

**参数一**：ADC名称

**参数二**：使能/失能

### ADC_GetConversionValue函数

**简介**：读取ADC转换值。

**参数**：ADC名称

### ADC_GetDualModeConversionValue函数

**简介**：双ADC模式读取转换值。暂时不用。

**参数**：void

### 中断标志位函数

**ADC_GetFlagStatus函数**

**ADC_ClearFlag函数**

**参数一**：ADC名称

**参数二**：ADC标志位

	ADC_FLAG_AWD, ADC_FLAG_EOC, ADC_FLAG_JEOC
	ADC_FLAG_JSTRT（注入组转换开始）, ADC_FLAG_STRT（规则组转换开始）

**ADC_GetITStatus函数**

**ADC_ClearITPendingBit函数**

**参数一**：ADC名称

**参数二**：ADC中断源

## 实验16 AD单通道

### 接线图

![](https://i-blog.csdnimg.cn/direct/f6d85786b98a4bc8bf47305d3091e128.jpeg)


### ADC驱动

**AD.h**

```c
#ifndef __AD_H
#define __AD_H

void AD_Init(void);
uint16_t AD_GetValue(void);

#endif
```

**AD.c**

```c
#include "stm32f10x.h"

void AD_Init(void)
{
	//配置时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	//ADC时钟为六分频12MHz
	RCC_ADCCLKConfig(RCC_PCLK2_Div6);
	
	//配置PA0
	GPIO_InitTypeDef GPIO_InitStructure;
	//ADC使用模拟输入模式
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	
	//选择规则组输入通道
	//将通道0写入序列1，采样时间要求不高随意
	ADC_RegularChannelConfig(ADC1, ADC_Channel_0, 1, ADC_SampleTime_55Cycles5);
	
	//初始化ADC
	//单次转换-非扫描模式-右对齐-无外部触发源-独立模式
	ADC_InitTypeDef ADC_InitStructure;
	ADC_InitStructure.ADC_ContinuousConvMode = DISABLE;
	ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;
	ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;
	ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;
	ADC_InitStructure.ADC_NbrOfChannel = 1;
	ADC_InitStructure.ADC_ScanConvMode = DISABLE;
	ADC_Init(ADC1, &ADC_InitStructure);
	
	//上电
	ADC_Cmd(ADC1, ENABLE);
	//校准
	ADC_ResetCalibration(ADC1);
	//校准完成后对应位清零跳出循环
	while (ADC_GetResetCalibrationStatus(ADC1));
	ADC_StartCalibration(ADC1);
	while (ADC_GetCalibrationStatus(ADC1));
}

//启动转换获取结果
uint16_t AD_GetValue(void)
{
	//软件触发转换
	ADC_SoftwareStartConvCmd(ADC1, ENABLE);
	//等待转换完成，转换结束标志位置一
	while (!ADC_GetFlagStatus(ADC1, ADC_FLAG_EOC));
	return ADC_GetConversionValue(ADC1);
	//读取后硬件自动清除转换结束标志位
}
```

### 主程序

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"
#include "AD.h"

uint16_t ADValue;
float Voltage;

int main(void)
{
	OLED_Init();
	AD_Init();
	OLED_ShowString(1, 1, "ADValue:");
	OLED_ShowString(2, 1, "Voltage:0.00V");
	while(1)
	{
		ADValue = AD_GetValue();
		//将AD值转换为电压值，为保留小数对ADValue做强制转换
		Voltage = (float)ADValue / 4095 * 3.3;
		OLED_ShowNum(1, 9, ADValue, 4);
		//尚未有OLED显示浮点数函数，将整数部分和小数部分分批显示
		OLED_ShowNum(2, 9, Voltage, 1);
		//取余需将浮点数转换为整数
		OLED_ShowNum(2, 11, (uint16_t)(Voltage * 100) % 100, 2);
		//降低刷新频率
		Delay_ms(100);
	}
}
```

### 连续转换

连续转换需对AD.c做如下改动：

```c
#include "stm32f10x.h"

void AD_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	RCC_ADCCLKConfig(RCC_PCLK2_Div6);
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	
	ADC_RegularChannelConfig(ADC1, ADC_Channel_0, 1, ADC_SampleTime_55Cycles5);
	
	ADC_InitTypeDef ADC_InitStructure;
	//启用连续转换
	ADC_InitStructure.ADC_ContinuousConvMode = ENABLE;
	ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;
	ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;
	ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;
	ADC_InitStructure.ADC_NbrOfChannel = 1;
	ADC_InitStructure.ADC_ScanConvMode = DISABLE;
	ADC_Init(ADC1, &ADC_InitStructure);
	
	ADC_Cmd(ADC1, ENABLE);
	ADC_ResetCalibration(ADC1);
	while (ADC_GetResetCalibrationStatus(ADC1));
	ADC_StartCalibration(ADC1);
	while (ADC_GetCalibrationStatus(ADC1));
	
	//软件触发只需在初始化执行一次
	ADC_SoftwareStartConvCmd(ADC1, ENABLE);
}

uint16_t AD_GetValue(void)
{
	//无需判断转换结束
	return ADC_GetConversionValue(ADC1);
}
```

## 实验17 AD多通道

### 接线图

![](https://i-blog.csdnimg.cn/direct/5fe635278d854f5b8ba80da822b1e6f7.jpeg)


### ADC驱动

本实验通过单次转换-非扫描模式每次触发更改输入通道实现AD多通道，扫描模式与DMA将在下一节介绍。

**AD.h**

```c
#ifndef __AD_H
#define __AD_H

void AD_Init(void);
uint16_t AD_GetValue(uint8_t ADC_Channel);

#endif
```

**AD.c**

```c
#include "stm32f10x.h"

void AD_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	RCC_ADCCLKConfig(RCC_PCLK2_Div6);
	
	//初始化PA0~PA3
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1 | GPIO_Pin_2 | GPIO_Pin_3;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
		
	ADC_InitTypeDef ADC_InitStructure;
	ADC_InitStructure.ADC_ContinuousConvMode = DISABLE;
	ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;
	ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;
	ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;
	ADC_InitStructure.ADC_NbrOfChannel = 1;
	ADC_InitStructure.ADC_ScanConvMode = DISABLE;
	ADC_Init(ADC1, &ADC_InitStructure);
	
	ADC_Cmd(ADC1, ENABLE);
	
	ADC_ResetCalibration(ADC1);
	while (ADC_GetResetCalibrationStatus(ADC1));
	ADC_StartCalibration(ADC1);
	while (ADC_GetCalibrationStatus(ADC1));
}

//通过参数传入指定的通道
uint16_t AD_GetValue(uint8_t ADC_Channel)
{
	ADC_RegularChannelConfig(ADC1, ADC_Channel, 1, ADC_SampleTime_55Cycles5);
	ADC_SoftwareStartConvCmd(ADC1, ENABLE);
	while (!ADC_GetFlagStatus(ADC1, ADC_FLAG_EOC));
	return ADC_GetConversionValue(ADC1);
}
```

### 主程序

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"
#include "AD.h"

uint16_t AD0, AD1, AD2, AD3;

int main(void)
{
	OLED_Init();
	AD_Init();
	OLED_ShowString(1, 1, "AD0:");
	OLED_ShowString(2, 1, "AD1:");
	OLED_ShowString(3, 1, "AD2:");
	OLED_ShowString(4, 1, "AD3:");
	while(1)
	{
		AD0 = AD_GetValue(ADC_Channel_0);
		AD1 = AD_GetValue(ADC_Channel_1);
		AD2 = AD_GetValue(ADC_Channel_2);
		AD3 = AD_GetValue(ADC_Channel_3);
		OLED_ShowNum(1, 5, AD0, 4);
		OLED_ShowNum(2, 5, AD1, 4);
		OLED_ShowNum(3, 5, AD2, 4);
		OLED_ShowNum(4, 5, AD3, 4);
		Delay_ms(100);
	}
}
```
