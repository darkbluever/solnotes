title: "也谈素性测试"
date: 2015-04-21 15:48:05
categories:
- 数学
tags:
- 素数
- 数学
mathjax: true
---

## 前言
很早之前读过 Matrix67的[素数与素性测试](http://www.matrix67.com/blog/archives/234)和 张洋的[聊聊如何检测素数](http://blog.codinglabs.org/articles/prime-test.html)，读了之后获益匪浅，不过一直没有自己动手写一次试试。

最近和朋友聊天，谈到如何找到距离某个数最近的素数，勾起了我的兴趣。遂重新读了这两篇博客，又查阅了一些其他的资料，自己动手写了一个简单的素数检测程序。下面记录了一些这次实践的经历以及脑洞。
<!-- more -->

## 素数定义
既然是素性测试，首先要明确素数的定义：
> A natural number (i.e. 1, 2, 3, 4, 5, 6, etc.) is called a prime number (or a prime) if it has exactly two positive divisors, 1 and the number itself.[1]
> Natural numbers greater than 1 that are not prime are called composite.
> -- [wikipedia](http://en.wikipedia.org/wiki/Prime_number)

即，一个自然数如果只能被1和它自身整除，那么这个自然数是素数。
大于1的自然数不是素数就是合数。

例如：
- 2是素数，因为它只能被1和2整除。
- 3是素数。
- 4不是素数，因为它可以被2整除。
- 5是素数。
- 6不是素数，因为它可以被2和3整除。

因此，还有一个结论，以十进制书写时，大于5的素数都是以1、3、7、9结尾的。可以用于快速判断一个数是否 ***不是*** 素数。

## 1的素性
我一直有一个疑问，1到底是素数还是合数。以前一直是因为看到的素数定义里面都指明了大于1的自然数作为前提，也就没有深究。1这个自然数就一直作为素数定义中的特例被我接受了下来。

这次认真看了下wiki，里面单独提及到了1的素性说明。
> Most early Greeks did not even consider 1 to be a number,[4] and so they did not consider it a prime. In the 19th century, however, many mathematicians did consider the number 1 a prime. For example, Derrick Norman Lehmer's list of primes up to 10,006,721, reprinted as late as 1956,[5] started with 1 as its first prime.[6] Henri Lebesgue is said to be the last professional mathematician to call 1 prime.[7]
>
> Although a large body of mathematical work would still be valid when calling 1 a prime, the fundamental theorem of arithmetic (mentioned above) would not hold as stated. For example, the number 15 can be factored as 3 · 5 and 1 · 3 · 5; if 1 were admitted as a prime, these two presentations would be considered different factorizations of 15 into prime numbers, so the statement of that theorem would have to be modified. Similarly, the sieve of Eratosthenes would not work correctly if 1 were considered a prime: a modified version of the sieve that considers 1 as prime would eliminate all multiples of 1 (that is, all numbers) and produce as output only the single number 1. Furthermore, the prime numbers have several properties that the number 1 lacks, such as the relationship of the number to its corresponding value of Euler's totient function or the sum of divisors function.

原来在19世纪有一些数学教认为1是素数。只不过很多数学成果在1是素数的条件下会有问题。例如：
- 算术基本定理（[Fundamental theorem of arithmetic](http://en.wikipedia.org/wiki/Fundamental_theorem_of_arithmetic)），又称为正整数的唯一分解定理，即：每个大于1的自然数均可写为素数的积，而且这些素因子按大小排列之后，写法仅有一种方式。例如: $6936 = 2^3 \times 3 \times 17^2，1200 = 2^4 \times 3 \times 5^2$。
如果1也是素数的话，15可以写成 $3 \times 5$，也可以写成 $1 \times 3 \times 5$。这两个表达式是不同的，这样算数基本定理就需要修改了。
- 埃拉托斯特尼筛法（[sieve of Eratosthenes](http://en.wikipedia.org/wiki/Sieve_of_Eratosthenes)），简称埃氏筛，是一种公元前250年由古希腊数学家埃拉托斯特尼所提出的一种简单检定素数的算法。
它通过从2的倍数开始，迭代地将每一个素数的倍数标记为合数来筛选出某个范围中的素数。
例如，如果要找出小于等于n的所有素数，埃拉托斯特尼的方法是：
1.创建一个从2到n的连续数列。
2.首先，令p等于2，即第一个素数。
3.将数列中等于p的倍数的自然数标记为合数，p本身不标记。
4.找到未被标记的大于p的第一个数，如果没有这样的数，方法结束。否则，令p等于找到的新数（就是下一个素数），然后重复步骤3。
![sieve of Eratosthenes](http://upload.wikimedia.org/wikipedia/commons/b/b9/Sieve_of_Eratosthenes_animation.gif)
当这个方法结束时，所有未被标记的数就是素数。
还有一个优化方式，在第4步中，若$p^2$大于n，则可以直接结束而不必寻找大于p的新数。因为小于等于$p^2$的数中的合数都已经标记过了。
如果1也是素数的话，在第三步结束后，列表中就只剩下1了。这样这个筛选法也需要修改。

另外，素数的一些特性1也没有，比如[欧拉函数](http://en.wikipedia.org/wiki/Euler%27s_totient_function)和[除数函数](http://en.wikipedia.org/wiki/Divisor_function)中的一些性质。关于1是否是素数的争论一直没有结束，只不过大多数人都倾向于1不是素数，因为1实在太特殊了，对很多理论都有影响。

写到这里发现这个坑太深了，欧拉函数和除数函数我没有仔细往下看，因为越往深追溯遇到的不知道的理论越多。看来这个坑不是一天两天能填上的，我决定先补一下数学基础再说T_T。如果对此感兴趣的朋友可以看下[What is the Smallest Prime?](https://cs.uwaterloo.ca/journals/JIS/VOL15/Caldwell1/cald5.html)和[The History of the Primality of One: A Selection of Sources](https://cs.uwaterloo.ca/journals/JIS/VOL15/Caldwell2/cald6.html)。

## 素性测试
### 试除法（Trial division）
试除法是判断一个给定整数n是否是素数的最基本的方法。该方法判断所有大于1且小于等于$\sqrt{n}$的整数是否可以整除n，如果有可以整除的数，则n是合数，否则判断n为素数。因为如果$n = a \times b, a \leq b$，则$a \leq \sqrt{n}$，所以只要试除到小于等于$\sqrt{n}$的整数即可。

有一种实现起来更高效的方式，只要试除2到$\sqrt{n}$之间的所有素数，即可判断出n是否是素数。因为合数都可以写成更小的素数的乘积（参考上面提到的算术基本定理）。

虽然试除法逻辑简单，实现比较容易，但是在面对大整数的时候，需要试除的素数数量会极速增长。根据[素数计数函数](http://en.wikipedia.org/wiki/Prime-counting_function), 小于$\sqrt{n}$的素数数量约等于 $\sqrt{n} / \ln(\sqrt{n})$，所以试除法需要至少判断这些数是否能整除n。如果 $n = 10^{20}$，则需要试除4亿多个素数。这在现实工程中是难以接受的。
<br />

### 费马素性测试（[Fermat_primality_test](http://en.wikipedia.org/wiki/Fermat_primality_test)）
既然试除法的效率无法接受，那么就需要寻找其他的替代方法。基于费马小定理，人们提出了一种基于随机化算法的检测方法来判断一个数是合数还是 ***可能是*** 素数，即费马素性测试。

首先我们还是先看一下理论基础，[费马小定理](http://en.wikipedia.org/wiki/Fermat%27s_little_theorem)：
***如果p是素数，a是小于p的正整数，那么 $a^{p-1} \equiv 1 \pmod p$。***

（关于费马小定理， Matrix67写得深入浅出，我应该不会写得比这个更好了，所以这部分直接引用自 Matrix67同学的[素数与素性测试](http://www.matrix67.com/blog/archives/234)）
> 这个证明就有点麻烦了。
> 首先我们证明这样一个结论：如果 $p$ 是一个素数的话，那么对任意一个小于 $p$ 的正整数$a$，$a, 2a, 3a, ..., (p-1)a$ 除以 $p$ 的余数正好是一个 $1$ 到 $p-1$ 的排列。例如，5是素数，3, 6, 9, 12除以5的余数分别为3, 1, 4, 2，正好就是1到4这四个数。
>
> 反证法，假如结论不成立的话，那么就是说有两个小于 $p$ 的正整数 $m$ 和 $n$ 使得 $na$ 和 $ma$ 除以 $p$ 的余数相同。不妨假设 $n > m$，则 $p$ 可以整除 $a(n-m)$。但 $p$ 是素数，那么 $a$ 和 $n-m$ 中至少有一个含有因子 $p$。这显然是不可能的，因为 $a$ 和 $n-m$ 都比 $p$ 小。
>
> 用同余式表述，我们证明了：
> $(p-1)! \equiv a \times 2a \times 3a \times ... \times (p-1)a \pmod p$
> 也即：
> $(p-1)! \equiv (p-1)! \times a^{p-1} \pmod p$
> 两边同时除以 $(p-1)!$，就得到了我们的最终结论：
> $1 ≡ a^{p-1} \pmod p$


费马素性测试，即选取小于p的正整数a，测试费马小定理是否成立，如果不成立，则p一定不是素数，如果成立，则p可能是素数。将上述过程重复n次，每次随机选取整数a进行测试。一般情况下，选取的a越多，测试结果越准确。

而有一些合数p，对于所有大于1且小于p的正整数都符合费马小定理，这样的数p称为卡迈克尔数（[Carmichael number](http://en.wikipedia.org/wiki/Carmichael_number)）。保罗·艾狄胥猜想有无限个卡迈克尔数，1994年 William Alford 、 Andrew Granville 及 Carl Pomerance 证明了这个命题。最小的卡迈克尔数是561。因为这些数的存在，使得费马素性检验变得不可靠。
<br />

### 米勒-拉宾素性测试（[Miller–Rabin_primality_test](http://en.wikipedia.org/wiki/Miller%E2%80%93Rabin_primality_test)）
米勒-拉宾素性测试也是一个基于随机化算法的素性测试方法，判断了一个数是合数还是 ***可能是*** 素数。和费马素性测试相似，米勒-拉宾素性测试也是通过判断一组对素数成立的等式是否对我们想测试的数也成立来实现素性测试的。

这个方法的原始版本来自卡内基梅隆大学的计算机系教授Gary Lee Miller，他首先提出了基于广义黎曼猜想的确定性算法，由于广义黎曼猜想并没有被证明，其后由以色列耶路撒冷希伯来大学的Michael O. Rabin教授作出修改，提出了不依赖于该假设的随机化算法。
<br />

#### 定理
米勒-拉宾素性测试基于下面的定理：
如果对模 $n$ 存在 $1$ 的非平凡平方根，则 $n$ 是合数。即对方程 $x^2 \equiv 1 \pmod n$，方程的解除了 $1$ 和 $n-1$ 两个平凡根之外，还有其它的根，那么 $n$ 肯定是合数。

假设 $p$是大于2的素数， $x$ 是摸 $p$ 等于1的平方根，则：
$x^2 \equiv 1 \pmod p$ ，即 $ (x - 1)( x + 1) \equiv 0 \pmod{p}$
也就是说素数 $p$ 整除 $(x - 1)( x + 1)$，因为 $p$ 是素数，它只能被 1 和 $p$ 整除，所以一定有 $x - 1 \equiv 0 \pmod p$ 或 $x - 1 \equiv 0 \pmod p$，所以 $x$ 模 $p$ 等于 1 或者 $p - 1$。
<br />

#### 方法
假设 $n$ 是素数，且 $n > 2$，则 $n - 1$是偶数，我们可以将其写为 $2^s \times d$ 的形式，其中 $s$ 和 $d$ 都是正整数，$d$ 是奇数。

结合费马小定理，$a^{p-1} \equiv 1 \pmod p$。如果 $p$ 是素数，则 $p - 1$是偶数，我们不断计算 $a^{p-1}$的平方根，那么最后一定会得到 1 或者 $p - 1$。

根据以上的理论，米勒-拉宾素性测试的等效表达为：
1.把 $n-1$ 表示成 $d \times 2^s$
2.计算 $a^{d \times 2^r} \pmod n ( 0 \leq r \leq s - 1 )$，（$r = s$ 时，此算式等效于费马小定理）
3.若结果是 1 和 $n-1$ 以外的值，测试结束，$n$是合数
4.若结果是 $n-1$，那么测试结束， $n$可能是素数，可以换一个底数 $a$ 进行下一轮计算
5.若结果是 1，那么仍然符合费马小定理，继续计算开方的值。即令 $r = r - 1$，$r \neq 0$ 的情况下回到第2步
6.直到计算到 $r = 0$，即 $a^d \pmod n$
7.若结果是 1 或 $n-1$，那么$n$可能是素数，可以换一个底数 $a$ 进行下一轮计算。否则测试结束，$n$是合数。

所以如果 $n$ 是一个素数，那么或者 $a^d \equiv 1 \pmod n$，或者存在某个 $r$ 使得 $a^{d \times 2^r} \equiv n-1 \pmod n ( 0 \leq r \leq s-1 )$。
<br />

以上面提到的卡迈克尔数 561为例，我们来看一下米勒-拉宾素性测试的实际应用：
1.560可以表示成 $35 \times 2^4$, 我们知道 $2^{560} = 1 \pmod {561}$。
2.计算得到 $2^{35 \times 2 ^ 3} = 1 \pmod {561}$
3.结果仍然符合费马小定理，继续计算，得到 $2^{35 \times 2 ^ 2} = 67 \pmod {561}$
4.测试结束，561是合数

下面引用 Matrix67同学的[素数与素性测试](http://www.matrix67.com/blog/archives/234)补充说明：
> Miller-Rabin素性测试同样是不确定算法，我们把可以通过以a为底的Miller-Rabin测试的合数称作以a为底的强伪素数(strong pseudoprime)。第一个以2为底的强伪素数为2047。第一个以2和3为底的强伪素数则大到1 373 653。
>
> 对于大数的素性判断，目前Miller-Rabin算法应用最广泛。一般底数仍然是随机选取，但当待测数不太大时，选择测试底数就有一些技巧了。比如，如果被测数小于4 759 123 141，那么只需要测试三个底数2, 7和61就足够了。当然，你测试的越多，正确的范围肯定也越大。如果你每次都用前7个素数(2, 3, 5, 7, 11, 13和17)进行测试，所有不超过341 550 071 728 320的数都是正确的。如果选用2, 3, 7, 61和24251作为底数，那么10^16内唯一的强伪素数为46 856 248 255 981。这样的一些结论使得Miller-Rabin算法在OI中非常实用。
>
> 通常认为，Miller-Rabin素性测试的正确率可以令人接受，随机选取k个底数进行测试算法的失误率大概为 $4^{-k}$。

<br />
### AKS素数测试
> AKS素数测试（又被称为Agrawal–Kayal–Saxena素数测试和Cyclotomic AKS test）是一个决定型素数测试算法。由三个来自印度坎普尔理工学院的计算机科学家，Manindra Agrawal、Neeraj Kayal 和 Nitin Saxena 提出，在2002年8月6日发表于一篇题为 PRIMES is in P 的论文。作者们因此获得了许多奖项，包含了2006年的哥德尔奖和2006年的 Fulkerson Prize。这个算法可以在多项式时间之内，决定一个给定整数是素数或者合数。
>
> AKS最关键的重要性在于它是第一个被发表的**一般的**、**多项式的**、**确定性的**和**无仰赖的**素数判定算法。先前的算法至多达到了其中三点，但从未达到全部四个。
>
> AKS算法可以被用于检测任何一般的给定数字是否为素数。很多已知的高速判定算法只适用于满足特定条件的素数。例如，卢卡斯-莱默检验法仅对梅森素数适用，而Pépin测试仅对费马数适用。
> 算法的最长运行时间可以被表为一个目标数字长度的多项式。ECPP和APR能够判断一个给定数字是否为素数，但无法对所有输入给出多项式时间范围。
> 算法可以确定性地判断一个给定数字是否为素数。随机测试算法，例如米勒-拉宾检验和Baillie–PSW，可以在多项式时间内对给定数字进行校验，但只能给出概率性的结果。
> AKS算法并未“仰赖”任何未证明猜想。一个反例是确定性米勒检验：该算法可以在多项式时间内对所有输入给出确定性结果，但其正确性却基于尚未被证明的广义黎曼猜想。
> -- [wikipedia](http://zh.wikipedia.org/wiki/AKS%E8%B3%AA%E6%95%B8%E6%B8%AC%E8%A9%A6)

此处留坑，有生之年。。。
[PRIMES is in P](http://www.cse.iitk.ac.in/users/manindra/algebra/primality_v6.pdf)
<br />

### 模幂运算（[Modular exponentiation](http://en.wikipedia.org/wiki/Modular_exponentiation)）
上面提到的费马素性测试和米勒-拉宾素性测试都涉及到了大量的模幂运算。
模幂运算计算了正整数 $b$ 的 $e$ 次方除以正整数 $m$ 的余数。用符号表示为，给定底数 $b$, 指数 $e$, 和模数 $m$, 模幂运算就是 $ c \equiv b^e \pmod{m}$。
<br />

#### 直接计算
最基础的办法就是直接计算 $ b^e $，然后对 $m$ 取余。假设 $b = 4, e = 13, m = 497$，则 $ c \equiv 4^{13} \pmod{497} $。计算得到 $ 4^{13} $ 等于67,108,864，对497取模计算出 $c$, 为445。

这个例子中 $b$ 和 $e$ 都是一位数，计算得到的 $ b^e $ 却是8位数。如果判断一个大数是否是素数的时候，$b$ 和 $e$ 都会很大，直接计算的计算效率将变得无法接受，所以需要更高效的计算方法。
<br />

#### 内存优化计算方法
这个方法相比于直接计算，计算次数增多了，但是因为每次计算大大节省了内存的使用，所以总的计算速度还是比直接计算更快。

这个方法的理论基础是，
$ c \equiv (a \cdot b) \pmod{m} $ 等价于 $ c \equiv (a \cdot (b\ (\mbox{mod}\ m))) \pmod{m} $
这样，不直接计算 $ b^e $，而是计算 $ \prod\_{i=0}^{e} b $，这样就可以根据上面的等式将每次乘以 $b$ 以后的结果取余，从而使计算过程中的中间变量一直小于模数 $m$，确保了内存使用效率。

具体的步骤如下：
1.令结果 $c = 1, e′ = 0.$
2.令 $ e′ = e′ + 1 $
3.计算 $ c \equiv (b \cdot c) \pmod{m} $
4.如果 $ e′ < e $，回到第二步；否则计算结束，此时 $ c \equiv b^e \pmod{m} $。

我们还用上面的例子来展示计算的具体过程：
$b = 4, e = 13, m = 497$
- $ e′ = 1. c = (1 \times 4) \pmod {497} = 4 \pmod {497} = 4.$
- $ e′ = 2. c = (4 \times 4) \pmod {497} = 16 \pmod {497} = 16.$
- $ e′ = 3. c = (16 \times 4) \pmod {497} = 64 \pmod {497} = 64.$
- $ e′ = 4. c = (64 \times 4) \pmod {497} = 256 \pmod {497} = 256.$
- $ e′ = 5. c = (256 \times 4) \pmod {497} = 1024 \pmod {497} = 30.$
- $ e′ = 6. c = (30 \times 4) \pmod {497} = 120 \pmod {497} = 120.$
- $ e′ = 7. c = (120 \times 4) \pmod {497} = 480 \pmod {497} = 480.$
- $ e′ = 8. c = (480 \times 4) \pmod {497} = 1920 \pmod {497} = 429.$
- $ e′ = 9. c = (429 \times 4) \pmod {497} = 1716 \pmod {497} = 225.$
- $ e′ = 10. c = (225 \times 4) \pmod {497} = 900 \pmod {497} = 403.$
- $ e′ = 11. c = (403 \times 4) \pmod {497} = 1612 \pmod {497} = 121.$
- $ e′ = 12. c = (121 \times 4) \pmod {497} = 484 \pmod {497} = 484.$
- $ e′ = 13. c = (484 \times 4) \pmod {497} = 1936 \pmod {497} = 445.$

最终计算得到 $ c = 445 $，和直接计算的结果相同。
这种方法的时间复杂度是 O(e)。
<br />

#### 位运算方法
这个方法大大减少了模幂运算的计算次数，并保持了和上一种方法一致的内存使用，是上面两种方法的结合。

首先，需要把指数 $e$ 写成二进制，即
$ e = \sum\_{i=0}^{n-1} a\_i \cdot 2^i $

其中， $e$ 可写为 $n$ 位的二进制数，$ a\_i $ 可以等于 0 或者 1，$0 ≤ i < n - 1$, 默认 $ a\_{n-1} = 1$。
于是，$ b^e $可以改写如下：
$ b^e = b^{\left( \sum\_{i=0}^{n-1} a\_i \cdot 2^i \right)} = \prod\_{i=0}^{n-1} \left( b^{2^i} \right) ^ {a\_i}$

所以，最后 $c$ 可写为：
$ c \equiv \prod\_{i=0}^{n-1} \left( b^{2^i} \right) ^ {a\_i}\ (\mbox{mod}\ m)$
<br />

如果 $e$ 很大的话，$n$ 也会比较大，此时 $b^{2^i}$ 仍然是一个比较耗费资源的幂运算。所以需要进一步优化：
$ \left({b^{2^i}}\right) ^ {a\_i} =
\begin{cases}
1 & \text{ if } a\_i = 0 \\\\
b^{2^i} = ({b^2})^{2^{i-1}} = ({b^2} \cdot {b^2}) ^ {2^{i-2}} \pmod m & \text{ if } a\_i = 1
\end{cases}
$

这样，通过将幂运算转化为底数 $b$ 的模幂运算，在保证了底数不大于 $m$ 的前提下，使得原公式中的 $2^i$ 部分始终等于1，就避免了高次幂运算。
例如：(此处我们暂不考虑 $a\_i$ )
- $i=0$ 时，$b^{2^i} = b ^ 1 = b \pmod m = b\_0$
- $i=1$ 时，$b^{2^i} = (b ^ 2)^{2^{i-1}} = b ^ 2 \pmod m = b\_0^2 \pmod m = b\_1$
- $i=2$ 时，$b^{2^i} = (b ^ 2)^{2^{i-1}} = (b ^ 2) ^ 2 = b\_1 ^ 2 \pmod m = b\_2$
- $i=3$ 时，$b^{2^i} = (b ^ 2)^{2^{i-1}} = (b\_1^2) ^ 2 = b\_2 ^ 2 \pmod m = b\_3$

这样就可以将 $c$ 改写为：
$ c \equiv \prod\_{i=0}^{n-1} \left( b\_i \right) ^ {a\_i}\ (\mbox{mod}\ m), b\_i = b\_{i-1}^2 \pmod m$
<br />

我们还以上面的例子展示计算过程：
$b = 4, e = 13 = 1101_{[2]}, m = 497$
- 令$c = 1$，此时 $i = 0, a\_i = 1$，所以 $c = (1 \times 4) \pmod {497} = 4 , b = (4 \times 4) \pmod {497} = 16$
- $i = 1, a\_i = 0$，所以 $c$ 不变，$ b = (16 \times 16) \pmod {497} = 256$
- $i = 2, a\_i = 1$，所以 $c = (4 \times 256) \pmod {497} = 30 , b = (256 \times 256) \pmod {497} = 429$
- $i = 3, a\_i = 1$，所以 $c = (30 \times 429) \pmod {497} = 445 , b = (429 \times 429) \pmod {497} = 151$，计算结束。
最终计算得到 $ c = 445 $，和上面的方法计算结果相同。

这个方法的时间复杂度由 O(e) 降为 O(log e).


## 结语
通过这次实践，又挖出了很多坑，发现了很多自己的不足之处。有一些坑还没能填上，留到以后数学水平提高了再回来填吧。另外，通过整理这些素性测试方法，对他们的理解也更深了一些。

纸上得来终觉浅，须知此事要躬行。


## References
1.Matrix67 [素数与素性测试](http://www.matrix67.com/blog/archives/234)
2.张洋 [聊聊如何检测素数](http://blog.codinglabs.org/articles/prime-test.html)
3.bindog [RSA周边——大素数是怎样生成的？](http://bindog.github.io/blog/2014/07/19/how-to-generate-big-primes/)
4.[wikipedia](http://en.wikipedia.org/wiki/Prime_number)
