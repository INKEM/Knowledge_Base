> 主要参考学习资料：
>
> B站@江协科技
>
> STM32入门教程-2023版 细致讲解 中文字幕
>
> 开发资料下载链接：https://pan.baidu.com/s/1h_UjuQKDX9IpP-U1Effbsw?pwd=dspb
>
> 单片机套装：STM32F103C8T6开发板单片机C6T6核心板 实验板最小系统板套件科协

本章是根据江协科技STM32入门教程编写的最终章，STM32单片机系列的更新至此将告一段落。由于博主学习时间紧迫，因此在文章面向读者的内容呈现方面没有太周全的考虑，看到了部分读者的建议也不便再进行优化，还请多多包涵。当然更完整的学习体验仍然推荐跟随江协科技的视频通过循序渐进的实践学习，纯粹的图文难免带来枯燥和欠缺的展示效果，本文的功能更多地还是在于帮助大家整理出视频中的知识点，以便高效地回顾和复盘。

关于STM32更进一步的更新计划博主视余力进行，可能的内容有：包括江协科技后续推出的更多模块教程和博主打算自行拓展学习的知识总结的【STM32+】系列，基于HAL库的开发方式教程【STM32HAL库】系列，以及对过往更新内容全面优化、拓展原文未展开介绍内容的〖STM32〗二期系列。当然博主现在还有更多的领域知识需要学习，这些在目前也只是空中楼阁的构想，可能要经过数年才能回过头来实现。

@[TOC](目录)

# FLASH简介

- STM32F1系列的FLASH包含程序存储器、系统存储器和选项字节三个部分，通过闪存存储器接口（外设）可以对程序存储器和选项字节进行擦除和编程。
- 读写FLASH的用途
	- 利用程序存储器的剩余空间来保存掉电不丢失的用户数据。
	- 通过在程序中编程（IAP），实现程序的自我更新。
- 在线编程（In-Circuit Programming，ICP）用于更新程序存储器的全部内容，它通过JTAG、SWD协议或系统加载程序（Bootloader）下载程序。
- 在程序中编程（In-Application Programming，IAP）可以使用微控制器支持的任意一种通信接口下载程序，需要通过自行设计Bootloader程序来实现，本节不涉及。

## 闪存模块组织

