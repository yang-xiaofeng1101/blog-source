---
title: 树莓派3——EMMC访问SD卡的实现与分析
tags:
  - Linux
  - EMMC
  - 树莓派
originContent: ''
categories:
  - 技术
toc: false
date: 2022-05-24 23:54:21
---

# 相关知识
The External Mass Media Controller（EMMC)是 Arasan™ 提供的嵌入式 MultiMedia™ 和 SD™ 卡接口。

卡的接口使用它自己的时钟 clk_emmc，它由时钟管理器模块提供。该时钟的频率应在 50 MHz 和 100 MHz 之间选择。即使 VideoCore 以降低的时钟频率运行，拥有单独的时钟也可以对卡进行高性能访问。 EMMC 模块包含自己的内部时钟分频器，用于从 clk_emmc 生成卡的时钟。
此外，来自卡的响应和数据的采样时钟最多可以延迟 40 步，可配置的延迟通常在每步 200 ps 到 1100 ps 之间。延迟是为了取消读取时卡内部的内部延迟（最长 14ns）。每步的延迟将随温度和电源电压而变化。因此，最好使用比必要更大的延迟，因为最大延迟没有限制。
EMMC 模块自动处理命令和数据线上的握手过程以及所有 CRC 处理。
在将任何所需的参数加载到 ARG1 寄存器后，通过将命令和适当的标志写入 CMDTM 寄存器来开始命令执行。 EMMC 模块计算 CRC 校验和，将命令传送到卡，接收响应并检查其 CRC。一旦命令已执行或超时，寄存器 INTERRUPT 的位 0 将被设置。请注意，中断寄存器不是自清零的，因此软件必须先通过写入 1 来复位它，然后才能使用它来检测命令是否完成。该软件负责检查卡响应的状态位，以验证卡是否成功处理。
为了从/向卡寄存器传输数据，在配置主机并使用 CMDTM寄存器向卡发送相应的命令后访问数据。因为 EMMC 模块不解释发送到卡的命令，所以将其配置为与使用 CONTROL0 寄存器的卡设置相同是很重要的。应特别注意确保主机和卡的数据总线宽度配置相同。通过适当地关闭其时钟，卡与数据流同步。握手信号 dma_req 可用于有节奏的数据传输。 INTERRUPT 寄存器的位 1 可用于确定数据传输是否完成。请注意，中断寄存器不是自清零的，所以软件在使用它来检测数据传输是否完成之前，必须首先通过写入 1 来复位它。
EMMC 模块将最大块大小限制为内部数据 FIFO 的大小，即 1k 字节。为了获得数据传输的最大性能，有必要使用多块数据传输。在这种情况下，EMMC 模块在乒乓模式下使用两个 FIFO，即一个用于向/从卡传输数据，而另一个则由 DMA 通过 AXI 总线同时访问。如果 EMMC 模块配置为单块传输，则仅使用一个 FIFO，因此在向卡传输数据或从卡传输数据时无法进行 DMA 访问，反之亦然，从而导致较长的死区时间。
＃ EMMC
EMMC 模块寄存器只能作为 32 位寄存器访问
树莓派3
EMMC寄存器基地址为0x7E300000
树莓派4
EMMC寄存器基地址为0x7E300000
![image.png](/images/2022/05/24/1605a73f-7bef-4742-a6c1-dbdf4873ba0a.png)
此处在总线地址 0x7Ennnnnn 相当于在物理地址0x3Fnnnnnn可用
EMMC相关寄存器及其地址如下表所示：
![image.png](/images/2022/05/24/317ef3ea-0417-4710-a1a2-a9035e0df8e3.png)
![image.png](/images/2022/05/24/061679a6-8d04-4db2-ab6e-3bededfacabe.png)
在这些寄存器里，经常被使用（指的是读写数据情景）的是ARG1，CMDTM，DATA，INTERRUPT这四个寄存器。其他寄存器在EMMC initialization阶段使用过后一般不再涉及。
## 读写数据场景
### 寄存器介绍
1.	**ARG1 Register**
This register contains the arguments for all commands except for the SD card specific command ACMD23 which uses ARG2. ARG1 must be set before the command is issued using the CMDTM register.
![image.png](/images/2022/05/24/75dd6da9-6829-458f-abde-1eb435208705.png)
ARG1寄存器是负责保存参数的，用于和EMMC模块进行命令式的通信时使用
2.	**CMDTM Register**
该寄存器用于向卡发出命令。除了命令之外，它还包含标志，通知 EMMC 模块什么卡响应和期望的数据传输类型。不正确的标志会导致错误的行为

