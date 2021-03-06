---
title: TCP协议
date: 2018-06-05 14:57:39
tags:
- tcp 
- ip
- mac
- arp
- icmp
---
## TCP 协议

TRANSMISSION CONTROL PROTOCOL

### 网络包收发
![](/images/tcp-ip-mac.png)

包收发操作的起点是 TCP 模块委托 IP 模块发送包的操作。这个委托的过程就是 TCP 模块在数据块的前面加上 TCP 头部，然后整个传递给 IP 模块，这部分就是网络包的内容。与此同时， TCP 模块还需要指定通信对象的 IP 地址，也就是需要写清楚“将什么内容发给谁”。

收到委托后，IP 模块会将包的内容当作一整块数据，在前面加上包含控制信息的 IP 头部和 MAC 头部。IP 头部中包含 IP 协议规定的、根据 IP 地址将包发往目的地所需的控制信息；MAC 头部包含通过以太网的局域网将包传输至最近的路由器所需的控制信息。

封装好的包会传递给网卡，传递给网卡的网络包是由一连串 0 和 1 组成的数字信息，网卡会将这些数字信息转换为电信号或光信号，并通过网线（或光纤）发送出去，然后这些信号就会到达集线器、路由器等转发设备，再由转发设备一步一步地送达接收方。

### 连接-收发-断开

1. 连接

    以太网的网线都是一直连接的状态，我们并不需要来回插拔网线，那么这里`连接`到底是什么意思呢？连接实际上是通信双方交换控制信息，在套接字中记录这些必要信息并准备数据收发的一连串操作。

    ![](/images/tcp-connection.png)

2. 收发

    
3. 断开

    TCP 断开是双方协商的，任何一端都可以发起断开。套接字并不会立即被删除，而是会等待一段时间之后再被删除。
    ![](/images/tcp-close.png)
    
    > 断开需要双方各自发起并确认，共四步操作。有些情况下三步也能完成，第2、3步合并后和四步操作的效果是一样的。


### 协议分析

```
$ curl http://css.kekek.cc
```

(tcp.dstport == 80 and ip.dst == 47.94.202.145) or (ip.addr == 47.94.202.145 and tcp.port == 80)
![](/images/wireshark.png)

1. Frame:   物理层的数据帧概况
2. Ethernet II: 数据链路层以太网帧头部信息
3. Internet Protocol Version 4: 互联网层IP包头部信息
4. Transmission Control Protocol:  传输层的数据段头部信息，此处是TCP
5. Hypertext Transfer Protocol:  应用层的信息，此处是HTTP协议

