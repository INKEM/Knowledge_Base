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
- OLED调试
- 对射式红外传感器计次
- 旋转编码器计次

新函数：
- GPIO库函数（剩余）
- EXTI库函数
- NVIC库函数

@[TOC](目录)
# 调试方式

- 串口调试：通过串口通信，将调试信息发送到电脑端，电脑使用串口助手显示调试信息。
- 显示屏调试：直接将显示屏连接到单片机，将调试信息打印在显示屏上。
- Keil调试模式：借助Keil软件的调试模式，可使用单步运行、设置断点、查看寄存器及变量等功能。

其他调试方式：

- 点灯调试：如果不清除程序是否执行到了某个位置，可以在该位置写一个点灯的代码，执行到了灯就会亮。
- 注释调试：如果程序添加了一段代码就跑不动了，则把添加的代码全部注释，再一行行解除注释运行，直到错误出现。
- 对照调试：参照他人能正常运行的代码，将其中的程序逻辑逐步替换成自己的。

# OLED简介

- OLED：有机发光二极管
- OLED显示屏：性能优异的新型显示屏，具有功耗低、响应速度快、宽视角、轻薄柔韧等特点。
- 0.96寸OLED模块：小巧玲珑、占用接口少、简单易用，是电子设计中非常常见的显示屏模块。
- 供电：3\~5.5V；通信协议：I2C/SPI；分辨率：128×64

OLED及本期实验效果图：

