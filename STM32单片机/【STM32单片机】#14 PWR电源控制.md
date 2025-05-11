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

# PWR简介

- PWR（Power Control）电源控制
- PWR负责管理STM32内部的电源供电部分，可以实现可编程电压监测器和低功耗模式的功能。
- 可编程电压监测器（PVD）可以监控VDD电源电压，当VDD下降到PVD阈值以下或上升到PVD阈值之上时，PVD会触发中断，用于执行紧急关闭任务（非本节重点）。
- 低功耗模式包括睡眠模式（Sleep）、停机模式（Stop）和待机模式（Standby），可在系统空闲时降低STM32的功耗，延长设备使用时间。
- 主频和电流消耗大致呈正比，降低主频也能显著降低功耗。

## 电源框图

![](https://i-blog.csdnimg.cn/direct/f67cda4dc4d649e1a1431d4e88c11186.png)


上图为STM32供电方案，可分为模拟部分供电（VDDA）、数字部分供电（VDD+1.8V）和后备供电。各供电区域负责的设备已列出。

VDDA供电区域主要负责模拟部分的供电，供电正极为VDDA，负极为VSSA。其中AD转换器还有参考电压供电引脚VREF+和VREF-，在引脚多的芯片型号会单独引出，而在F103C8T6中已经在内部分别接到了VDDA和VSSA。

中间部分分为VDD供电区域和1.8V供电区域，VDD经电压调节器降压到1.8V提供给1.8V供电区域。后备供电区域中，低电压检测器控制开关，在VDD有电时由VDD供电，VDD没电时由VBAT供电。

## 电压监测（了解）

### 上电复位和掉电复位

![](https://i-blog.csdnimg.cn/direct/360c3240993e44eaae291b15c1d301e1.png)


当VDD或VDDA电压过低时，STM32内部电路直接产生复位。电压小于下限PDR时复位，大于上限POR时解除复位，中间有40mV的迟滞电压防抖，复位信号Reset低电平有效。POR、PDR和复位滞后时间均可在芯片数据手册查询。

### 可编程电压监测器

![](https://i-blog.csdnimg.cn/direct/225a6355cc9944d297387bf1929595e4.png)


可编程电压监测器的工作流程与上电/掉电复位类似，但阈值电压可以手动调节，调节范围可在芯片数据手册查询，一般高于上电/掉电复位电压。PVD输出在电压过低时为1，电压正常时为0，可在上升/下降沿申请外部中断提醒程序进行适当处理。

## 低功耗模式

![](https://i-blog.csdnimg.cn/direct/17226566e9ad4be5ad3f916feda88923.png)


上表对三种低功耗模式进行了详细的对比。其中第二列为进入对应模式所需的配置，第三列为进入对应模式后的唤醒方法，后三列为进入对应模式后关闭的电路（电压调节器相当于1.8V供电区域的电源）。三种模式从上到下，关闭的电路越来越多，也越来越省电，越来越难唤醒。对于具体的省电效果，正常模式和睡眠模式电流消耗（数量级）均在10mA，停机模式为10$\mu$A，待机模式为1$\mu$A。

### 模式选择

执行WFI（Wait For Interrupt）或WFE（Wait For Event）指令后，STM32进入低功耗模式。在触发低功耗模式之前，配置寄存器进行模式选择的方式如下图所示，其中寄存器的每个位已由库函数封装好，无需自行配置。

![](https://i-blog.csdnimg.cn/direct/e4a12fa1a4e443fbbcb62dade4347fc0.png)


### 睡眠模式

- 执行完WFI/WFE指令后，STM32进入睡眠模式，程序暂停运行，唤醒后程序从暂停的地方继续运行。
- SLEEPONEXIT位决定STM32执行完WFI或WFE后，是立刻进入睡眠，还是等STM32从最低优先级的中断处理程序中退出时进入睡眠。
- 在睡眠模式下，所有的I/O引脚都保持它们在运行模式时的状态。
- WFI指令进入睡眠模式，可被任意一个NVIC响应的中断唤醒。
- WFE指令进入睡眠模式，可被唤醒事件唤醒。

### 停机模式

- 执行完WFI/WFE指令后，STM32进入停机模式，程序暂停运行，唤醒后程序从暂停的地方继续运行。
- 1.8V供电区域的所有时钟都被停止，PLL、HSI和HSE被禁止，SRAM和寄存器内容被保留下来。
- 在停机模式下，所有的I/O引脚都保持它们在运行模式时的状态。
- 当一个中断或唤醒事件导致退出停机模式时，HSI被选为系统时钟，因此唤醒后需重新启动HSE配置主频为72MHz。
- 当电压调节器处于低功耗模式下，系统从停机模式退出时，会有一段额外的启动延时。
- WFI指令进入停机模式，可被任意一个EXTI中断唤醒。
- WFE指令进入停机模式，可被任意一个EXTI事件唤醒。

### 待机模式

- 执行完WFI/WFE指令后，STM32进入待机模式，唤醒后程序从头开始运行。
- 整个1.8V供电区域被断电，PLL、HSI和HSE也被断电，SRAM和寄存器内容丢失，只有备份的寄存器和待机电路维持供电。
- 在待机模式下，所有的I/O引脚变为高阻态（浮空输入）。
- WKUP引脚的上升沿、RTC闹钟事件的上升沿、NRST引脚上外部复位、IWDG复位退出待机模式。

# 函数详解

## PWR_DeInit函数

**简介**：恢复缺省配置。

**参数**：void

## PWR_PVDCmd函数

**简介**：使能可编程电压监测器。

**参数**：使能/失能

## PWR_PVDLevelConfig函数

**简介**：配置PVD阈值。

**参数**：阈值

	PWR_PVDLevel_2V2/.../9（2.2~2.9V）

## PWR_WakeUpPinCmd函数

**简介**：使能WKUP引脚（PA0），配合待机模式的WKUP上升沿唤醒。

**参数**：使能/失能

## PWR_EnterSTOPMode函数

**简介**：进入停机模式。

**参数一**：电压调节器状态

	PWR_Regulator_ON（开启）
	PWR_Regulator_LowPower（低功耗）

**参数二**：唤醒方式

	PWR_STOPEntry_WFI/WFE（中断/事件）

## PWR_EnterSTANDBYMode函数

**简介**：进入待机模式。

**参数**：void

## 标志位函数

**PWR_GetFlagStatus函数**

**PWR_ClearFlag函数**

**参数**：PWR标志位

	PWR_FLAG_WU/SB/PWDO（唤醒/待机/电压监测器输出）

# 实验30 修改主频

## 接线图

![](https://i-blog.csdnimg.cn/direct/76f0ae4f02b14abbbf691748bee52b10.jpeg)


## system_stm32f10x文件

system文件提供了两个外部可调用的函数和一个外部可调用的变量：

- **SystemInit函数**：配置时钟树，复位后在启动文件中自动调用。
- **SystemCoreClock变量**：主频频率。
- **SystemCoreClockUpdate函数**：更新主频频率。

源文件system_stm32f10x.c中如下部分可更改主频的宏定义，解除对应的注释即可选择想要的系统主频。其中else之前的部分为VL超值系列可选主频。当前主频为72MHz。

```c
#if defined (STM32F10X_LD_VL) || (defined STM32F10X_MD_VL) || (defined STM32F10X_HD_VL)
/* #define SYSCLK_FREQ_HSE    HSE_VALUE */
 #define SYSCLK_FREQ_24MHz  24000000
#else
/* #define SYSCLK_FREQ_HSE    HSE_VALUE */
/* #define SYSCLK_FREQ_24MHz  24000000 */ 
/* #define SYSCLK_FREQ_36MHz  36000000 */
/* #define SYSCLK_FREQ_48MHz  48000000 */
/* #define SYSCLK_FREQ_56MHz  56000000 */
#define SYSCLK_FREQ_72MHz  72000000
#endif
```

若文件为只读状态，可以在文件资源管理器中右键对应文件更改属性，取消勾选只读。

SystemInit()首先启动HSI并恢复缺省配置，随后调用SetSysClock()函数会根据不同的宏定义执行相应的SetSysClockToxx()函数完成相应的时钟配置。

## 主程序

以下代码显示主频，并以72MHz下的1s为周期闪烁Running字符串。

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"

int main(void)
{
	OLED_Init();
	
	OLED_ShowString(1, 1, "SYSCLK:");
	OLED_ShowNum(1, 8, SystemCoreClock, 8);
	
	while(1)
	{
		OLED_ShowString(2, 1, "Running");
		Delay_ms(500);
		OLED_ShowString(2, 1, "       ");
		Delay_ms(500);
	}
}
```

此时Running以现实的1s为周期闪烁。

现通过宏定义修改主频为36MHz：

```c
#if defined (STM32F10X_LD_VL) || (defined STM32F10X_MD_VL) || (defined STM32F10X_HD_VL)
/* #define SYSCLK_FREQ_HSE    HSE_VALUE */
 #define SYSCLK_FREQ_24MHz  24000000
#else
/* #define SYSCLK_FREQ_HSE    HSE_VALUE */
/* #define SYSCLK_FREQ_24MHz  24000000 */ 
#define SYSCLK_FREQ_36MHz  36000000
/* #define SYSCLK_FREQ_48MHz  48000000 */
/* #define SYSCLK_FREQ_56MHz  56000000 */
/* #define SYSCLK_FREQ_72MHz  72000000 */
#endif
```

此时Running闪烁周期延长一倍。

由于修改主频会要求很多涉及精准计时的计算做好匹配工作，一般不建议采用。

# 实验31 睡眠模式+串口发送+接收

## 接线图

![](https://i-blog.csdnimg.cn/direct/936ee892a0ad43bfaf8a7c2cef2d2020.jpeg)


## 主程序

在实验21 串口发送+接收的基础上修改。

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"
#include "Serial.h"

uint8_t RxData;

int main(void)
{
	OLED_Init();
	Serial_Init();
	OLED_ShowString(1, 1, "RxDara:");

	while(1)
	{
		if (Serial_GetRxFlag())
		{
			RxData = Serial_GetRxData();
			Serial_SendByte(RxData);
			OLED_ShowHexNum(1, 8, RxData, 2);
		}
		//监测低功耗状态
		OLED_ShowString(2, 1, "Running");
		Delay_ms(100);
		OLED_ShowString(2, 1, "       ");
		Delay_ms(100);
		//默认配置开启睡眠模式
		__WFI();
	}
}
```

此时只有当我们向串口发送数据时，Running才会闪烁一次，并回传一次数据。

程序运行时，第一次循环即进入睡眠状态，程序停止在WFI指令，CPU睡眠，但各个外设例如USART仍处于工作状态。USART外设在收到数据时产生中断唤醒CPU，程序在暂停的地方继续运行，先进入中断服务函数，再回到主循环执行一轮循环进入下一次睡眠，如此循环往复。

# 实验32 停机模式+对射式红外传感器计次

## 接线图

![](https://i-blog.csdnimg.cn/direct/3cfa97333ad44fecb419191c4a4ed334.jpeg)


## 主程序

在实验6 对射式红外传感器计次的基础上修改。

原驱动程序中博主采用了延时100ms消抖，经实测中断处理时间过长会导致唤醒失败，因此将中断服务函数修改如下：

```c
//中断函数名称在启动文件startup_stm32f10x_md.s中规定
void EXTI15_10_IRQHandler(void)
{
	//检查中断挂起标志位
	if(EXTI_GetITStatus(EXTI_Line14) == SET)
	{
		//再次判断引脚电平
		if (GPIO_ReadInputDataBit(GPIOB, GPIO_Pin_14) == 0)
		{
			CountSensor_Count ++;
		}
		EXTI_ClearITPendingBit(EXTI_Line14);
	}
}
```

主程序修改如下：

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"
#include "CountSensor.h"

int main(void)
{
	OLED_Init();
	CountSensor_Init();
	
	//开启PWR时钟
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR, ENABLE);
	
	OLED_ShowString(1, 1, "Count:");
	while(1)
	{
		OLED_ShowNum(1, 7, CountSensor_Get(), 5);
		
		//监测低功耗状态
		OLED_ShowString(2, 1, "Running");
		Delay_ms(100);
		OLED_ShowString(2, 1, "       ");
		Delay_ms(100);
		
		//进入停机模式（电压调节器开启，中断唤醒）
		PWR_EnterSTOPMode(PWR_Regulator_ON, PWR_STOPEntry_WFI);
	}
}
```

此时每遮挡一次对射式红外传感器，计数加一，Running闪烁一次。有时闪烁两次是因为信号抖动导致遮挡和移开均产生中断信号。

除此之外，复位后Running第一次闪烁时间短暂，后续遮挡Running闪烁变慢，这是由于前文介绍的停机模式唤醒后HSI被选为系统时钟，只需在进入停机模式之后加上$\texttt{SystemInit();}$重新配置时钟为HSE即可。

# 实验33 待机模式+实时时钟

## 接线图

![](https://i-blog.csdnimg.cn/direct/cd7df354e5f24ac1996f6e3f18f0d919.jpeg)


## 主程序

在实验29 实时时钟的基础上修改。

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"
#include "MyRTC.h"

int main(void)
{
	OLED_Init();
	MyRTC_Init();
	
	//开启PWR时钟
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR, ENABLE);
	
	//更改显示布局
	//计数值
	OLED_ShowString(1, 1, "CNT :");
	//闹钟值
	OLED_ShowString(2, 1, "ALR :");
	//闹钟标志位
	OLED_ShowString(3, 1, "ALRF:");
	
	//设置闹钟值为复位10秒后
	//闹钟寄存器只写不读，先定义变量存储闹钟值
	uint32_t Alarm = RTC_GetCounter() + 10;
	RTC_SetAlarm(Alarm);
	OLED_ShowNum(2, 6, Alarm, 10);

	while(1)
	{
		OLED_ShowNum(1, 6, RTC_GetCounter(), 10);
		//显示闹钟标志位
		OLED_ShowNum(3, 6, RTC_GetFlagStatus(RTC_FLAG_ALR), 1);
		
		//监测低功耗状态
		OLED_ShowString(4, 1, "Running");
		Delay_ms(100);
		OLED_ShowString(4, 1, "       ");
		Delay_ms(100);
		
		//待机前关闭所有外部耗电电路以达到省电目的
		OLED_Clear();
		
		PWR_EnterSTANDBYMode();
	}
}
```

待机模式唤醒后程序从头开始运行，因此该程序现象为复位后OLED闪烁一次，十秒之后触发闹钟事件再次闪烁，往后每隔十秒闪烁一次。

在while循环之前加上$\texttt{PWR\_WakeUpPinCmd(ENABLE);}$可实现WKUP引脚（PA0）唤醒，此时每次将该引脚用跳线接到高电平即可观察到OLED闪烁。
