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
- 按键控制LED
- 光敏传感器控制蜂鸣器

新函数：
- GPIO_ReadInputDataBit
- GPIO_ReadInputData
- GPIO_ReadOutputDataBit
- GPIO_ReadOutputData

@[TOC](目录)
# 外部设备与电路

## 按键简介

![](https://i-blog.csdnimg.cn/direct/93bfb756237f45d383e97df480af7bfb.png)


- 按键：常见的输入设备，有两个引脚，按下导通，松手断开。
- 按键抖动：由于按键内部使用的事机械式弹簧片来进行通断的，所以在按下和松手的瞬间会伴随有一连串的抖动。抖动持续5\~10ms，需要对其进行过滤。

![](https://i-blog.csdnimg.cn/direct/c907bbe8f30f424f8167ccd9f22035e2.png)


## 传感器模块简介

![](https://i-blog.csdnimg.cn/direct/69e1a6e55d7d4fe8a75d3dc9417b7ffe.png)


- 传感器模块：传感器元件的电阻会随外界模拟量的变化而变化，通过与定值电阻分压即可得到模拟电压输出，再通过电压比较器进行二值化即可得到数字电压输出。

![](https://i-blog.csdnimg.cn/direct/10c2bd54bbb741678639f7cceead761f.png)


上图③为分压电路，N1为传感器元件可变电阻，R1为分压定值电阻，C2为滤波电容。N1与R1分压得到的模拟电压通过AO接到④中引脚。

①为数字输出电路，通过LM393芯片实现。LM393为电压比较器芯片，内含两个独立的电压比较器（运算放大器）电路，通过VCC和GND供电，C1为电源供电的滤波电容。电压比较器当同相输入端电压大于反向输入端电压时，输出会瞬间升高为最大值（等效于接VCC），当同相输入端电压小于反向输入端电压时，输出会瞬间降低为最小值（等效于接GND），以此对模拟电压二值化。同相输入端通过IN+接到AO模拟电压端，反相输入端通过IN-接到电位器②，通过调节电位器改变阈值电压。两个电压进行比较生成数字电压输出DO并接到④中引脚。

④中有两个指示灯电路，LED1为电源指示灯，通电即亮，LED2为DO输出指示灯，指示DO的输出电平，低电平点亮，高电平熄灭，R5上拉电阻保证默认输出为高电平。

## 硬件电路

![](https://i-blog.csdnimg.cn/direct/0897a4f2ef2e4b0896e94c5caa552d60.png)


按键电路有四种接法，①②为下接按键方式，③④为上接按键方式，通常使用下接。

以①为例，K1按下时，PA0接GND，端口为低电平。K1松开时，PA0悬空，此时引脚电压不确定，因此该接法要求PA0为上拉输入模式，使得此时端口为高电平。而②外接上拉电阻，PA0可以配置为浮空输入或上拉输入，上拉输入时PA0受双上拉作用，高电平更稳定，但接GND损耗也会更大。③④则与①②相反，K1按下为高电平，松开为低电平，且要求下拉输入模式或外接下拉电阻。

![](https://i-blog.csdnimg.cn/direct/7ad416d3b2914faabcb479b0593f8861.png)


传感器的模块化使得其电路较为简单，GND接GND，VCC接3V3，用于供电，DO数字输出端接到任一引脚，AO模拟输出端暂时不用。

# C语言补充

本节介绍在单片机中经常使用的有助于工程管理的C语言功能。

## 数据类型关键字

出于部分数据类型名字较长和单片机对数据类型的特殊使用，C语言的stdint头文件和ST库函数对其进行了重命名：

| 关键字                | stdint关键字 | ST关键字 |
| ------------------ | --------- | ----- |
| char               | int8_t    | s8    |
| unsigned char      | uint8_t   | u8    |
| short              | int16_t   | s16   |
| unsigned short     | uint16_t  | u16   |
| int                | int32_t   | s32   |
| unsigned int       | uint32_t  | u32   |
| long long          | int64_t   |       |
| unsigned long long | uint64_t  |       |

ST关键字新版本已与stdint关键字统一，但表格中旧版现在仍然保留。

## 宏定义

- 关键字：$\texttt{\#define}$
- 用途：用字符串代替数字，便于理解，防止出错；提取程序中经常出现的参数，便于快速修改。
- 定义宏定义：$\texttt{\#define ABC 12345}$
- 引用宏定义：$\texttt{int a = ABC}$

## 声明类型

- 关键字：$\texttt{typedef}$
- 用途：将较长的变量类型名重命名，便于使用。
- 定义$\texttt{typedef}$：$\texttt{typedef unsigned char uint8\_t}$
- 引用$\texttt{typedef}$：$\texttt{uint8\_t a}$

## 结构体

- 关键字：$\texttt{struct}$
- 用途：数据打包，不同类型变量的集合。
- 定义结构体变量：
	- $\texttt{struct\{char x; int y; float z;\} StructName;}$
	- 结构体变量类型较长，通常使用$\texttt{typedef}$更改变量类型名：
	- $\texttt{typedef struct\{}$
	- $\texttt{    char x;}$
	- $\texttt{    int y;}$
	- $\texttt{    float z;}$
	- $\texttt{\} StructName\_t;}$
	- $\texttt{StructName\_t StructName;}$
- 引用结构体成员：
	- $\texttt{StructName.x = 'A';}$
	- $\texttt{StructName.y = 66;}$
	- $\texttt{StructName.z = 1.23;}$
	- 或使用结构体指针名：
	- $\texttt{pStructName->x = 'A';}$
	- $\texttt{pStructName->y = 66;}$
	- $\texttt{pStructName->z = 1.23;}$

## 枚举

- 关键字：$\texttt{enum}$
- 用途：定义一个取值受限制的整型变量，用于限制变量取值范围；宏定义的集合。
- 定义枚举变量：
	- $\texttt{enum\{FALSE = 0, TRUE = 1\} EnumName;}$
	- 枚举变量类型较长，通常使用$\texttt{typedef}$更改变量类型名。
- 引用枚举成员：
	- $\texttt{EnumName = FALSE;}$
	- $\texttt{EnumName = TRUE;}$

# 实验4 按键控制LED

## 电路连接

![](https://i-blog.csdnimg.cn/direct/69eade7254a940dab46a633e28a9a2fd.jpeg)


LED接A1、A2，按键接B1、B11。

## 驱动编写

在工程文件夹中创建Hardware文件夹以存放硬件相关驱动，并配置好组和头文件路径。

### LED驱动

**LED.h**

```c
#ifndef __LED_H
#define __LED_H

void LED_Init(void);
void LED1_ON(void);
void LED1_OFF(void);
void LED1_Turn(void);
void LED2_ON(void);
void LED2_OFF(void);
void LED2_Turn(void);

#endif

```

**LED.c**

```c
#include "stm32f10x.h"

//LED端口初始化函数，参见实验1
void LED_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_1 | GPIO_Pin_2;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	
	GPIO_SetBits(GPIOA, GPIO_Pin_1 | GPIO_Pin_2);
}

//点亮LED1
void LED1_ON(void)
{
	GPIO_ResetBits(GPIOA, GPIO_Pin_1);
}

//熄灭LED1
void LED1_OFF(void)
{
	GPIO_SetBits(GPIOA, GPIO_Pin_1);
}

//切换LED1状态
void LED1_Turn(void)
{
	//GPIO_ReadInputDataBit函数读取输出寄存器的某一位
	if(GPIO_ReadInputDataBit(GPIOA, GPIO_Pin_1) == 0)
	{
		GPIO_SetBits(GPIOA, GPIO_Pin_1);
	}
	else
	{
		GPIO_ResetBits(GPIOA, GPIO_Pin_1);
	}
}

//LED2同理
void LED2_ON(void)
{
	GPIO_ResetBits(GPIOA, GPIO_Pin_2);
}

void LED2_OFF(void)
{
	GPIO_SetBits(GPIOA, GPIO_Pin_2);
}

void LED2_Turn(void)
{
	if(GPIO_ReadInputDataBit(GPIOA, GPIO_Pin_2) == 0)
	{
		GPIO_SetBits(GPIOA, GPIO_Pin_2);
	}
	else
	{
		GPIO_ResetBits(GPIOA, GPIO_Pin_2);
	}
}
```

### 按键驱动

**Key.h**

```c
#ifndef __KEY_H
#define __KEY_H

void Key_Init(void);
uint8_t Key_GetNum(void);

#endif
```

**Key.c**

```c
#include "stm32f10x.h"
#include "Delay.h"

//按键端口初始化函数，与LED同理
void Key_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStructure;
	//配置上拉输入模式
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_1 | GPIO_Pin_11;
	//速度对输入无影响，默认配置即可
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOB, &GPIO_InitStructure);
	
	GPIO_SetBits(GPIOB, GPIO_Pin_1 | GPIO_Pin_11);
}

//读取键值函数，B1端口键值为1，B11端口键值为2
uint8_t Key_GetNum(void)
{
	//存放键值变量
	uint8_t KeyNum = 0;
	//检验按键1是否按下
	if(GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_1) == 0)
	{
		//按键按下和松开都需要延时消抖
		Delay_ms(20);
		//等待按键松开
		while(GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_1) == 0);
		Delay_ms(20);
		//存储键值
		KeyNum = 1;
	}
	//检验按键2是否按下
	if(GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_11) == 0)
	{
		Delay_ms(20);
		while(GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_11) == 0);
		Delay_ms(20);
		KeyNum = 2;
	}
	//返回键值
	return KeyNum;
}
```

## 主程序

**main.c**

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "LED.h"
#include "Key.h"

//存储键值的全局变量
uint8_t KeyNum;

int main(void)
{
	//端口初始化
	LED_Init();
	Key_Init();
	while(1)
	{
		//读取键值
		KeyNum = Key_GetNum();
		//按键1按下，切换LED1
		if(KeyNum == 1)
		{
			LED1_Turn();
		}
		//按键2按下，切换LED2
		if(KeyNum == 2)
		{
			LED2_Turn();
		}
	}
}
```

## 函数详解

### GPIO_ReadInputDataBit函数

**简介**：读取输入寄存器某一位的数据。

**参数一**：GPIO外设名称

**参数二**：GPIO引脚编号

### GPIO_ReadInputData函数

**简介**：读取整个输入寄存器的数据。

**参数一**：GPIO外设名称

### GPIO_ReadOutputDataBit函数

**简介**：读取输出寄存器某一位的数据。

**参数一**：GPIO外设名称

**参数二**：GPIO引脚编号

### GPIO_ReadOutputData函数

**简介**：读取整个输出寄存器的数据。

**参数一**：GPIO外设名称

# 实验5 光敏传感器控制蜂鸣器

## 电路连接

![](https://i-blog.csdnimg.cn/direct/59426640057c4fd9a5dff3c4e35117e6.jpeg)


蜂鸣器接B12，光敏传感器接B13。

## 驱动编写

### 蜂鸣器驱动

将LED驱动复制粘贴，再将GPIO外设名称和引脚编号改为GPIOB、GPIO_Pin_12即可。

**Buzzer.h**

```c
#ifndef __BUZZER_H
#define __BUZZER_H

void Buzzer_Init(void);
void Buzzer_ON(void);
void Buzzer_OFF(void);
void Buzzer_Turn(void);

#endif
```

**Buzzer.c**

```c
#include "stm32f10x.h"

void Buzzer_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_12;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOB, &GPIO_InitStructure);
	
	GPIO_SetBits(GPIOB, GPIO_Pin_12);
}

void Buzzer_ON(void)
{
	GPIO_ResetBits(GPIOB, GPIO_Pin_12);
}

void Buzzer_OFF(void)
{
	GPIO_SetBits(GPIOB, GPIO_Pin_12);
}

void Buzzer_Turn(void)
{
	if(GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_12) == 0)
	{
		GPIO_SetBits(GPIOB, GPIO_Pin_12);
	}
	else
	{
		GPIO_ResetBits(GPIOB, GPIO_Pin_12);
	}
}
```

### 光敏传感器驱动

**LightSensor.h**

```c
#ifndef __LIGHT_SENSOR_H
#define __LIGNT_SENSOR_H

void LightSensor_Init(void);
uint8_t LightSensor_Get(void);

#endif
```

**LightSensor.c**

```c
#include "stm32f10x.h"

//移用按键初始化函数，将GPIO引脚编号改为GPIO_Pin_13即可
void LightSensor_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_13;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOB, &GPIO_InitStructure);
	
	GPIO_SetBits(GPIOB, GPIO_Pin_13);
}

//读取端口返回值函数
uint8_t LightSensor_Get(void)
{
	return GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_13);
}
```

## 主程序

**main.c**

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "Buzzer.h"
#include "LightSensor.h"

int main(void)
{
	//端口初始化
	Buzzer_Init();
	LightSensor_Init();
	while(1)
	{
		//光敏传感器遮光，蜂鸣器响
		if(LightSensor_Get() == 1)
		{
			Buzzer_ON();
		}
		else
		{
			Buzzer_OFF();
		}
	}
}
```
