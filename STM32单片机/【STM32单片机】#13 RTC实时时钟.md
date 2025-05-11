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

# Unix时间戳

- Unix时间戳定义为从UTC/GMT的1970年1月1日0时0分0秒开始所经过的秒数，不考虑闰秒。
- 时间戳存储在一个秒计数器中，秒计数器为32位/64位的整型变量。只使用秒而不采取进位的优点是硬件电路简单，仅需秒寄存器和一个变量，且便于计算时间间隔。
- 世界上所有时区的秒计数器相同，不同时区通过添加偏移来得到当地时间。
- 32位（有符号）时间戳将在2038年1月19日溢出，现今的电子设备基本采用64位时间戳，可以运作上千亿年。

![](https://i-blog.csdnimg.cn/direct/1f29c1d1d5024736811fdc0a35262123.png)


## UTC/GMT

- GMT（Greenwich Mean Time）格林尼治标准时间是一种以地球自转为基础的时间计量系统，它将地球自转一周的时间间隔等分为24小时，以此确定计时标准。但由于地球自转周期不固定，GMT定义的时间基准也将不断变化，不利于科学研究，因此UTC作为新的时间系统被提出。
- UTC（Universal Time Coordinated）协调世界时是一种以原子钟为基础的时间计量系统，它规定铯133原子基态的两个超精细能级间在零磁场下跃迁辐射9192631770周所持续的时间为1秒。当原子钟计时一天的时间与地球自转一周的时间相差超过0.9秒时，UTC会执行闰秒来保证其计时与地球自转协调一致。
- 日常生活中大多不会追求极致的严谨，GMT和UTC可视作相同的时间系统。

## 时间戳转换

C语言的time.h模块提供了时间获取和时间戳转换的相关函数，可以方便地进行秒计数器、日期时间和字符串之间的转换。本节使用表中localtime和mktime函数。

![](https://i-blog.csdnimg.cn/direct/ab5cd2873ed240aaa206f7c1c202cec5.png)


![](https://i-blog.csdnimg.cn/direct/233e031845a146a8beefaba693f6753c.png)


其中time_t默认为有符号64位整型，char\*指向表示时间的字符串，struct tm结构体成员如下：

```c
struct tm {
   int tm_sec;         /* 秒，范围从 0 到 59        */
   int tm_min;         /* 分，范围从 0 到 59        */
   int tm_hour;        /* 小时，范围从 0 到 23        */
   int tm_mday;        /* 一月中的第几天，范围从 1 到 31    */
   int tm_mon;         /* 月，范围从 0 到 11        */
   int tm_year;        /* 自 1900 年起的年数        */
   int tm_wday;        /* 一周中的第几天，范围从 0 到 6    */
   int tm_yday;        /* 一年中的第几天，范围从 0 到 365    */
   int tm_isdst;       /* 夏令时                */
};
```

# BKP简介

- BKP（Backup Registers）备份寄存器
- BKP可用于存储用户应用程序数据。当VDD（2.0\~3.6V）电源被切断，它们仍由VBAT（1.8\~3.6V）维持供电。当系统在待机模式下被唤醒，或系统/电源复位时，它们也不会被复位。
- 以下是BKP的主要功能，其中后两个与RTC关联：
	- TAMPER引脚（PC13）产生的侵入事件将所有备份寄存器内容清除。
	- RTC引脚（PC13）输出RTC校准时钟、RTC闹钟脉冲或者秒脉冲。外部设备测量RTC校准脉冲可以对其内部RTC微小误差进行校准，闹钟脉冲和秒脉冲可以为别的设备提供信号。
	- 存储RTC时钟校准寄存器。
- 用户数据存储容量：20字节（中小容量）/84字节（大容量和互联型）

## BKP基本结构

![](https://i-blog.csdnimg.cn/direct/ea68c1aaa77b49b1a4aeb647c8a925a7.png)


图中橙色部分为后备区域，包括BKP和RTC相关电路。后备区域在VDD主电源掉电时，仍然可以由VBAT的备用电池供电；当VDD主电源上电时，后备区域供电会由VBAT切换到VDD。

BKP的数据寄存器用于存储数据，每个DR可存储两个字节，中小容量有10个DR，大容量和互联型有42个DR。侵入检测从TAMPER引脚引入一个检测信号，当TAMPER产生上升/下降沿时，BKP所有内容被清除以保证安全。时钟输出将RTC相关时钟从RTC引脚输出供外部使用，输出校准时钟时配合RTC时钟校准寄存器可以对RTC的误差进行校准。

# RTC简介

- RTC（Real Time Clock）实时时钟
- RTC是一个独立的定时器，可为系统提供时钟和日历功能。
- RTC和时钟配置系统处于后备区域，系统复位时数据不清零，VDD断电后可借助VBAT供电继续走时。
- 32位可编程计数器，可对应Unix时间戳的秒计数器。
- 20位可编程预分频器，可适配不同频率的输入时钟，分频为1Hz可用于驱动秒计数器。
- 可选择三种RTC时钟源：
	- HSE时钟（高速外部时钟信号，8MHz）除以128
	- LSE振荡器时钟（低速外部时钟信号，通常为32.768KHz）
	- LSI振荡器时钟（低速内部时钟信号，40KHz）
	- 最常用的时钟源为LSE，32.768KHz经$2^{15}$分频即可得到1Hz，且只有LSE可以通过VBAT备用电池供电。

## RTC框图

![](https://i-blog.csdnimg.cn/direct/452eb08270084797bb8af47f73bb63ae.png)


框图左侧为核心的分频计数计时部分，右侧为中断输出使能和NVIC部分，上方为APB1总线读写部分，下方为PWR关联部分，灰色背景部分均处于后备区域。睡眠、停机、待机等低功耗相关内容将在下一节PWR部分讲解。

输入时钟为RTCCLK，由于可选时钟源的频率各不相同，且远大于所需的1Hz秒计数频率，因此RTCCLK需要首先经过RTC预分频器进行分频。分频器由重装载寄存器RTC_PRL和余数寄存器RTC_DIV（自减计数器）组成，本质上和定时器中的重装值ARR和计数器CNT作用相同，计几个数溢出一次即为几分频，由于计数器包含0，因此分频系数为重装值加一。

32位可编程计数器RTC_CNT为计时最核心的部分，可视为Unix时间戳的秒计数器。RTC_ALR为32位闹钟寄存器，可在ALR写入一个秒数设定闹钟，当CNT的值与ALR设定值相等时产生RTC_Alarm信号通往中断系统，或使STM32退出待机模式。其余两个中断信号，秒中断RTC_Second每秒触发，溢出中断RTC_Overflow在32位CNT计满溢出时触发。RTC_CR中，F后缀为中断标志位，IE后缀为中断使能，三个中断信号通过或门前往NVIC中断控制器。

上方APB1总线和APB1接口为程序读写寄存器的地方。下方闹钟信号和WKUP引脚（PA0）均可唤醒STM32，将在下一节学习。

## RTC基本结构

除去多余内容，本实验所用到RTC结构如下：

![](https://i-blog.csdnimg.cn/direct/04e5681cb03d40ad938f02b765e9109e.png)


## 硬件电路

![](https://i-blog.csdnimg.cn/direct/beb20160a2254073b2bed4d6fd0cc35e.png)


为了配合STM32的RTC，需要连接备用电池供电和外部低速晶振两个外部电路。本节实验最小系统板上已包含外部低速晶振，备用电池使用STLINK的3.3V供电。

备用电池供电中，简单连接直接使用3V电池，负极和系统共地，正极连接VBAT引脚；芯片参考手册的推荐连接同时使用电池和3.3V主电源通过二极管（防止电流倒灌）向VBAT供电，同时使用滤波电容。实验使用简单连接即可，板子绘制和产品设计使用推荐连接更保险。

根据参考手册，外部低速晶振中，X1为32.768KHz晶振，两端分别接在OSC32两个引脚上，并再各自接一个起振电容到GND。

## RTC操作注意事项

- 执行以下操作将使能对BKP和RTC的访问：
	- 设置RCC_APB1ENR的PWREN和BKPEN，使能PWR和BKP时钟（RTC无单独开启时钟选项）
	- 设置PWR_CR的DBP，使能对BKP和RTC的访问
- 若在读取RTC寄存器时，RTC的APB1接口曾经处于禁止状态，则软件首先必须等待RTC_CRL寄存器中的RSF位（寄存器同步标志）被硬件置1。这是由于RTC由RTCCLK（32.768KHz）驱动，而读取RTC寄存器的APB1总线由PCLK1（36MHz）驱动，存在时钟不同步问题，在APB1刚开启时立刻读取RTC寄存器，有可能RTC寄存器尚未更新到APB1总线上。
- 必须设置RTC_CRL寄存器中的CNF位，使RTC进入配置模式后，才能写入RTC_PRL、RTC_CNT、RTC_ALR寄存器。该操作在每个写寄存器的库函数中自动配置。
- 对RTC任何寄存器的写操作，都必须在前一次写操作结束后进行。可以通过查询RTC_CR寄存器中的RTOFF状态位，判断RTC寄存器是否处于更新中。仅当RTOFF状态位是1时，才可以写入RTC寄存器。这也是由于时钟不同步导致APB1写入的值无法立刻更新到RTC寄存器中。

# 函数详解

## BKP库函数

### BKP_DeInit函数

**简介**：恢复缺省配置（手动清空数据寄存器）。

**参数**：void

### BKP_TamperPinLevelConfig函数

**简介**：配置TAMPER引脚有效电平。

**参数**：高/低电平

	BKP_TamperPinLevel_High/Low

### BKP_TamperPinCmd函数

**简介**：侵入检测使能。

**参数**：使能/失能

### BKP_ITConfig函数

**简介**：BKP侵入中断配置。

**参数**：使能/失能

### BKP_RTCOutputConfig函数

**简介**：RTC引脚输出配置。

**参数**：输出信号

	BKP_RTCOutputSource_None（无）
	BKP_RTCOutputSource_CalibClock（校准时钟）
	BKP_RTCOutputSource_Alarm（闹钟脉冲）
	BKP_RTCOutputSource_Second（秒脉冲）

### BKP_SetRTCCalibrationValue函数

**简介**：设置RTC校准值（写入RTC校准寄存器）。

**参数**：0\~0x7F校准值。

### BKP_WriteBackupRegister函数

**简介**：写备份寄存器。

**参数一**：数据寄存器

	BKP_DR1, ..., BKP_DR42

**参数二**：写入数据

### BKP_ReadBackupRegister函数

**简介**：读备份寄存器。

**参数**：数据寄存器

### 中断标志位函数

**BKP_GetFlagStatus函数**

**BKP_ClearFlag函数**

**BKP_GetITStatus函数**

**BKP_ClearITPendingBit函数**

**参数**：void（BKP只有侵入检测中断）

## PWR_BackupAccessCmd函数

PWR库函数在下一节讲解，本节只使用其中的该函数。

**简介**：备份寄存器访问使能。

**参数**：使能/失能

## RCC库函数

RCC库函数包含部分RTC时钟相关函数。

### RCC_LSEConfig函数

**简介**：配置外部低速时钟。

**参数**：LSE工作状态

	RCC_LSE_OFF/ON, 
	RCC_LSE_ByPass（使用OSC32_IN引脚输入的外部时钟信号）

### RCC_LSICmd函数

**简介**：配置内部低速时钟。

**参数**：使能/失能

### RCC_RTCCLKConfig函数

**简介**：配置RCCCLK。

**参数**：RCCCLK时钟源

	RCC_RTCCLKSource_LSE/LSI
	RCC_RTCCLKSource_HSE_Div128

### RCC_RTCCLKCmd函数

**简介**：使能RTCCLK。

**参数**：使能/失能

### RCC_GetFlagStatus函数

**简介**：获取标志位。

**参数**：标志位，本节只用到一个。

	RCC_FLAG_LSERDY（LSE启动完成）

## RTC库函数

### RTC_ITConfig函数

**简介**：配置中断输出。

**参数一**：RTC中断源

	RTC_IT_OW/ALE/SEC（溢出/闹钟/秒）

**参数二**：使能/失能

### RTC_EnterConfigMode函数

**简介**：进入配置模式（对应注意事项第三点）。

**参数**：void

### RTC_ExitConfigMode函数

**简介**：退出配置模式。

**参数**：void

### RTC_GetCounter函数

**简介**：读取CNT计数器（读取时钟）。

**参数**：void

### RTC_SetCounter函数

**简介**：写入CNT计数器（设置时钟）。

**参数**：计数值

### RTC_SetPrescaler函数

**简介**：写入预分频器中的重装载寄存器。

**参数**：重装值（分频系数减一）

### RTC_SetAlarm函数

**简介**：写入闹钟。

**参数**：闹钟值

### RTC_GetDivider函数

**简介**：读取预分频器中的余数寄存器（得到比秒更细致的时间）。

**参数**：void

### RTC_WaitForLastTask函数

**简介**：等待上次操作完成（对应注意事项第四点）。

**参数**：void

### RTC_WaitForSynchro函数

**简介**：等待同步（对应注意事项第二点）。

**参数**：void

### 中断标志位函数

**RTC_GetFlagStatus函数**

**RTC_ClearFlag函数**

**参数**：RTC标志位

	RTC_FLAG_RTOFF（写完成）
	RTC_FLAG_RSF（同步）
	RTC_FLAG_OW
	RTC_FLAG_ALR
	RTC_FLAG_SEC

**RTC_GetITStatus函数**

**RTC_ClearITPendingBit函数**

**参数**：RTC中断源

# 实验28 读写备份寄存器

## 接线图

![](https://i-blog.csdnimg.cn/direct/6b0c446183574ffba54854ed7e1593d7.jpeg)


## 主程序

BKP的代码较少，因此不单独进行封装。

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"
#include "Key.h"

uint8_t KeyNum;

uint16_t ArrayWrite[] = {0x1234, 0x5678};
uint16_t ArrayRead[2];

int main(void)
{
	OLED_Init();
	Key_Init();
	
	OLED_ShowString(1, 1, "W:");
	OLED_ShowString(2, 1, "R:");
	
	//开启时钟
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR, ENABLE);
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_BKP, ENABLE);
	
	//使能访问
	PWR_BackupAccessCmd(ENABLE);
	
	while(1)
	{
		KeyNum = Key_GetNum();
		if(KeyNum)
		{
			//按下按键后变换测试数据并写入
			ArrayWrite[0] ++;
			ArrayWrite[1] ++;
			
			BKP_WriteBackupRegister(BKP_DR1, ArrayWrite[0]);
			BKP_WriteBackupRegister(BKP_DR2, ArrayWrite[1]);
			
			OLED_ShowHexNum(1, 3, ArrayWrite[0], 4);
			OLED_ShowHexNum(1, 8, ArrayWrite[1], 4);
			
		}
		//读取数据
		ArrayRead[0] = BKP_ReadBackupRegister(BKP_DR1);
		ArrayRead[1] = BKP_ReadBackupRegister(BKP_DR2);
		
		OLED_ShowHexNum(2, 3, ArrayRead[0], 4);
		OLED_ShowHexNum(2, 8, ArrayRead[1], 4);
	}
}
```

# 实验29 实时时钟

接线在实验28的基础上去掉按键。

## MyRTC驱动

驱动文件放在System中。

**MyRTC.h**

```c
#ifndef __MYRTC_H
#define __MYRTC_H

extern uint16_t MyRTC_Time[];

void MyRTC_Init(void);
void MyRTC_SetTime(void);
void MyRTC_ReadTime(void);

#endif
```

**MyRTC.c**

```c
#include "stm32f10x.h"
#include <time.h>

//年月日时分秒
uint16_t MyRTC_Time[] = {2025, 5, 5, 17, 29, 00};
//提前声明设置时间函数用于初始化
void MyRTC_SetTime(void);

void MyRTC_Init(void)
{
	//开启外设时钟
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_PWR, ENABLE);
	RCC_APB1PeriphClockCmd(RCC_APB1Periph_BKP, ENABLE);
	
	//使能访问
	PWR_BackupAccessCmd(ENABLE);
	
	//利用BKP特性判断系统是否完全断电过
	//仅在完全断电后的复位操作重置时间
	//自定义一个初始化时保存在BKP中的值
	if(BKP_ReadBackupRegister(BKP_DR1) != 0xA5A5)
	{
		//开启LSE时钟
		RCC_LSEConfig(RCC_LSE_ON);
		//等待LSE准备完成
		while(!RCC_GetFlagStatus(RCC_FLAG_LSERDY));
		
		//选择RTCCLK时钟源
		RCC_RTCCLKConfig(RCC_RTCCLKSource_LSE);
		RCC_RTCCLKCmd(ENABLE);
		
		//等待同步
		RTC_WaitForSynchro();
		//等待上一次操作完成
		RTC_WaitForLastTask();
		
		//配置预分频器
		//配置函数自带进入/退出配置模式代码，无需额外调用
		RTC_SetPrescaler(32768 - 1);
		RTC_WaitForLastTask();
		//设置时间
		MyRTC_SetTime();
		
		//写入BKP
		BKP_WriteBackupRegister(BKP_DR1, 0xA5A5);
	}
	else
	{
		//不初始化时调用等待代码防止意外
		RTC_WaitForSynchro();
		RTC_WaitForLastTask();
	}
}

//设置时间
void MyRTC_SetTime(void)
{
	time_t time_cnt;
	struct tm time_date;
	
	//将数组时间信息转移到结构体
	//time.h中年份从1900开始计算，月份为0-11
	time_date.tm_year = MyRTC_Time[0] - 1900;
	time_date.tm_mon = MyRTC_Time[1] - 1;
	time_date.tm_mday = MyRTC_Time[2];
	time_date.tm_hour = MyRTC_Time[3];
	time_date.tm_min = MyRTC_Time[4];
	time_date.tm_sec = MyRTC_Time[5];
	//转换为秒计数值
	time_cnt = mktime(&time_date);
	//设置时间
	RTC_SetCounter(time_cnt);
	RTC_WaitForLastTask();
}

//读取时间，流程与设置时间相反
void MyRTC_ReadTime(void)
{
	time_t time_cnt;
	struct tm time_date;
	
	time_cnt = RTC_GetCounter();
	
	time_date = *localtime(&time_cnt);
	MyRTC_Time[0] = time_date.tm_year + 1900;
	MyRTC_Time[1] = time_date.tm_mon + 1;
	MyRTC_Time[2] = time_date.tm_mday;
	MyRTC_Time[3] = time_date.tm_hour;
	MyRTC_Time[4] = time_date.tm_min;
	MyRTC_Time[5] = time_date.tm_sec;
}
```

如果严格按照Unix时间戳的规则，需要在伦敦时间的基础上添加偏移来得到不同时区的时间。直接对struct tm结构体的tm_hour成员操作可能导致进位错误，可以通过time_cnt转换到time_date时加上8\*60\*60（北京时间UTC+8）实现偏移（逆向转换则减去相应的值）。

## 主程序

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"
#include "MyRTC.h"

int main(void)
{
	OLED_Init();
	MyRTC_Init();
	
	OLED_ShowString(1, 1, "Date:XXXX-XX-XX");
	OLED_ShowString(2, 1, "Time:XX:XX:XX");
	OLED_ShowString(3, 1, "CNT :");
	OLED_ShowString(3, 1, "DIV :");

	while(1)
	{
		MyRTC_ReadTime();
		OLED_ShowNum(1, 6, MyRTC_Time[0], 4);
		OLED_ShowNum(1, 11, MyRTC_Time[1], 2);
		OLED_ShowNum(1, 14, MyRTC_Time[2], 2);
		OLED_ShowNum(2, 6, MyRTC_Time[3], 2);
		OLED_ShowNum(2, 9, MyRTC_Time[4], 2);
		OLED_ShowNum(2, 12, MyRTC_Time[5], 2);
		OLED_ShowNum(3, 6, RTC_GetCounter(), 10);
		//通过对余数寄存器线性变换得到毫秒
		OLED_ShowNum(4, 6, (32767 - RTC_GetDivider()) / 32767.0 * 999, 10);
	}
}
```
