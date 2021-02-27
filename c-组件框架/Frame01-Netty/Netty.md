# Netty

为什么选择Netty?

* I/O 模型、线程模型和事件处理机制
* 易用性API接口
* 对数据协议、序列化的支持



**I/O 模型**

I/O 请求可以分为两个阶段，分别为调用阶段和执行阶段。

* 第一个阶段为I/O 调用阶段，即用户进程向内核发起系统调用。

* 第二个阶段为I/O 执行阶段。此时，内核等待 I/O 请求处理完成返回。该阶段分为两个过程：首先等待数据就绪，并写入内核缓冲区；随后将内核缓冲区数据拷贝至用户态缓冲区。

![img](images/Netty/Ciqc1F-NAZ6Ae3bPAAHigveMsIQ514.png)

 **Linux 的 5 种主要 I/O 模式**

1.  同步阻塞 I/O（BIO）

   <img src="images/Netty/CgqCHl-OnUKAeEELAAEnHU3FHGA343.png" alt="1.png" style="zoom: 67%;" />

   如上图所表现的那样，应用进程向内核发起 I/O 请求，发起调用的线程一直等待内核返回结果。一次完整的 I/O 请求称为BIO（Blocking IO，阻塞 I/O），所以 BIO 在实现异步操作时，只能使用多线程模型，一个请求对应一个线程。但是，线程的资源是有限且宝贵的，创建过多的线程会增加线程切换的开销。

2. 同步非阻塞 I/O（NIO）

   <img src="images/Netty/Ciqc1F-OnTeAFLNhAAFptS-OxRY266.png" alt="2.png" style="zoom:67%;" />

   应用进程向内核发起 I/O 请求后不再会同步等待结果，而是会立即返回，通过轮询的方式获取请求结果。NIO 相比 BIO 虽然大幅提升了性能，但是轮询过程中大量的系统调用导致上下文切换开销很大。所以，单独使用非阻塞 I/O 时效率并不高，而且随着并发量的提升，非阻塞 I/O 会存在严重的性能浪费。

3. I/O 多路复用

   <img src="images/Netty/CgqCHl-OnV2ADXBhAAFUZ6oiz6U529.png" alt="3.png" style="zoom:67%;" />

   多路复用实现了一个线程处理多个 I/O 句柄的操作。多路指的是多个数据通道，复用指的是使用一个或多个固定线程来处理每一个 Socket。select、poll、epoll 都是 I/O 多路复用的具体实现，线程一次 select 调用可以获取内核态中多个数据通道的数据状态。多路复用解决了同步阻塞 I/O 和同步非阻塞 I/O 的问题，是一种非常高效的 I/O 模型。

4. 信号驱动 I/O

   <img src="images/Netty/CgqCHl-OnWqAddLWAAFUtZ6YHDA683.png" alt="4.png" style="zoom:67%;" />

   信号驱动 I/O 并不常用，它是一种半异步的 I/O 模型。在使用信号驱动 I/O 时，当数据准备就绪后，内核通过发送一个 SIGIO 信号通知应用进程，应用进程就可以开始读取数据了。

5. 异步 I/O

   <img src="images/Netty/Ciqc1F-OnXSAHOGVAACvxV3_3Mk188.png" alt="5.png" style="zoom:67%;" />

   异步 I/O 最重要的一点是从内核缓冲区拷贝数据到用户态缓冲区的过程也是由系统异步完成，应用进程只需要在指定的数组中引用数据即可。

   > 异步 I/O 与信号驱动 I/O 这种半异步模式的主要区别：信号驱动 I/O 由内核通知何时可以开始一个 I/O 操作，而异步 I/O 由内核通知 I/O 操作何时已经完成。



Netty 的 I/O 模型是基于非阻塞 I/O 实现的，底层依赖的是 JDK NIO 框架的多路复用器 Selector。一个多路复用器 Selector 可以同时轮询多个 Channel，采用 epoll 模式后，只需要一个线程负责 Selector 的轮询，就可以接入成千上万的客户端。

