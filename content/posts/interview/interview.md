---
title: "2022 年初 | 后端开发两年经验社招面经"
date: 2022-01-26T15:37:36+08:00
draft: false
categories: ["interview"]
---

# 2022 年初 | 后端开发两年经验社招面经

## 字节

### 一面

- coding: 对于一个数组，仅用一次遍历，等概率地随机出一个元素（对于每一个元素，从全局看，他们被选择的概率都应该是 1/n）
  - 对于第 i 个元素，它在第 i 轮被选中的概率是 1/i
  - 往后，只要选择了新的元素，它就会被淘汰；以第 i+1 轮为例，它被淘汰的概率是 1/(i+1)，那么反过来它被留下的概率就是 1 - 1/(i+1)
  - 最终每一个元素被选择的概率如下，第一个 1/i 代表它在第 i 次被选中，其他数代表它在后续的每一轮被留下
  - ![1](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/interview/1.png)

- followup：等概率地随机出 k 个元素
  - 对于第 i 个元素，它在第 i 轮被选中的概率是 k/i
  - 往后，它唯一会被淘汰的场景是：选择了新的元素，同时从已有的选择中，等概率地选择到了它；以第 i+1 轮为例，它被淘汰的概率是 k/(i+1) * 1/k = 1/(i+1)，那么反过来它被留下的概率就是 1 - k/(i+1) * 1/k = 1 - 1/(i+1)
  - 最终每一个元素被选择的概率如下，第一个 k/i 代表它在第 i 次被选中，其他数代表它在后续的每一轮被留下
  - ![2](https://raw.githubusercontent.com/chr1sc2y/warehouse-deprecated/refs/heads/main/resources/interview/2.png)

- coding: 实现 Fisher–Yates Suffle

    ```cpp
    void shuffle(vector<int> v)
    {
        int n = v.size();
        for (int i = n - 1; i >= 1; --i)
        {
            int j = rand() % (i + 1);
            swap(v[i], v[j]);
        }
    }
    ```

### 二面

- system design：主播开播，如何把这个消息推送给他的千万级 follower
  - 设计数据仓库（数据库也可以）
  - 消息队列
- followup：推送之后，主播已经下播，怎么处理
- followup：怎么保证消息推送或被消费
- followup：如何保证消息不重复

### 三面

- DB 迁移流程
  1. 设置快照，离线搬迁存量数据
  2. 双写，将快照后的新数据流写入新 DB
  3. 切换路由，将读的请求引入新 DB

- coding：写一个服务的大致框架

## Hotstar

### 一面

- 对协程的理解
- csrf token 是什么
  - Cross-site request forgery 跨站请求伪造；攻击者诱导受害者进入第三方网站，在第三方网站中，向被攻击网站发送跨站请求，并获取注册凭证，绕过后台验证，冒充用户对被攻击网站执行某项操作
- 对启发式搜索的理解，对启发函数的研究和优化
- coding：实现 Trie
- coding：实现 AC 自动机

### 二面

- 项目经历，讲细节
- 对 tf 的了解
- coding：实现对象池
- coding：实现内存池

### 三面

- 如何将一个长URL转换为一个短URL
  - 用发号期为每个长地址分配一个短号码
  - 0-9, a-z, A-Z 一共 62 个字符构成 62 进制，节省长度
  - 分布式系统中不同发号期使用不同号码段（A 发号器用 0-999，B 发号器用 1000-1999）
- coding：设计模式 decorator pattern 实现
- coding：一段可编译通过的 cpp 代码编码保存后，做字符串处理，去掉其中的所有注释
  - // 后注释的内容，/\* \*/ 中的内容，都要去掉
  - \ 换行要保留

### 四面

- 跳槽动机
- 未来长短期职业规划
- 没了

## 微软

### 一面

- 讲讲项目
- coding：实现一个类 `TextProcessor`，包含以下函数
  - `void set_variable(string k, string v)`: 将 k 映射为 v
  - `void get_text(string s)`: 将字符串 s 中用 `{` 和 `}` 中间的内容进行格式化替换
  - 例如：

  ```cpp
  auto text_processor = new TextProcessor();
  text_processor->set_variable("Name", "abc");
  text_processor->set_variable("Date", "2021-11-23");
  auto content = text_process->get_text("Dear {Name}, welcome! {Date}");
  printf("%s\n", content);
  // "Dear abc, welcome! 2021-11-23"
  ```

  - 很简单的字符串处理，需要注意包含多个转义字符 `\` 的情况

### 二面

- 讲讲项目
- Prometheus 原理，和 MySQL，ElasticSearch 的区别
- 对 kubernetes 的理解
- coding：24 点游戏，参考 leetcode 679 题

### 三面

- cpp 和 java 的区别
- cpp 中动态绑定的机制，实现原理
- 构造函数能否调用虚函数，为什么
- coding：一段可编译通过的 cpp 代码编码保存后，做字符串处理，去掉其中的所有注释（包括 // 和 /* */）；和 Hotstar 三面的 coding 一模一样

### 四面

- 闲聊
- 跳槽动机，未来规划
- system design：地图导航 app 的后端设计
  - 从立项到上线，过程中各个阶段需要考虑哪些方面
  - 设计各个技术方案时需要考虑哪些细节

### 五面

面试官是印度人，还好口音不是很重

- 自我介绍，教育背景
- talk about the project you've done with the most sense of achievement, why
- coding: 计算一个图里面的矩形个数（没找到原题）

## Amazon

### 笔试

1. 有两组剧，每组给出开始时间和持续时间，求每组各看一部剧的最快看完的时间点；举个例，A 组有 3 部，开始时间是 [1, 2, 3]，持续时间是 [1, 1, 1]；B 组有 3 部，开始时间是 [1, 2, 3]，持续时间是 [10, 5, 1]，那么最快看完的时间点是 4，其中 A 组看第一部，在 1+1=2 时间点结束，等到 3 点的时候看 B 组的第三部，3+1=4
    - 用贪心，先找到第一组最快看完的时间点，再从这个时间点往后找第二组最快看完的时间点。除此之外还要在先 B 后 A 再找一遍

2. 有一组服务器，长度为 n，各个服务器的开机耗电为 A[n]，持续耗电为 B[n]；只有连续的服务器才能组成集群，集群的总开机耗电为 max(A[i...j])，总持续耗电为 sum(A[i...j]) \* (j-i+1)；给出一个 p，求能够组成集群且总开机耗电加总持续耗电小于等于 p 的最大服务器个数。e.g. A[n] = {3,6,1,3,4}, B[n] = {2,1,3,4,5}, p = 25，那由第四个和第五个组成的集群总耗电是 4 + (4+5)*2 = 22，22 小于 25，所以是一个可行的解，返回这个集群的服务器数量 2 就可以了（不需要给出详细的方案）
    - 服务器需要连续，因此用滑动窗口确定范围；开机耗电用单调队列计算

### 一面

- coding：leetcode 252 meeting-rooms，1 遍历，2 差分数组
- coding：leetcode 103 zigzag tree，1 两个栈层序遍历，2 递归前序遍历，再反转偶数层数组

### 二面

- system design: 购物网站

### 三面

- system design: 设计浏览记录功能，考虑高并发

### 四面

- coding：leetcode 101 对称二叉树
- followup：对称多叉树
