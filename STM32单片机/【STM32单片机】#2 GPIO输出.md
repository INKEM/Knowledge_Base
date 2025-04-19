> 主要参考学习资料：
> 
> B站@江协科技
> 
> STM32入门教程-2023版 细致讲解 中文字幕
>
>开发资料下载链接：https://pan.baidu.com/s/1h_UjuQKDX9IpP-U1Effbsw?pwd=dspb
>
> 单片机套装：STM32F103C8T6开发板单片机C6T6核心板 实验板最小系统板套件科协


@[TOC](目录)
# GPIO简介

- GPIO（General Purpose Input Output）通用输入输出口
- 根据使用场景可配置为8种输入输出模式
- 引脚电平：0\~3.3V，部分引脚可容忍5V
- 输出模式下课控制端口输出高低电平，用以驱动LED、控制蜂鸣器、模拟通信协议输出时序等
- 输入模式下可读取端口的高低电平或电压，用于读取按键输入、外接模块电平信号输入、ADC电压采集、模拟通信协议接收数据等

## GPIO基本结构

**GPIO总体结构**

![](https://i-blog.csdnimg.cn/direct/05f78687819c48ce9ae7bf8fc52d8ee8.png)


所有GPIO挂载在APB2外设总线上，GPIO外设的名称按照GPIOA、GPIOB、GPIOC等来命名，每个GPIO外设总共有16个引脚，编号从0到15。

每个GPIO模块内主要包含寄存器和驱动器，寄存器是一段特殊的存储器，内核可以通过APB2总线对寄存器进行读写，完成输出电平和读取电平的功能。寄存器的每一位对应一个引脚，输出寄存器写1，对应的引脚输出高电平，写0输出低电平；输入寄存器读取为1，证明对应的引脚目前是高电平，读取为0就是低电平。STM32是32位单片机，因此内部的寄存器都是32位的，但每个GPIO外设只有16位端口，因此寄存器只有低16位有对应的端口，高16位空闲。

驱动器增加信号的驱动能力，寄存器只负责存储数据，如果要进行点灯操作，需要驱动器增大驱动能力。

**GPIO一位的具体结构**

![](https://i-blog.csdnimg.cn/direct/bbabf714787d4a3aa43f75c4e40e2601.png)


IO引脚连接两个保护二极管以对输入电压进行限幅，上方接VDD=3.3V，下方接VSS=0V。输入电压大于3.3V时上方二极管导通，小于0V时下方二极管导通，在0\~3.3V间两个二极管均不会导通。

输入驱动器右侧连接一个上拉电阻和一个下拉电阻，上拉电阻至VDD，下拉电阻至VSS，开关可以通过程序进行配置。上拉和下拉给输入提供默认输入电平，上拉导通下拉断开为上拉输入模式（默认为高电平的输入方式），下拉导通上拉断开为下拉输入模式（默认为低电平的输入方式），两者皆断开为浮空输入模式。

输入驱动器中施密特触发器（肖特基为翻译错误）对输入电压进行整形，避免波形失真。如果输入电压大于某一阈值，输出就会瞬间升为高电平；如果输入电压小于某一阈值，输出就会瞬间降为低电平。经过施密特触发器整形的波形即可直接写入输入数据寄存器，再用程序读取输入数据寄存器对应某一位的数据即可知道端口的输入电平。输入驱动器还接入了两条连接到其他需要读取端口的外设上的线，读取模拟量的线接在触发器之前，读取数字量的线接在触发器之后。

输出驱动器可以通过输出数据寄存器或其他片上外设控制，两种控制方式通过数据选择器连接输出控制部分。输出数据寄存器控制即普通IO口输出，由于输出数据寄存器只能整体读写，为了单独控制其中一个端口而不影响其他端口，只能采用特殊方式：

- 在程序中使用按位与/或的方式，较为麻烦。
- 是通过位设置/清除寄存器，如果要对某一位进行置1操作，在位设置寄存器的对应位写1、其余位写0即可，如果要对某一位进行清0操作，在位清除寄存器的对应位写1、其余位写0即可。
- 读写STM32中的“位带”区域，其映射了RAM和外设寄存器的所有位，读写其中的数据就相当于读写所映射位置的某一位，在本次学习中暂时不会用到。

输出控制之后连接两个MOS管，是一种电子开关，由信号控制开关的导通和关闭，开关负责将IO口接到VDD或VSS。可以选择推挽、开漏或关闭三种输出模式：

- 在推挽模式下，P-MOS和N-MOS均有效，数据寄存器为1时，上管导通，下管断开，输出接VDD，即高电平；数据寄存器为1时，上管断开，下管导通，输出接VSS，即低电平。该模式下高低电平均有较强的驱动能力，因此推挽输出模式也叫强推输出模式。
- 在开漏模式下，P-MOS无效，N-MOS有效，数据寄存器为1时，下管断开，输出断开，即高阻态；数据寄存器为0时，下管导通，输出接VSS，即低电平。该模式下只有低电平有驱动能力，高电平没有驱动能力，可以作为通信协议的驱动方式或结合外部上拉电阻输出5V的电平信号。
- 在关闭模式下，P-MOS和N-MOS均无效，端口的电平由外部信号控制。

## GPIO模式

![](https://i-blog.csdnimg.cn/direct/b07f0b88a84e448bbd7f5143846f85b9.png)


## GPIO寄存器

**端口配置低/高寄存器**（GPIOx_CRL/GPIOx_CRH）

端口配置寄存器每4位配置一个端口，低8位端口由端口配置低寄存器配置，高8位端口由端口配置高寄存器配置。

![](https://i-blog.csdnimg.cn/direct/1c476c0668284a2bbff3d0248463ad27.png)


![](https://i-blog.csdnimg.cn/direct/01cbf5be5ecb46bbaea27a30c4fba721.png)


![](https://i-blog.csdnimg.cn/direct/994bcc34b4db4619a7225f99661d7896.png)


GPIO的输出速度限制输出引脚的最大翻转速度以实现低功耗和稳定性，要求不高时配置成50MHz。

**端口输入/输出数据寄存器**（GPIOx_IDR/GPIOx_ODR）

端口输入/输出数据寄存器低16位对应16个引脚，高16位保留。

![](https://i-blog.csdnimg.cn/direct/9a780c8b73b64c6f9542ecfd137236bc.png)


![](https://i-blog.csdnimg.cn/direct/926dc2cdd0b84198a5969c54ef91641d.png)


**端口位设置/清除寄存器**（GPIO_BSRR）

端口位设置/清除寄存器高16位进行位清除，低16位进行位设置，写1进行设置/清除，写0不产生影响。

![](https://i-blog.csdnimg.cn/direct/e39d6d09313c4b3498011d9fab82fcda.png)


**端口位清除寄存器**（GPIOx_BRR）

端口位清除寄存器低16位和端口位设置/清除寄存器，该寄存器为了方便操作而设置。

![](https://i-blog.csdnimg.cn/direct/867650ff961a46ed8cfe3ee9a30eac72.png)


**端口配置锁定寄存器**（GPIOx_LCKR）

端口配置锁定寄存器对端口的配置进行锁定，防止意外更改，暂时用得不多。

![](https://i-blog.csdnimg.cn/direct/c01220315b42406b94c3beed5c3ff2a9.png)


# 外部设备和电路

## LED和蜂鸣器简介

- LED：发光二极管，正向通电点亮，反向通电不亮。
- 有源蜂鸣器：内部自带振荡源，将正负极接上直流电压即可持续发声，频率固定。
- 无源蜂鸣器：内部不带振荡源，需要控制器提供振荡脉冲才可发声，调整提供振荡脉冲的频率可发出不同频率的声音。

## 硬件电路

根据电路连接方式不同，外部设备有高低电平两种驱动方式，其选择由IO口高低电平驱动能力决定。推挽输出模式下二者均可，但很多单片机或芯片使用了高电平弱驱动、低电平强驱动的规则以避免高低电平打架，因此倾向使用第一种方法。限流电阻防止外部设备烧坏，也可调整LED亮度。

**LED（低电平驱动）**

![](https://i-blog.csdnimg.cn/direct/293b120d97cd4951820a3c2dfdfb6378.png)


**LED（低电平驱动）**

![](https://i-blog.csdnimg.cn/direct/11ed6ad7a45641bcaf1a0ac8b37176e9.png)


蜂鸣器采用三极管开关驱动以防止IO口因蜂鸣器功率大而负担过重。三极管开关是最简单的驱动电路，PNP由低电平驱动，NPN由高电平驱动。由于三极管的通断需要发射极和基极之间产生一定的开启电压，因此负载最好接在集电极。

**蜂鸣器（PNP三极管）**

![](https://i-blog.csdnimg.cn/direct/9c519fc946ab42d8abbe498373f348c2.png)


**蜂鸣器（NPN三极管）**

![](https://i-blog.csdnimg.cn/direct/cfe291bd0f114ce6a316fc5c47d38e50.png)


## 面包板

![](https://i-blog.csdnimg.cn/direct/d9b57cf6e88a4017b9b264c7b9f990ce.png)


![](https://i-blog.csdnimg.cn/direct/7ec7082df2e14caeb1ca929dfcdd26e7.png)


面包板背面有横向和纵向的金属爪，用于夹住引脚并连接线路。纵向金属爪每列分立，当元件插在一横排的不同孔位时，内部的金属爪就实现了线路的连接；横向金属爪整体相连，用于供电，从正负极对应的上下孔位中各引出一条跳线即可。

# 实验1 LED闪烁

## 电路连接

![](https://i-blog.csdnimg.cn/direct/37425b27446e4cdf86f376ae0852ed33.jpeg)


## 点亮LED

```c
#include "stm32f10x.h" 

int main(void)
{
	//开启GPIOA的时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	//GPIO初始化
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	//将A0设为低电平
	GPIO_ResetBits(GPIOA, GPIO_Pin_0);
	while(1)
	{
		
	}
}
```

## 函数详解

### RCC_APB2PeriphClockCmd函数

**简介**：APB2外设时钟使能/失能函数

**参数一**：APB2外设名称

	RCC_APB2Periph_AFIO, RCC_APB2Periph_GPIOA, RCC_APB2Periph_GPIOB,
    RCC_APB2Periph_GPIOC, RCC_APB2Periph_GPIOD, RCC_APB2Periph_GPIOE,
    RCC_APB2Periph_GPIOF, RCC_APB2Periph_GPIOG, RCC_APB2Periph_ADC1,
    RCC_APB2Periph_ADC2, RCC_APB2Periph_TIM1, RCC_APB2Periph_SPI1,
    RCC_APB2Periph_TIM8, RCC_APB2Periph_USART1, RCC_APB2Periph_ADC3,
    RCC_APB2Periph_TIM15, RCC_APB2Periph_TIM16, RCC_APB2Periph_TIM17,
    RCC_APB2Periph_TIM9, RCC_APB2Periph_TIM10, RCC_APB2Periph_TIM11

**参数二**：使能/失能

	ENABLE, DISABLE

### GPIO_Init函数

**简介**：GPIO外设初始化函数

**参数一**：GPIO外设名称

	GPIOA, ..., GPIOG

**参数二**：指向初始化信息结构体GPIO_InitTypeDef的指针

#### GPIO_InitTypeDef结构体

**成员GPIO_Mode**：输入/输出模式

	模拟输入：GPIO_Mode_AIN
	浮空输入：GPIO_Mode_IN_FLOATING
	下拉输入：GPIO_Mode_IPD
	上拉输入：GPIO_Mode_IPU
	开漏输出：GPIO_Mode_Out_OD
	推挽输出：GPIO_Mode_Out_PP
	复用开漏：GPIO_Mode_AF_OD
	复用推挽：GPIO_Mode_AF_PP

**成员GPIO_Pin**：GPIO引脚编号

	GPIO_Pin_0, ..., GPIO_Pin_15, GPIO_Pin_All

由于GPIO引脚编号本质还是二进制存储，因此可以通过或运算同时选中多个引脚。

GPIO_Pin_All选中全部16个引脚。

**成员GPIO_Speed**：输出速度

	GPIO_Speed_10MHz, GPIO_Speed_2MHz, GPIO_Speed_50MHz

### GPIO_SetBits函数

简介：将指定端口设置为高电平

参数一：GPIO外设名称

参数二：GPIO引脚编号

### GPIO_ResetBits函数

简介：将指定端口设置为低电平

参数一：GPIO外设名称

参数二：GPIO引脚编号

### GPIO_WriteBit函数

简介：向指定端口写入数据值

参数一：GPIO外设名称

参数二：GPIO引脚编号

参数三：设置/清楚端口值（置高/低电平）

	Bit_SET, Bit_RESET

或者使用数字+强制类型转换

	(BitAction)1, (BitAction)0
## LED闪烁

将资料包程序源码3-1中包含延时函数库的System文件夹复制粘贴到工程文件夹中，并在工程中添加对应的组和头文件路径。

```c
#include "stm32f10x.h" 
#include "Delay.h"

int main(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_0;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	while(1)
	{
		GPIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_RESET);
		Delay_ms(500);
		GPIO_WriteBit(GPIOA, GPIO_Pin_0, Bit_SET);
		Delay_ms(500);
	}
}
```

# 实验2 LED流水灯

## 电路连接

![](https://i-blog.csdnimg.cn/direct/2ed6a343ecbd4d28bc826a8a5123a04c.jpeg)


## LED流水灯

```c
#include "stm32f10x.h" 
#include "Delay.h"

int main(void)
{
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOA, ENABLE);
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_All;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOA, &GPIO_InitStructure);
	while(1)
	{
		//LED点亮对应位写1，其余写0，取反以低电平驱动
		GPIO_Write(GPIOA, ~0x0001);
		Delay_ms(500);
		GPIO_Write(GPIOA, ~0x0002);
		Delay_ms(500);
		GPIO_Write(GPIOA, ~0x0004);
		Delay_ms(500);
		GPIO_Write(GPIOA, ~0x0008);
		Delay_ms(500);
		GPIO_Write(GPIOA, ~0x0010);
		Delay_ms(500);
		GPIO_Write(GPIOA, ~0x0020);
		Delay_ms(500);
		GPIO_Write(GPIOA, ~0x0040);
		Delay_ms(500);
		GPIO_Write(GPIOA, ~0x0080);
		Delay_ms(500);
	}
}
```

## 函数详解

### GPIO_Write函数

简介：向端口输出数据寄存器(GPIOx_ODR)写入数据

参数一：GPIO外设名称

参数二：需写入的四位十六进制数据

# 实验3 蜂鸣器

## 电路连接

![](https://i-blog.csdnimg.cn/direct/04ff2101f12f442eb4afb4be6efae005.jpeg)


## 蜂鸣器

```c
#include "stm32f10x.h" 
#include "Delay.h"

int main(void)
{
	//本实验使用PB12端口，对初始化程序做相应修改
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOB, ENABLE);
	GPIO_InitTypeDef GPIO_InitStructure;
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_12;
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	GPIO_Init(GPIOB, &GPIO_InitStructure);
	while(1)
	{
		//每次连续响两下
		GPIO_ResetBits(GPIOB, GPIO_Pin_12);
		Delay_ms(100);
		GPIO_SetBits(GPIOB, GPIO_Pin_12);
		Delay_ms(100);
		GPIO_ResetBits(GPIOB, GPIO_Pin_12);
		Delay_ms(100);
		GPIO_SetBits(GPIOB, GPIO_Pin_12);
		Delay_ms(700);
	}
}
```

# 补充

使用库函数的方法：

- 打开.h文件，到最后查看有哪些函数，再右键转到定义查看函数和参数的用法。
- 打开资料文件里的库函数用户手册或固件库压缩包中的帮助文档。
- 浏览器搜索参考网上程序。