对于数据传输，支持两种模式：传输单个数据块或多个相同大小的块。 SD 卡使用两组不同的命令来区分它们，但需要使用 TM_MULTI_BLOCK 额外配置主机。重要的是，对于发送到卡的命令，该位设置正确，即 CMD18 和 CMD25 为 1，CMD17 和 CMD24 为 0。多块传输提供了更好的性能。

BLKSIZECNT 寄存器用于配置要传输的块的大小和数量。如果该寄存器的 TM_BLKCNT_EN 位置位，则在传输了 BLKSIZECNT 寄存器中配置的数据块数量后传输自动停止。

TM_AUTO_CMD_EN 位可用于使主机在 BLKSIZECNT 寄存器中的 BLKCNT 位为 0 时自动向卡发送命令，告知其数据传输已完成。

CMDTM寄存器具体每个bit的含义见下表

![image.png](/images/2022/05/24/15104761-25ba-448d-8dcb-54872d8fc406.png)
![image.png](/images/2022/05/24/640b30e8-bb81-4613-becb-8ca25a531e0e.png)
3.	**DATA Register**
该寄存器用于向/从卡传输数据。 INTERRUPT 寄存器的位 1 可用于检查数据是否可用。对于paced DMA 传输，可以使用高电平有效信号 dma_req。
![image.png](/images/2022/05/24/cc941877-7b8c-4abc-8b34-0eaa9a894599.png)
4.	**INTERRUPT Register**
该寄存器保存中断标志。可以使用 IRPT_MASK 寄存器中的相应位禁用每个标志。

其中与读写数据场景相关的bit见下表
![image.png](/images/2022/05/24/7c5a29c7-ab8b-4e0d-b761-9b121a6e30dc.png)
![image.png](/images/2022/05/24/5a58a19e-968d-42b6-98b3-623d7d7d899a.png)

READ_RDY位于interrupt 寄存器的第5 bit，值为1表示已经准备好要被读取来自SD卡的数据，一般用在读请求返回数据时
WRITE_RDY位于interrupt 寄存器的第4bit，值为1表示已经准备好接收要被写入SD卡的数据，一般用在写请求写入数据时

DATA_DONE 位于interrupt 寄存器第1 bit，值为1表示数据传输完成

### 工作流程
```cpp
#define CMD_READ_SINGLE 0x11220010
#define CMD_READ_MULTI 0x12220032
#define CMD_WRITE_SINGLE 0x18220000
#define CMD_WRITE_MULTI 0x19220022
```
CMD命令指令值宏定义，更多CMD格式见[EMMC_JESD84-A441.pdf](EMMC_JESD84-A441.pdf)的7.10章节
![image.png](/images/2022/05/24/bfe85e92-d4e6-4d11-8ea8-23efb2d3b679.png)
![image.png](/images/2022/05/24/a01dc938-46e8-410a-9570-1bb67b022f27.png)
在一些项目中，宏定义命名为CMDn，这里用的命名是Abbreviation名称
#### READ
（代码只是基本流程，并非实际编码实现）
1.	发送读命令，传入lba参数
```c
sd_cmd(CMD_READ_SINGLE, lba);
```
sd_cmd的实现如下
```c
*EMMC_ARG1 = lba;
*EMMC_CMDTM = CMD_READ_SINGLE;

```
sd_cmd运行后等待结果
```c
while(times < timeout) {
    u32 reg = *EMMC_INTERRUPT;
    if (reg & 0x8001) {		//CMD_DONE
        break;
    }
    timer_sleep(1);
    times++;
}
while(times < timeout) {
    u32 reg = *EMMC_INTERRUPT;
    if (reg & 0x8020) {		//READ_RDY
        break;
    }
    timer_sleep(1);
    times++;
}

long buffer[128];
for (d = 0; d < 128; d++)
    buffer[d] = *EMMC_DATA;


```
检查interrupt 寄存器的CMD_DONE 所在bit是否为1，如果是0说明命令未完成
继续等待直到CMD_DONE bit值为1
检查interrupt 寄存器的READ_RDY 所在bit是否为1，如果是0说明数据没ready
继续等待直到READ_ RDY bit值为1
READ_RDY满足条件后，从EMMC的DATA寄存器中读一个block的数据到缓冲区数组buffer中。

