---
title: "分布式系统"
date: 2015-07-05T08:57:52+10:00
draft: true
categories: ["Computer Science"]
---

# 分布式系统

## 分布式系统

### CAP 原则

- 分布式系统不可能同时满足一致性（C：Consistency）、可用性（A：Availability）和分区容忍性（P：Partition Tolerance），最多只能同时满足其中两项

#### 一致性

- 一致性指的是多个数据副本是否能保持一致的特性，在一致性的条件下，系统在执行数据更新操作之后能够从一致性状态转移到另一个一致性状态
- 对系统的一个数据更新成功之后，如果所有用户都能够读取到最新的值，该系统就被认为具有强一致性

#### 可用性

- 可用性指分布式系统在面对各种异常时可以提供正常服务的能力，可以用系统可用时间占总时间的比值来衡量，4 个 9 的可用性表示系统 99.99% 的时间是可用的
- 在可用性条件下，要求系统提供的服务一直处于可用的状态，对于用户的每一个操作请求总是能够在有限的时间内返回结果

#### 分区容忍性

- 网络分区指分布式系统中的节点被划分为多个区域，每个区域内部可以通信，但是区域之间无法通信
- 在分区容忍性条件下，分布式系统在遇到任何网络分区故障的时候，仍然需要能对外提供一致性和可用性的服务，除非是整个网络环境都发生了故障

#### 一致性 + 可用性：两阶段提交
#### 一致性 + 分区容忍性：多版本并发控制 MVCC
#### 可用性 + 分区容忍性：Paxos

### BASE

- BASE 是基本可用（Basically Available）、软状态（Soft State）和最终一致性（Eventually Consistent）三个短语的缩写
- BASE 理论是对 CAP 中一致性和可用性权衡的结果，它的核心思想是：即使无法做到强一致性，但每个应用都可以根据自身业务特点，采用适当的方式来使系统达到最终一致性。

#### 基本可用

- 指分布式系统在出现故障的时候，保证核心可用，允许损失部分可用性
- 例如促销时，为了保证购物系统的稳定性，部分消费者可能会被引导到一个降级的页面

#### 软状态

- 指允许系统中的数据存在中间状态，并认为该中间状态不会影响系统整体可用性，即允许系统不同节点的数据副本之间进行同步的过程存在时延

#### 最终一致性

- 最终一致性强调的是系统中所有的数据副本，在经过一段时间的同步后，最终能达到一致的状态
- ACID 要求强一致性，通常运用在传统的数据库系统上。而 BASE 要求最终一致性，通过牺牲强一致性来达到可用性，通常运用在大型分布式系统中
- 在实际的分布式场景中，不同业务单元和组件对一致性的要求是不同的，因此 ACID 和 BASE 往往会结合在一起使用

## 集群

### 负载均衡

- 集群中的应用服务器（节点）通常被设计成无状态，用户可以请求任何一个节点

- 负载均衡器会根据集群中每个节点的负载情况，将用户请求转发到合适的节点上

- 负载均衡器可以用来实现高可用以及伸缩性
    - 高可用：当某个节点故障时，负载均衡器会将用户请求转发到另外的节点上，从而保证所有服务持续可用
    - 伸缩性：根据系统整体负载情况，可以很容易地添加或移除节点

- 负载均衡器运行过程包含两个部分
    - 根据负载均衡算法得到转发的节点
    - 进行转发

#### 1. 轮询（Round Robbin）

- 轮询算法把每个请求轮流发送到每个服务器上
- 假设有 2 个服务器和 6 个客户端，客户端发送了 6 个请求，这 6 个请求按 (1, 2, 3, 4, 5, 6) 的顺序发送，那么 (1, 3, 5) 的请求会被发送到服务器 1，(2, 4, 6) 的请求会被发送到服务器 2
- 该算法比较适合每个服务器的性能差不多的场景，如果有性能存在差异的情况下，那么性能较差的服务器可能无法承担过大的负载

#### 2. 加权轮询（Weighted Round Robbin）

- 加权轮询是在轮询的基础上，根据服务器的性能差异，为服务器赋予一定的权值，性能高的服务器分配更高的权值

#### 3. 最少连接（Least Connections）

- 由于每个请求的连接时间不一样，使用轮询或者加权轮询算法的话，可能会让一台服务器当前连接数过大，而另一台服务器的连接过小，造成负载不均衡
- 最少连接算法就是将请求发送给当前最少连接数的服务器上

#### 4. 加权最少连接（Weighted Least Connection）

- 在最少连接的基础上，根据服务器的性能为每台服务器分配权重，再根据权重计算出每台服务器能处理的连接数

