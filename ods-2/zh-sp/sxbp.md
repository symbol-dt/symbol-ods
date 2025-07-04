# 简易 16 位处理器 (SIMPLE 16 BITS PROCESSOR) -- ODS 2

> - Author: [DiannaoJun](mailto:aheiwuchang@163.com)
> - Date: 2025/04/30 20:04 UTC+08
> - Version: 0.1.0

## 简介
简易 16 位处理器 (SIMPLE 16 BITS PROCESSOR) 是一个简单的 16 位处理器，用于学习处理器设计。

## 寻址
| 代号 | 寻址模式 | 描述 |
| :-:       | :-: | :--- |
| `immx`    | 16 位立即数 | 直接操作值 |
| `immb`    | 8 位立即数 | 直接操作值 |
| `immh`    | 4 位立即数 | 直接操作值，通常用于位移操作 |
| `regx`    | 16 位寄存器 | 寄存器代号作为操作值 |
| `regp`    | 指针寄存器 | 指针寄存器代号作为操作值 |
| `rega`    | 通用寄存器 | 通用寄存器代号作为操作值 |
| `regb`    | 8 位寄存器 | 8 位寄存器代号作为操作值 |
| `regl`    | 低 8 位寄存器 | 低 8 位寄存器代号作为操作值 |
| `regh`    | 高 8 位寄存器 | 高 8 位寄存器代号作为操作值 |
| `adxx`    | XX 偏移地址 | 16 位寄存器作为基址，16 位寄存器作为偏移量 `[regx + regx]` |
| `adxb`    | XB 偏移地址 | 16 位立即数作为基址，8 位寄存器作为偏移量 `[regx + regb]` |
| `adab`    | 绝对地址 | 16 位立即数直接作为地址 `[immx]` |
| `addb`    | 双偏移地址 | 16 位寄存器作为基址，16 位寄存器作为偏移量，8 位立即数作为第二偏移量 `[regx + regx + immb]` |

## 内存模型
该处理器使用平坦内存，以 16 位地址寻址的 16KB 作为全部内存空间直接寻址。
可通过扩展集实现 16 KB/PAGE * 65536 PAGES = 4 GB 的内存访问空间。

## 寄存器
### 16 位寄存器
| 助记符 | reg | regp | regx | 描述 | 备注 |
| :-:           | :-: | :-: | :-: | :-- | :-- |
| `AX`          | 000 | | 000 | 运算寄存器 A | |
| `BX`          | 001 | | 001 | 运算寄存器 B | |
| `CX`          | 002 | | 002 | 计数寄存器 | 用于 `REPCx` 循环控制 |
| `DX`          | 003 | | 003 | 数据寄存器 | |
| `EX`          | 004 | | 004 | 溢出寄存器 | 用于乘法、除法运算的溢出量 |
| `FG`          | 005 | | 005 | 标志寄存器 | 用于存储状态标志 |
| `XX`          | 006 | | 006 | 拓展通用寄存器 |
| `LX`          | 007 | | 007 | 逻辑寄存器 | 用于逻辑运算，`REPLx` 循环控制 |
| `CP`          | 010 | 000 | | 代码指针寄存器 | 下一条指令的起始地址（自动增长） |
| `DP`          | 011 | 001 | | 数据指针寄存器 | |
| `SP`          | 012 | 002 | | 栈顶指针寄存器 | 当前栈顶地址（自动向下增长，字对齐） |
| `SI`          | 013 | 003 | | 源索引寄存器 | 批量传输时的源地址（自动增长） |
| `XP / CS`   | 014 | 004 | | 拓展指针寄存器 X | 启用地址扩展是作为代码页面选择子寄存器 |
| `YP / DS`   | 015 | 005 | | 拓展指针寄存器 Y | 启用地址扩展是作为数据页面选择子寄存器 |
| `ZP / SS`   | 016 | 006 | | 拓展指针寄存器 Z | 启用地址扩展是作为栈顶页面选择子寄存器 |
| `DI`          | 017 | 007 | | 目的索引寄存器 | 批量传输时的目的地址（自动增长） |
### 8 位寄存器
只有通用寄存器有对应的 8 位寄存器，记为 `?L` 和 `?H`，分别表示低 8 位和高 8 位。
其编号通用寄存器编号，如果是 `reg8` 的高位寄存器，则需要加上 8。
例如：
- `AL` 的 `regb` 编号为 `000`，`regl` 编号为 `000`
- `BH` 的 `regb` 编号为 `011`，`regh` 编号为 `001`

