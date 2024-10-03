---
title: ThreadSanitizer
author: Mao Chencheng
date: '2023-07-20'
categories:
  - LLVM
tags:
  - note
slug: tsan-note
---

# ThreadSanitizer

- Tid: a unique thread ID

- ID: a unique memory location ID

- Event: READ, WRITE, WRLOCK, RDLOCK, WrUnlock, RdUnlock, Signal, Wait.

- Lock:

  In a thread $T$

  - A lock L is write-held ←→ $\text{NUM}(\text{WrLock}_T (L))> \text{NUM}(\text{WrUnLock}_T (L))$
  - A lock L is read-held ←→ $\text{NUM}(\text{RDLock}_T (L))> \text{NUM}(\text{RDUnLock}_T (L))$

- Lock sets: $LS$
 


**Happens-before arc:** 对于事件$X = Signal_{T_X}(A_X)$ 和 $X=Wait_{T_Y}(A_Y)$, 如果 $(A_X=A_Y) ∩ (T_X\neq T_Y)$, 并且$X$先被观察到，那么$(X,Y)$构成一条弧

X和Y在不同线程，X发送信号，Y接受相同的信号，

**Happens-before:** 如果两个事件$X=TYPEX_{T_X}(A_X)$, $Y=TYPEY_{T_Y}(A_Y)$,$X$ 在 $Y$ 之前发生，并且满足下列3个条件之一

- $T_X=T_Y$
- $(X,Y)$ 是Happens-before arc
- 传递性

**Segment Set**: Segment集合 $\{S_1,S_2...,S_N\}$ 满足对 $\forall i,j:S_i\not\preceq S_j$. 即SS里所有的segments都不相关

**Concurrent**(并行): 两个事件 $X,Y$, $X\not\preceq Y$ AND $Y\not\preceq X$ AND $LS(X)∩LS(Y)=\emptyset$

*即X与Y不存在逻辑的偏序性，且没有相同的锁对其进行同步*

**Data Race:** 两个并行的线程同时访问某个内存，并且至少有一个是 $\text{WRITE}$



读写段集合$SS_{Rd}$, $SS_{Wr}$ 满足条件A

$\forall S_r\in SS_{Rd},S_w\in SS_{Wr}:S_r\not\preceq S_w$

*注：$S_r\not\preceq S_w$ 的意味：(1) $S_r$ happens-after $S_w$; (2) $S_r$ 与 $S_w$ 不相关*



算法

![img](https://pic4.zhimg.com/80/v2-449d53eb7ce09ce48284d86999300f3f_1440w.webp)

在个内存事件发生后维护 $SS_{Rd}$, $SS_{Wr}$ ，然后检测data race。

**更新过程(Line4~Line9)：**

- 如果内存事件是读，$SS_{Rd}$ 更新为 $\{s:s\in SS_{Rd}∩ s\not\preceq \text{seg} \}$; $SS_{Wr}$ 更新为 $\{s:s\in SS_{Rd}∩ s\not\preceq \text{seg} \}∩ \{\text{seg}\}$
- 如果内存事件是写，$SS_{Rd}$ 更新为 $\{s:s\in SS_{Rd}∩ s\not\preceq \text{seg} \}∩\{\text{seg}\}$

更新过程使得$SS_{Rd}$, $SS_{Wr}$ 永远满足条件A

**检测冲突：**![img](https://pic4.zhimg.com/80/v2-eae33324acb54b57c78be189f108ed5b_1440w.webp)

循环遍历$SS_{Wr}$中每个片段(Line3)，记片段为$W_1$ (Line4)，记该片段的锁集合为$LS_1$ (Line5), 检查每个可能和$W_1$冲突的写片段（Line7\~Line12)，检查每个可能和$W_1$冲突的读片段（Line14~Line17)

在这里检测的是相并行的两个Segment。更新过程完成的是 $X\not\preceq Y$ AND $Y\not\preceq X$, 冲突检查完成的是$LS(X)∩LS(Y)=\emptyset$

> **Concurrent**(并行): 两个事件 $X,Y$, $X\not\preceq Y$ AND $Y\not\preceq X$ AND $LS(X)∩LS(Y)=\emptyset$