如果要修改block大小，可以通过CMD16来实现，sd_cmd(CMD16,  BlockSizeInBytes)

#### WRITE
（代码只是基本流程，并非实际编码实现）
1.	发送读命令，传入lba参数
```c
sd_cmd(CMD_WRITE_SINGLE, lba);
```
sd_cmd的实现如下
```c
*EMMC_ARG1 = lba;
*EMMC_CMDTM = CMD_WRITE_SINGLE;
```
sd_cmd运行后等待结果
```c
while(times < timeout) {
    u32 reg = *EMMC_INTERRUPT;
    if (reg & 0x8001) {		//CMD_DONE
        break;
    }
    timer_sleep(1);
    times++;
}
while(times < timeout) {
    u32 reg = *EMMC_INTERRUPT;
    if (reg & 0x8010) {  	//WRITE_RDY
        break;
    }
    timer_sleep(1);
    times++;
}

long buffer[128];
for (d = 0; d < 128; d++)
    *EMMC_DATA = buf[d];
```
检查interrupt 寄存器的CMD_DONE 所在bit是否为1，如果是0说明命令未完成
继续等待直到CMD_DONE bit值为1
检查interrupt 寄存器的READ_RDY 所在bit是否为1，如果是0说明数据没ready
继续等待直到WRITE_ RDY bit值为1
WRITE_RDY满足条件后，从到缓冲区数组buffer向EMMC的DATA寄存器中写一个block的数据。

## EMMC初始化场景
### 寄存器介绍
#### EMMC寄存器

1.	**CONTROL0 Register**
该寄存器用于配置 EMMC 模块。
![image.png](/images/2022/05/24/346c5c7e-053f-4a35-b137-820f04469e3c.png)
![image.png](/images/2022/05/24/bda4d54a-c3a9-4c4a-a361-7ec366b9ed49.png)
2.	**CONTROL1 Register**
该寄存器用于配置 EMMC 模块。CLK_STABLE 似乎与其名称相反，只是表示 clk_emmc 输入上有一个上升沿，但并不表示该时钟的频率实际上是稳定的。
![image.png](/images/2022/05/24/65567723-b015-4533-aaae-8974468dd6e7.png)
![image.png](/images/2022/05/24/1991b0a1-887f-4323-951a-08a69114c958.png)
其中的SRST_DATA，SRST_CMD，SRST_HC是Reset设备的控制bit，在一开始就要被处理CLK_XXX是时钟相关的Field
3.	**IRPT_MASK Register**
该寄存器用于屏蔽 INTERRUPT 寄存器中的中断标志。

4.	**IRPT_EN Register**
该寄存器用于启用 INTERRUPT 寄存器中的不同中断，以在 int_to_arm 输出上产生中断。

5.	**STATUS Register**
该寄存器包含用于调试的信息。它的值会根据硬件自动改变。由于它涉及不同时钟域之间的重新同步，它仅在一些延迟后才会改变，并且很容易过早地对值进行采样。因此不推荐使用该寄存器进行轮询。取而代之的是使用实现握手机制的INTERRUPT Register，这使得轮询时不可能错过更改。
![image.png](/images/2022/05/24/a3599248-57d3-464b-af65-09002b5e0759.png)
6.	**INTERRUPT Register**
该寄存器保存中断标志。可以使用 IRPT_MASK 寄存器中的相应位禁用每个标志