![](https://i-blog.csdnimg.cn/direct/7b5122976d314317958162ceb00f8c5d.png)


闪存模块组织分为三个块。主存储器存放程序代码，是最主要、容量最大的块（表中为128K，C8T6型号只有前64K）。信息块可分为启动程序代码和用户选择字节，启动程序代码存放原厂写入的Bootloader，用于串口下载，用户选择字节（选项字节）存放独立的参数。闪存存储器接口寄存器是不属于闪存的普通外设，存储介质为SRAM，用于控制闪存的擦除和编程过程，而闪存的读取使用指针即可。

与SPI通信中的W25Q64闪存一样，主存储器进行了分页以更好地管理闪存，擦除和写保护均以页为单位，其操作特性也与W25Q64闪存相同。

## FLASH基本结构

![](https://i-blog.csdnimg.cn/direct/08b96b75fb034d9eb4563b0e7d8750b3.png)


上图为整个闪存的基本结构。闪存存储器接口（FPEC）可以对程序存储器和选项字节进行擦除和编程，但无法操作系统存储器。选项字节有很大一部分配置位用于配置主程序存储器的读写保护，还有一些配置功能在后文讲解。

# FLASH操作

## FLASH解锁

操控FPEC对程序存储器和选项字节进行擦除和编程的第一步是FLASH解锁，相当于W25Q64的写使能。

- FPEC共有三个键值：
	- RDPRT键=0x000000A5（在选项字节中解除读保护）
	- KEY1=0x45670123
	- KEY2=0xCDEF89AB
- 解锁：
	- 复位后，FPEC被保护，不能写入FLASH_CR。
	- 在FLASH_KEYR先写入KEY1，再写入KEY2，解锁。
	- 错误的操作序列会在下次复位前锁死FPEC和FLASH_CR。
- 加锁：
	- 设置FLASH_CR中的LOCK位锁住FPEC和FLASH_CR。
	- 操作完成后需尽快完成加锁，以防止意外情况。

## 使用指针访问存储器

使用指针读指定地址下的寄存器：

```c
uint16_t Data = *((__IO uint16_t *)(0x08000000));
```

0x08000000为要访问的寄存器地址，括号保证优先级（在对地址进行运算的情况下）。$\texttt{(\_\_IO uint16\_t *)}$将地址强制转换为uint16_t的指针类型，随后使用$\texttt*$将指针指向的存储器取出来。$\texttt{\_\_IO}$在STM32中有如下宏定义：

```c
#define __IO volatile
```

在数据类型前加上volatile是一种防止编译器优化的安全保障措施。编译器优化会去除无用的繁杂代码，降低代码空间，提升运行效率，但在某些地方可能会弄巧成拙。在无意义加减变量、多线程更改变量、读写与硬件相关的寄存器时都需要加上volatile。

使用指针写指定地址下的存储器：

```c
*((__IO uint16_t *)(0x08000000)) = 0x1234;
```

## 程序存储器操作

接下来展示的程序框图均已在库函数中封装好，了解流程即可。

### 程序存储器全擦除

![](https://i-blog.csdnimg.cn/direct/f3d5f5d4d0f94c18bcd8ab624cb45615.png)


首先读取LOCK位检测芯片是否上锁，如果LOCK位=1则执行解锁过程，但在库函数中无论是否上锁都会执行解锁过程。解锁后置控制寄存器的MER（Mass Erase）位为1，再置STRT（Start）位为1。STRT置一为触发条件，满足条件后芯片开始执行控制寄存器相应位的操作（此处为全擦除）。擦除过程需要时间，因此开始后程序要执行等待，判断状态寄存器的BSY（Busy）位是否为1，直到BSY位=0跳出循环。读出并验证所有页的数据为测试程序的任务，正常情况下全擦除完成默认为成功。

### 程序存储器页擦除

![](https://i-blog.csdnimg.cn/direct/ee3c9d5dad224c2f8753fe8601b7f956.png)


除了寄存器配置不同，其余程序存储器操作与全擦除流程大体相同。在页擦除中，需置控制寄存器的PER（Page Erase）位为1，然后在地址寄存器中写入一个页起始地址选择要擦除的页，最后置控制寄存器的STRT位为1。

### 程序存储器编程

![](https://i-blog.csdnimg.cn/direct/313ce6eba2ac446e9852a2c3eae22ab9.png)


STM32闪存在写入之前会检查指定地址有没有被擦除，若没有擦除则STM32不执行写入操作，除非全写入0。写入操作置控制寄存器的PG（Programming）位为1，然后通过使用指针写指定地址下的存储器的代码在指定的地址写入半字16位数据。32位数据需分两次写入，而8位数据为了不影响其他数据，最好的方法是将闪存的一页读到SRAM中写入，再擦除闪存中的一页，将SRAM中的一页数据整体写回去。写入不需要STRT位的触发条件。

## 选项字节操作（了解）

### 选项字节

![](https://i-blog.csdnimg.cn/direct/102adc2e54414943a48da22049f8e3f6.png)


地址为写入选项字节的起始地址。表中各名称含义如下：

- RDP：写入RDPRT键解除读保护，默认为解除状态。
- USER：配置硬件看门狗和进入停机/待机模式是否产生复位。
- Data0/1：用户自定义使用。
- WRP0/1/2/3：配置写保护，每一个位对应保护4个存储页（中容量）。小容量产品只需WRP0，大容量产品使用WRP3的位7保护剩余的页。

前缀n要求我们在写入存储器时，要同时在对应的带n名称处写入数据的反码，写入操作才有效。若芯片检测到二者不是反码的关系，则不执行对应功能，是一种安全保障措施。写入反码的过程由硬件自动完成，封装在库函数中。

### 选项字节编程

- 检查FLASH_SR的BSY位，确认没有其他正在进行的编程操作。
- 解锁FLASH_CR的OPTWRE位。
	- 选项字节有单独的小锁，通过在FLASH_OPTKEYR先后写入KEY1和KEY2即可解锁。
- 设置FLASH_CR的OPTPG位为1。
- 写入要编程的半字到指定的地址。
- 等待BSY位变为0。
- 读出写入的地址并验证数据。

### 选项字节擦除

- 检查FLASH_SR的BSY位，确认没有其他正在进行的编程操作。
- 解锁FLASH_CR的OPTWRE位。
- 设置FLASH_CR的OPTER位为1。
- 设置FLASH_CR的STRT位为1。
- 等待BSY位变为0。
- 读出写入的地址并验证数据。

## 补充

在一个闪存地址写入一个半字将启动一次编程，在编程过程中，任何读写闪存的操作都会使CPU暂停，直到此次闪存编程结束。如果程序中有频繁执行且对时间要求严格的中断函数需慎用内部闪存存储。

# 器件电子签名

- 电子签名存放在闪存存储器模块的系统存储区域，包含的芯片识别信息在出厂时编写，不可更改，使用指针读指定地址下的存储器可获取电子签名。
- 闪存容量寄存器：
	- 基地址：0x1FFFF7E0
	- 大小：16位
- 产品唯一身份标识寄存器：
	- 基地址：0x1FFFF7E8
	- 大小：96位

# 函数详解

## FLASH_Unlock函数

**简介**： 解锁FLASH。

**参数**：void

## FLASH_Lock函数

**简介**：加锁FLASH。

**参数**：void

## FLASH_ErasePage函数

**简介**：FLASH页擦除。

**参数**：页起始地址

**返回值**：执行状态

	FLASH_BUSY = 1（忙）
	FLASH_COMPLETE（执行完成）
	FLASH_ERROR_PG（编程错误）
	FLASH_ERROR_WRP（写保护错误）
	FLASH_TIMEOUT（等待超时）

## FLASH_EraseAllPages函数

**简介**：全擦除。

**参数**：void

**返回值**：执行状态

## FLASH_EraseOptionBytes函数

**简介**：擦除选项字节。

**参数**：void

**返回值**：执行状态

## FLASH_Program(Half)Word函数

**简介**：写入（半）字。

**参数一**：写入地址

**参数二**：（半）字数据

**返回值**：执行状态

## FLASH_GetStatus函数

**简介**：获取FLASH执行状态。

**参数**：void

**返回值**：执行状态

# 实验36 读写内部FLASH

实现功能：按下Key1，数组自增并即时更新到闪存中；按下Key2，数组清零。断电后上电，数组数据不丢失。

## 接线图

![](https://i-blog.csdnimg.cn/direct/b737b43def1c49aa8fe66a5c1a0c7714.jpeg)


## MyFLASH驱动

驱动文件放在System中。

**MyFLASH.h**

```c
#ifndef __MYFLASH_H
#define __MYFLASH_H

uint32_t MyFLASH_ReadWord(uint32_t Address);
uint16_t MyFLASH_ReadHalfWord(uint32_t Address);
uint32_t MyFLASH_ReadByte(uint32_t Address);
void MyFLASH_EraseAllPages(void);
void MyFLASH_ErasePage(uint32_t PageAddress);
void MyFLASH_ProgramWord(uint32_t Address, uint32_t Data);
void MyFLASH_ProgramHalfWord(uint32_t Address, uint16_t Data);

#endif
```

**MyFLASH.c**

```c
#include "stm32f10x.h"

//FLASH无需初始化
//读取字
uint32_t MyFLASH_ReadWord(uint32_t Address)
{
	//将指针转换为读取数据类型对应的指针类型
	return *((__IO uint32_t *)(Address));
}

//读取半字
uint16_t MyFLASH_ReadHalfWord(uint32_t Address)
{
	return *((__IO uint16_t *)(Address));
}

//读取字节
uint32_t MyFLASH_ReadByte(uint32_t Address)
{
	return *((__IO uint8_t *)(Address));
}

//全擦除
void MyFLASH_EraseAllPages(void)
{
	FLASH_Unlock();
	FLASH_EraseAllPages();
	FLASH_Lock();
}

//页擦除
void MyFLASH_ErasePage(uint32_t PageAddress)
{
	FLASH_Unlock();
	FLASH_ErasePage(PageAddress);
	FLASH_Lock();
}

//写入字
void MyFLASH_ProgramWord(uint32_t Address, uint32_t Data)
{
	FLASH_Unlock();
	FLASH_ProgramWord(Address, Data);
	FLASH_Lock();
}

//写入半字
void MyFLASH_ProgramHalfWord(uint32_t Address, uint16_t Data)
{
	FLASH_Unlock();
	FLASH_ProgramHalfWord(Address, Data);
	FLASH_Lock();
}
```

## Store驱动

由于直接对闪存进行读写不方便，效率低，且容易丢失数据，因此我们在SRAM中定义一个数组作为闪存的分身，直接对SRAM操作来读写数据。但是SRAM掉电丢失，所以我们需要闪存来配合，在SRAM每次更改时，都把数组整体备份到闪存中，在上电时，把闪存中的数据初始化加载回SRAM中。

驱动文件放在System中。

**Store.h**

```c
#ifndef __STORE_H
#define __STORE_H

extern uint16_t Store_Data[];

void Store_Init(void);
void Store_Save(void);
void Store_Clear(void);

#endif
```

**Store.c**

```c
#include "stm32f10x.h"
#include "MyFLASH.h"

//宏定义数组存储位置和数组长度
#define STORE_START_ADDRESS 0x0800FC00
#define STORE_COUNT 512

//一组数据对应闪存的一页1024字节
uint16_t Store_Data[STORE_COUNT];

void Store_Init(void)
{
	//根据自定义的0xA5A5标志位判断闪存是否存储过数据
	if(MyFLASH_ReadHalfWord(STORE_START_ADDRESS) != 0xA5A5)
	{
		//如果闪存未存储过数据则擦除后写入自定义标志位
		MyFLASH_ErasePage(STORE_START_ADDRESS);
		MyFLASH_ProgramHalfWord(STORE_START_ADDRESS, 0xA5A5);
		//闪存数据默认为0xFFFF
		//将标志位之外的数据写0，以此将SRAM数组初始化为0
		for(uint16_t i = 1;i < STORE_COUNT;i++)
		{
			//每个半字占用两个字节，因此i*2
			MyFLASH_ProgramHalfWord(STORE_START_ADDRESS + i * 2, 0x0000);
		}
	}
	//在上电时将闪存备份的数据（包括标志位）恢复到SRAM数组中
	for(uint16_t i = 0;i < STORE_COUNT;i++)
	{
		Store_Data[i] = MyFLASH_ReadHalfWord(STORE_START_ADDRESS + i * 2);
	}
}

//将SRAM数组（包括标志位）更新到闪存中
void Store_Save(void)
{
		MyFLASH_ErasePage(STORE_START_ADDRESS);
		for(uint16_t i = 0;i < STORE_COUNT;i++)
		{
			MyFLASH_ProgramHalfWord(STORE_START_ADDRESS + i * 2, Store_Data[i]);
		}
}

//数据清零（除了标志位）
void Store_Clear(void)
{
	for(uint16_t i = 1;i < STORE_COUNT;i++)
	{
		Store_Data[i] = 0x0000;
	}
	//每次更改SRAM数组均更新闪存保证数组和闪存一样
	Store_Save();
}
```

## 主程序

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"
#include "Store.h"
#include "Key.h"

uint8_t KeyNum;

int main(void)
{
	OLED_Init();
	Key_Init();
	Store_Init();
	
	//显示标志位
	OLED_ShowString(1, 1, "Flag:");
	//显示数据
	OLED_ShowString(1, 1, "Data:");
	
	while(1)
	{
		KeyNum = Key_GetNum();
		if(KeyNum == 1)
		{
			Store_Data[1] ++;
			Store_Data[2] += 2;
			Store_Data[3] += 3;
			Store_Data[4] += 4;
			Store_Save();
		}
		if(KeyNum == 2)
		{
			Store_Clear();
		}
		OLED_ShowHexNum(1, 6, Store_Data[0], 4);
		OLED_ShowHexNum(3, 1, Store_Data[1], 4);
		OLED_ShowHexNum(3, 6, Store_Data[2], 4);
		OLED_ShowHexNum(4, 1, Store_Data[3], 4);
		OLED_ShowHexNum(4, 6, Store_Data[4], 4);
	}
}
```

## 程序空间更改

为了避免程序占用空间与数组存储空间冲突，可以在Keil5中手动调整程序分配空间。

![](https://i-blog.csdnimg.cn/direct/f4192b07186a4ef1a0bd5a6f4555c242.png)


片上ROM起始地址（程序起始地址）为0x8000000（最高位0省略），空间大小Size为0x10000，默认全部64K闪存均为程序代码分配的空间。可将Size改小一些，将闪存的尾部空间保留。如果Size过小，编译时会报错，以便我们调整。右侧IRAM1为片上RAM的起始地址和大小。

通过全部编译程序，在编译输出可以得到程序大小：

![](https://i-blog.csdnimg.cn/direct/89cddfc407c949eba7a0de3eac463b6c.png)


前三个数之和为闪存占用大小，后两个数之和为SRAM占用大小。

也可在Target的map文件中查看：

![](https://i-blog.csdnimg.cn/direct/1eb4beec47834e17b7c304e52a48b567.png)


# 实验37 读取芯片ID

接线只需用到OLED模块。

本实验主要展示如何用指针读取寄存器数据。

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"

int main(void)
{
	OLED_Init();
	
	//显示闪存容量
	OLED_ShowString(1, 1, "F_SIZE:");
	OLED_ShowHexNum(1, 8, *((__IO uint16_t*)(0x1FFFF7E0)), 4);
	//显示芯片ID
	OLED_ShowString(2, 1, "U_ID:");
	//芯片ID分多个存储器存储，通过地址偏移依次读取
	//以下展示了16位读取和32位读取两种方式
	OLED_ShowHexNum(2, 6, *((__IO uint16_t*)(0x1FFFF7E8)), 4);
	OLED_ShowHexNum(2, 11, *((__IO uint16_t*)(0x1FFFF7E8 + 0x02)), 4);
	OLED_ShowHexNum(3, 1, *((__IO uint32_t*)(0x1FFFF7E8 + 0x04)), 8);
	OLED_ShowHexNum(4, 1, *((__IO uint32_t*)(0x1FFFF7E8 + 0x08)), 8);
	while(1)
	{
	}
}
```
