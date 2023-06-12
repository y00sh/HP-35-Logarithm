# HP-35-Logarithm
Logarithm function from HP-35 calculator in Vyper

HP-35 was the first device that could do scientific calculations. When the calculator was being created they had not yet developed a multiplying unit chip, they could only shift, add and subtract. They got around this by using lookup tables and specialized hardware that could shift in base 10. This could be useful for doing calculations in Ethereum EVM due to the high costs of arithmetic operations.

We will be calculating the logarithm base 2 of a given [decimal](https://docs.vyperlang.org/en/stable/types.html?highlight=decimal#decimals) type in vyper. If you are only solving for a given integer take a look at [snekmate](https://github.com/pcaversaccio/snekmate/tree/main) for a well written vyper contract. 

Let us first visualize the number we want to take the logarithm base 2 of. Let's call it 'M'

This number, M can be broken down as the sum of numbers as the numbers get smaller and approach zero

$$M = A_{0}^{q_{0}} + A_{1}^{q_{1}} + A_{2}^{q_{2}} + A_{3}^{q_{3}} + \ldots + A_{j}^{q_{j}}$$

Visually it would be like this, where the blocks gets smaller and smaller and you can add up all blocks to get M
<p align="center">
  <img width="400" height="300" src="https://github.com/y00sh/HP-35-Logarithm/assets/90585099/5a49b581-3cde-43ac-aaa1-6eb02ce5c18a"">
</p>
To decrease the block size each iteration let's use $`A_{j} = (1 + 2^{-j})`$

$$M = (1 + 2^{-1})^{q_{0}} + (1 + 2^{-2})^{q_{1}} + (1 + 2^{-3})^{q_{2}} + (1 + 2^{-4})^{q_{3}} + \ldots + (1 + 2^{-j})^{q_{j}}$$

log product rule says $` \log(M \times N)=\log(M)+log(N) `$ 

$$\log(M \times A_{j}^{q_{j}})= \log(M)+\log(A_{0}^{q_{0}})+\log(A_{1}^{q_{1}})+\log(A_{2}^{q_{2}})+\log(A_{3}^{q_{3}})+\ldots+\log(A_{j}^{q_{j}})$$

log exponent rule says $`\log(A_{j}^{q_{j}})=q_{j}\log(A_{j})`$ , and $`A_j`$ approaches 1 as j increases

$$\log(M \times 1^{j})= \log(M)+ q_{0}\log(A_{0})+q_{1}\log(A_{1})+q_{2}\log(A_{2}^{q_{3}})+q_{j}\log(A_{3})+\ldots+q_{j}\log(A_{j})$$

The Eureka moment is that even though $`A_j`$ approaches 1 as j increases. Henry Briggs in writing lookup tables for his book-Arithmetica Logarithmica (1624)-found that $`\log(1+x) \approx x`$ for example $`\log(1 + 2^{-33})=\log(1 + 0.00000000011641532182) \approx 0.00000000016795180747`$. They both have 9 leading zeros so if $`q_{33}=1`$ there would be a 'digit' at that decimal place.

M could be a large number, let's fold the value M to be between 1-2. If we do that we can revise $`\log(M)=\log(2^k)=k\log(2)+\log(r)`$

## $$\log(M)= k\log(2)+ q_{0}\log(A_{0})+q_{1}\log(A_{1})+q_{2}\log(A_{2}^{q_{3}})+q_{j}\log(A_{3})+\ldots+q_{j}\log(A_{j}) \enspace Eq. 1$$

Eq. 1 above can be broken up into two parts and we can think of our answer to $`log(M)`$ as a fixed binary point number with an integer part and decimal part. The $`k\log(2)`$ part will find the integer and if $`\log(M)=\log(2^k)=k\log(2)`$ can't fit into a power of 2 perfectly there'll be a remainder. The remainder part will find the rest of the digits with the $`q_{0}\log(A_{0})+q_{1}\log(A_{1})+q_{2}\log(A_{2}^{q_{3}})+q_{j}\log(A_{3})+\ldots+q_{j}\log(A_{j})`$ part 

Let's start our code with the first part: $`k\log(2)`$

To find log2(M), [bit twiddling hacks](http://graphics.stanford.edu/~seander/bithacks.html) says to keep dividing M by 2 and count the division performed until our value is below 2. But our goal in Vyper is to minimize gas and since the input to our smart contract is a type decimal, we could potentially be doing the division operation (5gas) 133 times.  
I took a peek at what [Vyperlang](https://github.com/vyperlang/vyper/pull/2501) did. Let's visualize the binary expansion of our value 'M' 

| 2<sup>7</sup> | 2<sup>6</sup> | 2<sup>5</sup> | 2<sup>4</sup> | 2<sup>3</sup> | 2<sup>2</sup> | 2<sup>1</sup> | 2<sup>0</sup> |
| --- | --- | --- | --- | --- | --- | --- | --- |
|  7  |  6  |  5  |  4  |  3  |  2  |  1  |  0  |

For example if M was 123.456 , we can convert it to $`2^4 \times 2^2 \times 1.929`$ We have now folded 123.456 to 1.929 and found our value k by adding up the exponents. $`\log(123.456)=\log(2^4\times2^2\times1.929)=\log(2^4)+\log(2^2)+\log(1.929)=4log(2)+2log(2)+log(1.929)=6log(2)+log(1.929)`$ ,  k=6

The HP-35 uses a lookup table to save on computation so instead of computing $`2^7 - 2^0`$ let's create a list. The max number of bits for the integer part of a decimal type in Vyper is 133 so we only need to check $`2^{128}`$ since $`2^{256}`$ would be too big

lookup table t
| 2\^7 | 2\^6 | 2\^5 | 2\^4 | 2\^3 | 2\^2 | 2\^1 | 2\^0 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 128 | 64 | 32 | 16 | 8 | 4 | 2 | 1 |

lookup table p
| 2\^128 | 2\^64 | 2\^32 | 2\^16 | 2\^8 | 2\^4 | 2\^2 | 2\^1 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| 340282366920938463463374607431768211456 | 18446744073709551616 | 4294967296 | 65536 | 256 | 16 | 4 | 2 |

```python
    k: uint8= 0     # will never exceed 256
    t: uint8[8]= [128, 64 ,32 ,16 ,8 ,4 ,2 ,1]
    p: decimal[8]= [340282366920938463463374607431768211456.0 ,18446744073709551616.0 ,4294967296.0 ,65536.0 ,256.0 ,16.0, 4.0 , 2.0]
    
    # not scaling up m here but need it to find k, use an temporary variable temp
    temp: decimal = m
    for i in range(8):  # iterate from 0 to 7
        if temp >= p[i]:
            temp /= p[i] 
            k += t[i]
    
    # the remainder is calculated here based on k      
    m = m / (convert((1 << k), decimal) / 10000000000.0)
    # to get the full decimals of the remainder scale it by 10^10
```
Now implementing the second part Eq. 1 $`q_{0}\log(A_{0})+q_{1}\log(A_{1})+q_{2}\log(A_{2}^{q_{3}})+q_{j}\log(A_{3})+\ldots+q_{j}\log(A_{j})`$. We need to find the values $`q_j`$. Remember that even though it's addition, the order is very important to find each q

Let's first set M to our remainder after folding it to be between 1-2.  In our last example it was 1.929
Remember that we are trying to find the rest of the binary digits. so far we have 110._ _ _ _ _ _ _ _ . Now let's find the rest of the digits:
If M_j < A, that binary digit will be 0.  If M_j > A the digit will be $`\frac{M_j}{A_{j}^{q_j}}`$. We have chosen to solve for logarithm base 2 so $`q_j`$ will only be 0 or 1. If we were solving for logarithm base 10 $`q_j`$ would be a value from 0 to 9. In python/vyper lists start at 0, let's work through the first couple of 'digits'. Remember that $` A_{j} = (1 + 2^{-j}) `$ 

$`j=0`$ $`M_{j-1}=1.929`$ now solve for $`q_0`$ 
### $\frac{1.929}{1.5^{q_0}}\therefore q_0=1, M=1.286$
### $\frac{1.286}{1.25^{q_0}}\therefore q_1=1, M=1.0288$
### $\frac{1.0288}{1.125^{q_0}}\therefore q_2=0, M=1.0288$
### $\frac{1.0288}{1.0625^{q_0}}\therefore q_3=0, M=1.0288$
### $\frac{1.0288}{1.03125^{q_0}}\therefore q_4=0, M=1.0288$
### $\frac{1.0288}{1.015625^{q_0}}\therefore q_5=1, M=1.129723076$
### $`\dots`$
### $\frac{M_{j-1}}{A_{j}^{q_j}}$

we now have log(123.456)=110.110001... but how many digits do we need? Since Vyper's decimal type has 10 decimals we need 10 digits but how many binary bits ($`q_j`$) is this? we need precision down to 0.0000000001 or $`1\times10^{10}`$. In fixed point the 'decimal' is known as fractional bits
<p align="center">
  <img width="400" alt="image" src="https://github.com/y00sh/HP-35-Logarithm/assets/90585099/24b54c8d-ec88-4409-9ec7-4e588c7ed92e">
</p>

$`\frac{1}{2^j}`$ we need to find j where $`\frac{1}{2^j} < 1\times10^{10}`$

```
1/2^j < 1e-10
1 < 2^j * 1e-10
1e10 < 2^j
log2(1e10) < j
33.2 < j
j is 34 
```

Knowing j we can go back and compute our lookup table for 
$$\log(M)= k\log(2)+ q_{0}\log(A_{0})+q_{1}\log(A_{1})+q_{2}\log(A_{2}^{q_{3}})+q_{j}\log(A_{3})+\ldots+q_{j}\log(A_{j}) \enspace Eq. 1$$

We'll find a list of q[] and multiply by our lookup table for $`\log(A_j)`$. To create the lookup table let's first compute a list of A[] with 34 elements and than find the logarithm base 2 of each element in A[]
```python
A: decimal[34] = [1.5, 1.25, 1.125, 1.0625, 1.03125, 1.015625, ... , 1.0000000000582076609134674072265625]
```

We have a problem. In Vyper the decimal type will truncate everything after 10 decimals. Our last value in list $`A_34`=1.0000000000 This means log(1)=0 and our last digit will be 0 even if $`q_34`=1$. We won't have accuracy in the 8th to 10th decimal place.

The HP-35 calculator went down to 6 decimals for computing the natural logarithm, in our smart contract we are going even further
<p align="center">
  <img width="400" alt="image" src="https://github.com/y00sh/HP-35-Logarithm/assets/90585099/a0eaf80a-cddc-4553-96eb-33e227cc22b8">
</p>
  
Luckily since we are working with with fixed point, there is a simple solution: we can shift the decimal aka radix point to the right. The integer part of the number will increase and we must ensure we dont overflow. But we now have more information while maintaining our 10 decimals places. Let's wait to shift M until after it is folded to less than 2. This helps avoid overflow since it is a small value. we can shift M a maximum of $`10^{39}`$ -since M<2- but to get accuracy up to the 10th decimal place we only need to double the number of decimals places we perform arithmetic on, we will shift the fixed point by $`10^{10}`$ Before returning the answer shift M back to the original fixed point.  Here's the Vyper code:
  
```python
  # initialize lists 
    A: decimal[34] = [
       15000000000.0000000000, 12500000000.0000000000, 
       11250000000.0000000000, 10625000000.0000000000, 
       10312500000.0000000000, 10156250000.0000000000, 
       10078125000.0000000000, 10039062500.0000000000, 
       10019531250.0000000000, 10009765625.0000000000, 
       10004882812.5000000000, 10002441406.2500000000, 
       10001220703.1250000000, 10000610351.5625000000, 
       10000305175.7812500000, 10000152587.8906250000, 
       10000076293.9453125000, 10000038146.9726562500, 
       10000019073.4863281250, 10000009536.7431640625, 
       10000004768.3715820312, 10000002384.1857910156, 
       10000001192.0928955078, 10000000596.0464477539, 
       10000000298.0232238770, 10000000149.0116119385, 
       10000000074.5058059692, 10000000037.2529029846, 
       10000000018.6264514923, 10000000009.3132257462, 
       10000000004.6566128731, 10000000002.3283064365, 
       10000000001.1641532183, 10000000000.5820766091
       ]
     
    log2A: decimal[34] = [
    5849625007.2115618145, 3219280948.8736234787, 
    1699250014.4231236290, 874628412.5033940825, 
    443941193.5845343765, 223678130.2845450826, 
    112272554.2325412033, 56245491.9387810691, 
    28150156.0705403815, 14081943.9280838890, 
    7042690.1124664325, 3521774.8030102723, 
    1760994.8644250603, 880524.3012217690, 
    440268.8682731671, 220136.1136034049, 
    110068.4766748144, 55034.3433064860, 
    27517.1978956128, 13758.6055084113, 
    6879.3043943584, 3439.6526072176, 
    1719.8264061184, 859.9132286866, 
    429.9566207501, 214.9783119767, 
    107.4891563888, 53.7445782945, 
    26.8722891722, 13.4361445924, 
    6.7180722977, 3.3590361492, 
    1.6795180747, 0.8397590373
    ]

    q: uint8[34] = [0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0]
    
    #find our list q 
    for j in range(34): 
        if m >= A[j]:
            m = (m * 10000000000.0)/A[j]
            q[j] = 1
    
    fracbits: decimal = 0.0
    for i in range(34):
        if q[i] == 1:
            fracbits += log2A[i]
    
    return (fracbits/10000000000.0) + convert(k, decimal)
```
Our $`q = [1, 1, 0, 0, 0, 1, 1, 1, 0, 1, 0, 0, 1, 1, 1, 1, 0, 0, 1, 1, 0, 1, 0, 0, 1, 1, 0, 1, 0, 1, 0, 1, 1, 0]`$ 

for every element in $`q`$ that has a 1 we take the corresponding element from the list log2A and add all the elements we pulled together.  The sum will be our answer scaled by $`10^{10}`$. We have found the result digit-by-digit. This approach is also similar to the more famous HP-35 CORDIC algorithm. 
  
After we scale it back to the original radix point, we now how our fractional bits and the answer is $`\log(123.456) = 6.9478531433`$

We now have a smart contract calculating the logarithm base 2 function. Based on this we can calculate the solution to any other logarithm base x by:
  
### $$\log_{x}(M)=\frac{\log_{2}(M)}{\log_{2}(x)} $$
     
We can also calculate exponents using our logarithm function:

### $$b^x = 2^{\log_2(b^x)} = 2^{x\log_2(b)}$$
  
This code is meant to introduce you to interesting mathematical algorithms. Gas savings for users are of greater importance and the HP-35 calculator used lookup tables to minimize computation. It also didn't utilize exponents in our code which are particular expensive. 

To solve for the largest decimal value, 18707220957835557353007165858768422651595.9365500927 used a total of 4275 gas

#the current proposal to have a builtin log base 2 function in vyper is [here](https://github.com/vyperlang/vyper/pull/2501)
  
  
  