7.	**SLOTISR_VER Register**
该寄存器包含版本信息和插槽中断状态。
![image.png](/images/2022/05/24/d4c71e35-de0d-451b-ae2c-2f21b6ba4357.png)
![image.png](/images/2022/05/24/26a38c62-2631-41c9-99a0-267f04bc2419.png)

8.	**BLKSIZECNT Register**
它包含要传输的数据块的数量和大小（以字节为单位）。请注意，EMMC 模块将最大块大小限制为内部数据 FIFO 的大小，即 1k 字节。 
BLKCNT 用于告诉主机要传输多少数据块。一旦数据传输开始并且 CMDTM 寄存器中的 TM_BLKCNT_EN 位置位，EMMC 模块会随着数据块的传输自动减小 BNTCNT 值，并在 BLKCNT 达到 0 时停止传输。
当卡和主机之间的任何数据传输正在进行时，不得访问或修改此寄存器。
![image.png](/images/2022/05/24/820ba369-8603-47d6-bb44-4e93de5da519.png)
#### GPIO 寄存器
![image.png](/images/2022/05/24/5c34c06a-6fd6-4732-91cf-f7d8eb09f93c.png)
![image.png](/images/2022/05/24/a2513119-8f03-49a0-9d3d-dcbc40b2ae9d.png)
GPIO High Detect Enable Registers (GPHENn)
高电平检测使能寄存器定义在事件检测状态寄存器 (GPEDSn) 中设置高电平位的引脚。如果尝试清除 GPEDSn 中的状态位时引脚仍然为高电平，则状态位将保持设置状态。
![image.png](/images/2022/05/24/319cc1d8-9d9f-43f6-b78c-8fb3db6abf48.png)
**GPIO Function Select Registers (GPFSELn)**
功能选择寄存器用于定义通用 I/O 引脚的操作。 54 个 GPIO 引脚中的每一个都至少具有第 16.2 节中定义的两个替代功能。 FSEL{n} 字段确定第 n 个 GPIO 引脚的功能。所有未使用的替代功能线都接地，如果选择，将输出“0”。所有引脚复位为正常 GPIO 输入操作。
![image.png](/images/2022/05/24/21fd0cb8-a577-46fe-8a1e-f923acde7a2d.png)

**GPIO Event Detect Status Registers (GPEDSn)**
事件检测状态寄存器用于记录 GPIO 引脚上的电平和边沿事件。事件检测状态寄存器中的相关位在以下情况下设置：
1) 检测到与上升/下降沿检测启用寄存器中编程的边沿类型相匹配的边沿，或 2) 检测到与编程电平类型相匹配的电平在高/低电平检测使能寄存器。通过向相关位写入“1”来清除该位。
可以对中断控制器进行编程，以在设置任何状态位时中断处理器。 

GPIO 外设具有三个专用中断线。每个 GPIO bank 可以产生一个独立的中断。每当设置任何位时，第三行都会生成一个中断。
![image.png](/images/2022/05/24/8092c18d-3258-4613-af4f-0f401358a5dd.png)
![image.png](/images/2022/05/24/50fca48d-f0cc-4df9-b3a0-0ed09b1a1cba.png)

**GPIO Pull-up/down Register (GPPUD)**
GPIO 上拉/下拉寄存器控制内部上拉/下拉控制线对所有 GPIO 引脚的驱动。该寄存器必须与 2 个 GPPUDCLKn 寄存器一起使用。请注意，无法回读当前的上拉/下拉设置，因此用户有责任“记住”哪些上拉/下拉处于活动状态。这样做的原因是，即使在掉电模式下，当内核关闭时，GPIO 上拉也保持不变，此时备用功能表也具有掉电后应用的上拉状态。
![image.png](/images/2022/05/24/146058c8-b575-40bf-bc19-a4328bbe016e.png)

