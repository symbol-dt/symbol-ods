# Minecraft 拓展指令集 (Minecraft Extended Instruction Set) - ODS 2.1

> - Author: [DiannaoJun](mailto:aheiwuchang@163.com)
> - Date: 2025/04/30 20:04 UTC+08
> - Version: 0.1.0

## 简介
作为一种较为简单的 Minecraft Computer Processor 设计方案。

## 内存模型
该处理器使用平坦内存，以 16 位地址寻址的 16KB 作为全部内存空间直接寻址。

## `STT` 状态寄存器
| 位 | 助记符 | 描述 |
| :-: | :-:     | :-- |
| 0 | `BUSY`    | 处理器忙 |
| 1 | `LOCK`    | 处理器锁定 |
| 2 | `POWER`   | 处理器电源 |

## 指令集
| 指令 | 助记符 | 描述 |
| :--                                       | :--                                       | :-- |
| `1111 0000`                               | `MC-LDX`                                  | 从纸带读取一个字到 `[DS]:DI` |
| `1111 0001`                               | `MC-LDB`                                  | 从纸带读取一个字节到 `[DS]:DI` |