## 编码

### 通信协议

所谓协议，就是**通信双方事先商量好的接口暗语**。目前主流的**通用协议**(通信)如 HTTP、HTTPS、JSON-RPC、FTP、IMAP、Protobuf 等。**通用协议**兼容性好，易于维护，各种异构系统之间可以实现无缝对接。

如果通用协议不能满足业务场景，可以自定义通信协议，自定义通信协议有以下优点：

* **极致性能**：通用的通信协议考虑了很多兼容性的因素，必然在性能方面有所损失
* **扩展性**：自定义的协议相比通用协议更好扩展，可以更好地满足自己的业务需求。
* **安全性**：通用协议是公开的，自定义协议更加安全

一个完备的网络协议需具备的基本要素：

![image-20210227153334757](images/Netty/image-20210227153334757.png)

### Netty 中的通信协议

![image-20210227161102754](images/Netty/image-20210227161102754.png)

### Netty 中支持的常用解码器

**固定长度解码器 FixedLengthFrameDecoder**

通过构造函数设置固定长度的大小 frameLength，无论接收方一次获取多大的数据，都会严格按照 frameLength 进行解码。通过构造函数设置固定长度的大小 frameLength，无论接收方一次获取多大的数据，都会严格按照 frameLength 进行解码。

* 构造器 
    ```java
    FixedLengthFrameDecoder(int frameLength)
    ```

**特殊分隔符解码器 DelimiterBasedFrameDecoder**

DelimiterBasedFrameDecoder 中的几个属性及作用

* **delimiters**：指定特殊分隔符，类型为 ByteBuf数组 的**入参**，可指定多个分割符，但最终会选择最短的分隔符进行拆分
* **maxLength**: 报文最大长度的限制。如果超过 maxLength 还没有检测到指定分隔符，将会抛出 TooLongFrameException
* **failFast**：failFast 与 maxLength 需要搭配使用，通过设置 failFast 可以控制抛出 TooLongFrameException 的时机，如果为 true 则立即抛出，否则当解码出一个完成的消息后抛出
* **stripDelimiter**：判断解码后得到的消息是否去除分隔符

------
* 构造器

    ```java
    DelimiterBasedFrameDecoder(
                int maxFrameLength, 
                boolean stripDelimiter, 
                boolean failFast, 
                ByteBuf... delimiters)
    ```

**长度域解码器 LengthFieldBasedFrameDecoder**

长度域解码器 LengthFieldBasedFrameDecoder 是解决 TCP 拆包/粘包问题最常用的**解码器**。

LengthFieldBasedFrameDecoder 包含的属性

* 长度域解码器特有属性

  ```java
  // 长度字段的偏移量，也就是存放长度数据的起始位置
  private final int lengthFieldOffset; 
  // 长度字段所占用的字节数
  private final int lengthFieldLength; 
  /* 消息长度的修正值
   *
   * 在很多较为复杂一些的协议设计中，长度域不仅仅包含消息的长度，而且包含其他的数据，如版本号、数据类型、数据状态等，那么这时候我们需要使用 lengthAdjustment 进行修正
   * 
   * lengthAdjustment = 包体的长度值 - 长度域的值
   */
  private final int lengthAdjustment; 
  // 解码后需要跳过的初始字节数，也就是消息内容字段的起始位置
  private final int initialBytesToStrip;
  // 长度字段结束的偏移量，lengthFieldEndOffset = lengthFieldOffset + lengthFieldLength
  private final int lengthFieldEndOffset;
  ```

* 固定长度解码器和特定分隔符解码器相似的属性

  ```java
  private final int maxFrameLength; // 报文最大限制长度
  private final boolean failFast; // 是否立即抛出 TooLongFrameException，与 maxFrameLength 搭配使用
  private boolean discardingTooLongFrame; // 是否处于丢弃模式
  private long tooLongFrameLength; // 需要丢弃的字节数
  private long bytesToDiscard; // 累计丢弃的字节数
  ```

