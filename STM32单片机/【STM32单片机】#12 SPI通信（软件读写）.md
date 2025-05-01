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

# SPI通信

- SPI（Serial Periheral Interface）是由Motorrola公司开发的一种通用数据总线。
- 四根通信线：SCK（Serial Clock）、MOSI（Master Output Slave Input）、MISO（Master Input Slave Output）、SS（Slave Select）
- 同步全双工
- 支持总线挂载多设备（一主多从）

## 硬件电路

![](https://i-blog.csdnimg.cn/direct/73cb299711984230b8f9a14d5226ca3b.png)


- 所有SPI设备的SCK、MOSI、MISO分别连在一起。
- 主机另外引出多条SS控制线，分别接到各从机的SS引脚。SS线低电平有效，要指定从机则将相应的SS线置低电平，同一时间只能指定一个从机以防冲突。
- 输出引脚配置为推挽输出，强高低电平驱动，上升沿下降沿均迅速，传输速度远高于I2C。但从机输出仍可能存在冲突，因此SPI规定从机未被选中时需输出高阻态。
- 输入引脚配置为浮空或上拉输入。

## 移位示意图

![](https://i-blog.csdnimg.cn/direct/e2da1310761140b59a3803d535b5f55f.png)


 SPI高位先行，时钟驱动移位寄存器左移，时钟源由主机的波特率发生器提供。主机移位寄存器移出的数据通过MOSI输入到从机移位寄存器的右端，从机移位寄存器移出的数据通过MISO输入到主机移位寄存器的右端。波特率发生器的上升沿（下降沿）驱动移位寄存器向左移出一位放在引脚上；下降沿（上升沿）驱动引脚上的位采样移入到移位寄存器空出的最低位。八个时钟之后，主机和从机交换一个字节的数据，交换字节是SPI通信的基础。只发不收则不读取从机输入的数据，只收不发则可向从机发送0x00或0xFF置换从机的数据。

## SPI时序基本单元

![](https://i-blog.csdnimg.cn/direct/88652084d25b48d1be57f3754933bb3f.png)


- 起始条件：SS高→低
- 终止条件：SS低→高

![](https://i-blog.csdnimg.cn/direct/747c82130d3048ed89c8759f043eb668.png)


SPI在移位时序上提供了CPOL（时钟极性）和CPHA（时钟相位）两个可配置的位以兼容更多芯片，不同的配置组合构成了四种模式，其中模式0应用最多。

- 交换一个字节（模式0）
- CPOL=0：空闲状态时，SCK为低电平
- CPHA=0：SCK第一个边沿移入数据（第一个边沿指SCK跳出空闲状态产生的边沿，此处为上升沿），第二个边沿移出数据（第二个边沿与第一个边沿相反）

![](https://i-blog.csdnimg.cn/direct/afe1f49d25ee431b822a4066c2c4f190.png)


- 交换一个字节（模式1）
- CPOL=0：空闲状态时，SCK为低电平
- CPHA=1：SCK第一个边沿移出数据，第二个边沿移入数据

![](https://i-blog.csdnimg.cn/direct/4694ab03125d4237961a4d27ae078e60.png)


- 交换一个字节（模式2）
- CPOL=1：空闲状态时，SCK为高电平
- CPHA=0：SCK第一个边沿移入数据，第二个边沿移出数据

![](https://i-blog.csdnimg.cn/direct/96ea0451492f488f98319b92264f47d2.png)


- 交换一个字节（模式3）
- CPOL=1：空闲状态时，SCK为高电平
- CPHA=1：SCK第一个边沿移出数据，第二个边沿移入数据

## SPI时序（W25Q64芯片）

SPI通常采用指令码+读写数据的流程，从机中会定义一个指令集。SPI起始后，第一个发送给从机的数据一般为指令码指导从机完成相应的功能，随后根据指令要求继续收发数据。

![](https://i-blog.csdnimg.cn/direct/7ae62dd70f8a4f2a91c934ba4c724a57.png)


- 发送指令
- 向SS指定的设备发送指令（0x06，对应W25Q64芯片写使能指令）

![](https://i-blog.csdnimg.cn/direct/fe14cd1e0961450f8c1cbbd6597848a5.png)


- 指定地址写
- 向SS指定的设备，发送写指令（0x02），随后在指定地址（包含24位地址的三个字节Address\[23:0\]，高位先行）下，写入指定数据（Data，图中为0x55）。SPI也有地址指针，可随地址自增连续写入多个字节。

![](https://i-blog.csdnimg.cn/direct/b89794aea91c4da89743dfa09bf4fd71.png)


- 指定地址读
- 向SS指定的设备，发送读指令（0x03），随后在指定地址（Address\[23:0\]）下，读取从机数据（Data，主机发送0xFF置换从机的0x55）。可随地址自增连续读取多个字节。

# W25Q64简介

- W25Qxx系列是一种小成本、小型化、使用简单的非易失性存储器，常应用于数据存储、字库存储、固件程序存储等场景。
- 存储介质：Nor Flash（闪存）
- 时钟频率：80MHz/160MHz（Dual SPI，二重SPI）/320MHz（Quad SPI，四重SPI），后两种类似使用多个SPI数据线并行传输，了解即可。
- W25Qxx的存储容量（24位地址）为xxMbit

![](https://i-blog.csdnimg.cn/direct/e3ecc45f6793427aadcaf637a0e2055b.png)


## 硬件电路

![](https://i-blog.csdnimg.cn/direct/93ef5cb813ea43fcbf7340d6d42f47ce.png)


| 引脚       | 功能            |
| -------- | ------------- |
| VCC、GND  | 电源（2.7\~3.6V） |
| CS（SS）   | SPI片选         |
| CLK（SCK） | SPI时钟         |
| DI（MOSI） | SPI主机输出从机输入   |
| DO（MISO） | SPI主机输入从机输出   |
| WP       | 写保护           |
| HOLD     | 数据保持          |

中途需要释放总线时，可以将HOLD引脚置低电平，芯片会记住当前时序，需要继续之前的时序时再将HOLD置回高电平。

括号内的IO$_x$为使用双重SPI和四重SPI时用于充当SPI数据线的引脚，了解即可。

![](https://i-blog.csdnimg.cn/direct/e7dd8149f00047cea6ef7d16253ab343.png)


上图为W25Qxx模块原理图，J1为引出的排针，D1为电源指示灯。

## W25Q64框图

![](https://i-blog.csdnimg.cn/direct/633183dfa2e7446c8ffcd43208bd3816.png)


红色方框为存储器规划示意图，存储器分为128×64KB块，每个块分为16×4KB扇区，每个扇区分为16×256Byte页。每个块和扇区的左下角和右上角为对应的起始和终止地址，块和扇区同一行的左端和右端为页的起始和终止地址。

左下角为SPI控制逻辑（SPI Command & Control Logic），自动完成地址锁存、数据读写操作。控制逻辑左侧为与主控芯片相连的通信引脚，主控芯片通过SPI协议将指令和数据发给控制逻辑，控制逻辑自动操作内部电路完成相应功能。控制逻辑上方为状态寄存器（Status Register），与忙状态、写使能/保护有关，后文详细介绍。再上方为配合WP引脚实现写保护的写控制逻辑（Write Control Logic）。SPI控制逻辑往右有高电压生成器（High Voltage Generator），配合Flash进行编程，通过高电压刺激实现掉电不丢失；页/字节地址锁存/计数器（Page/Byte Address Latch/Counter）用于指定地址，三字节地址的前两个字节进入页地址锁存/计数器，最后一个字节进入字节地址锁存/计数器。页地址通过写保护和行解码（Write Protect Logic & Row Decode）选择操作哪一页，字节地址通过列解码和256字节页缓存（Column Decode & 256-Byte Page Buffer）进行指定地址读写操作，写入数据先存储在页缓存区以跟上SPI传输速度，等数据写完（连续写入不超过256字节），芯片再将数据从缓存区转移到Flash存储器，此时芯片进入忙状态，将状态寄存器BUSY位置一，不会响应新的读写时序。计数器使地址指针在读写后自动加一。

## Flash操作注意事项

Flash存储器为了实现掉电不丢失，同时保证存储容量足够大、成本足够低，在操作的便捷性上做出了妥协。

**写入操作**

- 写入操作前，必须先进行写使能。
- 每个数据位只能由1改写为0，不能由0改写为1。
- 为弥补上一条限制，写入数据前必须先擦除，擦除后，所有数据位变为1
- 擦除必须按最小擦除单元（一个扇区）进行，若想保留部分字节必须先读取作为备份
- 由于页缓存区存在，连续写入多字节时，最多（从起始位置）写入一页256Byte的数据，超过页尾位置的数据，会回到页首覆盖写入
- 写入操作结束后，芯片进入忙状态，不响应新的读写操作，读取到状态寄存器BUSY位为0时再进行

**读取操作**
- 直接调用读取时序，无需使能，无需额外操作，没有页的限制，读取操作结束后不会进入忙状态，但不能在忙状态时读取。

## 状态寄存器

W25Q64有两个状态寄存器，我们只关注重要的状态寄存器1的前两位。第一个位为前文介绍过的BUSY位，第二个位为写使能锁存位WEL。执行写使能指令后，WEL置一，代表芯片可以进行写入操作。写失能则使WEL清零，芯片刚上电、执行写失能、页编程、擦除指令都会触发写失能，因此任何写入操作前都需要写使能。

## 指令集

本实验涉及以下指令：

| 功能        | 指令   | 数据                                 |
| --------- | ---- | ---------------------------------- |
| 写使能       | 0x06 |                                    |
| 写失能       | 0x04 |                                    |
| 读状态寄存器1   | 0x05 | 交换读取一字节状态寄存器1配置                    |
| 页编程（写数据）  | 0x02 | 写入三字节地址和一字节数据<br>后续字节随地址自增依次存储     |
| 扇区擦除      | 0x20 | 写入三字节地址                            |
| 读JEDEC ID | 0x9F | 交换三字节ID（一字节厂商ID+两字节设备ID）           |
| 读取数据      | 0x03 | 写入三字节地址，交换读取一字节数据<br>后续字节随地址自增依次读取 |

# 实验26 软件SPI读写W25Q64

## 接线图

![](https://i-blog.csdnimg.cn/direct/0f51aa5cc6254ff993551020bdc9d7af.jpeg)


## SPI协议层

**MySPI.h**

```c
#ifndef __MYSPI_H
#define __MYSPI_H

void MySPI_Init(void);
void MySPI_Start(void);
void MySPI_Stop(void);
uint8_t MySPI_SwapByte(uint8_t ByteSend);

#endif
```

**MySPI.c**

```c
#include "stm32f10x.h"

//封装输出引脚写
void MySPI_W_SS(uint8_t BitValue)
{
	GPIO_WriteBit(GPIOA, GPIO_Pin_4, (BitAction)BitValue);
}

void MySPI_W_SCK(uint8_t BitValue)
{
	GPIO_WriteBit(GPIOA, GPIO_Pin_5, (BitAction)BitValue);
}

void MySPI_W_MOSI(uint8_t BitValue)
{
	GPIO_WriteBit(GPIOA, GPIO_Pin_7, (BitAction)BitValue);
}

//封装输入引脚读
uint8_t MySPI_R_MISO(void)
{
	return GPIO_ReadInputDataBit(GPIOA, GPIO_Pin_6);
}

void MySPI_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	//除MISO为上拉输入，其余为推挽输出
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_4 | GPIO_Pin_5 | GPIO_Pin_7;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	GPIO_SetBits(GPIOA, GPIO_Pin_4 | GPIO_Pin_5 | GPIO_Pin_7);
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_6;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	GPIO_SetBits(GPIOA, GPIO_Pin_6);
	
	//初始化引脚默认电平
	//不选中从机
	MySPI_W_SS(1);
	//模式0时钟空闲低电平
	MySPI_W_SCK(0);
}

//起始信号
void MySPI_Start(void)
{
	MySPI_W_SS(0);
}

//终止信号
void MySPI_Stop(void)
{
	MySPI_W_SS(1);
}

//交换字节（模式0）
uint8_t MySPI_SwapByte(uint8_t ByteSend)
{
	uint8_t i, ByteReceive = 0x00;
	for(i = 0;i < 8;i++)
	{
		//软件无法同时执行两条语句，因此先产生边沿再操作数据
		MySPI_W_MOSI(ByteSend & (0x80 >> i));
		MySPI_W_SCK(1);
		//上升沿移入字节
		if(MySPI_R_MISO())ByteReceive |= (0x80 >> i);
		MySPI_W_SCK(0);
		//下降沿移出字节（接循环开头）
	}
	//若使用其他模式
	//改变极性则交换时钟写1和写0语句
	//改变相位则将时钟提前到读写操作之前
	return ByteReceive;
}
```

## W25Q64驱动层

**W25Q64.h**

```c
#ifndef __W25Q64_H
#define __W25Q64_H

void W25Q64_Init(void);
void W25Q64_ReadID(uint8_t *MID, uint16_t *DID);
void W25Q64_PageProgram(uint32_t Address, uint8_t *DataArray, uint16_t Count);
void W25Q64_SectorErase(uint32_t Address);
void W25Q64_ReadData(uint32_t Address, uint8_t *DataArray, uint32_t Count);

#endif
```

**W25Q64.c**

```c
#include "stm32f10x.h"
#include "MySPI.h"
#include "W25Q64_Ins.h"

void W25Q64_Init(void)
{
	MySPI_Init();
}

//读JEDEC ID，指针实现多参数返回，MID厂商ID，DID设备ID
void W25Q64_ReadID(uint8_t *MID, uint16_t *DID)
{
	MySPI_Start();
	//指令
	MySPI_SwapByte(W25Q64_JEDEC_ID);
	//置换ID数据
	*MID = MySPI_SwapByte(W25Q64_DUMMY_BYTE);
	//DID先高八位后低八位
	*DID = MySPI_SwapByte(W25Q64_DUMMY_BYTE);
	*DID <<= 8;
	*DID |= MySPI_SwapByte(W25Q64_DUMMY_BYTE);	
	MySPI_Stop();
}

//写使能，在页编程和扇区擦除开头使用
void W25Q64_WriteEnable(void)
{
	MySPI_Start();
	MySPI_SwapByte(W25Q64_WRITE_ENABLE);
	MySPI_Stop();
}

//等待BUSY，在页编程和扇区擦除末尾使用
void W25Q64_WaitBusy(void)
{
	MySPI_Start();
	MySPI_SwapByte(W25Q64_READ_STATUS_REGISTER_1);
	//等待状态寄存器1最低位BUSY清零
	while((MySPI_SwapByte(W25Q64_DUMMY_BYTE) & 0x01) == 0x01);
	MySPI_Stop();
}

//页编程
//参数为地址（C语音无24位使用32位）、字节数组、写入字节个数（0-256使用16位）
void W25Q64_PageProgram(uint32_t Address, uint8_t *DataArray, uint16_t Count)
{
	uint16_t i;
	W25Q64_WriteEnable();
	MySPI_Start();
	MySPI_SwapByte(W25Q64_PAGE_PROGRAM);
	//地址高位先行，移位后程序自动接收低八位
	MySPI_SwapByte(Address >> 16);
	MySPI_SwapByte(Address >> 8);
	MySPI_SwapByte(Address);
	//写入数据
	for(i = 0;i < Count;i++)
	{
		MySPI_SwapByte(DataArray[i]);
	}
	MySPI_Stop();
	W25Q64_WaitBusy();
}

//扇区擦除
void W25Q64_SectorErase(uint32_t Address)
{
	W25Q64_WriteEnable();
	MySPI_Start();
	MySPI_SwapByte(W25Q64_SECTOR_ERASE_4KB);
	MySPI_SwapByte(Address >> 16);
	MySPI_SwapByte(Address >> 8);
	MySPI_SwapByte(Address);
	MySPI_Stop();
	W25Q64_WaitBusy();
}

//读取数据
//Count无限制，给最大类型
void W25Q64_ReadData(uint32_t Address, uint8_t *DataArray, uint32_t Count)
{
	uint32_t i;
	MySPI_Start();
	MySPI_SwapByte(W25Q64_READ_DATA);
	MySPI_SwapByte(Address >> 16);
	MySPI_SwapByte(Address >> 8);
	MySPI_SwapByte(Address);
	for(i = 0;i < Count;i++)
	{
		DataArray[i] = MySPI_SwapByte(W25Q64_DUMMY_BYTE);
	}
	MySPI_Stop();
}
```

## 主程序

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"
#include "W25Q64.h"

uint8_t MID;
uint16_t DID;
//写入数组
uint8_t ArrayWrite[] = {0x01, 0x02, 0x03, 0x04};
uint8_t ArrayRead[4];

int main(void)
{
	OLED_Init();
	W25Q64_Init();
	
	OLED_ShowString(1, 1, "MID:   DID:");
	OLED_ShowString(2, 1, "W:");
	OLED_ShowString(3, 1, "R:");
	
	W25Q64_ReadID(&MID, &DID);
	
	OLED_ShowHexNum(1, 5, MID, 2);
	OLED_ShowHexNum(1, 12, DID, 4);	
	//写入前擦除，地址尽量对齐扇区起始地址（低三位为000）
	W25Q64_SectorErase(0x000000);
	W25Q64_PageProgram(0x000000, ArrayWrite, 4);
	W25Q64_ReadData(0x000000, ArrayRead, 4);
	
	OLED_ShowHexNum(2, 3, ArrayWrite[0], 2);
	OLED_ShowHexNum(2, 6, ArrayWrite[1], 2);
	OLED_ShowHexNum(2, 9, ArrayWrite[2], 2);
	OLED_ShowHexNum(2, 12, ArrayWrite[3], 2);
	OLED_ShowHexNum(3, 3, ArrayRead[0], 2);
	OLED_ShowHexNum(3, 6, ArrayRead[1], 2);
	OLED_ShowHexNum(3, 9, ArrayRead[2], 2);
	OLED_ShowHexNum(3, 12, ArrayRead[3], 2);
	while(1)
	{
	}
}
```
