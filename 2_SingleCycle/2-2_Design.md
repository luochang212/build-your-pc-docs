[👈 Previous](./2-1_Basic.md) · [👉 Next](./2-3_Verilog.md) · [🚩 Home](../README.md)

# 单周期 CPU 的设计思路

- [单周期 CPU 的设计思路](#%e5%8d%95%e5%91%a8%e6%9c%9f-cpu-%e7%9a%84%e8%ae%be%e8%ae%a1%e6%80%9d%e8%b7%af)
  - [基本的数据通路图](#%e5%9f%ba%e6%9c%ac%e7%9a%84%e6%95%b0%e6%8d%ae%e9%80%9a%e8%b7%af%e5%9b%be)
  - [主要控制逻辑的真值表](#%e4%b8%bb%e8%a6%81%e6%8e%a7%e5%88%b6%e9%80%bb%e8%be%91%e7%9a%84%e7%9c%9f%e5%80%bc%e8%a1%a8)

熟悉了我们需要实现的 8 条指令之后，我们就需要设计 CPU 的数据通路。

## 基本的数据通路图

我在最终设计的数据通路图中，主要的数据通路包括：

- 指令存储器：Instruction Memory
- 数据存储器：Data Memory
- 寄存器堆：Register File
- 程序计数器：Program Counter
- 专用于处理跳转指令的 NPC：Next Program Counter
- 控制单元：Control Unit
- 算术逻辑单元：ALU
- 符号扩展、移位二合一模块：Extend Module
- 多处多路选择器：Multiplexer

**值得注意的是**：单周期 CPU 在实际实现的过程中并不需要指令寄存器（IR）。

具体数据通路图大致如下：

![](https://i.loli.net/2019/09/02/LIoGAp8nbwtxV1e.png)

其中，主要的逻辑控制信号 Control Signals（蓝色）有：

|    信号    |                                          功能                                           |
| :--------: | :-------------------------------------------------------------------------------------: |
| `RegWrite` |                                    写寄存器使能信号                                     |
| `MemWrite` |                                  读数据存储器使能信号                                   |
|  `ALUOp`   |                   ALU 控制信号（区分算术指令，比如 `ADD`、`SUB` 等）                    |
|  `RegSrc`  |             选择将 ALU 计算结果、数据存储器输出或 Extend 模块输出写入寄存器             |
|  `RegDst`  |                                写入寄存器 rt 、rd 二选一                                |
|  `ALUSrc`  | 选择 ALU 源操作数来自寄存器或符号扩展的立即数（区分算术指令结果与 `LW`、`SW` 指令结果） |
|  `NPCOp`   |            决定下一指令地址 NPC，或为 PC + 4，或为 BEQ 目标指令或 J 目标指令            |
|  `ExtOp`   |                 Extend 模块控制信号源（`LUI` 移位 16 位、有无符号扩展）                 |

由于需要：

- 实现 `BEQ` 和 `J`，立即数的位数不一样，因此 15 - 0 位的 `imm16` 代表 `BEQ` 指令的偏移量，25 - 0 位的 `imm26` 代表 `J` 指令的偏移量
- 实现 `BEQ`，需要将 rs、rt 寄存器的值相减得到并判断是否为 0，因此引入 `Zero` 控制信号，用来判断是否跳转

## 主要控制逻辑的真值表

![](https://i.loli.net/2019/09/02/5vR41AYxGjQsiwV.png)

其中，对于 ALU 算术指令，我们只需要实现加法和减法，`ALUOp` 功能表如下（为了方便后续扩展，我设计的 `ALUOp` 有三位）：

| `ALUOp[2:0]` |  功能   |  描述  |
| :----------: | :-----: | :----: |
|     000      | Default | 缺省值 |
|     001      |  Y=A+B  |   加   |
|     010      |  Y=A-B  |   减   |

对于 Extend 模块（用于统一进行符号扩展或移位操作），`ExtOp` 功能表如下：

| `ExtOp[1:0]` |               功能                |  指令  |
| :----------: | :-------------------------------: | :----: |
|      00      |              Default              | 缺省值 |
|      01      | 将 16 位立即数 imm 向左移位 16 位 |  LUI   |
|      10      |   立即数 imm 有符号扩展至 32 位   | ADDIU  |
|      11      | 立即数 offset 无符号扩展至 32 位  | LW、SW |

对于 `RegSrc` 信号，我们需要选择写入寄存器的源：

| `RegSrc[1:0]` |         功能         |      指令       |
| :-----------: | :------------------: | :-------------: |
|      00       |       Default        |     缺省值      |
|      01       |       来自 ALU       | ADDIU、ADD、SUB |
|      10       |   来自 Data Memory   |       LW        |
|      11       | 来自 Extend 模块输出 |       LUI       |

对于 NPC 模块，`NPCOp` 信号需要决定我们执行正常下一条取值（PC + 4）、BEQ 跳转或 J 跳转：

| `NPCOp[2:0]` |     功能     |   指令   |
| :----------: | :----------: | :------: |
|     000      |   Default    |  缺省值  |
|     001      |   普通跳转   | 正常指令 |
|     010      | J 型直接跳转 |    J     |
|     011      |  BEQ 型跳转  |   BEQ    |

[👈 Previous](./2-1_Basic.md) · [👉 Next](./2-3_Verilog.md) · [🚩 Home](../README.md)