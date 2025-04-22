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
- 串口发送
- 串口发送+接收

@[TOC](目录)

# 通信接口

- 通信的目的：将一个设备的数据传送到另一个设备，扩展硬件系统。
- 通信协议：制定通信的规则，通信双方按照协议规则进行数据收发。

![](https://i-blog.csdnimg.cn/direct/495bcabd441349c79ea65375a4b88df9.png)


上表所列通信协议会在后续逐个深入分析。

**全双工**：通信双方能够同时进行双向通信，一般有发送和接收两条数据线。

**半双工**：通信双方只能分时进行双向通信，一般发送和接收共用一条数据线。

**单工**：数据只能从一个设备到另一个设备，而不能反向传输。

**同步**：拥有时钟线，接收方在时钟的指引下对数据进行采样。

**异步**：没有时钟线，双方约定采样频率，并添加帧头帧尾进行采样位置的对齐。

**单端**：引脚的高低电平是对GND的电压差，通信双方必须共地。

**差分**：靠两个差分引脚的电压差传输信号，无需GND（除部分USB协议）。使用差分信号可以极大地提高抗干扰特性，所以差分信号一般有较高的传输速度和较长的传输距离。

**点对点**：只有两个设备相互通信。

**多设备**：可以在总线上挂载多个设备，需要通过寻址确定通信对象。

# 串口协议

## 串口通信

- 串口是一种应用十分广泛的通讯接口，成本低、容易使用、通信线路简单，可实现两个设备的互相通信。
- 单片机的串口可以使单片机与单片机、单片机与电脑、单片机与各种模块互相通信，极大地扩展了单片机的应用范围，增强了单片机系统的硬件实力。

## 硬件电路

![](https://i-blog.csdnimg.cn/direct/6b41bdd93ec146b99a67a4eb6edbceae.png)


- 简单双向串口通信有两根通信线，TX为发送端，RX为接收端。
- TX与RX交叉连接。
- 当只需单向的数据传输时，可以只接一根通信线。
- 如果两个设备都有独立供电可不接VCC。
- 当电平标准不一致时，需要加电平转换芯片。

## 电平标准

电平标准是数据1和数据0的表达方式，是传输线缆中人为规定的电压与数据的对应关系。串口常用的电平标准有如下三种：

- TTL电平：+3.3V或+5V表示1，0V表示0，是单片机中最常见的电平标准。
- RS232电平：-3\~-15V表示1，+3\~+15V表示0。
- RS485电平：（差分信号）两线压差+2\~+6V表示1，-2\~-6V表示0。

本系列基于TTL电平，其他电平使用电平转换芯片即可，在软件层面没有变化。

## 串口参数及时序

![](https://i-blog.csdnimg.cn/direct/4a7ed7fb166e430aa8f59cdd3350e969.png)


图中为串口发送一个字节的格式，由串口协议规定。串口中每一个字节都装载在一个数据帧里面，每个数据帧都由起始位、数据位和停止位组成。左侧8位数据位代表一个字节的8位，右侧在数据位末尾增加了奇偶校验位，共9位数据位，其中有效载荷为前8位。

- 波特率：串口通信的速率，单位为码元/s，在二进制调制下等于比特率单位bit/s。
- 起始位：标志一个数据帧的开始，固定为低电平。串口的空闲状态为高电平，起始位产生下降沿告诉接收设备数据帧开始。
- 数据位：数据帧的有效载荷，1为高电平，0为低电平，从低位开始发送。
- 停止位：用于数据帧间隔，固定为高电平，恢复空闲状态，为下一个起始位做准备。
- 校验位：用于数据验证，根据数据位计算得来，可以判断数据传输是否正确。如果数据出错，可以选择丢弃或要求重传。

校验方式有无校验、奇校验、偶校验三种，无校验不需要校验位；若使用奇校验，前8位有偶数个1则校验位补1，反之补0，保证1的个数为奇数，接收方在接收数据后会验证数据位和校验位，若1的个数为奇数则认为没有出错，若传输干扰使1的个数变为偶数则认为传输出错；偶校验保证1的个数为偶数，方法同理。

如果两位数据同时出错，则奇偶特性不变，因此奇偶校验的检出率并不高。要求更高的检出率可以了解更为复杂的CRC校验。

**串口时序实例（波特率9600）**

0xAA，1位停止，无校验：

![](https://i-blog.csdnimg.cn/direct/322a28a06fd44b06a56683fdcf4fb877.png)


0x55，1位停止，偶校验：

![](https://i-blog.csdnimg.cn/direct/69d72eab2b694d729e30ac0a523dfff4.png)


0x55，2位停止，无校验：

![](https://i-blog.csdnimg.cn/direct/a388624812b344bdaf1d15db34c050d6.png)


# USART简介

- USART（Universal Synchronous/Asynchronous Receiver/Transmitter）通用同步/异步收发器
- 相应地，UART为异步收发器。同步功能一般是为了兼容别的协议或特殊用途设计，串口很少使用，因此两者区别不大。
- USART是STM32内部集成的硬件外设，可根据数据寄存器的一个字节数据自动生成数据帧时序从TX引脚发送，也可自动接收RX引脚的数据帧时序，拼接为一个字节数据存放在数据寄存器中。
- 自带波特率发生器（分频器），最高达4.5Mbits/s。
- 可配置数据位长度（8/9），停止位长度（0.5/1/1.5/2）。
- 可选校验位（无检验/奇校验/偶校验）。
- 支持同步模式、硬件流控制（接收方通过另接线路反馈准备信号，防止因处理慢而导致数据丢失）、DMA和智能卡（刷卡）、IrDA（红外通信）、LIN（局域网）等其他与串口相似的协议。
- STM32F103C8T6 USART资源：USART1（APB2）、USART2/3（APB1）。
## USART框图

![](https://i-blog.csdnimg.cn/direct/a8e0e76820a944869a8148c3ebbc75a2.png)


**收发工作部分**

左上角TX/RX为发送/接收引脚，其余为智能卡和IrDA协议引脚，包括其编解码模块在内都无需考虑。TX由发送移位寄存器引出，RX通向接收移位寄存器。发送/接收的字节数据存储在发送/接收数据寄存器（TDR/RDR）中，两者共用一个数据寄存器DR的地址，TDR只写，RDR只读，DR自动根据操作是读还是写进行区分。发送移位寄存器将一个字节的数据一位一位移出。

当硬件检测到数据写入TDR时，会检查当前发送移位寄存器是否有数据正在移位，若没有则将TDR的数据全部移动到发送移位寄存器准备发送，并将TXE（TX Empty）标志位置一，此时可以向TDR写入下一个数据。随后发送移位寄存器在发送器控制的驱动下向右移位，将数据一位一位第输出到TX引脚。当数据移位完成后，新的数据会再次从TDR转移到发送移位寄存器中。TDR和移位寄存器的双重缓存可以保证连续发送数据的时候，数据帧之间不会有空闲，提高工作效率。

当数据到达RX引脚时，在接收器控制的驱动下，接收移位寄存器会一位一位地读取RX电平，先放在最高位再向右移。当一个字节的数据移位完成时，该字节的数据会整体转移到RDR中，并将RXNE（RX Not Empty）标志位置一，此时可以读取RDR的数据，接收移位寄存器也能直接接收下一帧数据。

**增强部分与控制部分**

下方发送器控制控制发送移位寄存器工作，接收器控制控制接收移位寄存器工作。左侧硬件数据流控实现硬件流控制功能，nRTS（Request To Send）为输出脚，告诉对方本设备能否接收，nCTS（Clear To Send）为输入脚，接收其他设备的nRTS信号，二者交叉连接。n指低电平有效，本设备能接收数据时则RTS置低电平请求对方发送，发送时也需通过CTS判断对方能否接收。该功能不常使用，了解即可。

SCLK控制及其引脚用于产生同步的时钟信号，发送移位寄存器每移位一次，同步时钟电平就跳变一个周期。该功能用于兼容其他协议或产生自适应波特率告知接收设备所使用的波特率，不常使用，了解即可。

唤醒单元通过给串口分配一个USART地址实现串口挂载多设备，仅在接收到发送端发送的指定地址时才会唤醒开始工作，否则保持沉默。该功能不常使用，了解即可。

中断控制部分中，中断申请位为SR寄存器中的标志位，其中已经提到过的TXE和RXNE较为重要，其余了解即可。USART中断控制配置中断能否通向NVIC。

波特率发生器部分中，时钟输入为来自APB1/2分频后的$f_{PCLK1/2}$，挂载在APB2的USART1一般为72MHz，其余挂载在APB1的一般为36MHz。之后该时钟除以USARTDIV分频系数，USARTDIV分为整数部分12位和小数部分4位。分频得到的波特率会再进行16分频（因为后文介绍的数据采样需要16倍频）得到发送器时钟和接收器时钟，通向控制部分。USART_BRR中，TE置一则发送使能（发送器波特率有效），RE置一则接收使能（接收器波特率有效）。

关于图中未详细介绍的寄存器，需要时参见芯片数据手册。

## USART基本结构

以下为程序控制USART工作相关的基本结构：

![](https://i-blog.csdnimg.cn/direct/62d1e6631ce84a12a4b776b7b597a8d2.png)


## 数据采样

![](https://i-blog.csdnimg.cn/direct/7121cabdd3984cf1842ad9978d77839c.png)


当输入电路侦测到一个数据帧的起始位后，就会以波特率的频率连续采样一帧数据。同时，从起始位开始，采样位置就要对齐位的正中间。为了实现上述功能，接收电路会以波特率的16倍频率进行采样。空闲时刻采样为1，采样到0则检测为下降沿，接下来的16次采样若无噪声（全为0）则判断为起始位。但为了包容实际电路可能存在的噪声，接收电路还会检验第3/5/7次采样和第8/9/10次采样，若这两批采样各自至少有2个0则仍判断为起始位，但在2个0和1个1的情况下，接收电路会将NE（Noise Error）噪声标志位置一作为提醒。其余噪声过多的情况下，接收电路将忽略之前的数据，重新开始捕捉下降沿。

![](https://i-blog.csdnimg.cn/direct/be7c60d064884ef8a9a2eb8bb040cb3c.png)


如果检测到起始位，则第8/9/10次采样正好对应位的正中间，之后接收每个数据位时都在第8/9/10次进行采样，并按照少数服从多数原则判断该数据位的数据，以此包容噪声，若存在噪声则将NE标志位置一。

## 波特率发生器

![](https://i-blog.csdnimg.cn/direct/0eb854bae8c14ac6a5573b970fe5f503.png)


发送器和接收器的波特率由波特率寄存器BRR中的DIV确定，计算公式为

波特率=$f_{PCLK1/2}$/(16\*DIV)

# USB转串口模块

![](https://i-blog.csdnimg.cn/direct/7dad51020dfb420a96f7b3976d423848.png)


串口相关实验将通过USB转串口模块连接STM32与电脑实现。

USB端口以UD+和UD-为通信线，通过CH340芯片将USB协议转换为串口协议，输出TXD和RXD。

该模块靠USB端口的VCC+5V供电，并由左上角稳压管电路降压得到VCC+3.3V，与VCC+5V一同通过排针J1第4脚和第6脚引出。第5脚的VCC为CH340的电源输入脚，通过跳线帽与第4脚或第6脚连接，并且TTL电平与供电电平相同。

右下角为PWR电源指示灯和TXD/RXD指示灯，后者会在引脚上有数据传输时闪烁。

# 函数详解

## USART_DeInit函数

**简介**：恢复缺省配置。

**参数**：USART名称

	USART1/2/3, UART4/5

## USART_Init函数

**简介**：初始化USART。

**参数一**：USART名称

**参数二**：USART_InitStruct结构体指针

### USART_InitStruct结构体

**成员USART_BaudRate**：波特率

**成员USART_HardwareFlowControl**：硬件流控制

	USART_HardwareFlowControl_None（不使用）
	USART_HardwareFlowControl_RTS（使用RTS）
	USART_HardwareFlowControl_CTS（使用CTS）
	USART_HardwareFlowControl_RTS_CTS（都使用）

**成员USART_Mode**：发送/接收模式

	USART_Mode_Rx（接收模式）, USART_Mode_Tx（发送模式）
	若都使用则进行或运算

**成员USART_Parity**：校验位

	USART_Parity_No（无校验）
	USART_Parity_Odd（奇校验）
	USART_Parity_Even（偶校验）

**成员USART_StopBits**：停止位

	USART_StopBits_0_5, USART_StopBits_1_5
	USART_StopBits_1, USART_StopBits_2

**成员USART_WordLength**：字长（数据位长度）

	USART_WordLength_8b（无校验位）
	USART_WordLength_9b（使用校验位）

## USART_StructInit函数

**简介**：初始化USART_InitStruct结构体。

**参数**：USART_InitStruct结构体指针

## USART_Cmd函数

**简介**：USART使能。

**参数一**：USART名称

**参数二**：使能/失能

## USART_ITConfig函数

**简介**：USART中断使能。

**参数一**：USART名称

**参数二**：USART中断源

	USART_IT_TXE
	USART_IT_RXNE
	其余暂时不用

**参数三**：使能/失能

## USART_DMACmd函数

**简介**：开启USART到DMA的触发通道。

**参数一**：USART名称

**参数二**：DMA请求类型

	USART_DMAReq_Tx（发送请求）
	USART_DMAReq_Rx（接受请求）

**参数三**：使能/失能

## USART_SendData函数

**简介**：写DR寄存器（发送）。

**参数一**：USART名称

**参数二**：数据

## USART_ReceiveData函数

**简介**：读DR寄存器（接收）。

**参数**：USART名称

## 中断标志位函数

**USART_GetFlagStatus函数**

**USART_ClearFlag函数**

**参数一**：USART名称

**参数二**：USART标志位

	USART_FLAG_TXE
	USART_FLAG_RXNE
	其余暂时不用


**USART_GetITStatus函数**

**USART_ClearITPendingBit函数**

**参数一**：USART名称

**参数二**：USART中断源

# 实验20 串口发送

## 接线图

![](https://i-blog.csdnimg.cn/direct/8d6b9fc4b019416da74c02e8b3147d30.jpeg)


跳线帽接VCC和3V3，RXD和TXD分别接与USART1对应的TX引脚PA9和RX引脚PA10，GND共地。

## 编译器配置

本实验部分功能需要对编译器进行配置。

**printf函数移植**

![](https://i-blog.csdnimg.cn/direct/55085f565d46445e8a643fff15d481a7.png)


**显示汉字**

编译器代码部分输入汉字不报错方法，如下输入--no-multibyte-chars：

![](https://i-blog.csdnimg.cn/direct/a025e70233d74c619179cd1240d0c787.png)


将UTF8编码切换为兼容性更好的GB2312编码：

![](https://i-blog.csdnimg.cn/direct/979441f80bb74fdabddaf8c572ab9f21.png)


## 串口助手

验证程序需要用到资料包工具软件中的串口助手。

![](https://i-blog.csdnimg.cn/direct/848055fca4cc491d8c5288e8fe2046a8.png)


插入USB转串口模块，串口号与设备管理器中一致，其余配置与程序一致，打开串口即可接收数据。

接收区配置中，HEX模式显示16进制数据，文本模式将16进制数据按ASCII码转换为对应的字符，汉字则通过文本编码选项选择GB2312或UTF8编码。

发送区配置中，HEX模式只能发送两位一组的原始16进制数据，文本模式将发送区内容视作字符，编码为16进制数据发送。

**Serial.h**

```c
#ifndef __SERIAL_H
#define __SERIAL_H

#include <stdio.h>

void Serial_Init(void);
void Serial_SendByte(uint8_t Byte);
void Serial_SendArray(uint8_t *Array, uint16_t Length);
void Serial_SendString(char *String);
void Serial_SendNumber(uint32_t Number, uint8_t Length);
void Serial_Printf(char *format, ...);

#endif
```

**Serial.c**

```c
#include "stm32f10x.h"
//对stdio.h的printf重定向的串口输出方法所需头文件
#include <stdio.h>
//使用可变参数封装sprintf功能的串口输出方法所需头文件
#include <stdarg.h>

void Serial_Init(void)
{
	//开启时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);	
	
	//初始化TX引脚PA9
	GPIO_InitTypeDef GPIO_InitStructure;
	//复用推挽
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure); 
	
	//初始化USART
	USART_InitTypeDef USART_InitStructure;
	//9600波特率
	USART_InitStructure.USART_BaudRate = 9600;
	//无流控
	USART_InitStructure.USART_HardwareFlowControl = USART_HardwareFlowControl_None;
	//发送模式
	USART_InitStructure.USART_Mode = USART_Mode_Tx;
	//无校验
	USART_InitStructure.USART_Parity = USART_Parity_No;
	//1位停止位
	USART_InitStructure.USART_StopBits = USART_StopBits_1;
	//8位字长
	USART_InitStructure.USART_WordLength = USART_WordLength_8b;
	USART_Init(USART1, &USART_InitStructure);
	
	USART_Cmd(USART1, ENABLE);
}

//发送字节函数
void Serial_SendByte(uint8_t Byte)
{
	//发送数据
	USART_SendData(USART1, Byte);
	//等待发送数据寄存器空
	while(!USART_GetFlagStatus(USART1, USART_FLAG_TXE));
	//TXE在DR写操作自动清零，无需手动清零
}

//发送数组函数，Length指定数组长度以判断数组结束
void Serial_SendArray(uint8_t *Array, uint16_t Length)
{
	uint16_t i;
	for(i = 0; i < Length; i++)
	{
		Serial_SendByte(Array[i]);
	}
}

//发送字符串函数
void Serial_SendString(char *String)
{
	uint8_t i;
	//可以利用空字符作为结束标志位
	for(i = 0; String[i] != '\0'; i++)
	{
		Serial_SendByte(String[i]);
	}
}

//求幂函数，为发送字符串数字函数做准备
uint32_t Serial_Pow(uint32_t X, uint32_t Y)
{
	uint32_t Result = 1;
	while(Y--)
	{
		Result *= X;
	}
	return Result;
}

//发送字符串形式数字函数
void Serial_SendNumber(uint32_t Number, uint8_t Length)
{
	uint8_t i;
	for(i = 0; i < Length; i++)
	{
		//从高位到低位提取数位
		//ASCII码表中数字从0x30对应的'0'开始
		Serial_SendByte(Number / Serial_Pow(10, Length - i - 1) % 10 + '0');
	}
}

//以下为移植printf函数的三种串口输出方法

/*
一
如果想使用printf一样的格式化输出功能则可以对其重定向
printf通过其底层函数fputc以字符为单位打印到屏幕
单片机没有屏幕，重定向printf函数的底层fputc到串口
*/
int fputc(int ch, FILE *f)
{
	Serial_SendByte(ch);
	return ch;
}

//第二种方法为主程序中未封装的sprintf代码

/*
三
使用可变参数函数封装sprintf功能
C语言可变参数是较为深入的内容，这里仅展示代码
*/
void Serial_Printf(char *format, ...)
{
	char String[100];
	va_list arg;
	va_start(arg, format);
	vsprintf(String, format, arg);
	va_end(arg);
	Serial_SendString(String);
}
```

## 主程序

```c
#include "stm32f10x.h" 
#include "Delay.h"
#include "OLED.h"
#include "Serial.h"

int main(void)
{
	OLED_Init();
	Serial_Init();
	
	//不同类型的串口输出函数
	Serial_SendByte('A');
	uint8_t MyArray[] = {0x42, 0x43, 0x44, 0x45};
	Serial_SendArray(MyArray, 4);
	//通过/r/n换行
	Serial_SendString("\r\nNum1=");
	Serial_SendNumber(111, 3);
	
	//printf重定向串口输出方式
	printf("\r\nNum2=%d", 222);
	
	//重定向只能被一个串口占用
	//无需重定向实现格式化输出的sprintf方式
	char String[100];
	//sprintf将格式化内容转化到字符串变量中
	sprintf(String, "\r\nNum3=%d", 333);
	//再发送字符串
	Serial_SendString(String);
	
	//封装后的sprintf方式
	Serial_Printf("\r\nNum4=%d", 444);
	Serial_Printf("\r\n");
	while(1)
	{
	}
}
```

# 实验21 串口发送+接收

接线保持不变。

## 查询方式

仅需更改Serial_Init函数：

```c
void Serial_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);	
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure); 
	//初始化RX引脚PA10
	//串口空闲为高电平，选上拉或浮空输入
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_10;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure); 
	
	USART_InitTypeDef USART_InitStructure;
	USART_InitStructure.USART_BaudRate = 9600;
	USART_InitStructure.USART_HardwareFlowControl = USART_HardwareFlowControl_None;
	//发送/接收模式
	USART_InitStructure.USART_Mode = USART_Mode_Tx | USART_Mode_Rx;
	USART_InitStructure.USART_Parity = USART_Parity_No;
	USART_InitStructure.USART_StopBits = USART_StopBits_1;
	USART_InitStructure.USART_WordLength = USART_WordLength_8b;
	USART_Init(USART1, &USART_InitStructure);
	
	USART_Cmd(USART1, ENABLE);
}
```

主程序：

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

	while(1)
	{
		//检测是否接收到数据
		if (USART_GetFlagStatus(USART1, USART_FLAG_RXNE))
		{
			//读取数据
			RxData = USART_ReceiveData(USART1);
			OLED_ShowHexNum(1, 1, RxData, 2);
			//RXNE标志位在读取DR时自动清零，无需手动清零
		}
	}
}
```

## 中断方式

**Serial.h**

```c
#ifndef __SERIAL_H
#define __SERIAL_H

#include <stdio.h>

void Serial_Init(void);
void Serial_SendByte(uint8_t Byte);
void Serial_SendArray(uint8_t *Array, uint16_t Length);
void Serial_SendString(char *String);
void Serial_SendNumber(uint32_t Number, uint8_t Length);
void Serial_Printf(char *format, ...);
//新增函数
uint8_t Serial_GetRxFlag(void);
uint8_t Serial_GetRxData(void);

#endif
```

添加两个全局变量，更改Serial_Init函数：

```c
//在驱动中暂存接受数据和接收标志位
uint8_t Serial_RxData;
uint8_t Serial_RxFlag;

void Serial_Init(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_USART1, ENABLE);
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);	
	
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_AF_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_9;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure); 
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_IPU;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_10;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure); 
	
	USART_InitTypeDef USART_InitStructure;
	USART_InitStructure.USART_BaudRate = 9600;
	USART_InitStructure.USART_HardwareFlowControl = USART_HardwareFlowControl_None;
	USART_InitStructure.USART_Mode = USART_Mode_Tx | USART_Mode_Rx;
	USART_InitStructure.USART_Parity = USART_Parity_No;
	USART_InitStructure.USART_StopBits = USART_StopBits_1;
	USART_InitStructure.USART_WordLength = USART_WordLength_8b;
	USART_Init(USART1, &USART_InitStructure);
	
	//开启中断
	USART_ITConfig(USART1, USART_IT_RXNE, ENABLE);
	//配置NVIC
	NVIC_PriorityGroupConfig(NVIC_PriorityGroup_2);
	NVIC_InitTypeDef NVIC_InitStructure;
	//打开USART1通道
	NVIC_InitStructure.NVIC_IRQChannel = USART1_IRQn;
	NVIC_InitStructure.NVIC_IRQChannelCmd = ENABLE;
	NVIC_InitStructure.NVIC_IRQChannelPreemptionPriority = 1;
	NVIC_InitStructure.NVIC_IRQChannelSubPriority = 1;
	NVIC_Init(&NVIC_InitStructure);
	
	USART_Cmd(USART1, ENABLE);
}
```

添加新函数：

```c
//外接调用接收数据函数
uint8_t Serial_GetRxData(void)
{
	return Serial_RxData;
}

//中断函数实现读取功能
void USART1_IRQHandler(void)
{
	if(USART_GetITStatus(USART1, USART_IT_RXNE))
	{
		Serial_RxData = USART_ReceiveData(USART1);
		Serial_RxFlag = 1;
		USART_ClearITPendingBit(USART1, USART_IT_RXNE);
	}
}
```

主程序：

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
		//检测是否接收到数据
		if (Serial_GetRxFlag())
		{
			RxData = Serial_GetRxData();
			//数据回传功能
			Serial_SendByte(RxData);
			OLED_ShowHexNum(1, 8, RxData, 2);
			//RXNE标志位在读取DR时自动清零，无需手动清零
		}
	}
}
```