**GPIO Pull-up/down Clock Registers (GPPUDCLKn)**
GPIO 上拉/下拉时钟寄存器控制相应 GPIO 引脚上内部下拉的驱动。这些寄存器必须与 GPPUD 寄存器一起使用，以实现 GPIO 上拉/下拉更改。需要以下事件序列： 
1. 写入 GPPUD 以设置所需的控制信号（即上拉或下拉或都不删除当前的上拉/下拉） 
2. 等待 150 个周期——这提供了所需的控制信号的建立时间 
3. 写入 GPPUDCLK0/1 以将控制信号时钟输入您希望修改的 GPIO 焊盘 – 注意只有接收时钟的焊盘将被修改，所有其他焊盘将保持其先前状态。 
4. 等待 150 个周期——这为控制信号提供了所需的保持时间 
5. 写入 GPPUD 以移除控制信号 
6. 写入 GPPUDCLK0/1 以移除时钟
![image.png](/images/2022/05/24/868bfc3e-457f-488e-8795-d119d3dc7022.png)

#### SD卡接口寄存器
在卡接口内定义了六个寄存器：OCR、CID、CSD、EXT_CSD、RCA 和 DSR。这些只能通过相应的命令访问（参见EMMC_JESD84-A441.pdf第85 页的第 7.10 节）。 OCR、CID 和 CSD 寄存器携带卡/内容特定信息，而 RCA 和 DSR 寄存器是存储实际配置参数的配置寄存器。 EXT_CSD 寄存器携带卡特定信息和实际配置参数。

1.	**OCR register**
32 位操作条件寄存器 (OCR) 存储卡的 VDD 电压曲线和访问模式指示。此外，该寄存器包含一个状态信息位。如果卡上电过程已完成，则设置此状态位。 OCR寄存器应由所有卡实现。

2.	**CID register**
卡识别 (CID) 寄存器为 128 位宽。它包含在卡识别阶段（MultiMediaCard 协议）使用的卡识别信息。每个单独的闪存或 I/O 卡都应具有唯一的标识号。每种类型的 MultiMediaCard ROM 卡（由内容定义）都应具有唯一的标识号。第 114 页的表 42 列出了这些标识符。

3.	**CSD register**
卡特定数据 (CSD) 寄存器提供有关如何访问卡内容的信息。 CSD 定义了数据格式、纠错类型、最大数据访问时间、数据传输速度、是否可以使用 DSR 寄存器等。寄存器的可编程部分（由 W 或 E 标记的条目，见下文）可以通过以下方式更改CMD27。

4.	**Extended CSD register**
扩展 CSD 寄存器定义卡属性和选择的模式。它有 512 个字节长。最重要的 320 字节是 Properties 段，它定义了卡的功能并且不能被主机修改。低 192 字节是 Modes 段，它定义了卡的工作配置。主机可以通过 SWITCH 命令更改这些模式。

5.	**RCA register**
可写的 16 位相对卡地址（RCA）寄存器携带主机在识别卡时分配的卡地址。该地址用于在卡识别过程之后寻址的主机卡通信。 RCA 寄存器的默认值为 0x0001。保留值 0x0000 以使用 CMD7 将所有卡设置为待机状态。

6.	**DSR register**
16 位驱动级寄存器 (DSR) 在第 169 页的第 12.4 节中进行了详细描述。它可以选择性地用于提高扩展操作条件下的总线性能（取决于总线长度、传输速率或牌）。 CSD 寄存器携带有关 DSR 寄存器使用情况的信息。 DSR寄存器的默认值为0x404

