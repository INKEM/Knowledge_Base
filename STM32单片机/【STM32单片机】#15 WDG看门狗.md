> 主要参考学习资料：
>
> B站@江协科技
>
> STM32入门教程-2023版 细致讲解 中文字幕
>
> 开发资料下载链接：https://pan.baidu.com/s/1h_UjuQKDX9IpP-U1Effbsw?pwd=dspb
>
> 单片机套装：STM32F103C8T6开发板单片机C6T6核心板 实验板最小系统板套件科协

@[TOC](目录)

# WDG简介

- WDG（Watchdog）看门狗
- 看门狗可以监控程序的运行状态，当程序因为设计漏洞、硬件故障、电磁干扰等原因，出现卡死或跑飞现象时，看门狗能及时复位程序，避免程序陷入长时间的罢工状态，保证系统的可靠性和安全性。
- 看门狗本质上是一个定时器，当指定时间范围内，程序没有执行喂狗（重置计数器）操作时，看门狗硬件电路就自动产生复位信号。
- STM32内置两个看门狗
	- 独立看门狗（IWDG）：独立工作（时钟为LSI，独立于系统主时钟），对时间精度要求较低（只有最晚时间界限）。
	- 窗口看门狗（WWDG）：要求看门狗在精确计时窗口起作用（喂狗时间既有最早界限也有最晚界限）。

## IWDG

### IWDG框图

