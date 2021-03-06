---
layout: post
title: "平摊分析"
date: 2018-03-27 10:47:00 +0800
categories: 算法分析
---
## 为什么需要平摊分析
请看以下例子。假设一个栈(stack)支持以下操作：

* PUSH(S, x): 将对象x压入栈S
* POP(S): 弹出S的栈顶元素，并将其返回

PUSH和POP操作的代价都是O(1)。现在我们新增一种操作MULTIPOP：

* MULTIPOP(S, k): 弹出栈S的k个栈顶对象

其伪代码为：

```
MULTIPOP(S, k)
    while not STACK-EMPTY(S) and k != 0
        do POP(S)
           k = k - 1
```

可见MULTIPOP(S, k)调用了k次POP(S)，其代价为O(k)。

现在来分析一个由n个PUSH、POP、MULTIPOP操作所构成的序列，其作用于一个初始为空的栈。序列中一次MULTIPOP的代价为O(n)，因为栈的大小至多为n。最坏情况下，每个操作都是MULTIPOP，共O(n)个MULTIPOP操作，因此总代价是O(n^2)。虽然这一分析是正确的，但通过单独地考虑每个操作的最坏情况代价而得到的O(n^2)结论却是不够紧确的。这时我们就需要平摊分析。

## 简介
在平摊分析中，执行一系列数据结构操作所需要的时间是通过对执行的所有操作求平均而得出的。平摊分析可以证明，在一系列操作中，通过对所有的操作求平均之后，即使其中单一的操作具有较大的代价，平均代价还是很小的。

平摊分析与平均情况分析的区别在与，前者不涉及概率，保证在最坏情况下，每个操作具有平均性能。

常用的有3种方法：

* 聚集方法(aggregate method)
* 记账方法(accounting method)
* 势能方法(potential method)

### 聚集方法
在聚集分析中，要证明对所有的n，由n个操作所构成的序列的总时间在最坏情况下为T(n)。因此，最坏情况下每个操作的平均代价(平摊代价amortized cost)为T(n)/n。请注意平摊代价对每个操作都是成立的，即使序列中存在几种不同类型的操作时也一样。

### 记账方法
在记账方法中，我们对不同的操作赋予不同的平摊代价(amortized cost)。当平摊代价超过实际代价时，两者的差值就被当作存款(credit)，存款可以在以后用于补偿那些平摊代价低于实际代价的操作。平摊代价可以看作两部分：实际代价+存款。

所有操作的平摊代价的总和，必须大于等于实际代价的总和，这样才能保证存款不为负数，才能用廉价操作的存款来补偿昂贵操作的代价。平摊代价的总和，应该是实际代价总和的一个上界。

#### MULTIPOP stack
用记账方法来分析MULTIPOP stack。栈的各操作的实际代价为：

* PUSH      1
* POP       1
* MULTIPOP  min(k, s), k为MULTIPOP的参数，s为stack里的元素数量

其中k为MULTIPOP的一个参数，s为调用该操作时栈的大小。现在对它们赋予平摊代价：

* PUSH      2
* POP       0
* MULTIPOP  0

每次PUSH元素时，平摊代价为2，除去支付PUSH的实际代价1以外，还剩下1作为存款(credit)放在元素上。任何时候，栈内元素上都有数量为1的存款。当执行POP操作时，其平摊代价为0，因为只需要用之前的存款来支付实际代价即可。同样，MULTIPOP的平摊代价也是0，弹出k个元素，就用这k个元素上的存款来支付实际代价即可。

因为栈里每个元素都有1存款，且栈中总有非负个数的盘子，这就保证了存款的总量是非负的。这样，对任意的包含n次PUSH、POP、MULTIPOP操作的序列，总的平摊代价就是其总的实际代价的一个上界。又因为总的平摊代价为O(n)，故总的实际代价也为O(n)。

### 势能法
与记账法类似，势能方法(potential method)将预先支付的代价表示为一种“势能”或“势”，它在需要时可以释放出来，以支付后面的操作。势能是与整个数据结构而不是其中个别对象发生联系的。其工作方法如下：

