> 主要参考学习资料：
> 
> B站@江协科技
> 
> STM32入门教程-2023版 细致讲解 中文字幕
>
>开发资料下载链接：https://pan.baidu.com/s/1h_UjuQKDX9IpP-U1Effbsw?pwd=dspb
>
> 单片机套装：STM32F103C8T6开发板单片机C6T6核心板 实验板最小系统板套件科协
>
>建议有51单片机基础再着手学习STM32单片机

@[TOC](目录)
# STM32简介

STM32是ST公司基于ARMCortex-M内核开发的32位微控制器，功能强大、性能优异、片上资源丰富、功耗低，是一款经典的嵌入式微控制器。

本次学习所用型号：

- 系列：主流系列STM32F1
- 内核：ARM Cortex-M3
- 主频：72MHz
- RAM：20K（SRAM）
- ROM：64K（Flash）
- 供电：2.0\~3.6V（标准3.3V）
- 封装：LQFP48

以下资料前期大致了解即可：

## 片上资源/外设

| 英文缩写    | 名称        | 英文缩写    | 名称        |
| ------- | --------- | ------- | --------- |
| NVIC    | 嵌套向量中断控制器 | CAN     | CAN通信     |
| SysTick | 系统滴答定时器   | USB     | USB通信     |
| RCC     | 复位和时钟控制   | RTC     | 实时时钟      |
| GPIO    | 通用IO口     | CRC     | CRC校验     |
| AFIO    | 复用IO口     | PWR     | 电源控制      |
| EXTI    | 外部中断      | BKP     | 备份寄存器     |
| TIM     | 定时器       | IWDG    | 独立看门狗     |
| ADC     | 模数转换器     | WWDG    | 窗口看门狗     |
| DMA     | 直接内存访问    | DAC     | 数模转换器     |
| USART   | 同步\异步串口通信 | SDIO    | SD卡接口     |
| I2C     | I2C通信     | FSMC    | 可变静态存储控制器 |
| SPI     | SPI通信     | USB OTG | USB主机接口   |

具体外设资源还需参考数据手册。

## 命名规则

