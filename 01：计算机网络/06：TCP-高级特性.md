### TCP 高级特性

------

[TOC]

##### 01：滑动窗口协议（Sliding Window Protocol）

- TCP协议的一种应用，用于**网络数据传输时的流量控制**的技术，以避免拥塞的发生；
- 该协议允许发送方在**停止并等待确认前发送多个数据分组**；由于发送方不必每发一个分组（分片）就停下来等待确认，因此该协议可以加速数据的传输速度，提高网络吞吐量；底层是用TCP报文段中的**窗口大小字段**来控制；
- 因为 TCP 是全双工传输，因此通信的双方都拥有两个滑动窗口，一个发送窗口，一个接收窗口；
  - 接收窗口 >= 发送窗口

###### 发送窗口：

- 接收方允许发送方一次能容纳的未确认的字节数，决定了发送方允许传送的字节数大小，当收到接收方新的ACK（确认号）时，确认字节已经被接受，窗口向前滑动；
- ![](/Users/likang/Code/Git/Network/01：计算机网络/photos/sliding-window.png)

###### 发送端：

1. 已发送已确认
   - 数据流中最早的字节已经发送并得到确认
2. 已发送但未收到确认
   - 已发送但尚未得到确认的字节，发送方在未被确认之前，认为这些数据还没有被处理
3. 允许发送但尚未发送
   - **数据已经被加载到缓存中(窗口）**
4. 不允许发送
   - 数据属于未发送，同时接收端也不允许发送的，因为这些数据已经超出了发送端窗口范围；

###### 接收端：

- 已接收
- 未接收允许接收（接受窗口）
- 未接收并不允许接收（由于ACK直接由TCP协议栈回复，默认无应用延迟，**不存在已接收未回复ACK**）；

##### 02：滑动窗口协议的应用：

###### 停止-等待协议（1 bit 滑动窗口协议）

- **接受方的窗口和发送方的窗口大小都是1bit**，所以也叫 1比特滑动窗口协议
- 发送方这时自然每次只能发送1bit，并且必须等待这个数据包的ACK，才能发送下一个，所以**效率太低**，一直需要等待

###### 回退n步协议

- 发送方在发完一个数据帧后，不会停下来等待应答帧，而是连续发送若干个数据帧，即使在连续发送过程中收到了接收方发来的应答帧，也可以继续发送。且**发送方在每发送完一个数据帧时都要设置超时定时器**，但是如果在设置的超时时间内仍未收到确认帧，就要**重发相应的数据帧，回退了n步**；
- 一旦网路情况比较糟糕时，反而**重复发送很多已经发送失败的帧**，效果反而不如1bit协议；

###### 选择重传协议

- 当出现错误帧后，总是要重发该帧之后的所有帧，所以**接收端总会缓存所有收到的帧，当某个帧出现错误时，只会要求重传这一个帧**，只有当某个序号后的所有帧都正确收到后，才会一起提交给上层应用
- 缺点：接受端需要**更多的缓存**；

##### 03：拥塞机制

- 某一时刻，若对网络中某一个资源的需求超过了资源所能提供的负载，网络的性能就会变坏，叫做拥塞；
- 拥塞控制：防止过多的请求数据注入网络中，可以使网络中的路由器不会导致过载；
- **是一个全局性的过程和流量控制不同，流量控制指点对点通信量的控制**；
- **发送方维持一个拥塞窗口(cwnd)的状态变量**，拥塞窗口的大小取决于网络的拥塞程度，并且动态地变化，发送方让自己的发送窗口等于拥塞窗口，另外考虑到接受方的接收能力，发送窗口可能小于拥塞窗口；

###### 慢开始算法

- 先发送少量的数据，**探测一下网络的拥塞程度**，也就是由小到大逐渐增加拥塞窗口的大小；
- 为了防止拥塞窗口(cwnd)增长过大引起网络拥塞，还需设置一个慢开始门限(ssthresh)状态变量
  - 当cwnd < ssthresh，执行慢开始算法
  - 当cwnd = ssthresh，慢开始和拥塞避免算法任意
  - 当cwnd > ssthresh，拥塞避免算法

###### 拥塞避免算法

- 让拥塞窗口缓慢增长，即**每经过一个往返时间 RTT 就把发送方的拥塞窗口cwnd 加 1，不是加倍**，让拥塞窗口按线性规律缓慢增长
- 无论是在慢开始阶段还是在拥塞避免阶段，只要发送方判断网络拥塞，就把**慢开始门限设置为出现拥塞时的发送窗口大小的一半**，然后把拥塞窗口设置为1，执行慢开始算法；

##### 04：快重传、快恢复

- 快重传
  - 要求接收方在**收到一个失序的报文时就立即发出重复确认**（使发送尽早知道有报文段没有到达对方）而不要等到自己发送数据时捎带确认；
  - 一般来说，重传发生在超时之后，但是如果**发送端接收到3个以上的重复确认ACK报文**，就应该意识到，需要重新传递，这个机制不需要等到**重传定时器溢出**，所以叫做快重传；
- 快重传以后，因为走的不是慢开始而是拥塞避免算法，所以叫快速恢复算法；
- 快重传和快恢复目的：快速恢复丢失的数据包；