### 工作流程
#### GPIO引脚初始化
结合BCM2837说明文档，每条指令的含义标注在代码注释中
```c
long r, cnt, ccs = 0;
// GPIO_CD
r = *GPFSEL4;    // GPIO Alternate function select register 4
r &= ~(7 << (7 * 3)); //FSEL47 GPIO Pin 47 takes alternate function 3
*GPFSEL4 = r;
*GPPUD = 2; //enable pull up control
wait_cycles(150);
*GPPUDCLK1 = (1 << 15);  //assert clock on line 47
wait_cycles(150);
*GPPUD = 0; //OFF disable pull down/up
*GPPUDCLK1 = 0; //No Effect
r = *GPHEN1;
r |= 1 << 15;
*GPHEN1 = r; //High on GPIO pin 47 sets corresponding bit in GPEDS

// GPIO_CLK, GPIO_CMD
r = *GPFSEL4;    // GPIO Alternate function select register 4
r |= (7 << (8 * 3)) | (7 << (9 * 3)); //GPIO Pin 48,49 takes alternate function 3
*GPFSEL4 = r;
*GPPUD = 2;  //enable pull up control
wait_cycles(150);
*GPPUDCLK1 = (1 << 16) | (1 << 17); //assert clock on line 48,49
wait_cycles(150);
*GPPUD = 0;  //OFF disable pull down/up
*GPPUDCLK1 = 0; //No Effect

// GPIO_DAT0, GPIO_DAT1, GPIO_DAT2, GPIO_DAT3
r = *GPFSEL5; // GPIO Alternate function select register 5
r |= (7 << (0 * 3)) | (7 << (1 * 3)) | (7 << (2 * 3)) | (7 << (3 * 3));
// GPIO Pin 50,51,52,53 takes alternate function 3
*GPFSEL5 = r;
*GPPUD = 2; //enable pull up control
wait_cycles(150);
*GPPUDCLK1 = (1 << 18) | (1 << 19) | (1 << 20) | (1 << 21); //assert clock on line 50,51,52,53
wait_cycles(150);
*GPPUD = 0; //OFF disable pull down/up
*GPPUDCLK1 = 0; //No Effect
uart_puts("EMMC: GPIO set up\n");
```
这段代码的含义是，将GPIO47~53 引脚全部 pull Up；
目的是将GPIO47 引脚设置为GPIO_CD；将GPIO48，49引分别设置为CLK和CMD，GPIO50~53做dat0~3四路数据line

#### EMMC初始化
```c
sd_hv = (*EMMC_SLOTISR_VER & HOST_SPEC_NUM) >> HOST_SPEC_NUM_SHIFT;
```
读取Host Controller specification version信息
```c
/ Reset the card.
*EMMC_CONTROL0 = 0;
*EMMC_CONTROL1 |= C1_SRST_HC;
cnt = 10000;
do
{
    wait_msec(10);
} while ((*EMMC_CONTROL1 & C1_SRST_HC) && cnt--);
if (cnt <= 0)
{
    uart_puts("ERROR: failed to reset EMMC\n");
    return SD_ERROR;
}
```
![image.png](/images/2022/05/24/d145b8b6-e45f-42ff-92d5-c56163ec4713.png)
通过control0寄存器来reset complete host circuit，reset成功后SRST_HC bit将变成0
```c
//This enabled VDD1 bus power for SD card, needed for RPI 4.
    #ifdef RPI_VERSION == 4
    *EMMC_CONTROL1 |= 0x0F << 8;
    wait_msec(10);
    #endif
```
如果是树莓派4，则需要给control1的第8~11 bit设置为1，来enable VDD1 bus power for SD card
![image.png](/images/2022/05/24/e85ea6cb-64e1-4272-8af3-d7449625ac48.png)
![image.png](/images/2022/05/24/35bbfbce-f50e-4a34-b162-19de537eb679.png)

```c
*EMMC_CONTROL1 |= C1_CLK_INTLEN | C1_TOUNIT_MAX;
wait_msec(10);
```
设置了Data timeout unit exponent:和Clock enable for internal EMMC clocks for power saving
```c
// Set clock to setup frequency.
    if ((r = sd_clk(400000)))
        return r;
//允许所有中断产生，设置中断掩码，作用于interrupt 寄存器
    *EMMC_INT_EN = 0xffffffff;
    *EMMC_INT_MASK = 0xffffffff;
```
设置了时钟频率，开启了中断和mask
sd_clk的实现如下
```c
*EMMC_CONTROL1 &= ~C1_CLK_EN; //disable clk
    wait_msec(10);

/***************************/
计算频率
/***************************/

if (sd_hv > HOST_SPEC_V2)  //判断是否是sdhc卡，也就是二代SD卡
    d = c;
else
    d = (1 << s);
if (d <= 2)
{
    d = 2;
    s = 0;
}
uart_puts("sd_clk divisor ");
uart_hex(d);
uart_puts(", shift ");
uart_hex(s);
if (sd_hv > HOST_SPEC_V2)
    h = (d & 0x300) >> 2;
d = (((d & 0x0ff) << 8) | h);
*EMMC_CONTROL1 = (*EMMC_CONTROL1 & 0xffff003f) | d; //设置时钟频率
wait_msec(10);
*EMMC_CONTROL1 |= C1_CLK_EN;  //enable clk
wait_msec(10);
cnt = 10000;
while (!(*EMMC_CONTROL1 & C1_CLK_STABLE) && cnt--) //判断是否生效
    wait_msec(10);
```
sd_clk到此结束