设D<sub>0</sub>为初始的数据结构。我们对其执行i=1,2,3...,n个操作，c<sub>i</sub>为第i个操作的实际代价，D<sub>i</sub>为第i个操作后的结果，P(D<sub>i</sub>)为D<sub>i</sub>相联系的势能。第i个操作的平摊代价c<sub>i</sub>'为:

c<sub>i</sub>' = c<sub>i</sub> + P(D<sub>i</sub>) - P(D<sub>i-1</sub>)

每个操作的平摊代价等于其实际代价加上由于该操作所增加的势能。n个操作的总的平摊代价为:

sum(c<sub>i</sub>') = sum(c<sub>i</sub>) + P(D<sub>n</sub>) - P(D<sub>0</sub>)

如果我们能定义一个势能函数P使得P(D<sub>n</sub>) >= P(D<sub>0</sub>)，则总的平摊代价就是总的实际代价的一个上界。实际上，我们并不总是知道要执行多少个操作，即n未知，所以我们只要要求对于所有i，都有P(D<sub>i</sub>) >= P(D<sub>0</sub>)即可。为了方便起见，定义P(D<sub>0</sub>)为0,然后证明对所有的i，都有P(D<sub>i</sub>) >= 0。

#### MULTIPOP
以MULTIPOP栈为例，定义栈上的势能函数P为栈中对象的个数。开始时栈D<sub>0</sub>为空，P(D<sub>0</sub>) = 0。因为栈中的元素数量始终非负，故在第i个操作之后，栈D<sub>i</sub>就具有非负的势能，且有P(D<sub>i</sub>) >= 0 = P(D<sub>0</sub>)。因此，势能函数P所表示的n个操作的平摊代价的总和就表示了实际代价总和的一个上界。

一次PUSH操作的实际代价c<sub>i</sub>为1，平摊代价为：

c<sub>i</sub>' = c<sub>i</sub> + P(D<sub>i</sub>) - P(D<sub>i-1</sub>) = 1 + 1 = 2

假设栈上第i个操作为MULTIPOP(S, k)，且弹出了k' = min(k, s)个对象，s为栈中目前元素的数量。操作的实际代价c<sub>i</sub>为k'，平摊代价为:

c<sub>i</sub>' = c<sub>i</sub> + P(D<sub>i</sub>) - P(D<sub>i-1</sub>) = k' - k' = 0

类似的，普通POP操作的代价也是0。

#### 二进制计数器递增1
定义势能函数P(D<sub>i</sub>)为计数器中1的个数。由于计数器中1的数量始终为非负，且P(D<sub>0</sub>)为0，因此对任意i，都有P(D<sub>i</sub>) >= P(D<sub>0</sub>) = 0，可知总平摊代价为总实际代价的一个上界。

定义第i次INCREMENT操作后计数器的势能为b<sub>i</sub>，即第i次操作后计数器中1的个数。我们来计算INCREMENT操作的平摊代价。设第i次INCREMENT操作对t<sub>i</sub>个位进行了复位(置为0)，该操作的实际代价至多是t<sub>i</sub>+1，因为除了将t<sub>i</sub>个位复位外，它至多将1个位设为1。如果b<sub>i</sub>=0，那么第i个操作复位k个位，则b<sub>i-1</sub>=t<sub>i</sub>=k。如果b<sub>i</sub>>0，则b<sub>i</sub>=b<sub>i-1</sub>-t<sub>i</sub>+1。在这两种情况中，都有b<sub>i</sub><=b<sub>i-1</sub>-t<sub>i</sub>+1，而且势能差为：

P(D<sub>i</sub>) - P(D<sub>i-1</sub>) <= 1 - t<sub>i</sub>

因此平摊代价为：

c<sub>i</sub>' = c<sub>i</sub> + P(D<sub>i</sub>) - P(D<sub>i-1</sub>) <= t<sub>i</sub> + 1 + 1 - t<sub>i</sub> = 2

可见，INCREMENT操作的平摊代价为常数。因此，任意n次INCREMENT操作的总代价为O(n)。