### 标志寄存器 `FG`
#### 位定义
| 位 | 助记符 | 描述 |
| :-: | :-: | :--- |
| 0 | `ZF` | 零标志 |
| 1 | `IF` | 中断许可标志 |
| 2 | `SF` | 符号标志 |
| 3 | `PF` | 奇偶标志 |
| 4 | `CF` | 进位标志 |
| 5 | `OF` | 溢出标志 |
| 6 | `EF` | 错误标志 |
| 7 | `DF` | 调试标志 |
#### 条件码
| 值 | 助记符 | 条件 | 描述 |
| :-: | :-:                 | :--                       | :-- |
| 000 | `Z / E / F`         | `ZF = 0`                  | 零、等于 |
| 001 | `NI`                | `IF = 0`                  | 中断禁止 |
| 002 | `NS`                | `SF = 0`                  | 正数 |
| 003 | `NP`                | `PF = 0`                  | 偶数 |
| 004 | `NC / NB / AE`      | `CF = 0`                  | 无进位、无符号不小于、无符号大于等于 |
| 005 | `NO`                | `OF = 0`                  | 无溢出 |
| 006 | `OK`                | `EF = 0`                  | 无错误 |
| 010 | `NZ / NE / T`       | `ZF = 1`                  | 非零、不等于 |
| 011 | `I`                 | `IF = 1`                  | 中断许可 |
| 012 | `S`                 | `SF = 1`                  | 负数 |
| 013 | `P`                 | `PF = 1`                  | 奇数 |
| 014 | `C / B / LE`        | `CF = 1`                  | 进位、无符号小于、无符号大于等于 |
| 015 | `O`                 | `OF = 1`                  | 溢出 |
| 016 | `ER`                | `EF = 1`                  | 错误 |
| 017 | `DBG`               | `DF = 1`                  | 调试 |
| 020 | `NL / GE`           | `SF = OF`                 | 有符号不小于、有符号大于等于 |
| 021 | `G`                 | `ZF = 1 & SF = OF`        | 有符号大于 |
| 022 | `L / GT`            | `SF != OF`                | 有符号小于、有符号大于等于 |
| 023 | `NG / LE`           | `ZF = 0 \| SF != OF`      | 有符号不大于、有符号小于等于 |
| 024 | `NA / BE`           | `ZF = 0 \| CF = 1`        | 无符号不大于、无符号小于等于 |
| 025 | `A`                 | `ZF = 1 & CF = 0`         | 无符号大于 |
| 030 | `CF`                | `CX = 0`                  | 计数寄存器为零 |
| 031 | `CT`                | `CX != 0`                 | 计数寄存器不为零 |
| 032 | `LF`                | `LX = 0`                  | 逻辑寄存器为假 |
| 033 | `LT`                | `LX!= 0`                  | 逻辑寄存器为真 |
### 内部寄存器
| 助记符 | 尺寸 | 描述 | 备注 |
| :-:   | :-:   | :-- | :-- |
| `INS` | 8 B   | 指令缓冲寄存器 | 队列结构 |
| `STT` |       | 状态寄存器 | |
#### `STT` 状态寄存器
| 位 | 助记符 | 描述 |
| :-: | :-:     | :-- |
| 0 | `BUSY`    | 处理器忙 |
| 1 | `LOCK`    | 处理器锁定 |
| 2 | `POWER`   | 处理器电源 |
| ... | | 未定义，依照具体实现需要 |