![](https://i-blog.csdnimg.cn/direct/6d00ac9ab2434c3986c4048bd157bca4.png)


# 硬件电路

![](https://i-blog.csdnimg.cn/direct/938dcf7fc270417f9f39476fbfd15a5c.png)


除了GND和VCC，4针脚OLED的剩下两个引脚为I2C通信引脚，7针脚OLED的剩下五个引脚为SPI通信引脚。外设的通信引脚需要与单片机对应通信协议的引脚相连，如果使用GPIO口模拟通信也可以接在任意GPIO口上。本次使用4针脚OLED。

# OLED驱动函数

由于通信协议在后期学习，我们目前先使用资料包提供的硬件驱动，其包括以下函数：

![](https://i-blog.csdnimg.cn/direct/aa3bf4d2d3984e61ad15a6ba2481f79f.png)


其中显示函数的前两个参数为显示起始位置的坐标，数字显示函数的第四个参数为显示数字长度，过长则高位补零，过短则不显示高位。

# 实验5 OLED调试

## 电路连接

![](https://i-blog.csdnimg.cn/direct/69f2fcd6daaf478d9602fffe2fb25c3a.jpeg)


SCL接PB8，SDA接PB9。

## OLED驱动

将资料包中程序源码 > STM32Project-无注释版 > 1-4 OLED驱动函数模块 > 4针脚I2C版本的三个文件复制粘贴到工程的Hardware文件夹并在Keil5中添加到组。

如果SCL和SDA接在了其他GPIO引脚上，则需要修改图中OLED.c对应的部分：

![](https://i-blog.csdnimg.cn/direct/79af51354c904c0487520afd34c86690.png)


## 主程序

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"

int main(void)
{
	OLED_Init();
	OLED_ShowChar(1, 1, 'A');
	OLED_ShowString(1, 3, "HelloWorld!");
	OLED_ShowNum(2, 1, 12345, 5);
	OLED_ShowSignedNum(2, 7, -66, 2);
	OLED_ShowHexNum(3, 1, 0xAA55, 4);
	OLED_ShowBinNum(4, 1, 0xAA55, 16);
	while(1)
	{
	}
}
```
# 中断系统

- **中断**：在主程序运行过程中，出现了特定的中断触发条件（中断源），使得CPU暂停当前正在运行的程序，转而去处理中断程序，处理完成后又返回原来被暂停的位置继续运行。
- **中断优先级**：当有多个中断源同时申请中断时，CPU会根据中断源的轻重缓急进行裁决，优先响应更加紧急的中断源。
- **中断嵌套**：当一个中断程序正在运行时，又有新的更高优先级的中断源申请终端，CPU再次暂停当前中断程序，转而去处理新的中断程序，处理完成后依次进行返回。

**中断执行流程**

![](https://i-blog.csdnimg.cn/direct/7f97879f4dd843b1a100550a668552b0.png)


中断程序在子函数里，该子函数不需要我们调用，在触发中断时硬件会自动调用。

## STM32中断

- F1系列最多有68个可屏蔽终端通道，包括EXTI、TIM、ADC、USART、SPI、I2C、RTC等多个外设。
- 使用NVIC统一管理中断，每个中断通道都拥有16个可编程的优先等级，可对优先等级进行分组，进一步设置抢占优先级和响应优先级。

**STM32中断向量表**

![](https://i-blog.csdnimg.cn/direct/15718093940941baa7d0629aaf3f6eb0.png)


灰色部分为内核中断，一般用不到。白色部分为外设中断，本节用到EXTI9_5和EXTI15_10。

**中断地址**：由于程序中的中断函数的地址由编译器分配，是不固定的，但中断跳转由于硬件限制只能跳到固定的地址执行程序。为了让硬件跳转到地址不固定的中断函数里，需要再内存中定义一个地址列表（中断向量表），中断发生后跳转到固定地址，在固定地址再由编译器加上一条跳转到中断函数的代码。在C语言编程中编译器已经帮我们配置好了，无需中断向量表。

## NVIC介绍

![](https://i-blog.csdnimg.cn/direct/14343b07bbdd4d3992368ee890c6402d.png)


NVIC是嵌套中断向量控制器，用来统一分配中断优先级和管理中断，并告诉CPU应该处理哪个中断，为CPU分担任务。

**NVIC优先级分组**

- NVIC的中断优先级由优先级寄存器的4位决定，这4位可以进行切分，分为高n位的抢占优先级和低4-n位的响应优先级。
- 抢占优先级高的可以中断嵌套，响应优先级高的可以优先排队，抢占优先级和响应优先级均按中断号排队。

![](https://i-blog.csdnimg.cn/direct/a21160efd9454d898fbc28e25a20cd79.png)


# EXTI简介

- EXTI（Extern Interrupt）外部中断
- EXTI可以监测指定GPIO口的电平信号，当其指定的GPIO口产生电平变化时，EXTI将立即向NVIC发出中断申请，经过NVIC裁决后即可中断CPU主程序，使CPU执行EXTI对应的中断程序。
- 支持的触发方式：上升沿/下降沿/双边沿/软件触发
- 支持的GPIO口：所有GPIO口，但相同的Pin不能同时触发中断
- 通道数：16个Pin，外加PVD输出、RTC闹钟、USB唤醒、以太网唤醒，后四个功能均为从低功耗模式的停止模式下唤醒STM32。
- 触发响应方式：中断响应/事件响应（不会触发中断而是触发别的外设操作）

## EXTI基本结构

![](https://i-blog.csdnimg.cn/direct/ab2ff087ecfc442a8e361e5456eea9cd.png)


对于GPIO的16个Pin，AFIO从所有GPIO外设中选择其中一个连接到EXTI通道，因此相同的Pin不能同时触发中断。16个Pin和4个外加组成了EXTI的20个输入通道，而输出通道中EXTI的9\~5、15\~10被分配到一个通道里，需在中断函数中要根据标志位区分到底是哪个中断，20个连接到其他外设的输出通道为事件响应。

## AFIO复用IO口

- AFIO主要用于引脚复用功能的选择和重定义。
- 在STM32中，AFIO主要完成两个任务：复用功能引脚重映射、中断引脚选择。

![](https://i-blog.csdnimg.cn/direct/dbbe4011f75c47dea007c4953618e5c6.png)


## EXTI框图

![](https://i-blog.csdnimg.cn/direct/34042ff7cbb04f19b621981f5951532d.png)


上升沿/下降沿触发选择寄存器控制上升沿/下降沿/双边沿触发方式是否有效，经边沿检测后，硬件触发和软件中断事件寄存器接到一个或门，随后分为两路，上一路触发中断响应，下一路触发事件响应。触发中断响应首先会经过请求挂起寄存器，相当于中断标志位，通过读取该寄存器判断中断由哪个通道触发。请求挂起寄存器置一后，中断信号和中断屏蔽寄存器接到一个与门，只有中断屏蔽寄存器置一才允许中断，事件屏蔽寄存器同理。

# 硬件模块

## 旋转编码器介绍

- 旋转编码器：用来测量位置、速度或旋转方向的装置，当其旋转轴旋转时，其输出端可以输出与旋转速度和方向对应的方波信号，读取方波信号的频率和相位信息即可得知旋转轴的速度和方向。
- 类型：机械触点式/霍尔传感器式/光栅式

![](https://i-blog.csdnimg.cn/direct/6bd78e650d9643829aabc72aaf3f5345.png)


## 硬件电路

![](https://i-blog.csdnimg.cn/direct/f1f59b6b7b28416aaa80f22306704625.png)


本次使用机械触点式旋转编码器，方框中上方按键模块悬空并未使用，下方即编码器内部的两个触点，旋转轴旋转时，这两个触点以相位差90°的方式交替导通，通过两个触点产生下降沿的先后次序判断旋转方向。

A、B端口各有一个上拉电阻、输出限流电阻和滤波电容，C端口接GND，触点导通为低电平，断开为高电平。使用模块时上方VCC、GND接电源，下方A、B接到两个Pin不一样的GPIO口，C暂时不用。

# 函数详解

## GPIO库函数

本次展示剩余的GPIO库函数，其中包括AFIO函数。

### GPIO_AFIODeInit函数

**简介**：清除AFIO外设配置。

**参数**：空

### GPIO_PinLockConfig函数（不常用）

**简介**：锁定GPIO配置。

**参数一**：GPIO外设名称

**参数二**：引脚编号

### GPIO_EventOutputConfig函数（不常用）

**简介**：选择GPIO引脚用作事件输出。

**参数一**：GPIO外设源

	GPIO_PortSourceGPIOA, ..., GPIO_PortSourceGPIOE

**参数二**：GPIO引脚源

	GPIO_PinSource0, ..., GPIO_PinSource15

### GPIO_EventOutputCmd函数（不常用）

**简介**：使能或失能事件输出，跟在GPIO_EventOutputConfig函数之后，对相应引脚进行使能或失能。

**参数**：使能/失能

	ENABLE, DISABLE

### GPIO_PinRemapConfig函数

**简介**：引脚重映射。目前未学到需要映射引脚的外设，实际调用之后展示。

**参数一**：重映射方式

**参数二**：新的状态（使能/失能）

### GPIO_EXTILineConfig函数

**简介**：配置AFIO数据选择器来选择想要的中断引脚。

**参数一**：GPIO外设源

**参数二**：GPIO引脚源

### GPIO_ETH_MediaInterfaceConfig函数（不常用）

**简介**：选择以太网接口。套件没有以太网外设。

**参数**：GPIO以太网接口

	GPIO_ETH_MediaInterface_MII, GPIO_ETH_MediaInterface_RMII

## EXTI库函数

### EXTI_DeInit函数

**简介**：清除EXTI配置，恢复上电默认状态。

**参数**：空

### EXTI_Init函数

**简介**：EXTI外设配置函数，使用方法和GPIO_Init同理。

**参数**：指向初始化信息EXTI_InitTypeDef结构体的指针

#### EXTI_InitTypeDef结构体

**成员EXTI_Line**：中断线路

	EXTI_Line0, ..., EXTI_Line19

**成员EXTI_LineCmd**：中断线路状态（开启/关闭）

	ENABLE, DISABLE

**成员EXTI_Mode**：中断线路模式

	中断模式：EXTI_Mode_Interrupt
	事件模式：EXTI_Mode_Event

**成员EXTI_Trigger**：触发信号的有效边沿

	上升沿：EXTI_Trigger_Rising
	下降沿：EXTI_Trigger_Falling
	双边沿：EXTI_Trigger_Rising_Falling

### EXTI_StructInit函数

**简介**：给EXTI_InitTypeDef结构体赋默认值。

**参数**：指向初始化信息EXTI_InitTypeDef结构体的指针

### EXTI_GenerateSWInterrupt函数

**简介**：软件触发指定中断线路。

**参数**：中断线路

### EXTI_GetFlagStatus函数

**简介**：在主程序中获取指定的挂起标志位状态。

**参数**：中断线路

**返回值**：SET/RESET

### EXTI_ClearFlag函数

**简介**：在主程序中对置一的挂起标志位进行清除。

**参数**：中断线路

### EXTI_GetITStatus函数

**简介**：在中断程序中获取指定的挂起标志位状态。

**参数**：中断线路

**返回值**：SET/RESET

### EXTI_ClearITPendingBit函数

**简介**：在中断程序中对置一的挂起标志位进行清除。

**参数**：中断线路

## NVIC库函数

以下NVIC库函数声明于负责杂项的misc.h中。

NVIC_SetVectorTable函数（设置向量表的位置和偏移）和NVIC_SystemLPConfig函数（选择系统进入低功耗模式的条件）不常用，不单独解释。

### NVIC_PriorityGroupConfig函数

**简介**：中断优先级分组。

**参数**：分组方式

	NVIC_PriorityGroup_0, ..., NVIC_PriorityGroup_4
	（数字为分配给抢占优先级的位数）

### NVIC_Init函数

**简介**：初始化NVIC。

**参数**：指向初始化信息NVIC_InitTypeDef结构体的指针

#### NVIC_InitTypeDef结构体

**成员NVIC_IRQChannel**：中断通道

	库函数兼容所有F1系列芯片，不同芯片中断通道列表不一样，根据条件编译选择
	套件芯片对应库函数中STM32F10X_MD
	中断通道较多，不一一列举，基本为'通道名称_IRQn'格式，例如EXTI15_10_IRQn

**成员NVIC_IRQChannelCmd**：使能/失能

**成员NVIC_IRQChannelPreemptionPriority**：抢占优先级（取值范围取决于优先级分组方式）

**成员NVIC_IRQChannelSubPriority**：响应优先级（取值范围取决于优先级分组方式）
 
# 实验6 对射式红外传感器计次

## 电路连接

![](https://i-blog.csdnimg.cn/direct/ac86cdaaab434796b88bca45f26ac194.jpeg)


对射式红外传感器的DO口接B14，当挡光片或编码盘从对射式红外传感器中间经过，DO输出电平跳变信号，触发PB14口中断。

## 对射式红外传感器计数驱动

以后若无特殊说明，驱动都放Hardware文件夹及组中。

**CountSensor.h**

```c
#ifndef __COUNT_SENSOR_H
#define __COUNT_SENSOR_H

void CountSensor_Init(void);
uint16_t CountSensor_Get(void);
//中断函数自动调用，无需声明

#endif
```

**CountSensor.c**

```c
#include "stm32f10x.h"
#include "Delay.h"

//计数全局变量，默认初始化为0
uint16_t CountSensor_Count;

void CountSensor_Init(void)
{
	//配置时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);
	//EXTI和NVIC的时钟默认打开
	//配置GPIO
	GPIO_InitTypeDef GPIO_InitStructure;
	//EXTI可选浮空/上拉/下拉输入
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_14;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOB, &GPIO_InitStructure);
	//配置AFIO
	//将PB14接到EXTI14中断线路
	GPIO_EXTILineConfig(GPIO_PortSourceGPIOB, GPIO_PinSource14);
	//配置EXTI
	//将EXTI14中断线路配置为中断模式，下降沿触发，并开启中断
	EXTI_InitTypeDef EXTI_InitStructure;
	EXTI_InitStructure.EXTI_Line = EXTI_Line14;
	EXTI_InitStructure.EXTI_LineCmd = ENABLE;
	EXTI_InitStructure.EXTI_Mode = EXTI_Mode_Interrupt;
	EXTI_InitStructure.EXTI_Trigger = EXTI_Trigger_Falling;
	EXTI_Init(&EXTI_InitStructure);
	//配置NVIC
	//根据实际需求选择中断分组，中断不多时随意
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
	NVIC_InitTypeDef NVIC_InitStructure;
	NVIC_InitStructure.NVIC_IRQChannel = EXTI15_10_IRQn;
	NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
	//只有一个中断，优先级随意
	NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 1;
	NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;
	NVIC_Init(&NVIC_InitStructure);
}

//返回计数值
uint16_t CountSensor_Get(void)
{
	return CountSensor_Count;
}

//中断函数名称在启动文件startup_stm32f10x_md.s中规定
void EXTI15_10_IRQHandler(void)
{
	//检查中断挂起标志位
	if(EXTI_GetITStatus(EXTI_Line14) == SET)
	{
		//若传感器不稳定可消抖
		Delay_ms(100);
		CountSensor_Count ++;
		Delay_ms(100);
		//中断程序结束时清除标志位，否则会持续申请中断
		EXTI_ClearITPendingBit(EXTI_Line14);
	}
}
```

## 主程序

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"
#include "CountSensor.h"

int main(void)
{
	OLED_Init();
	CountSensor_Init();
	OLED_ShowString(1, 1, "Count:");
	while(1)
	{
		OLED_ShowNum(1, 7, CountSensor_Get(), 5);
	}
}
```

# 实验7 旋转编码器计次

## 电路连接

![](https://i-blog.csdnimg.cn/direct/0ef09bccf0604caca3a7c8c1ae315322.jpeg)


旋转编码器A端接PB0，B端接PB1。

## 旋转编码器驱动

**Encoder.h**

```c
#ifndef __ENCODER_H
#define __ENCODER_H

void Encoder_Init(void);
int16_t Encoder_Get(void);

#endif
```

**Encoder.c**

```c
#include "stm32f10x.h"

int16_t Encoder_Count;

//初始化函数框架沿用实验6，但要配置PB0和PB1两个端口
void Encoder_Init(void)
{
	//配置时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_AFIO, ENABLE);
	//配置GPIO
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0 | GPIO_Pin_1;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOB, &GPIO_InitStructure);
	//配置AFIO
	GPIO_EXTILineConfig(GPIO_PortSourceGPIOB, GPIO_PinSource0);
	GPIO_EXTILineConfig(GPIO_PortSourceGPIOB, GPIO_PinSource1);
	//配置EXTI
	EXTI_InitTypeDef EXTI_InitStructure;
	EXTI_InitStructure.EXTI_Line = EXTI_Line0 | EXTI_Line1;
	EXTI_InitStructure.EXTI_LineCmd = ENABLE;
	EXTI_InitStructure.EXTI_Mode = EXTI_Mode_Interrupt;
	EXTI_InitStructure.EXTI_Trigger = EXTI_Trigger_Falling;
	EXTI_Init(&EXTI_InitStructure);
	//配置NVIC
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
	
	NVIC_InitTypeDef NVIC_InitStructure;	
	NVIC_InitStructure.NVIC_IRQChannel = EXTI0_IRQn;
	NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
	NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 1;
	NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;
	NVIC_Init(&NVIC_InitStructure);
	
	//重复利用结构体变量初始化另一个引脚
	NVIC_InitStructure.NVIC_IRQChannel = EXTI1_IRQn;
	NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
	NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 2;
	NVIC_InitStructure.NVIC_IRQChannelSubPriority = 2;
	NVIC_Init(&NVIC_InitStructure);
}

//返回Encoder_Count自上次清零前的变化量
//该操作的好处是也可以通过每隔一段时间调用检测转速
int16_t Encoder_Get(void)
{
	int16_t Temp;
	Temp = Encoder_Count;
	Encoder_Count = 0;
	return Temp;
}

//中断函数由第一个下降沿触发，再判断另一个引脚是否产生第二个下降沿
//旋转方向自行指定
void EXTI0_IRQHandler(void)
{
	if(GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_1) == 0)
	{
		Encoder_Count --;
	}
	EXTI_ClearITPendingBit(EXTI_Line0);
}

void EXTI1_IRQHandler(void)
{
	if(GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_0) == 0)
	{
		Encoder_Count ++;
	}
	EXTI_ClearITPendingBit(EXTI_Line1);
}
```

## 主程序

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"
#include "Encoder.h"

int16_t Num;

int main(void)
{
	OLED_Init();
	Encoder_Init();
	OLED_ShowString(1, 1, "Num:");
	while(1)
	{
		Num += Encoder_Get();
		OLED_ShowSignedNum(1, 5, Num, 5);
	}
}
```
