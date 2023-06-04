# HP-35-Logarithm
Logarithm function from HP-35 calculator in Vyper

HP-35 was the first device that could do scientific calculations. It was able to do this by overcoming the technology at the time with a solid basis of mathematics and creativity. This could be useful for doing calculations in Ethereum EVM due to the high costs of arithmetic operations. 

Let us first visualize the number we want to take the logarithm base 2 of. Let's call it 'M'

This number, M can be broken down as the sum of numbers as the numbers get smaller and approach zero

$$M = A_{0}^{q_{0}} + A_{1}^{q_{1}} + A_{2}^{q_{2}} + A_{3}^{q_{3}} + \ldots + A_{j}^{q_{j}}$$

Visually it would be like this, where the blocks gets smaller and smaller and you can add up all blocks to get M
<p align="center">
  <img width="200" height="200" src="https://github.com/y00sh/HP-35-Logarithm/assets/90585099/8df22aa2-b181-409c-b5ea-15c1198f5f5d">
</p>

To decrease the block size each iteration let's use $` A_{j} = (1 + 2^{-j}) `$

$$M = (1 + 2^{-1})^{q_{0}} + (1 + 2^{-2})^{q_{1}} + (1 + 2^{-3})^{q_{2}} + (1 + 2^{-4})^{q_{3}} + \ldots + (1 + 2^{-j})^{q_{j}}$$

log product rule says $` \log(M \times N)=\log(M)+log(N) `$ but since $`(1 + 2^{-j}) \approx 1, \log(1)=\log(1 + 2^{-j})`$

$$\log(M \times 1^{j})= \log(M)+\log(A_{0}^{q_{0}})+\log(A_{1}^{q_{1}})+\log(A_{2}^{q_{2}})+\log(A_{3}^{q_{3}})+\ldots+\log(A_{j}^{q_{j}})$$

log exponent rule says $` \log(A_{j}^{q_{j}}=q_{j}\log(A_{j} `$

$$\log(M \times 1^{j})= \log(M)+ q_{0}\log(A_{0})+q_{1}\log(A_{1})+q_{2}\log(A_{2}^{q_{3}})+q_{j}\log(A_{3})+\ldots+q_{j}\log(A_{j})$$

M could be a large number, let's fold the value M to be between 1-2. If we do that we can revise $`\log(M)=\log(2^k)=k\log(2)`$

$$\log(M)= k\log(2)+ q_{0}\log(A_{0})+q_{1}\log(A_{1})+q_{2}\log(A_{2}^{q_{3}})+q_{j}\log(A_{3})+\ldots+q_{j}\log(A_{j}) \enspace Eq. 1$$

Eq. 1 above can be broken up into two parts and we can think of our answer to $`log(M)`$ as a fixed binary point number with an integer part and decimal part. The $`k\log(2)`$ part will find the integer and if $`\log(M)=\log(2^k)=k\log(2)`$ can't fit into a power of 2 perfectly there'll be a remainder. The remainder part will find the rest of the digits with the $`q_{0}\log(A_{0})+q_{1}\log(A_{1})+q_{2}\log(A_{2}^{q_{3}})+q_{j}\log(A_{3})+\ldots+q_{j}\log(A_{j})`$ part 

Let's start our code with the first part: $`k\log(2)`$

To find log2(M), [bit twiddling hacks](http://graphics.stanford.edu/~seander/bithacks.html) says to keep dividing M by 2 and count the division performed until our value is below 2. But our goal in Vyper is to minimize gas and since the input to our smart contract is a type [decimal](https://docs.vyperlang.org/en/stable/types.html?highlight=decimal#decimals) we could potentially be doing division operation (5gas) 133 times.  
I took a peak at what [Vyperlang](https://github.com/vyperlang/vyper/pull/2501) did. Let's visualize the binary expansion of our value 'M' 

| 2<sup>7</sup> | 2<sup>6</sup> | 2<sup>5</sup> | 2<sup>4</sup> | 2<sup>3</sup> | 2<sup>2</sup> | 2<sup>1</sup> | 2<sup>0</sup> |
| --- | --- | --- | --- | --- | --- | --- | --- |
|  7  |  6  |  5  |  4  |  3  |  2  |  1  |  0  |

For example if M was 11.1 , we can convert it to $`2^3 + 2^2 + 1.1`$ We have now folded 11.1 to 1.1 and found our value k by adding up the exponents. $`\log(11.1)=\log(2^3)+\log(2^2)+\log(1.1)=3log(2)+3log(2)+log(1.1)=5log(2)+log(1.1)`$

The HP-35 uses lookup table to save on computation so instead of computing $`2^7 - 2^0`$ let's create a list and call it. The max number of bits for the integer part is 133 so we only need to check $`2^128`$ since $`2^256`$ would be too big

lookup table t
| 2\^7 | 2\^6 | 2\^5 | 2\^4 | 2\^3 | 2\^2 | 2\^1 | 2\^0 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |

lookup table p
| 2\^128 | 2\^64 | 2\^32 | 2\^16 | 2\^8 | 2\^4 | 2\^2 | 2\^1 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 340282366920938463463374607431768211456 | 18446744073709551616 | 4294967296 | 65536 | 256 | 16 | 4 | 2 |

```python
n_exp: uint256= 0
    t: decimal[8]= [128.0, 64.0 ,32.0 ,16.0 ,8.0 ,4.0 ,2.0 ,1.0]
    p: decimal[8]= [340282366920938463463374607431768211456.0 ,18446744073709551616.0 ,4294967296.0 ,65536.0 ,256.0 ,16.0, 4.0 , 2.0]
    for i in range(8):  
        if M >= p[i]:
            M /= p[i]
            n_exp += convert(t[i], uint256)
```
Now implementing the second part Eq. 1 $`q_{0}\log(A_{0})+q_{1}\log(A_{1})+q_{2}\log(A_{2}^{q_{3}})+q_{j}\log(A_{3})+\ldots+q_{j}\log(A_{j})`$ part. We need to find the values $`Q_j`$. It's easier to reason when we look back at this equation, $'M = A_{0}^{q_{0}} + A_{1}^{q_{1}} + A_{2}^{q_{2}} + A_{3}^{q_{3}} + \ldots + A_{j}^{q_{j}}'$ $`q_0`$ can be find by Even though it is only addition








     
     \[\ln(M) = \pm (n\ln(2) + \sum_{j=0}^{33} q[j] \ln(A[j]))\]
     
     
     
     y = b^x = 2^{\log_2(b^x)}
     y = b^x = 2^{x\log_2(b)}



