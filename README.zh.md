# heatshrink

English: [![English](https://img.shields.io/badge/lang-english-blue.svg)](README.md)

简体中文: [![简体中文](https://img.shields.io/badge/lang-chinese-red.svg)](README.zh.md)

一个用于嵌入式/实时系统的数据压缩/解压库。

## 主要特性：

- **低内存占用（最低仅需 50 字节）**  
  在某些场景下，它在小于 50 字节的内存下仍能使用，并且在多数常规场景下小于 300 字节就足够了。
- **增量式、受限的 CPU 使用率**  
  可以用任意小的数据块处理输入数据。这个特性在硬实时环境中非常有用。
- **可使用静态或动态内存分配**  
  库不对内存管理方式做任何限制。
- **ISC 许可协议**  
  可自由使用，包括商业用途。

## 快速上手：

虽然有一个独立的命令行程序 `heatshrink`，但编码器和解码器也可以作为独立库使用。若要在项目中使用，只需将 `heatshrink_common.h`、`heatshrink_config.h` 和 `heatshrink_encoder.c` 或 `heatshrink_decoder.c`（以及对应的头文件）拷贝进你的项目中。如果项目同时使用编码器和解码器，也可以构建使用静态或动态内存的静态库。

默认使用动态分配，但在嵌入式环境中，你可能更倾向于使用静态分配。可在 `heatshrink_config.h` 中将 `HEATSHRINK_DYNAMIC_ALLOC` 设置为 0 来启用静态分配。

### 基本用法

1. 使用 `alloc` 函数分配 `heatshrink_encoder` 或 `heatshrink_decoder` 状态机，或者静态分配后用 `reset` 函数初始化。（配置选项见下文）

2. 使用 `sink` 函数将输入缓冲区的数据传入状态机。`input_size` 指针参数会被设置为实际消费的输入字节数。（如果是 0，说明缓冲区已满）

3. 使用 `poll` 函数将输出从状态机中转移到输出缓冲区。`output_size` 指针参数会显示输出了多少字节，函数的返回值指示是否还有更多输出。（状态机可能在未收到足够输入前不会输出数据）

重复步骤 2 和 3 来持续流式处理数据。由于是压缩操作，输入输出大小可能会有显著变化。在处理过程中，循环缓冲输入输出是必要的。

4. 当输入流结束时，调用 `finish` 通知状态机没有更多输入了。`finish` 的返回值会指示是否还有剩余输出可取，如有，继续调用 `poll` 获取。

继续交替调用 `finish` 和 `poll`，直到 `finish` 指示输出已全部完成。

一旦调用过 `finish`，再次进行 `sink` 输入操作将不会生效，除非先对状态机调用 `reset`。

## 配置选项

heatshrink 提供了若干配置项，这些选项会影响其资源使用和压缩效率。对于动态分配，可以在调用时设置；对于静态分配，在 `heatshrink_config.h` 中设置。

- `window_sz2`（命令行参数 `-w`）：设置窗口大小为 2^W 字节。  

  窗口大小决定了输入中可以回溯查找重复模式的最大距离。例如 `window_sz2` 设置为 8 时，窗口为 256 字节（2^8），而设置为 10 时为 1024 字节（2^10）。更大的窗口能提高压缩效率，但也需要更多内存。

  当前 `window_sz2` 的合法值范围为 4 到 15。

- `lookahead_sz2`（命令行参数 `-l`）：设置前瞻大小为 2^L 字节。  

  前瞻大小决定了压缩时可识别的最大重复模式长度。比如 `lookahead_sz2` 为 4，那么一段 50 字节的连续字符 'a' 会被分成多个 16 字节的重复块（2^4=16），而更大的前瞻值可能能一次性压缩全部。但由于使用的是定长比特表示，过大的前瞻值可能因浪费比特而导致压缩效果变差。

  `lookahead_sz2` 当前必须在 3 到 `window_sz2 - 1` 之间。

- `input_buffer_size`：解码器的输入缓冲区大小。影响每步能处理的数据量。更大的缓冲区消耗更多内存，太小（例如 1 字节）会频繁进入 suspend/resume 状态，但不会影响压缩比。

### 推荐默认值

对于嵌入式/低内存环境，建议使用 8 到 10 之间的 `window_sz2`，具体视内存限制而定。也可针对具体数据调整以取得最佳效果。

`lookahead_sz2` 建议从 `window_sz2 / 2` 开始设定，例如：`-w 8 -l 4` 或 `-w 10 -l 5`。可使用命令行程序测试不同参数对数据压缩效果的影响。

## 更多信息与基准测试：

heatshrink 基于 [LZSS] 算法，非常适合在低内存环境中使用。可选的轻量级 [索引机制] 可大幅加快压缩速度，但即使不使用索引，也可以在低于 100 字节内存下运行。启用索引时会额外使用 2^(窗口大小 + 1) 字节内存，并在构建索引时临时占用 512 字节栈空间。

更多信息见这篇 [博客文章]。API 说明请参阅 `heatshrink_encoder.h` 和 `heatshrink_decoder.h` 头文件。

[博客文章]: http://spin.atomicobject.com/2013/03/14/heatshrink-embedded-data-compression/  
[索引机制]: http://spin.atomicobject.com/2014/01/13/lightweight-indexing-for-embedded-systems/  
[LZSS]: http://en.wikipedia.org/wiki/Lempel-Ziv-Storer-Szymanski  

## 构建状态

[![Build Status](https://travis-ci.org/atomicobject/heatshrink.png)](http://travis-ci.org/atomicobject/heatshrink)
