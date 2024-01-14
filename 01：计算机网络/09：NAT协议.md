### NAT 协议

------

[TOC]

##### 01：NAT：Network Address Translation 

- 将私有IP 转换为公网IP

##### 02：SNAT

- NAT路由器会将IP数据包里的**源IP地址**修改一下，私有IP地址`192.168.30.5`改写为公网IP地址`20.20.20.20`，这叫**SNAT**（**S**ource **N**etwork **A**ddress **T**ranslation，源地址转换）

##### 03：DNAT

- 回传数据时会将这个数据包的**目的IP地址**修改一下，变成内网IP地址`192.168.30.5`, 这也叫**DNAT**（**D**estination **N**etwork **A**ddress **T**ranslation，目的地址转换）

##### 04：NAPT

- 通过同时去区分内网里的各个网络连接，转换IP和端口的技术，就是NAPT（Network Address Port Transfer）, 网络地址端口转换 

##### 05：内网穿透技术

- 因为NAT的存在，我们只能从内网主动发起连接（内网访问外网），否则NAT设备不会记录相应的映射关系，没有映射关系也就不能转发数据。
  - **没有什么是加中间层不能解决的，如果有，那就再加一层**
- 解决方式：在**公网上**加一台服务器x，并暴露一个访问域名，再让内网的服务**主动**连接服务器x，这样NAT路由器上就有对应的**映射关系**。接着，所有人都去访问服务器x，服务器x将数据转发给内网机器，再原路返回响应，这样数据就都通了。这就是所谓的**内网穿透**。

<img src="https://github.com/likang315/Network/blob/master/01：计算机网络/photos/nat.png?raw=true" alt="nat" style="zoom:50%;" />

##### 06：抛开第三方，直连通信

- 假设还是A和B两个**局域网内**的机子，A内网对应的NAT设备叫`NAT_A`，B内网里的NAT设备叫`NAT_B`，和一个第三方服务器`server`。

###### 流程如下

- **step1和2**: A主动去连server，此时A对应的`NAT_A`就会留下A的内网地址和外网地址的**映射关系**，server也拿到了A对应的外网IP地址和端口。
- **step3和4**: B的操作和A一样，主动连第三方server，`NAT_B`内留下B的内网地址和外网地址的**映射关系**，然后server也拿到了B对应的外网IP地址和端口。
- **step5和step6以及step7**: 重点来了。此时server发消息给A，让A主动发`UDP`消息到B的外网IP地址和端口。此时NAT_B收到这个A的UDP数据包时，这时候**根据NAT_B的设置不同**，导致这时候**有可能**NAT_B能直接转发数据到B，那此时A和B就通了。但也**有可能不通**，直接丢包，不过丢包没关系，这个操作的**目的是**给NAT_A上留下**有关B的映射关系**。
- **step8和step9以及step10**: 跟step5一样熟悉的配方，此时server再发消息给B，让B主动发`UDP`消息到A的外网IP地址和端口。NAT_B上也留下了关于A到映射关系，这时候由于之前NAT_A上有过关于B的映射关系，此时NAT_A就能正常接受B的数据包，并将其转发给A。到这里A和B就能正常进行数据通信了。这就是所谓的**NAT打洞**。
- **step11**: 注意，之前我们都是用的**UDP数据包**，目的只是为了在两个局域网的NAT上**打个洞**出来，实际上大部分应用用的都是TCP连接，所以，这时候我们还需要在A主动向B发起TCP连接。到此，我们就完成了两端之间的通信。

<img src="https://github.com/likang315/Network/blob/master/01：计算机网络/photos/nat-direct.png?raw=true" alt="nat-direct" style="zoom:67%;" />