![](https://i-blog.csdnimg.cn/direct/b81768d61cfb4bd3bbe20aa8485d01d3.png)


## 系统结构

![](https://i-blog.csdnimg.cn/direct/2244eb864bd74471ae1cd4b4cd155061.png)


## 引脚定义

![](https://i-blog.csdnimg.cn/direct/2a8020d9d8f44c0fbfe5308e5a6affad.png)


## 启动配置

![](https://i-blog.csdnimg.cn/direct/af92018417784d6d8bb412f18e43b807.png)


主闪存存储器为最常用的启动模式。

## 最小系统电路

![](https://i-blog.csdnimg.cn/direct/568e843494a348fca8cee2091b22b601.png)


# 新建工程

## 开发方式

STM32的开发方式有基于寄存器的方式、基于库函数的方式和基于HAL库的方式。

基于寄存器的方式和开发51单片机的方式一样，是用程序直接配置寄存器来达到我们想要的功能。这种方式最底层、最直接、效率更高，但是STM32的结构复杂、寄存器太多，所以基于寄存器的方式目前不推荐。

基于库函数的方式是使用ST官方提供的封装好的函数，通过调用函数来间接配置寄存器。由于ST对寄存器封装得比较好，所以这种方式既能满足对寄存器的配置，对开发人员也比较友好，有利于提高开发效率。本次学习使用库函数开发方式。

基于HAL库的方式可以用图形化界面快速配置STM32，比较适合快速上手STM32的情况。但是这种方式隐藏了底层逻辑，只能停留在很浅的水平，推荐学过库函数之后再了解。

## 步骤

（软件安装和破解过程省略，参照[[2-1] 软件安装_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1th411z7sn?spm_id_from=333.788.videopod.episodes&vd_source=14840859b7706d40ba924de91194c603&p=3)）

### 创建多工程框架

新建工程：

![](https://i-blog.csdnimg.cn/direct/948cae943e3c4fd1b265f664a9da2224.png)


将工程新建在单独的工程文件夹中，工程名字不易更改，建议起通用名字：

![](https://i-blog.csdnimg.cn/direct/4d95406ba9964123960fc17938219612.png)


选择芯片型号：

![](https://i-blog.csdnimg.cn/direct/b98528d036204eb0969be15f6bf62a7d.png)


弹出新建工程小助手，可以帮助我们快速新建工程，暂时不用可以关闭：

![](https://i-blog.csdnimg.cn/direct/98495ce5a6d14027bce59524f4acdc10.png)


将固件库中STM32F10x_StdPeriph_Lib_V3.5.0 > STM32F10x_StdPeriph_Lib_V3.5.0 > Libraries > CMSIS > CM3 > DeviceSupport > ST > STM32F10x > startup > arm的启动文件复制粘贴到工程文件夹中单独建的Start文件夹里：

![](https://i-blog.csdnimg.cn/direct/8eddb8df6f594c2296884aaf2ce648ed.png)


![](https://i-blog.csdnimg.cn/direct/669288cb03674972be5c33ab4c841ddd.png)


将固件库中STM32F10x_StdPeriph_Lib_V3.5.0 > STM32F10x_StdPeriph_Lib_V3.5.0 > Libraries > CMSIS > CM3 > DeviceSupport > ST > STM32F10x的stm32f10x.h（外设寄存器描述文件）、system_stm32f10x.c和system_stm32f10x.h（时钟配置文件）复制粘贴到Start文件夹：

![](https://i-blog.csdnimg.cn/direct/c738a86cd768437bb0b9eaa7e5b40927.png)


将固件库中STM32F10x_StdPeriph_Lib_V3.5.0 > STM32F10x_StdPeriph_Lib_V3.5.0 > Libraries > CMSIS > CM3 > CoreSupport中的内核寄存器描述文件复制粘贴到Start文件夹：

![](https://i-blog.csdnimg.cn/direct/34d6feef86614aaabdc0244ab7ade1d5.png)


准备将刚才复制的文件添加到工程里，将Source Group 1重命名为Start：

![](https://i-blog.csdnimg.cn/direct/813c113080454ea1bf8b35e75c0e3d0c.png)


![](https://i-blog.csdnimg.cn/direct/83f2288015a14e58a5c37379398183f8.png)


启动文件添加startup_stm32f10x_md.s，然后将.c和.h文件都添加进来：

![](https://i-blog.csdnimg.cn/direct/d669149dd63549558b4e82422881c217.png)


添加头文件路径：

![](https://i-blog.csdnimg.cn/direct/642abeb273584400bcf8ff53b9de9255.png)


在工程文件夹里创建User文件夹用于存放main函数，并创建User组和main函数：

![](https://i-blog.csdnimg.cn/direct/9d53ff1a0ec544299890ab5260edeb98.png)


![](https://i-blog.csdnimg.cn/direct/d250f930bead4e73b878538a5c9c6a68.png)


![](https://i-blog.csdnimg.cn/direct/4932e77e0b5b424bbf236e7f08de0957.png)


插入头文件：

![](https://i-blog.csdnimg.cn/direct/93e8a35ba95343049ae7f266fe71ab29.png)


编写一个main函数框架：

```c
#include "stm32f10x.h"                  // Device header

int main(void)
{
	while(1)
	{
		
	}
}

```

注意：

- main函数是一个int型返回值、void参数的函数。
- 文件的最后一行必须是空行，否则会报警告。

编译并建立工程：

![](https://i-blog.csdnimg.cn/direct/7da979fe654d4f5fa661d8f7edef8e34.png)


### 配置STLINK调试

将STLINK与STM32最小系统板使用4根母对母杜邦线连接：

- 3.3V——3.3/3V3
- SWCLK——DCLK
- SWDIO——DIO/SWIO
- GND——GND/G

![](https://i-blog.csdnimg.cn/direct/ddb7153e5df84a088eb27803539e842d.jpeg)


将STLINK插在电脑上可看到电源灯常亮，PC13口的灯闪烁，这是芯片的测试程序。

更改调试选项：

![](https://i-blog.csdnimg.cn/direct/056691caeeee4b239431e01799d79c4f.png)


重新编译后点击Load将程序下载到STM32：

![](https://i-blog.csdnimg.cn/direct/fec20f8905b9447fa01911c321f35949.png)


此时PC13口的灯熄灭。

### 点亮PC13口的灯

#### 寄存器方式

点亮PC13口的灯需要配置3个寄存器。

RCC寄存器APB2ENR外设时钟使能寄存器：使能GPIO时钟。

![](https://i-blog.csdnimg.cn/direct/5a602ab9d4324422bb1872664db97776.png)


第4位写1即可打开GPIO时钟，其余无关项写0。

端口配置高寄存器GPIOx_CRH：配置PC13口模式。

![](https://i-blog.csdnimg.cn/direct/fc4c3dc604a045f3a64794004a5c2a96.png)


第20\~23位的MODE13和CNF13用于配置PC13口：

![](https://i-blog.csdnimg.cn/direct/0736725370674749b822cc6f17361d89.png)


其中CNF13配置为通用推免输出模式（00），MODE13配置为输出模式最大速度50MHz（11），其余无关项写0。

端口输出数据寄存器GPIOx_ODR：给PC13口输出数据。

![](https://i-blog.csdnimg.cn/direct/fc5c752616ab4fc589ba96a0abf4c7ce.png)


第13位ODR13对应PC13口，写1为高电平，写0为低电平，低电平点亮，全写0。

```c
#include "stm32f10x.h"                  // Device header

int main(void)
{
	//以十六进制配置寄存器
	RCC -> APB2ENR = 0x00000010;
	GPIOC -> CRH = 0x00300000;
	GPIOC -> ODR = 0x00000000;
	while(1)
	{
		
	}
}

```

编译并下载，PC13口的灯点亮。

寄存器方式需要频繁查看数据手册，而且如果不想影响寄存器中其他位的配置还需要与操作和或操作，虽然代码简洁但不太方便。

#### 库函数方式

在工程文件夹中创建Library文件夹存放库函数，将固件库中STM32F10x_StdPeriph_Lib_V3.5.0 > STM32F10x_StdPeriph_Lib_V3.5.0 > Libraries > STM32F10x_StdPeriph_Driver > src的库函数源文件全部复制粘贴到Library文件夹：

![](https://i-blog.csdnimg.cn/direct/030aa50ab7164254872eb05d920c5ee7.png)


将固件库中STM32F10x_StdPeriph_Lib_V3.5.0 > STM32F10x_StdPeriph_Lib_V3.5.0 > Libraries > STM32F10x_StdPeriph_Driver > inc的库函数头文件全部复制粘贴到Library文件夹：

![](https://i-blog.csdnimg.cn/direct/f51c86f08810499d890940e72501e81d.png)


新建Library组并将Library文件夹中的所有文件添加进去：

![](https://i-blog.csdnimg.cn/direct/dc95d986caa746608aa3ce4a189a489f.png)


要使用库函数，还需要将固件库中STM32F10x_StdPeriph_Lib_V3.5.0 > STM32F10x_StdPeriph_Lib_V3.5.0 > Project > STM32F10x_StdPeriph_Template的stm32f10x_conf.h（配置库函数头文件包含关系和检查参数的函数定义）、stm32f10x_it.c和stm32f10x_it.h（中断函数）复制粘贴到User目录下并添加到User组中：

![](https://i-blog.csdnimg.cn/direct/645937c8eebd4071b4a7ddc10d006a4a.png)


头文件stm32f10x.h中有一个条件编译语句：

```c
#ifdef USE_STDPERIPH_DRIVER
  #include "stm32f10x_conf.h"
#endif
```

因此需要宏定义USE_STDPERIPH_DRIVER才能使stm32f10x_conf.h有效，顺便添加刚才新增的头文件路径：

![](https://i-blog.csdnimg.cn/direct/ecfbf8d0d1404f27af5678342b688624.png)


```c
#include "stm32f10x.h"                  // Device header

int main(void)
{
	//RCCAPB2外设时钟控制函数，使能GPIOC时钟
	RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);
	//GPIO使用结构体配置参数
	GPIO_InitTypeDef GPIO_InitStructure;
	//通用推免输出模式
	GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
	//指定PC13管脚
	GPIO_InitStructure.GPIO_Pin = GPIO_Pin_13;
	//输出模式最大速度50MHz
	GPIO_InitStructure.GPIO_Speed = GPIO_Speed_50MHz;
	//通过传入结构体地址配置GPIO模式
	GPIO_Init(GPIOC, &GPIO_InitStructure);
	//将PC13设为低电平
	GPIO_ResetBits(GPIOC, GPIO_Pin_13);
	//高电平为GPIO_SetBits函数
	while(1)
	{
		
	}
}
```

查阅函数定义以理解使用方法：

![](https://i-blog.csdnimg.cn/direct/a6ff454efbb04ea8a7893739841b4222.png)


![](https://i-blog.csdnimg.cn/direct/b7c99d72434f469f94fe397f6e098bbf.png)


- @brief：功能简介
- @param：参数
- @retval：返回值

查询GPIO_InitStructure结构体中GPIO_Mode成员的定义时，遇到参考@ref在注释中的情况，选中后Ctrl+F点Find Next定位：

![](https://i-blog.csdnimg.cn/direct/4f2825c613f24a448a099cc59ef4f47f.png)


typedef中找到该成员枚举出的值：

![](https://i-blog.csdnimg.cn/direct/c42382ab54f840fdb6ccae3121d508c1.png)


# 补充与总结

不同型号的启动文件后缀选择方法：

![](https://i-blog.csdnimg.cn/direct/93c720f05d6549b7be9255498c96158a.png)


新建工程的步骤：

- 建立工程文件夹，Keil中新建工程，选择型号
- 工程文件夹里建立Start、Library、User等文件夹，复制固件库里面的文件到工程文件夹
- 工程里对应建立Start、Library、User等同名称的分组，然后将文件夹内的文件添加到工程分组里
- 工程选项，C/C++，Include Paths内声明所有包含头文件的文件夹
- 工程选项，C/C++，Define内定义USE_STDPERIPH_DRIVER
- 工程选项，Debug，下拉列表选择对应调试器，Settings，Flash Download里勾选Reset and Run

工程架构：

![请添加图片描述](https://i-blog.csdnimg.cn/direct/64633f6f6900491b811d7ef93fbf9d1b.png)