#### 5. 随机算法（Random）

- 把请求随机发送到服务器上。
- 和轮询算法类似，该算法比较适合服务器性能差不多的场景

#### 6. 源地址哈希法 (IP Hash)

- 源地址哈希通过对客户端 IP 计算哈希值之后，再对服务器数量取模得到目标服务器的序号

- 可以保证同一 IP 的客户端的请求会转发到同一台服务器上，用来实现会话保持/会话粘滞（Sticky Session）

##### 会话粘滞

- 可以识别客户端与服务器之间交互过程的关连性，在作负载均衡的同时还保证一系列相关连的访问请求会保持分配到一台服务器上
- 如果用户需要登录，那么就可以简单的理解为会话；如果不需要登录，那么就是连接
- 在某些要求登录状态的情境下，要求客户端和服务器之间保持一个会话以记录客户端的各种信息；一个客户与服务器经常经过好几次的交互过程才能完成一笔交易；由于几次交互过程是密切相关的，所以需要服务器知道前几次的处理结果
- 会话保持的意义是确保将来自相同客户端的请求转发至相同的服务器

#### 转发实现

##### 1. HTTP 重定向

- 负载均衡服务器得到服务器的 IP 地址之后，将该地址写入 HTTP 重定向报文中发送给客户端，状态码为 302；客户端收到重定向报文之后，需要重新向服务器发起请求

- 缺点
    - 需要两次请求，因此访问延迟比较高
    - HTTP 负载均衡器处理能力有限，会限制集群的规模
    - 该负载均衡转发的缺点比较明显，实际场景中很少使用它

##### 2. DNS 域名解析

- 在 DNS 解析域名的同时使用负载均衡算法计算服务器 IP 地址

- 优点
    - DNS 能够根据地理位置进行域名解析，返回离用户最近的服务器 IP 地址
- 缺点
    - 由于 DNS 具有多级结构，每一级的域名记录都可能被缓存，当下线一台服务器需要修改 DNS 记录时，需要过很长一段时间才能生效

- 大型网站基本使用了 DNS 做为第一级负载均衡手段，然后在内部使用其它方式做第二级负载均衡。也就是说，域名解析的结果为内部的负载均衡服务器 IP 地址

##### 3. 反向代理服务器

- 反向代理服务器位于源服务器前面，用户的请求需要先经过反向代理服务器才能到达源服务器。反向代理可以用来进行缓存、日志记录等，同时也可以用来做为负载均衡服务器

- 在这种负载均衡转发方式下，客户端不直接请求源服务器，因此源服务器不需要外部 IP 地址，而反向代理需要配置内部和外部两套 IP 地址。

- 优点：与其它功能集成在一起，部署简单
- 缺点：所有请求和响应都需要经过反向代理服务器，它可能会成为性能瓶颈

##### 4. 网络层

- 在操作系统内核进程获取网络数据包，根据负载均衡算法计算源服务器的 IP 地址，并修改请求数据包的目的 IP 地址，最后进行转发

- 源服务器返回的响应也需要经过负载均衡服务器，通常是让负载均衡服务器同时作为集群的网关服务器来实现

- 优点：在内核进程中进行处理，性能比较高
- 缺点：和反向代理一样，所有的请求和响应都经过负载均衡服务器，会成为性能瓶颈

##### 5. 链路层

- 在链路层根据负载均衡算法计算源服务器的 MAC 地址，并修改请求数据包的目的 MAC 地址，并进行转发
- 通过配置源服务器的虚拟 IP 地址和负载均衡服务器的 IP 地址一致，从而不需要修改 IP 地址就可以进行转发。也正因为 IP 地址一样，所以源服务器的响应不需要转发回负载均衡服务器，可以直接转发给客户端，避免了负载均衡服务器的成为瓶颈
- 这是一种三角传输模式，被称为直接路由。对于提供下载和视频服务的网站来说，直接路由避免了大量的网络传输数据经过负载均衡服务器
- 这是目前大型网站使用最广负载均衡转发方式，在 Linux 平台可以使用的负载均衡服务器为 LVS（Linux Virtual Server）

## 哈希

- 将输入通过哈希算法，映射成输出（哈希值）

### 哈希冲突

- 经过哈希得到的地址已经有其他数据

#### 解决方法

1. 开放定址法：为产生冲突的地址求一个地址序列，包括线性探测再散列，二次探测再散列，伪随机探测再散列

2. 链地址法：将所有地址相同的记录都链接在同一个链表中

3. 再哈希法：产生冲突时再进行一次哈希，直到没有冲突为止

4. 建立公共溢出区：把所有冲突都放在公共溢出区