## 指令集
| 指令 | 助记符 | 描述 |
| :--                                       | :--                                       | :-- |
| `0000 0000`                               | `NOP`                                     | 空操作 |
| `0000 0001`                               | `HLT`                                     | 处理器停止 |
| `0000 0010 0000 xxxx`                     | `FCL x<immh>`                             | 清除标志 |
| `0000 0010 0001 xxxx`                     | `FST x<immh>`                             | 设置标志 |
| `0000 0010 010x xxxx <...>`               | `IFx y<command>`                          | 满足条件则执行 |
| `0000 0010 011x xxxx <...>`               | `REPx y<command>`                         | 满足条件则反复执行 |
| `0000 0011`                               | `SYNC`                                    | 同步操作 |
| `0000 0100`                               | `LOCK`                                    | 锁定操作 |
| `0000 0101`                               | `UNLOCK`                                  | 解锁操作 |
| `0000 1000 xxxx yyyy`                     | `CALL x<regx>, y<regx>`                   | 调用 |
| `0000 1001 xxxx yyyy`                     | `CALL x<regx>, y<regb>`                   | 调用 |
| `0000 1010 xxxx yyyy zzzz zzzz`           | `CALL x<regx>, y<regx>, z<immb>`          | 调用 |
| `0000 1011 xxxx xxxx xxxx xxxx`           | `CALL x<immx>`                            | 调用 |
| `0000 1100 xxxx xxxx`                     | `INT x<immx>`                             | 调用软中断 |
| `0000 1110`                               | `RET`                                     | 从调用返回 |
| `0000 1111`                               | `RETI`                                    | 从软中断返回 |
| `0001 0000 0000 xxxx yyyy zzzz`           | `LD x<regx>, [y<regx>, z<regx>]`          | 从内存中加载字到寄存器 |
| `0001 0000 0001 xxxx yyyy zzzz`           | `LD x<regx>, [y<regx>, z<regb>]`          | 从内存中加载字到寄存器 |
| `0001 0000 0010 xxxx yyyy zzzz wwww wwww` | `LD x<regx>, [y<regx>, z<regx>, w<immb>]` | 从内存中加载字到寄存器 |
| `0001 0000 0011 xxxx yyyy yyyy yyyy yyyy` | `LD x<regx>, [y<immx>]`                   | 从内存中加载字到寄存器 |
| `0001 0000 0100 xxxx yyyy zzzz`           | `LD x<regb>, [y<regx>, z<regx>]`          | 从内存中加载字节到寄存器 |
| `0001 0000 0101 xxxx yyyy zzzz`           | `LD x<regb>, [y<regx>, z<regb>]`          | 从内存中加载字节到寄存器 |
| `0001 0000 0110 xxxx yyyy zzzz wwww wwww` | `LD x<regb>, [y<regx>, z<regx>, w<immb>]` | 从内存中加载字节到寄存器 |
| `0001 0000 0111 xxxx yyyy yyyy yyyy yyyy` | `LD x<regb>, [y<immx>]`                   | 从内存中加载字节到寄存器 |
| `0001 0000 1000 xxxx yyyy zzzz`           | `ST x<regx>, [y<regx>, z<regx>]`          | 从寄存器中存储字到内存 |
| `0001 0000 1001 xxxx yyyy zzzz`           | `ST x<regx>, [y<regx>, z<regb>]`          | 从寄存器中存储字到内存 |
| `0001 0000 1010 xxxx yyyy zzzz wwww wwww` | `ST x<regx>, [y<regx>, z<regx>, w<immb>]` | 从寄存器中存储字到内存 |
| `0001 0000 1011 xxxx yyyy yyyy yyyy yyyy` | `ST x<regx>, [y<immx>]`                   | 从寄存器中存储字到内存 |
| `0001 0000 1100 xxxx yyyy zzzz`           | `ST x<regb>, [y<regx>, z<regx>]`          | 从寄存器中存储字节到内存 |
| `0001 0000 1101 xxxx yyyy zzzz`           | `ST x<regb>, [y<regx>, z<regb>]`          | 从寄存器中存储字节到内存 |
| `0001 0000 1110 xxxx yyyy zzzz wwww wwww` | `ST x<regb>, [y<regx>, z<regx>, w<immb>]` | 从寄存器中存储字节到内存 |
| `0001 0000 1111 xxxx yyyy yyyy yyyy yyyy` | `ST x<regb>, [y<immx>]`                   | 从寄存器中存储字节到内存 |
| `0001 0001 0000 xxxx`                     | `LD x<regx>`                              | 加载 `[DS]SI` 到 16 位寄存器，并自增 |
| `0001 0001 0001 xxxx`                     | `LD x<regb>`                              | 加载 `[DS]SI` 到 8 位寄存器，并自增 |
| `0001 0001 0010 xxxx`                     | `PUSH x<regx>`                            | 压入 16 位寄存器到栈顶 |
| `0001 0001 0011 xxxx`                     | `PUSH x<regb>`                            | 压入 8 位寄存器到栈顶 |
| `0001 0001 0100 xxxx yyyy yyyy yyyy yyyy` | `LD x<regx>, y<immx>`                     | 加载立即数到 16 位寄存器 |
| `0001 0001 0101 xxxx yyyy yyyy`           | `LD x<regb>, y<immb>`                     | 加载立即数到 8 位寄存器 |
| `0001 0001 0110 0000`                     | `PUSHX`                                   | 压入所有 16 位寄存器 |
| `0001 0001 0110 0001`                     | `PUSHA`                                   | 压入所有通用寄存器 |
| `0001 0001 0110 0010`                     | `PUSHP`                                   | 压入所有指针寄存器 |
| `0001 0001 0110 0011`                     | `PUSHS`                                   | 依次压入 `FG LG CP DP SP SI XP YP ZP DI` 寄存器 |
| `0001 0001 0110 0100`                     | `POPX`                                    | 弹出所有 16 位寄存器 |
| `0001 0001 0110 0101`                     | `POPA`                                    | 弹出所有通用寄存器 |
| `0001 0001 0110 0110`                     | `POPP`                                    | 弹出所有指针寄存器 |
| `0001 0001 0110 0111`                     | `POPS`                                    | 依次弹出 `DI ZP YP XP SP SI DP CP LG FG` 寄存器 |
| `0001 0001 1000 xxxx`                     | `ST x<regx>`                              | 存储 16 位寄存器到 `[DS]DI`，并自增 |
| `0001 0001 1001 xxxx`                     | `ST x<regb>`                              | 存储 8 位寄存器到 `[DS]DI`，并自增 |
| `0001 0001 1010 xxxx`                     | `POP x<regx>`                             | 弹出栈顶到 16 位寄存器 |
| `0001 0001 1011 xxxx`                     | `POP x<regb>`                             | 弹出栈顶到 8 位寄存器 |
| `0001 0001 1100 xxxx yyyy zzzz`           | `LA x<regx>, [y<regx>, z<regx>]`          | 计算地址并存入 16 位寄存器 |
| `0001 0001 1101 xxxx yyyy zzzz`           | `LA x<regx>, [y<regx>, z<regb>]`          | 计算地址并存入 16 位寄存器 |
| `0001 0001 1110 xxxx yyyy zzzz wwww wwww` | `LA x<regx>, [y<regx>, z<regx>, w<immb>]` | 计算地址并存入 16 位寄存器 |
| `0001 0001 1111 xxxx yyyy yyyy yyyy yyyy` | `LA x<regx>, [y<immx>]`                   | 计算地址并存入 16 位寄存器 |
| `0001 0010`                               | `LDSTX`                                   | 从 `[DS]SI` 转移一个字到 `[DS]DI`，并自增 |
| `0001 0011`                               | `LDSTB`                                   | 从 `[DS]SI` 转移一个字节到 `[DS]DI`，并自增 |
| `0001 0100 ____ xxxx`                     | `FLZ x<regx>`                             | 将 16 位寄存器清零 |
| `0001 0101 ____ xxxx`                     | `FLZ x<regb>`                             | 将 8 位寄存器清零 |
| `0001 0110 ____ xxxx`                     | `FLS x<regx>`                             | 将 16 位寄存器置满 |
| `0001 0111 ____ xxxx`                     | `FLS x<regb>`                             | 将 8 位寄存器置满 |
| `0010 0000 0000 0000 xxxx yyyy`           | `ADD x<regx>, y<regx>`                    | 加法 |
| `0010 0000 0000 0001 xxxx yyyy`           | `ADD x<regb>, y<regb>`                    | 加法 |
| `0010 0000 0000 0010 xxxx yyyy`           | `ADD x<regx>, y<regb>`                    | 加法 |
| `0010 0000 0000 0011 xxxx yyyy`           | `IADD x<regx>, y<regb>`                   | 有符号加法 |
| `0010 0000 0000 0100 xxxx yyyy`           | `SUB x<regx>, y<regx>`                    | 减法 |
| `0010 0000 0000 0101 xxxx yyyy`           | `SUB x<regb>, y<regb>`                    | 减法 |
| `0010 0000 0000 0110 xxxx yyyy`           | `SUB x<regx>, y<regb>`                    | 减法 |
| `0010 0000 0000 0111 xxxx yyyy`           | `ISUB x<regx>, y<regb>`                   | 有符号减法 |
| `0010 0000 0000 1000 xxxx yyyy`           | `SGN x<regx>, y<regx>`                    | 将 16 位寄存器的符号位扩展 |
| `0010 0000 0010 xxxx`                     | `NEG x<regx>`                             | 取相反数 |
| `0010 0000 0011 xxxx`                     | `NEG x<regb>`                             | 取相反数 |
| `0010 0000 0100 xxxx`                     | `INC x<regx>`                             | 自增 |
| `0010 0000 0101 xxxx`                     | `INC x<regb>`                             | 自增 |
| `0010 0000 0110 xxxx`                     | `DEC x<regx>`                             | 自减 |
| `0010 0000 0111 xxxx`                     | `DEC x<regb>`                             | 自减 |
| `0010 0000 1000 0000 xxxx yyyy`           | `MUL x<regx>, y<regx>`                    | 乘法 |
| `0010 0000 1000 0001 xxxx yyyy`           | `MUL x<regb>, y<regb>`                    | 乘法 |
| `0010 0000 1000 0010 xxxx yyyy`           | `MUL x<regx>, y<regb>`                    | 乘法 |
| `0010 0000 1000 0100 xxxx yyyy`           | `IMUL x<regx>, y<regx>`                   | 有符号乘法 |
| `0010 0000 1000 0101 xxxx yyyy`           | `IMUL x<regb>, y<regb>`                   | 有符号乘法 |
| `0010 0000 1000 0110 xxxx yyyy`           | `IMUL x<regx>, y<regb>`                   | 有符号乘法 |
| `0010 0000 1000 1000 xxxx yyyy`           | `MULE x<regx>, y<regx>`                   | 乘法，溢出到 `EX` |
| `0010 0000 1000 1001 xxxx yyyy`           | `MULE x<regb>, y<regb>`                   | 乘法，溢出到 `EX` |
| `0010 0000 1000 1010 xxxx yyyy`           | `MULE x<regx>, y<regb>`                   | 乘法，溢出到 `EX` |
| `0010 0000 1000 1100 xxxx yyyy`           | `IMULE x<regx>, y<regx>`                  | 有符号乘法，溢出到 `EX` |
| `0010 0000 1000 1101 xxxx yyyy`           | `IMULE x<regb>, y<regb>`                  | 有符号乘法，溢出到 `EX` |
| `0010 0000 1000 1110 xxxx yyyy`           | `IMULE x<regx>, y<regb>`                  | 有符号乘法，溢出到 `EX` |
| `0010 0000 1001 0000 xxxx yyyy`           | `DIV x<regx>, y<regx>`                    | 除法，`x/y = x ... y` |
| `0010 0000 1001 0001 xxxx yyyy`           | `DIV x<regb>, y<regb>`                    | 除法，`x/y = x ... y` |
| `0010 0000 1001 0010 xxxx yyyy`           | `DIV x<regx>, y<regb>`                    | 除法，`x/y = x ... EX` |
| `0010 0000 1001 0100 xxxx yyyy`           | `IDIV x<regx>, y<regx>`                   | 有符号除法，`x/y = x ... y` |
| `0010 0000 1001 0101 xxxx yyyy`           | `IDIV x<regb>, y<regb>`                   | 有符号除法，`x/y = x ... y` |
| `0010 0000 1001 0110 xxxx yyyy`           | `IDIV x<regx>, y<regb>`                   | 有符号除法，`x/y = x ... EX` |
| `0010 0000 1001 1000 xxxx yyyy`           | `DIVE x<regx>, y<regx>`                   | 除法，`x/y = x ... y` |
| `0010 0000 1001 1001 xxxx yyyy`           | `DIVE x<regb>, y<regb>`                   | 除法，`x/y = x ... y` |
| `0010 0000 1001 1010 xxxx yyyy`           | `DIVE x<regx>, y<regb>`                   | 除法，`x/y = x ... EX` |
| `0010 0000 1001 1100 xxxx yyyy`           | `IDIVE x<regx>, y<regx>`                  | 有符号除法，`x/y = x ... y` |
| `0010 0000 1001 1101 xxxx yyyy`           | `IDIVE x<regb>, y<regb>`                  | 有符号除法，`x/y = x ... y` |
| `0010 0000 1001 1110 xxxx yyyy`           | `IDIVE x<regx>, y<regb>`                  | 有符号除法，`x/y = x ... EX` |
| `0010 0001 0000 xxxx`                     | `NOT x<regx>`                             | 取反 |
| `0010 0001 0001 xxxx`                     | `NOT x<regb>`                             | 取反 |
| `0010 0001 0010 0000 xxxx yyyy`           | `SHL x<regx>, y<regx>`                    | 左移 |
| `0010 0001 0010 0001 xxxx yyyy`           | `SHL x<regx>, y<regb>`                    | 左移 |
| `0010 0001 0010 0010 xxxx yyyy`           | `SHL x<regb>, y<regx>`                    | 左移 |
| `0010 0001 0010 0011 xxxx yyyy`           | `SHL x<regb>, y<regb>`                    | 左移 |
| `0010 0001 0010 0100 xxxx yyyy`           | `SHR x<regx>, y<regx>`                    | 右移 |
| `0010 0001 0010 0101 xxxx yyyy`           | `SHR x<regx>, y<regb>`                    | 右移 |
| `0010 0001 0010 0110 xxxx yyyy`           | `SHR x<regb>, y<regx>`                    | 右移 |
| `0010 0001 0010 0111 xxxx yyyy`           | `SHR x<regb>, y<regb>`                    | 右移 |
| `0010 0001 0010 1000 xxxx yyyy`           | `SAL x<regx>, y<regx>`                    | 算术左移 |
| `0010 0001 0010 1001 xxxx yyyy`           | `SAL x<regx>, y<regb>`                    | 算术左移 |
| `0010 0001 0010 1010 xxxx yyyy`           | `SAL x<regb>, y<regx>`                    | 算术左移 |
| `0010 0001 0010 1011 xxxx yyyy`           | `SAL x<regb>, y<regb>`                    | 算术左移 |
| `0010 0001 0010 1100 xxxx yyyy`           | `SAR x<regx>, y<regx>`                    | 算术右移 |
| `0010 0001 0010 1101 xxxx yyyy`           | `SAR x<regx>, y<regb>`                    | 算术右移 |
| `0010 0001 0010 1110 xxxx yyyy`           | `SAR x<regb>, y<regx>`                    | 算术右移 |
| `0010 0001 0010 1111 xxxx yyyy`           | `SAR x<regb>, y<regb>`                    | 算术右移 |
| `0010 0001 0011 0000 xxxx yyyy`           | `ROL x<regx>, y<regx>`                    | 循环左移 |
| `0010 0001 0011 0001 xxxx yyyy`           | `ROL x<regx>, y<regb>`                    | 循环左移 |
| `0010 0001 0011 0010 xxxx yyyy`           | `ROL x<regb>, y<regx>`                    | 循环左移 |
| `0010 0001 0011 0011 xxxx yyyy`           | `ROL x<regb>, y<regb>`                    | 循环左移 |
| `0010 0001 0011 0100 xxxx yyyy`           | `ROR x<regx>, y<regx>`                    | 循环右移 |
| `0010 0001 0011 0101 xxxx yyyy`           | `ROR x<regx>, y<regb>`                    | 循环右移 |
| `0010 0001 0011 0110 xxxx yyyy`           | `ROR x<regb>, y<regx>`                    | 循环右移 |
| `0010 0001 0011 0111 xxxx yyyy`           | `ROR x<regb>, y<regb>`                    | 循环右移 |
| `0010 0010 xxxx yyyy`                     | `AND x<regx>, y<regx>`                    | 与 |
| `0010 0011 xxxx yyyy`                     | `AND x<regb>, y<regb>`                    | 与 |
| `0010 0100 xxxx yyyy`                     | `OR x<regx>, y<regx>`                     | 或 |
| `0010 0101 xxxx yyyy`                     | `OR x<regb>, y<regb>`                     | 或 |
| `0010 0110 xxxx yyyy`                     | `XOR x<regx>, y<regx>`                    | 异或 |
| `0010 0111 xxxx yyyy`                     | `XOR x<regb>, y<regb>`                    | 异或 |
| `0010 1000 xxxx yyyy`                     | `SHL x<regx>, y<immh>`                    | 左移 |
| `0010 1001 xxxx _yyy`                     | `SHL x<regb>, y<immh>`                    | 左移 |
| `0010 1010 xxxx yyyy`                     | `SHR x<regx>, y<immh>`                    | 右移 |
| `0010 1011 xxxx _yyy`                     | `SHR x<regb>, y<immh>`                    | 右移 |
| `0010 1100 xxxx yyyy`                     | `SAL x<regx>, y<immh>`                    | 算术左移 |
| `0010 1101 xxxx _yyy`                     | `SAL x<regb>, y<immh>`                    | 算术左移 |
| `0010 1110 xxxx yyyy`                     | `SAR x<regx>, y<immh>`                    | 算术右移 |
| `0010 1111 xxxx _yyy`                     | `SAR x<regb>, y<immh>`                    | 算术右移 |
| `0011 0000 xxxx yyyy`                     | `CMP x<regx>, y<regx>`                    | 比较 |
| `0011 0001 xxxx yyyy`                     | `CMP x<regb>, y<regb>`                    | 比较 |
| `0011 0010 xxxx yyyy`                     | `TEST x<regx>, y<regx>`                   | 测试 |
| `0011 0011 xxxx yyyy`                     | `TEST x<regb>, y<regb>`                   | 测试 |
| `0011 0100 xxxx yyyy`                     | `ROL x<regx>, y<immh>`                    | 循环左移 |
| `0011 0101 xxxx _yyy`                     | `ROL x<regb>, y<immh>`                    | 循环左移 |
| `0011 0110 xxxx yyyy`                     | `ROR x<regx>, y<immh>`                    | 循环右移 |
| `0011 0111 xxxx _yyy`                     | `ROR x<regb>, y<immh>`                    | 循环右移 |
| `0100 0000 xxxx yyyy`                     | `OUT x<regx>, y<regx>`                    | 输出 |
| `0100 0001 xxxx yyyy`                     | `OUT x<regb>, y<regx>`                    | 输出 |
| `0100 0010 xxxx yyyy`                     | `OUT x<regx>, y<regb>`                    | 输出 |
| `0100 0011 xxxx yyyy`                     | `OUT x<regb>, y<regb>`                    | 输出 |
| `0100 0100 xxxx ____`                     | `OUTX x<regx>`                            | 输出 `[DS]SI` 处的字 |
| `0100 0101 xxxx ____`                     | `OUTX x<regb>`                            | 输出 `[DS]SI` 处的字 |
| `0100 0110 xxxx ____`                     | `OUTB x<regx>`                            | 输出 `[DS]SI` 处的字节 |
| `0100 0111 xxxx ____`                     | `OUTB x<regb>`                            | 输出 `[DS]SI` 处的字节 |
| `0100 1000 xxxx yyyy`                     | `IN x<regx>, y<regx>`                     | 输入 |
| `0100 1001 xxxx yyyy`                     | `IN x<regb>, y<regx>`                     | 输入 |
| `0100 1010 xxxx yyyy`                     | `IN x<regx>, y<regb>`                     | 输入 |
| `0100 1011 xxxx yyyy`                     | `IN x<regb>, y<regb>`                     | 输入 |
| `0100 1100 xxxx ____`                     | `INX x<regx>`                             | 输入字到 `[DS]DI` 处 |
| `0100 1101 xxxx ____`                     | `INX x<regb>`                             | 输入字到 `[DS]DI` 处 |
| `0100 1110 xxxx ____`                     | `INB x<regx>`                             | 输入字节到 `[DS]DI` 处 |
| `0100 1111 xxxx ____`                     | `INB x<regb>`                             | 输入字节到 `[DS]DI` 处 |
| `0101 0000 xxxx yyyy`                     | `IOX x<regx>, y<regx>`                    | 输入字并输出 |
| `0101 0001 xxxx yyyy`                     | `IOX x<regb>, y<regb>`                    | 输入字并输出 |
| `0101 0010 xxxx yyyy`                     | `IOX x<regx>, y<regx>`                    | 输入字到 `[DS]DI` 并输出 `[DS]SI` 处的字 |
| `0101 0011 xxxx yyyy`                     | `IOX x<regb>, y<regb>`                    | 输入字到 `[DS]DI` 并输出 `[DS]SI` 处的字 |
| `0101 0100 xxxx yyyy`                     | `IOB x<regx>, y<regx>`                    | 输入字节并输出 |
| `0101 0101 xxxx yyyy`                     | `IOB x<regb>, y<regb>`                    | 输入字节并输出 |
| `0101 0110 xxxx yyyy`                     | `IOB x<regx>, y<regx>`                    | 输入字节到 `[DS]DI` 并输出 `[DS]SI` 处的字节 |
| `0101 0111 xxxx yyyy`                     | `IOB x<regb>, y<regb>`                    | 输入字节到 `[DS]DI` 并输出 `[DS]SI` 处的字节 |
| `0101 1000 xxxx yyyy`                     | `OIX x<regx>, y<regx>`                    | 输出 `[DS]SI` 处的字并输入字到 `[DS]DI` |
| `0101 1001 xxxx yyyy`                     | `OIX x<regb>, y<regb>`                    | 输出 `[DS]SI` 处的字并输入字到 `[DS]DI` |
| `0101 1010 xxxx yyyy`                     | `OWIX x<regx>, y<regx>`                   | 输出 `[DS]SI` 处的字节，缓冲，并输入字节到 `[DS]DI` |
| `0101 1011 xxxx yyyy`                     | `OWIX x<regb>, y<regb>`                   | 输出 `[DS]SI` 处的字节，缓冲，并输入字节到 `[DS]DI` |
| `0101 1100 xxxx yyyy`                     | `OIB x<regx>, y<regx>`                    | 输出 `[DS]SI` 处的字节并输入字节到 `[DS]DI` |
| `0101 1101 xxxx yyyy`                     | `OIB x<regb>, y<regb>`                    | 输出 `[DS]SI` 处的字节并输入字节到 `[DS]DI` |
| `0101 1110 xxxx yyyy`                     | `OWIB x<regx>, y<regx>`                   | 输出 `[DS]SI` 处的字节，缓冲，并输入字节到 `[DS]DI` |
| `0101 1111 xxxx yyyy`                     | `OWIB x<regb>, y<regb>`                   | 输出 `[DS]SI` 处的字节，缓冲，并输入字节到 `[DS]DI` |

## 等效指令
| 指令 | 指令 |
| :-- | :-- |
| `JMP x<immx>` | `LD cp, x<immx>` |
| `JMP x<addr>` | `LA cp, x<addr>` |