------

* 构造器

  ```java
  public LengthFieldBasedFrameDecoder(
      ByteOrder byteOrder, 
      int maxFrameLength, 
      int lengthFieldOffset, 
      int lengthFieldLength,     
      int lengthAdjustment, 
      int initialBytesToStrip, 
      boolean failFast)
  ```


长度域解码器 示例

* 示例 1：典型的基于消息长度 + 消息内容的解码

  ```java
    BEFORE DECODE (14 bytes)         AFTER DECODE (14 bytes)
    +--------+----------------+      +--------+----------------+
    | Length | Actual Content |----->| Length | Actual Content |
    | 0x000C | "HELLO, WORLD" |      | 0x000C | "HELLO, WORLD" |
    +--------+----------------+      +--------+----------------+
        
  lengthFieldOffset = 0，因为 Length 字段就在报文的开始位置。
  lengthFieldLength = 2，协议设计的固定长度。
  lengthAdjustment = 0，Length 字段只包含消息长度，不需要做任何修正。
  initialBytesToStrip = 0，解码后内容依然是 Length + Content，不需要跳过任何初始字节。
  ```

* 示例 2：解码结果需要截断

  ```java
    BEFORE DECODE (14 bytes)         AFTER DECODE (12 bytes)
    +--------+----------------+      +----------------+
    | Length | Actual Content |----->| Actual Content |
    | 0x000C | "HELLO, WORLD" |      | "HELLO, WORLD" |
    +--------+----------------+      +----------------+
  
  lengthFieldOffset = 0，因为 Length 字段就在报文的开始位置。
  lengthFieldLength = 2，协议设计的固定长度。
  lengthAdjustment = 0，Length 字段只包含消息长度，不需要做任何修正。
  initialBytesToStrip = 2，跳过 Length 字段的字节长度，解码后 ByteBuf 中只包含 Content字段。
  ```

* 示例 3：长度字段包含消息长度和消息内容所占的字节

  ```java
  BEFORE DECODE (14 bytes)         AFTER DECODE (14 bytes)
  +--------+----------------+      +--------+----------------+
  | Length | Actual Content |----->| Length | Actual Content |
  | 0x000E | "HELLO, WORLD" |      | 0x000E | "HELLO, WORLD" |
  +--------+----------------+      +--------+----------------+
  lengthFieldOffset = 0，因为 Length 字段就在报文的开始位置。
  lengthFieldLength = 2，协议设计的固定长度。
  lengthAdjustment = -2，长度字段为 14 字节，需要减 2 才是拆包所需要的长度。
  initialBytesToStrip = 0，解码后内容依然是 Length + Content，不需要跳过任何初始字节。
  ```

  > 与前两个示例不同的是，示例 3 的 Length 字段包含 Length 字段自身的固定长度以及 Content 字段所占用的字节数，Length 的值为 0x000E（2 + 12 = 14 字节），在 Length 字段值（14 字节）的基础上做 lengthAdjustment（-2）的修正，才能得到真实的 Content 字段长度

* 示例 4：基于长度字段偏移的解码

  ```java
  BEFORE DECODE (17 bytes)                      AFTER DECODE (17 bytes)
  +----------+----------+----------------+      +----------+----------+----------------+
  | Header 1 |  Length  | Actual Content |----->| Header 1 |  Length  | Actual Content |
  |  0xCAFE  | 0x00000C | "HELLO, WORLD" |      |  0xCAFE  | 0x00000C | "HELLO, WORLD" |
  +----------+----------+----------------+      +----------+----------+----------------+
  lengthFieldOffset = 2，需要跳过 Header 1 所占用的 2 字节，才是 Length 的起始位置。
  lengthFieldLength = 3，协议设计的固定长度。
  lengthAdjustment = 0，Length 字段只包含消息长度，不需要做任何修正。
  initialBytesToStrip = 0，解码后内容依然是完整的报文，不需要跳过任何初始字节。
  ```

  Length 字段不再是报文的起始位置，Length 字段的值为 0x00000C，表示 Content 字段占用 12 字节