[ TCP Header Format](https://tools.ietf.org/html/rfc793#section-3.1)
``` 
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|          Source Port          |       Destination Port        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                        Sequence Number                        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Acknowledgment Number                      |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|  Data |           |U|A|P|R|S|F|                               |
| Offset| Reserved  |R|C|S|S|Y|I|            Window             |
|       |           |G|K|H|T|N|N|                               |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|           Checksum            |         Urgent Pointer        |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                    Options                    |    Padding    |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
|                             data                              |
+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- Source Port 源端口号 16位

- Destination Port 目标端口号 16位

- Sequence Number 序列号 32位
    
    代表偏移量，本包在整个信息中的偏移量，也就是发送的数据部分第一个字节的序号。TCP 模块在拆分数据时，会先算好每一块数据相当于从头开始的第几个字节。在实际的通信中，序号并不是从 1 开始的，而是需要用随机数计算出一个初始值，初始值在握手阶段发送给对方。

- Acknowledgment Number 确认号 32位 
    
    表示接收方希望收到对方下次发送的数据的第一个字节的序号。
    
    在TCP确认机制中，无法有效处理非连续TCP片段。确认号表明所有低于该编号的 Sequence Number 已经被发送该编号的设备接收。如果我们收到的字节数落在两个非连续的范围内，则无法只通过一个编号来确认。这可能导致潜在严重的性能问题，特别是高速或可靠性较差的网络。

- Data Offset 4位

    表示数据部分的起始位置，也可以认为表示头部的长度。`The number of 32 bit words in the TCP Header.` 它的基数是32 bit（即4个字节），例如`1011`表示头部共`0b0111 * 32 = 352 bits`(44 bytes)。其中主要是`Options`部分为可变长度。

- Reserved 预留6位

- Control Bits:  控制位 6 bits (from left to right):

    - URG:  Urgent Pointer field significant

        URG 为紧急标志，和紧急指针字段配合使用。当 URG 位置为1时，表名此报文要尽快传送，而不按原排队次序发送。
        
    - ACK:  Acknowledgment field significant

        确认标志，和 Acknowledgment Number 配合使用。

    - PSH:  Push Function
        
        PSH 为推送标志，当 PSH 位置为1时，发送方将立即发送缓冲区中的数据，而不等待后续数据构成一个更大的段。接收方一旦收到 PSH 位位1的段，就立即将接收缓冲区中的数据提交给应用程序，而不等待后续数据到达。

    - RST:  Reset the connection

        复位标志。当 RST 位置为1时，表名有严重差错，必须释放连接。
        
    - SYN:  Synchronize sequence numbers

        同步标志。当 SYN 为1是表示请求建立连接，此时会发送初始 Sequence Number。
        
    - FIN:  No more data from sender

        终止标志。当 FIN 位置为1时，表明是护具已经发送完，请求释放连接。

- Window 窗口 16位

    接收方告知发送方窗口大小（即无需等待确认可一起发送的数据量），用于通知对方当前机器接收区的大小。提高效率的一种方式，不用每次发包都要`ack`一下，可以发多个包后一次`ack`。

- Checksum 校验和 16位

    用来检查是否出现错误

- Urgent Pointer 紧急指针 16位

    表示应紧急处理的数据位置

- Options 可选字段

    除了上面的固定头部字段之外，还可以添加可选字段，但除了连接操作之外，很少使用可选字段。

    ```
       kid       len          
    +--------+
    |00000000|
    +--------+
    |00000001|
    +--------+--------+--------+--------+
    |00000010|00000100|   max seg size  |
    +--------+--------+--------+--------+
    |00000011|00000011| wscale |
    +--------+--------+--------+
    |00000100|00000010|
    +--------+--------+
    ```

    - 0：选项表结束（1字节）
    - 1：无操作（1字节）用于选项字段之间的字边界对齐。
    - 2：最大报文段长度（4字节，Maximum Segment Size，MSS）通常在建立连接而设置SYN标志的数据包中指明这个选项，指明本端所能接收的最大长度的报文段。通常将MSS设置为（MTU-40）字节，携带TCP报文段的IP数据报的长度就不会超过MTU，从而避免本机发生IP分片。只能出现在同步报文段中，否则将被忽略。
    - 3：窗口扩大因子（4字节，wscale），取值0-14。用来把TCP的窗口的值左移的位数，使窗口值乘倍。只能出现在同步报文段中，否则将被忽略。这是因为现在的TCP接收数据缓冲区（接收窗口）的长度通常大于65535字节。
    - 4：sackOK—发送端支持并同意使用SACK选项。
    - 5：SACK实际工作的选项。
    - 8：时间戳（10字节，TCP Timestamps Option，TSopt）
        
        - 发送端的时间戳（Timestamp Value field，TSval，4字节）
        - 时间戳回显应答（Timestamp Echo Reply field，TSecr，4字节）

    ``` javascript
    function parseTCPOptions(options) {
        let _readerIndex = 0;
        let mss, wscale, sackOK, sack, tsVal, tsEcr;

        while(_readerIndex < options.length) {
            let kid = options.readUInt8(_readerIndex);
            _readerIndex++;
            // console.log('kid: %d, _readerIndex: %d', kid, _readerIndex);
            
            if (kid === 0) {
                // 结束
                return;
            } else if (kid === 1) {
                // 无操作
                continue;
            } else if (kid === 2) {
                // MSS
                _readerIndex++; // skip length 值固定是4（包含类型和长度字段）

                mss = options.readUInt16BE(_readerIndex);
                _readerIndex += 2;
            } else if (kid === 3) {
                // 窗口扩大因子
                _readerIndex++; // skip length 值固定是3

                wscale = options.readUInt8(_readerIndex);
                _readerIndex ++;
            } else if (kid === 4) {
                // 允许SACK
                _readerIndex++; // skip length 值固定是2

                sackOK = true;
            } else if (kid === 5) {
                let len = options.readUInt8(_readerIndex);
                _readerIndex++;

                _readerIndex += (len -2); // skip 
            } else if (kid === 8) {
                _readerIndex++; // skip length 值固定为10
                tsVal = options.readUInt32BE(_readerIndex);
                _readerIndex += 4;
                tsEcr = options.readUInt32BE(_readerIndex);
                _readerIndex += 4;

            } else {
                // unknow
                let len = options.readUInt8(_readerIndex);
                _readerIndex++;

                // skip 
                _readerIndex += (len - 2);
            }
        }

        return {
            mss, wscale, sackOK, sack, tsVal, tsEcr
        };
    }
    ```

示例：

```
0000   d2 e6 00 50 f5 e9 1f 49 00 00 00 00 b0 02 ff ff
0010   5a 79 00 00 02 04 05 b4 01 03 03 05 01 01 08 0a
0020   ba 87 34 a1 00 00 00 00 04 02 00 00
```

上图中`Source Port`为`0xd2e6`对应十进制为`53990`，`Destination Port`为`0x0050`十进制为`80`。`Sequence Number`为`0xf5ef1f49`对应十进制为`4126089033`，`Acknowledgment Number`为`0x000000`。`Data Offset`、`Reserved`及`Control Bits`合并为`0xb002`，对应二进制`1011000000000010`其中`Data Offset`为`1011`则头部长度为44 bytes，`Control Bits`为`000010`对照为`SYN`类型。`Window`为`0xffff`。`Checksum`为为`0x5a79`。`Urgent Pointer`为`0x0000`。`Options`为`020405b4010303050101080aba8734a10000000004020000`解析出来为`"mss":1460,"wscale":5,"sackOK":true,"tsVal":3129423009,"tsEcr":0`。

> TCP 包的长度有 MTU 确定，在以太网中一般是 1500 bytes。MTU 是包含头部（TCP头部、IP头部）的总长度，因此需要从 MTU 减去头部的长度，然后得到的长度就是一个网络包中所能容纳的最大数据长度，这一长度叫作 MSS。

|      No.      |                       服务器:80                        | 流向 |                客户端:53990                 |                  说明                   |
| ------------- | ------------------------------------------------------ | ---- | ------------------------------------------- | ------------------------------------- |
| **TCP 连接**  |                                                        |      |                                             |                                       |
| 399           |                                                        | <--  | SYN:4125695817 SN:4125695817 Window:65535   | [连接] 客户端发起，初始SN为4125695817   |
| 400           | ACK:4125695818,SYN:787209242 SN:787209242 Window:14480 | -->  |                                             | [连接] 服务器ACK并发送初始SN为787209242 |
| 401           |                                                        | <--  | ACK:787209243 SN:4125695818 Window:4117     | [连接] 客户端ACK 更新窗口为4117         |
| **HTTP 请求** |                                                        |      |                                             |                                        |
| 402           |                                                        | <--  | ACK:787209243,PSH SN:4125695818 Window:4117 | [收发] 客户端发送HTTP请求数据           |
| 403           | ACK:4125695894 SN:787209243 Window:227                 | -->  |                                             | [收发] 服务器ACK客户端请求数据          |
| **HTTP 响应** |                                                        |      |                                             |                                       |
| 404           | ACK:4125695894 SN:787209243 Window:227                 | -->  |                                             | [收发] 服务器端发送数据                 |
| 405           | ACK:4125695894 SN:787210691 Window:227                 | -->  |                                             | [收发] 服务器端发送数据                 |
| 406           | ACK:4125695894,PSH SN:787212139 Window:227             | -->  |                                             | [收发] 服务器端发送数据                 |
| 407           |                                                        | <--  | ACK:787212139 SN:4125695894 Window:4050     | [收发] 客户端确认                       |
| 408           |                                                        | <--  | ACK:787213379 SN:4125695894 Window:4012     | [收发] 更新窗口                        |
| **TCP 关闭**  |                                                        |      |                                             |                                        |
| 409           |                                                        | <--  | ACK:787213379,FIN SN:4125695894 Window:4096 |                                       |
| 411           | ACK:4125695895,FIN SN:787213379 Window:227             | -->  |                                             |   这步是一个合并操作，效果等同于四次握手    |
| 412           |                                                        | <--  | ACK:787213380 SN:4125695895 Window:4096     |                                        |


解析TCP头部代码

``` javascript
function parseTCPHeader(header) {
    if (header.length < 20) throw new Error('Bad Frame');

    let _readerIndex = 0;
    let sourcePort = header.readUInt16BE(_readerIndex);
    _readerIndex += 2;
    let destinationPort = header.readUInt16BE(_readerIndex);
    _readerIndex += 2;
    let sequenceNumber = header.readUInt32BE(_readerIndex);
    _readerIndex += 4;
    let acknowledgmentNumber = header.readUInt32BE(_readerIndex);
    _readerIndex += 4;

    let dataOffsetAndFlags = header.readUInt16BE(_readerIndex);
    _readerIndex += 2;
    let dataOffset = dataOffsetAndFlags >> 12;
    let headerLength = dataOffset * 4; // 4 bytes 32 bits
    let urg = dataOffsetAndFlags >> 5 & 0x01;
    let ack = dataOffsetAndFlags >> 4 & 0x01;
    let psh = dataOffsetAndFlags >> 3 & 0x01;
    let rst = dataOffsetAndFlags >> 2 & 0x01;
    let syn = dataOffsetAndFlags >> 1 & 0x01;
    let fin = dataOffsetAndFlags & 0x01;

    let window = header.readUInt16BE(_readerIndex);
    _readerIndex += 2;
    let checksum = header.readUInt16BE(_readerIndex);
    _readerIndex += 2;
    urgentPointer = header.readUInt16BE(_readerIndex);
    _readerIndex += 2;

    let options;
    if (headerLength > 20) options = parseTCPOptions(header.slice(_readerIndex)); 

    return {
        sourcePort,
        destinationPort,
        sequenceNumber,
        acknowledgmentNumber,
        dataOffset,
        urg,
        ack,
        psh,
        rst,
        syn,
        fin,
        window,
        checksum,
        urgentPointer,
        options
    }
}
```

### 参考
- [TRANSMISSION CONTROL PROTOCOL](https://tools.ietf.org/html/rfc793#section-3.1)
- [网络是怎么连接的](https://book.douban.com/subject/26941639/)
- [TCP/IP详解 卷1：协议](https://book.douban.com/subject/1088054/)
- [TCP/IP网络与协议（第二版）](https://book.douban.com/subject/1683696/)
- [为什么标准以太网接口缺省的MTU为1500？](https://www.zhihu.com/question/21524257)
- [细说TCP确认机制](https://community.emc.com/message/842879#842879)