![](https://i-blog.csdnimg.cn/direct/17881f82a066442a866732a5dab2bad5.png)


看门狗的结构基本与定时器基本一样，主要依赖预分频器、计数器和重装载寄存器运行。40KHz的LSI时钟进入预分频器，由预分配寄存器配置最大为256的分频系数。时钟经分频后驱动递减计数器自减，当计数器自减到零后产生IWDG复位。为了避免复位，我们需要预先向重装载寄存器写入重装值，并在运行过程中向键寄存器写入特定数据控制电路喂狗。喂狗时重装值会复制到计数器中，使计数器回到重装值重新自减运行。寄存器均位于1.8V供电区，而看门狗功能位于VDD供电区，因此在停机和待机模式仍能正常工作，且可以唤醒STM32。

### IWDG键寄存器

键寄存器本质上是控制寄存器，用于控制硬件电路的工作。普通的控制寄存器通过对特定的位写零或一控制某个功能，而在可能存在干扰的情况下，一般通过向整个键寄存器写入特定值作为指令来代替控制寄存器写入一位的功能，实现较强的抗干扰能力。

![](https://i-blog.csdnimg.cn/direct/e11ae82ee06043049265927e759eb44c.png)


### IWDG超时时间

超时时间：$T_{\mathrm{IWDG}}=T_{\mathrm{LSI}}\times\mathrm{PR}预分频系数\times(\mathrm{RL}重装值+1)$

其中：$T_{\mathrm{LSI}}=1/f_{\mathrm{LSI}}$

![](https://i-blog.csdnimg.cn/direct/07c0f137a2ba4ec3a2f3f7eb9bdb725f.png)


在实际情况中，LSI的频率会在30KHz到60KHz之间变化，因此时间会有轻微浮动。

## WWDG

### WWDG框图

![](https://i-blog.csdnimg.cn/direct/6bce38f463d946aaafd525ae09c6ddae.png)


在计数部分，WWDG的时钟源为默认36MHz的PCLK1，先经过固定的4096分频，再经预分频后驱动递减计数器计数。计数器与控制寄存器合二为一，且没有重装载寄存器，计数器的重装通过直接向计数器写入重装值来完成，其中计数器的有效位只有T\[5:0\]共6位，T6为溢出标志位，在溢出时置零。也可将T\[6:0\]视为整体，当计数器自减到1000000时，下一次自减则会将T6置零。

在信号输出部分，控制寄存器的WDGA为窗口看门狗的使能位，和复位信号作用于最后的与门。复位信号的两个来源由或门连接，其中一路来自溢出标志位T6，即喂狗超出最晚时间界限。窗口值（最早时间界限）存储在配置寄存器W\[6:0\]中，当计数器还未自减到配置寄存器的值时（通过7位比较判断），喂狗操作（写入WWDG_CR）会使与之相连的与门产生复位信号通向或门。

### WWDG工作特性

- 喂狗太晚：递减计数器T\[6:0\]的值小于0x40时（此处包含T6位的1），WWDG产生复位。
- 喂狗太早：递减计数器T\[6:0\]在窗口W\[6:0\]外被重新装载时，WWDG产生复位。
- 递减计数器T\[6:0\]等于0x40时可以产生早期唤醒中断（EWI），用于重装载计数器以避免WWDG复位。该中断是即将溢出的提醒，也称为死前中断，一般用于执行保存重要数据、关闭危险设备等紧急操作，或者在非危险情况下执行喂狗阻止复位，仅给出提示信息。
- 定期写入WWDG_CR寄存器（喂狗）以避免WWDG复位。

以下时序图展示了WWDG从重装到复位的过程（刷新即喂狗）：

![](https://i-blog.csdnimg.cn/direct/56b0ac5987d0473f9fbaa015e75d6172.png)


### WWDG超时时间

- 超时（最晚）时间：$T_{\mathrm{WWDG}}=T_{\mathrm{PCLK1}}\times4096\times\mathrm{WDGTB}预分频系数\times(T[5:0]+1)$
- 窗口（最早）时间：$T_{\mathrm{WIN}}=T_{\mathrm{PCLK1}}\times4096\times\mathrm{WDGTB}预分频系数\times(T[5:0]-W[5:0])$
- 其中：$T_{\mathrm{PCLK1}}=1/f_{\mathrm{PCLK1}}$

![](https://i-blog.csdnimg.cn/direct/7ad873191e5d4de1a4b57bde24403917.png)


## IWDG和WWDG对比

![](https://i-blog.csdnimg.cn/direct/99739f6c71f44b5cac1758e7caf8a67d.png)


补充：

- 系统复位时，看门狗默认处于关闭状态，一旦被开启则无法关闭，直至下次复位。
- 看门狗中的递减计数器处于自由运行状态，即使看门狗被关闭，计数器仍然在计数，因此启用看门狗时T6位必须被设置，以防立即产生一个复位。
- LSI会随看门狗启动被强制打开，在稳定后自动为IWDG提供时钟，无需手动配置，也无法被关闭。

# 函数详解

## IWDG

### IWDG_WriteAccessCmd函数

**简介**：写使能。

**参数**：使能/失能（解除/启用写保护）

	IWDG_WriteAccess_Enable/Disable

### IWDG_SetPrescaler函数

**简介**：写预分频器。

**参数**：分频系数

	IWDG_Prescaler_4/8/16/32/64/128/256

### IWDG_SetReload函数

**简介**：写重装值。

**参数**：重装值

### IWDG_ReloadCounter函数

**简介**：喂狗。

**参数**：void

### IWDG_Enable函数

**简介**：启用IWDG。

**参数**：void

### IWDG_GetFlagStatus函数

**简介**：获取标志位。

**参数**：IWDG标志位

	IWDG_FLAG_PVU/RVU（更新预分频器/更新重装值）

## WWDG

### WWDG_DeInit函数

**简介**：恢复缺省配置。

**参数**：void

### WWDG_SetPrescaler函数

**简介**：写入预分频器。

**参数**：分频系数

	WWDG_Prescaler_1/2/4/8

### WWDG_SetWindowValue函数

**简介**：写入窗口值。

**参数**：窗口值（包括W6位置一）

### WWDG_EnableIT函数

**简介**：使能死前中断。

**参数**：void

### WWDG_SetCounter函数

**简介**：写入计数器（喂狗）。

**参数**：计数值（包括T6位置一）

### WWDG_Enable函数

**简介**：使能窗口看门狗。

**参数**：初始喂狗计数值（包括T6位置一）

### 标志位函数

**WWDG_GetFlagStatus函数**

**WWDG_ClearFlag函数**

只有死前中断一个标志位。

**参数**：void

# 实验34 独立看门狗

## 接线图

![](https://i-blog.csdnimg.cn/direct/e64507e74ec14657ac93a4b5b66e026c.jpeg)


## 主程序

看门狗代码较少，不独立封装驱动。

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"
#include "Key.h"

int main(void)
{
	OLED_Init();
	Key_Init();
	
	OLED_ShowString(1, 1, "IWDG TEST");
	
	//判断复位为系统复位还是IWDG复位，便于观察实验现象
	if(RCC_GetFlagStatus(RCC_FLAG_IWDGRST))
	{
		//IWDG复位
		OLED_ShowString(2, 1, "IWDGRST");
		Delay_ms(500);
		OLED_ShowString(2, 1, "       ");
		//该标志位不会在复位后清除，需手动清除
		RCC_ClearFlag();
	}
	else
	{
		//正常复位
		OLED_ShowString(2, 1, "RST");
		Delay_ms(500);
		OLED_ShowString(2, 1, "       ");
	}
	
	//解除写保护
	IWDG_WriteAccessCmd(IWDG_WriteAccess_Enable);
	//配置预分频和重装值
	//本实验超时时间1000ms
	IWDG_SetPrescaler(IWDG_Prescaler_16);
	IWDG_SetReload(2499);
	
	//启动前喂狗使计数器初始值为重装值
	IWDG_ReloadCounter();
	//启动看门狗自动开启写保护
	IWDG_Enable();
	
	while(1)
	{
		//按下按键时程序阻塞无法喂狗
		Key_GetNum();
		
		//正常情况每隔800ms喂狗
		IWDG_ReloadCounter();
		OLED_ShowString(4, 1, "FEED");
		Delay_ms(200);
		OLED_ShowString(4, 1, "    ");
		Delay_ms(600);
	}
}
```

# 实验35 窗口看门狗

接线不变。

## 主程序

在独立看门狗的基础上修改。

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"
#include "Key.h"

int main(void)
{
	OLED_Init();
	Key_Init();
	
	//更改OLED显示
	OLED_ShowString(1, 1, "WWDG TEST");
	
	//标志位判断改为WWDG
	if(RCC_GetFlagStatus(RCC_FLAG_WWDGRST))
	{
		OLED_ShowString(2, 1, "WWDGRST");
		Delay_ms(500);
		OLED_ShowString(2, 1, "       ");
		RCC_ClearFlag();
	}
	else
	{
		OLED_ShowString(2, 1, "RST");
		Delay_ms(500);
		OLED_ShowString(2, 1, "       ");
	}
	
	//开启时钟
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_WWDG, ENABLE);
	//配置预分频和窗口值
	//本实验超时时间50ms，窗口时间30ms
	WWDG_SetPrescaler(WWDG_Prescaler_8);
	//或运算将T6位置一
	WWDG_SetWindowValue(0x40 | 21);
	WWDG_Enable(0x40 | 54);
	
	while(1)
	{
		Key_GetNum();
		
		//每隔40ms喂狗
		//由于初始化已经喂过狗，因此喂狗需放在循环末尾防止喂狗过快
		OLED_ShowString(4, 1, "FEED");
		Delay_ms(20);
		OLED_ShowString(4, 1, "    ");
		Delay_ms(20);
		
		WWDG_SetCounter(0x40 | 54);
	}
}
```

喂狗过快情况难以通过实验操作模拟，可以在程序中缩短喂狗间隔延时观察。