```c
sd_cmd(CMD_GO_IDLE, 0);
```
![image.png](/images/2022/05/24/136827d0-1bb6-4fdb-aeed-1b2406b5f43d.png)
Resets the card to idle state
sd_cmd的实现与之前在读写数据场景章节中的类似，通过的是CMDTM传命令和ARG传参数实现。
```c
sd_cmd(CMD_SEND_IF_COND, 0x000001AA);   //The card sends its EXT_CSD register as a block of data. CMD8
```
![image.png](/images/2022/05/24/ea164ddf-e10b-4f20-9d42-ebd9495e0a9e.png)
用读data的方式获取EXT_CSD寄存器的内容，从接⼝状态命令判断是否⽀持SD V2.0

```c
r = sd_cmd(CMD_SEND_OP_COND, ACMD41_ARG_HC); //Asks the card, in idle state, to send its Operating Conditions Register(OCR register) contents in the response on the CMD line.
```
![image.png](/images/2022/05/24/8af5389c-be0e-4a5f-9520-c447032699de.png)
用CMD response 的方式让sd卡发送OCR寄存器的数据。
SD_SEND_OP_COND (ACMD41)命令来识别或者拒绝不匹配host主机供电电压范围的卡。如果SD卡在主机规定的电压范围内不能实现数据传输，卡将放弃下一步的总线操作而进入不活动状态(Inactive State)。


```c
sd_cmd(CMD_ALL_SEND_CID, 0);
```
![image.png](/images/2022/05/24/b27046d8-824f-44d1-95bc-fc2dcff9b5ff.png)
让SD卡发送它的CID寄存器的值到CMD line 上，在卡发送它的CID之后，卡进入识别状态（Identification State）。

```c
sd_rca = sd_cmd(CMD_SEND_REL_ADDR, 0);
```
![image.png](/images/2022/05/24/3cebd233-dd80-4b89-85fd-5c22f8c5f574.png)
获取 card的地址，RCA是一个比CID短的，并且将来在数据传输模式中使用的地址。

```c
sd_cmd(CMD_CARD_SELECT, sd_rca);
```
![image.png](/images/2022/05/24/8045e69b-d738-40db-82af-8bcb8018dc4d.png)
选择根据 sd_cra来选择card，并且将卡带入传输状态(Transfer State)。在同一个时间内，只有一张卡能进入传输状态。当发送的CMD7的RCA地址参数为"0x0000"，所有卡将跳回到准备状态（Stand-by State ）。

```c
*EMMC_BLKSIZECNT = (1 << 16) | 8;
sd_cmd(CMD_SEND_SCR, 0);    //SD Configuration Register
if (sd_int(INT_READ_RDY))
	return SD_TIMEOUT;
while (r < 2 && cnt)
{
if (*EMMC_STATUS & SR_READ_AVAILABLE)
        sd_scr[r++] = *EMMC_DATA;   //SD Configuration Register
}
```
读取SCR寄存器到sd_scr数组中，可以得到设备的版本信息，支持的bus width等。用于进一步可选的配置。
SCR 寄存器：SCR（SD 配置）寄存器总共 64 bits，定义了卡片的一些特殊功能；该寄存器由厂商编程，主机不能对它进行编程

至此 EMMC初始化完成
