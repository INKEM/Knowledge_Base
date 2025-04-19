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
- DMA数据转运
- DMA+AD多通道

新函数：
- DMA库函数

@[TOC](目录)
# DMA简介

- DMA（Direct Memory Access）直接存储器存取
- DMA可以提供外设和存储器或存储器和存储器之间的高速数据传输，无须CPU干预，节省了CPU的资源。
- STM32的DMA拥有12个独立可配置的通道：DMA1（7个通道），DMA2（5个通道）
- 每个通道都支持软件触发和特定的硬件触发。存储器和存储器之间通常使用软件触发一次性全部转运，而外设到存储器的转运通常由硬件触发源触发以在正确的时机转运。
- STM32F103C8T6资源：DMA1（7个通道）

# 存储器映像

存储器映像包含了STM32的所有存储器及其地址：

![](https://i-blog.csdnimg.cn/direct/fdd351f3344e4d1f939ecae1346329e2.png)


其中ROM是非易失性、掉电不丢失的只读存储器，RAM是易失性、掉电丢失的随机存储器。

从起始地址开始，存储器内部的每个字节都会分配一个独一无二的地址，终止地址取决于存储器的容量。外设寄存器中的每个外设又有自己的起始地址。

# DMA框图

![](https://i-blog.csdnimg.cn/direct/f3004cb5982946d8ae407659c3e429e4.png)


左上角为Cortex-M3内核，包含CPU和内核外设，其右侧设备均可看作存储器。总线矩阵可以高效有条理地访问存储器，左端是拥有存储器访问权的主动单元，右端是被主动单元读写的被动单元。DMA1和DMA2各有一条DMA总线，另一条DMA总线为以太网外设私有的DMA。DMA1和DMA2的各个通道可分别设置它们转运数据的源地址和目的地址，各自独立工作，但每个DMA的所有通道只能分时复用一条DMA总线，如果产生冲突，则由仲裁器根据通道的优先级决定先后顺序。总线矩阵也有一个仲裁器，如果DMA和CPU要访问同一个目标，则DMA会暂停CPU访问以防冲突，但总线仲裁器仍保证CPU得到一半的总线带宽以正常工作。AHB从设备是DMA自身的寄存器，作为被动单元由CPU配置。DMA请求是DMA的硬件触发源。

# DMA基本结构

![](https://i-blog.csdnimg.cn/direct/bfab68cf297246e5a1a8327e27ecc261.png)


在数据转运中，转运的方向由参数控制，但由于Flash只读，因此无法将数据转运到Flash。外设站点和存储器站点（此为STM32官方称呼，实际上站点无外设和存储器之分）各有3个参数，起始地址决定数据从哪来、到哪去；数据宽度指定一次转运要按多大的数据宽度来进行，可选字节（8位）、半字（16位）、字（32位）；地址是否自增指定一次转运完成后，下一次转运是否要把地址移动到下一个位置，以防数据覆盖。

传输计数器指定总共需要转运几次，是一个自减计数器，每转运一次计数器减一，减到零后不再进行数据转运，自增地址也会恢复为起始地址以便开始新一轮转运。自动重装器指定传输计数器减到零后是否要自动恢复到最初的值，即转运模式是单次还是循环。

触发决定DMA在什么时机转运，触发源由M2M决定，1为软件触发，0为硬件触发。软件触发会连续不断地触发转运直至传输计数器清零，因此无法与循环模式同时使用，适用于软件启动、无需时机、需尽快完成的存储器到存储器的转运。而外设到寄存器的转运需要一定的时机，通常使用硬件触发。

开关控制使能后则DMA可以转运。若为单次转运，则传输计数器清零后需关闭DMA，重新写入传输计数器的值再开启。

# DMA请求

![](https://i-blog.csdnimg.cn/direct/b91147be7b8f4da28549019518ffab15.png)


每个通道的硬件触发源不同，硬件触发必须使用相应的通道，而软件触发可以任意选择。每个通道选择哪个触发源由对应的外设是否开启了DMA输出决定。

# 数据宽度与对齐

![](https://i-blog.csdnimg.cn/direct/d3534a03e5f04cd7a491f1d48c536ea2.png)


该表列举了STM32在转运两端不同数据宽度配置下的处理方法。简而言之，目标宽度冗余则高位补零，目标宽度不足则舍弃高位。

# 函数详解

## DMA_DeInit函数

**简介**：恢复缺省配置。

**参数**：DMA通道

	DMA1_Channel1, ..., DMA1_Channel7
	DMA2_Channel1, ..., DMA2_Channel5

## DMA_Init函数

**简介**：初始化DMA。

**参数一**：DMA通道

**参数二**：DMA_InitTypeDef结构体指针

### DMA_InitTypeDef结构体

**成员DMA_PeripheralBaseAddr**：外设站点起始地址

**成员DMA_PeripheralDataSize**：外设站点数据宽度

	DMA_PeripheralDataSize_Byte（字节）
	DMA_PeripheralDataSize_HalfWord（半字）
	DMA_PeripheralDataSize_Word（字）

**成员DMA_PeripheralInc**：外设站点地址是否自增

	DMA_PeripheralInc_Enable（自增）
	DMA_PeripheralInc_Disable（不自增）

**成员DMA_MemoryBaseAddr**：存储器站点起始地址

**成员DMA_MemoryDataSize**：存储器站点数据宽度

	DMA_MemoryDataSize_Byte（字节）
	DMA_MemoryDataSize_HalfWord（半字）
	DMA_MemoryDataSize_Word（字）

**成员DMA_MemoryInc**：存储器站点地址是否自增

	DMA_MemoryInc_Enable（自增）
	DMA_MemoryInc_Disable（不自增）

**成员DMA_DIR**：传输方向

	DMA_DIR_PeripheralDST（存储器站点→外设站点）
	DMA_DIR_PeripheralSRC（外设站点→存储器站点）

**成员DMA_BufferSize**：传输计数器值

**成员DMA_Mode**：是否自动重装

	DMA_Mode_Circular（循环模式）
	DMA_Mode_Normal（单次模式）

**成员DMA_M2M**：触发方式

	DMA_M2M_Enable（软件触发）
	DMA_M2M_Disable（硬件触发）

**成员DMA_Priority**：优先级

	DMA_Priority_VeryHigh
	DMA_Priority_High
	DMA_Priority_Medium
	DMA_Priority_Low

## DMA_StructInit函数

**简介**：初始化DMA_InitTypeDef结构体。

**参数**：DMA_InitTypeDef结构体指针

## DMA_Cmd函数

**简介**：DMA使能。

**参数一**：DMA通道

**参数二**：使能/失能

## DMA_ITConfig函数

**简介**：DMA中断输出使能。

**参数一**：DMA通道

**参数二**：DMA中断源

	DMA_IT_TC（转运完成）
	DMA_IT_HT（转运过半）
	DMA_IT_TE（转运错误）

## DMA_SetCurrDataCounter函数

**简介**：设置当前传输计数器。

**参数一**：DMA通道

**参数二**：计数值

## DMA_GetCurrDataCounter函数

**简介**：获取当前传输计数器值。

**参数一**：DMA通道

## 中断标志位函数

**DMA_GetFlagStatus函数**

**DMA_ClearFlag函数**

**参数**：DMA标志位

	x为DMA名称，y为DMA通道
	DMAx_FLAG_GLy（全局）
	DMAx_FLAG_TCy（转运完成）
	DMAx_FLAG_HTy（转运过半）
	DMAx_FLAG_TEy（转运错误）

**DMA_GetITStatus函数**

**DMA_ClearITPendingBit函数**

**参数**：DMA中断源

# 实验18 DMA数据转运

## 任务分析

![](https://i-blog.csdnimg.cn/direct/7c5dd064d7194faa874f88b260bf0124.png)


该实验我们将SRAM中的数组DataA转运到另一个数组DataB中。外设地址和存储器地址分别为DataA和DataB的首地址；数组类型为uint8_t，因此数据宽度为8位字节；每次转运完数组的一个元素后，两个站点的地址都应该自增移动到下一个元素的位置继续转运；方向为外设站点到存储器站点；传输计数器初值为数组元素个数，不需要自动重装；存储器到存储器的转运使用软件触发。

## 接线图

![](https://i-blog.csdnimg.cn/direct/74cdfb151017475fae2cd62d05ddbb52.jpeg)


## DMA驱动

该驱动存放在System文件夹和组中。

**MyDMA.h**

```c
#ifndef __MYDMA_H
#define __MYDMA_H

void MyDMA_Init(uint32_t AddrA, uint32_t AddrB, uint16_t Size);
void MyDMA_Transfer(void);

#endif
```

**MyDMA.c**

```c
#include "stm32f10x.h"

//存储传输计数器初值的全局变量
uint16_t MyDMA_Size;

void MyDMA_Init(uint32_t AddrA, uint32_t AddrB, uint16_t Size)
{
	MyDMA_Size = Size;
	RCC_AHBPeriphClockCmd(RCC_AHBPeriph_DMA1, ENABLE);
	 
	DMA_InitTypeDef DMA_InitStructure;
	//外设站点
	//数组地址不固定，因此通过传入参数的数组名获取地址
	DMA_InitStructure.DMA_PeripheralBaseAddr = AddrA;
	//数据宽度8位字节
	DMA_InitStructure.DMA_PeripheralDataSize = DMA_PeripheralDataSize_Byte;
	//地址自增
	DMA_InitStructure.DMA_PeripheralInc = DMA_PeripheralInc_Enable;
	//存储器站点
	DMA_InitStructure.DMA_MemoryBaseAddr = AddrB;
	DMA_InitStructure.DMA_MemoryDataSize = DMA_MemoryDataSize_Byte;
	DMA_InitStructure.DMA_MemoryInc = DMA_MemoryInc_Enable;
	//传输计数器初值
	DMA_InitStructure.DMA_BufferSize = Size;
	//方向由外设到存储器
	DMA_InitStructure.DMA_DIR = DMA_DIR_PeripheralSRC;
	//软件触发
	DMA_InitStructure.DMA_M2M = DMA_M2M_Enable;
	//单次转运
	DMA_InitStructure.DMA_Mode = DMA_Mode_Normal;
	//优先级随意
	DMA_InitStructure.DMA_Priority = DMA_Priority_Medium;
	DMA_Init(DMA1_Channel1, &DMA_InitStructure);
	//初始化不转运，等待手动调用再转运
	DMA_Cmd(DMA1_Channel1, DISABLE);
}

//再次给传输计数器赋值启动DMA转运
void MyDMA_Transfer(void)
{
	DMA_Cmd(DMA1_Channel1, DISABLE);
	DMA_SetCurrDataCounter(DMA1_Channel1, MyDMA_Size);
	DMA_Cmd(DMA1_Channel1, ENABLE);
	//等待转运完成
	while(!DMA_GetFlagStatus(DMA1_FLAG_TC1));
	//手动清除标志位
	DMA_ClearFlag(DMA1_FLAG_TC1);
}
```

## 主程序

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"
#include "MyDMA.h"

uint8_t DataA[] = {0x01, 0x02, 0x03, 0x04};
uint8_t DataB[] = {0, 0, 0, 0};

int main(void)
{
	OLED_Init();
	MyDMA_Init((uint32_t)DataA, (uint32_t)DataB, 4);
	
	OLED_ShowString(1, 1, "DataA");
	OLED_ShowString(3, 1, "DataB");
	//显示数组地址
	OLED_ShowHexNum(1, 8, (uint32_t)DataA, 8);
	OLED_ShowHexNum(3, 8, (uint32_t)DataB, 8);
	OLED_ShowHexNum(2, 1, DataA[0], 2);
	OLED_ShowHexNum(2, 4, DataA[1], 2);
	OLED_ShowHexNum(2, 7, DataA[2], 2);
	OLED_ShowHexNum(2, 10, DataA[3], 2);
	OLED_ShowHexNum(4, 1, DataB[0], 2);
	OLED_ShowHexNum(4, 4, DataB[1], 2);
	OLED_ShowHexNum(4, 7, DataB[2], 2);
	OLED_ShowHexNum(4, 10, DataB[3], 2);
	
	while(1)
	{
		//DataA自增
		DataA[0] ++;
		DataA[1] ++;
		DataA[2] ++;
		DataA[3] ++;
		OLED_ShowHexNum(2, 1, DataA[0], 2);
		OLED_ShowHexNum(2, 4, DataA[1], 2);
		OLED_ShowHexNum(2, 7, DataA[2], 2);
		OLED_ShowHexNum(2, 10, DataA[3], 2);
		OLED_ShowHexNum(4, 1, DataB[0], 2);
		OLED_ShowHexNum(4, 4, DataB[1], 2);
		OLED_ShowHexNum(4, 7, DataB[2], 2);
		OLED_ShowHexNum(4, 10, DataB[3], 2);
		Delay_ms(1000);
		//将DataA转运到DataB
		MyDMA_Transfer();
		OLED_ShowHexNum(2, 1, DataA[0], 2);
		OLED_ShowHexNum(2, 4, DataA[1], 2);
		OLED_ShowHexNum(2, 7, DataA[2], 2);
		OLED_ShowHexNum(2, 10, DataA[3], 2);
		OLED_ShowHexNum(4, 1, DataB[0], 2);
		OLED_ShowHexNum(4, 4, DataB[1], 2);
		OLED_ShowHexNum(4, 7, DataB[2], 2);
		OLED_ShowHexNum(4, 10, DataB[3], 2);
		Delay_ms(1000);
	}
}
```

# 实验19 DMA+AD多通道

## 任务分析

![](https://i-blog.csdnimg.cn/direct/e455c031c15548b2978b8f489bbd1443.png)


左侧为ADC扫描模式执行流程，7个通道依次进行AD转换，转换结果都存入ADC_DR数据寄存器。我们需要在每个单独的通道转换完成之后进行一次DMA数据转运，并且目的地址自增，以防数据覆盖。DMA配置中，外设地址为ADC_DR寄存器地址，存储器地址是在SRAM中定义数组ADValue的地址；数据宽度根据uint16_t数据类型，选择半字传输；外设地址不自增，存储器地址自增；传输方向由外设站点到计数器站点；传输计数器初值与转换通道数相同；如果ADC为单次扫描，则DMA可不自动重装，如果为连续扫描，则DMA可自动重装；DMA转运时机需和ADC单个通道转换完成同步，因此触发源为ADC硬件触发。

## 接线图

![](https://i-blog.csdnimg.cn/direct/3261cfaa812f4a078248f19195aaf0e0.jpeg)


接线同实验17。

## 单次扫描-单次转运

### ADC驱动

ADC驱动沿用实验17。

**AD.h**

```c
#ifndef __AD_H
#define __AD_H

//供外部调用的SRAM数组
extern uint16_t AD_Value[4];

void AD_Init(void);
void AD_GetValue(void);

#endif
```

**AD.c**

```c
#include "stm32f10x.h"

uint16_t AD_Value[4];

void AD_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	RCC_ADCCLKConfig(RCC_PCLK2_Div6);
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1 | GPIO_Pin_2 | GPIO_Pin_3;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	
	//配置多通道序列
	ADC_RegularChannelConfig(ADC1, ADC_Channel_0, 1, ADC_SampleTime_55Cycles5);
	ADC_RegularChannelConfig(ADC1, ADC_Channel_1, 2, ADC_SampleTime_55Cycles5);
	ADC_RegularChannelConfig(ADC1, ADC_Channel_2, 3, ADC_SampleTime_55Cycles5);
	ADC_RegularChannelConfig(ADC1, ADC_Channel_3, 4, ADC_SampleTime_55Cycles5);

	ADC_InitTypeDef ADC_InitStructure;
	ADC_InitStructure.ADC_ContinuousConvMode = DISABLE;
	ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;
	ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;
	ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;
	//配置通道数为4
	ADC_InitStructure.ADC_NbrOfChannel = 4;
	//开启扫描模式
	ADC_InitStructure.ADC_ScanConvMode = ENABLE;
	ADC_Init(ADC1, &ADC_InitStructure);
	
	//DMA初始化沿用实验18
	RCC_AHBPeriphClockCmd(RCC_AHBPeriph_DMA1, ENABLE);
	DMA_InitTypeDef DMA_InitStructure;
	//外设站点
	//地址为ADC1的DR寄存器地址
	DMA_InitStructure.DMA_PeripheralBaseAddr = (uint32_t)&ADC1->DR;
	//数据宽度为半字
	DMA_InitStructure.DMA_PeripheralDataSize = DMA_PeripheralDataSize_HalfWord;
	//地址不自增
	DMA_InitStructure.DMA_PeripheralInc = DMA_PeripheralInc_Disable;
	//存储器站点
	//地址为SRAM数组地址
	DMA_InitStructure.DMA_MemoryBaseAddr = (uint32_t)AD_Value;
	//数据宽度为半字
	DMA_InitStructure.DMA_MemoryDataSize = DMA_MemoryDataSize_HalfWord;
	DMA_InitStructure.DMA_MemoryInc = DMA_MemoryInc_Enable;
	//传输数量为4
	DMA_InitStructure.DMA_BufferSize = 4;
	DMA_InitStructure.DMA_DIR = DMA_DIR_PeripheralSRC;
	//不使用软件触发
	DMA_InitStructure.DMA_M2M = DMA_M2M_Disable;
	DMA_InitStructure.DMA_Mode = DMA_Mode_Normal;
	DMA_InitStructure.DMA_Priority = DMA_Priority_Medium;
	//ADC1的硬件触发只与通道1连接，其余通道不行
	DMA_Init(DMA1_Channel1, &DMA_InitStructure);
	DMA_Cmd(DMA1_Channel1, ENABLE);
	
	//开启ADC到DMA的输出信号
	ADC_DMACmd(ADC1, ENABLE);
	
	ADC_Cmd(ADC1, ENABLE);
	ADC_ResetCalibration(ADC1);
	while (ADC_GetResetCalibrationStatus(ADC1));
	ADC_StartCalibration(ADC1);
	while (ADC_GetCalibrationStatus(ADC1));
}

void AD_GetValue(void)
{
	//重写传输计数器
	DMA_Cmd(DMA1_Channel1, DISABLE);
	DMA_SetCurrDataCounter(DMA1_Channel1, 4);
	DMA_Cmd(DMA1_Channel1, ENABLE);
	//启动转换
	ADC_SoftwareStartConvCmd(ADC1, ENABLE);
	//等待DMA转运完成
	while(!DMA_GetFlagStatus(DMA1_FLAG_TC1));
	DMA_ClearFlag(DMA1_FLAG_TC1);
}
```

### 主程序

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"
#include "AD.h"

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
		AD_GetValue();
		OLED_ShowNum(1, 5, AD_Value[0], 4);
		OLED_ShowNum(2, 5, AD_Value[1], 4);
		OLED_ShowNum(3, 5, AD_Value[2], 4);
		OLED_ShowNum(4, 5, AD_Value[3], 4);
		Delay_ms(100);
	}
}
```

## 连续扫描-循环转运

### ADC驱动

**AD.h**

```c
#ifndef __AD_H
#define __AD_H

extern uint16_t AD_Value[4];

void AD_Init(void);

#endif
```

**AD.c**

```c
#include "stm32f10x.h"

uint16_t AD_Value[4];

void AD_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_ADC1, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	RCC_ADCCLKConfig(RCC_PCLK2_Div6);
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AIN;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1 | GPIO_Pin_2 | GPIO_Pin_3;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	
	ADC_RegularChannelConfig(ADC1, ADC_Channel_0, 1, ADC_SampleTime_55Cycles5);
	ADC_RegularChannelConfig(ADC1, ADC_Channel_1, 2, ADC_SampleTime_55Cycles5);
	ADC_RegularChannelConfig(ADC1, ADC_Channel_2, 3, ADC_SampleTime_55Cycles5);
	ADC_RegularChannelConfig(ADC1, ADC_Channel_3, 4, ADC_SampleTime_55Cycles5);

	ADC_InitTypeDef ADC_InitStructure;
	//使用ADC连续模式
	ADC_InitStructure.ADC_ContinuousConvMode = ENABLE;
	ADC_InitStructure.ADC_DataAlign = ADC_DataAlign_Right;
	ADC_InitStructure.ADC_ExternalTrigConv = ADC_ExternalTrigConv_None;
	ADC_InitStructure.ADC_Mode = ADC_Mode_Independent;
	ADC_InitStructure.ADC_NbrOfChannel = 4;
	ADC_InitStructure.ADC_ScanConvMode = ENABLE;
	ADC_Init(ADC1, &ADC_InitStructure);
	
	RCC_AHBPeriphClockCmd(RCC_AHBPeriph_DMA1, ENABLE);
	DMA_InitTypeDef DMA_InitStructure;
	DMA_InitStructure.DMA_PeripheralBaseAddr = (uint32_t)&ADC1->DR;
	DMA_InitStructure.DMA_PeripheralDataSize = DMA_PeripheralDataSize_HalfWord;
	DMA_InitStructure.DMA_PeripheralInc = DMA_PeripheralInc_Disable;
	DMA_InitStructure.DMA_MemoryBaseAddr = (uint32_t)AD_Value;
	DMA_InitStructure.DMA_MemoryDataSize = DMA_MemoryDataSize_HalfWord;
	DMA_InitStructure.DMA_MemoryInc = DMA_MemoryInc_Enable;
	DMA_InitStructure.DMA_BufferSize = 4;
	DMA_InitStructure.DMA_DIR = DMA_DIR_PeripheralSRC;
	DMA_InitStructure.DMA_M2M = DMA_M2M_Disable;
	//使用DMA循环模式
	DMA_InitStructure.DMA_Mode = DMA_Mode_Circular;
	DMA_InitStructure.DMA_Priority = DMA_Priority_Medium;
	//ADC1的硬件触发只与通道1连接，其余通道不行
	DMA_Init(DMA1_Channel1, &DMA_InitStructure);
	DMA_Cmd(DMA1_Channel1, ENABLE);
	
	//开启ADC到DMA的输出信号
	ADC_DMACmd(ADC1, ENABLE);
	
	ADC_Cmd(ADC1, ENABLE);
	ADC_ResetCalibration(ADC1);
	while (ADC_GetResetCalibrationStatus(ADC1));
	ADC_StartCalibration(ADC1);
	while (ADC_GetCalibrationStatus(ADC1));
	
	//ADC触发放在初始化最后，之后ADC连续转换，DMA循环转运
	ADC_SoftwareStartConvCmd(ADC1, ENABLE);
}
//不需要GetValue函数
```

### 主程序

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"
#include "AD.h"

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
		//不需要调用GetValue函数
		OLED_ShowNum(1, 5, AD_Value[0], 4);
		OLED_ShowNum(2, 5, AD_Value[1], 4);
		OLED_ShowNum(3, 5, AD_Value[2], 4);
		OLED_ShowNum(4, 5, AD_Value[3], 4);
		Delay_ms(100);
	}
}
```