* 示例 5：长度字段与内容字段不再相邻

  ```java
  BEFORE DECODE (17 bytes)                      AFTER DECODE (17 bytes)
  +----------+----------+----------------+      +----------+----------+----------------+
  |  Length  | Header 1 | Actual Content |----->|  Length  | Header 1 | Actual Content |
  | 0x00000C |  0xCAFE  | "HELLO, WORLD" |      | 0x00000C |  0xCAFE  | "HELLO, WORLD" |
  +----------+----------+----------------+      +----------+----------+----------------+
  lengthFieldOffset = 0，因为 Length 字段就在报文的开始位置。
  lengthFieldLength = 3，协议设计的固定长度。
  lengthAdjustment = 2，由于 Header + Content 一共占用 2 + 12 = 14 字节，所以 Length 字段值（12 字节）加上 lengthAdjustment（2 字节）才能得到 Header + Content 的内容（14 字节）。
  initialBytesToStrip = 0，解码后内容依然是完整的报文，不需要跳过任何初始字节
  ```

  > Length 字段之后是 Header 1，Length 与 Content 字段不再相邻。Length 字段所表示的内容略过了 Header 1 字段，所以也需要通过 lengthAdjustment 修正才能得到 Header + Content 的内容

* 示例 6：基于长度偏移和长度修正的解码

  ```java
  BEFORE DECODE (16 bytes)                       AFTER DECODE (13 bytes)
  +------+--------+------+----------------+      +------+----------------+
  | HDR1 | Length | HDR2 | Actual Content |----->| HDR2 | Actual Content |
  | 0xCA | 0x000C | 0xFE | "HELLO, WORLD" |      | 0xFE | "HELLO, WORLD" |
  +------+--------+------+----------------+      +------+----------------+
  lengthFieldOffset = 1，需要跳过 HDR1 所占用的 1 字节，才是 Length 的起始位置。
  lengthFieldLength = 2，协议设计的固定长度。
  lengthAdjustment = 1，由于 HDR2 + Content 一共占用 1 + 12 = 13 字节，所以 Length 字段值（12 字节）加上 lengthAdjustment（1）才能得到 HDR2 + Content 的内容（13 字节）。
  initialBytesToStrip = 3，解码后跳过 HDR1 和 Length 字段，共占用 3 字节。
  ```

* 示例 7：长度字段包含除 Content 外的多个其他字段

  ```java
  BEFORE DECODE (16 bytes)                       AFTER DECODE (13 bytes)
  +------+--------+------+----------------+      +------+----------------+
  | HDR1 | Length | HDR2 | Actual Content |----->| HDR2 | Actual Content |
  | 0xCA | 0x0010 | 0xFE | "HELLO, WORLD" |      | 0xFE | "HELLO, WORLD" |
  +------+--------+------+----------------+      +------+----------------+
  lengthFieldOffset = 1，需要跳过 HDR1 所占用的 1 字节，才是 Length 的起始位置。
  lengthFieldLength = 2，协议设计的固定长度。
  lengthAdjustment = -3，Length 字段值（16 字节）需要减去 HDR1（1 字节） 和 Length 自身所占字节长度（2 字节）才能得到 HDR2 和 Content 的内容（1 + 12 = 13 字节）。
  initialBytesToStrip = 3，解码后跳过 HDR1 和 Length 字段，共占用 3 字节。
  ```

  > 示例 7 与 示例 6 的区别在于 Length 字段记录了整个报文的长度，包含 Length 自身所占字节、HDR1 、HDR2 以及 Content 字段的长度，解码器需要知道如何进行 lengthAdjustment 调整，才能得到 HDR2 和 Content 的内容