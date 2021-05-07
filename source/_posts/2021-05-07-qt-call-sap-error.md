---
title: 一次愉(折)快(腾)的问题排查之旅：Qt在Linux下调用SAP接口拿不到数据？
toc: false
date: 2021-05-07 09:40:00
description: ...
tags:
- Linux
- Qt
---

五一前，无锡的同事告诉我，有个诡异的现象。

用TopJS3（我们用Qt写的一个程序）在Linux下调SAP的某一个特定的接口，拿不到数据。一次愉快的问题排查之旅开始了。

1. **验证**：TA说的现象是真的吗？

   到客户服务器上执行了一下，确实如此。状态码为200，且有返回值，只是返回的值是空的。

2. **缩小范围**：将同事给的Demo进一步简化，写了一个最小Demo程序，以避免其他干扰因素的影响。

3. **找逻辑点**：用TopJS3在Windows上试了一下，结果是对的。

4. 找逻辑点：用Curl在Linux上试了一下，结果也是对的。

5. 找逻辑点：用Python写了一个最小的Demo，结果也是对的。

6. **尝试串联逻辑点，形成逻辑树**。

7. **排除法**：输出在某一个特定的情况下不对，其他情况都OK，排除SAP服务端不对的可能性。

8. 排除法：只可能与输入有关，输入包括Header和Body。

9. 排除法：找工具查看了一下，排除了Body中有隐藏字符的可能。

10. 所以，大概率与Header有关。

11. **我猜我猜我猜猜猜**，猜测可能的Header不对，试了一遍，结果仍然和之前一样，返回值为空。

12. 尝试抓包，用tcpdump。结果用tcpdump开两个Termial居然抓不到任何东西。同样的方式在我本地测试OK。

13. 放弃用tcpdump，改为wireshack。

14. 因为客户的服务器环境没有网络，都得离线装，蛮折腾的。先安装了一个nomachine用来远程，再装了一个wireshack用来抓包。

15. 安装好wireshack后，分别用TopJS3和Curl试一下。

16. **破案**，TopJS3多了一个头 `Accept-Language: en-US, *`。手动添加一个头`Accept-Language: zh-CN`后拿到了正确的结果。

    查看了Qt关于该部分的源码，确实加了Accept-Language。

    ![](/images/qt-call-sap-error-1.png)

    

