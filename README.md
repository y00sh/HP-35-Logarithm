# HP-35-Logarithm
Logarithm function from HP-35 calculator in Vyper

HP-35 was the first device that could do scientific calculations. It was able to do this by overcoming the technology at the time with a solid basis of mathematics and creativity. This could be useful for doing calculations in Ethereum EVM due to the high costs of arithmetic operations. 

Let us first visualize the number we want to take the logarithm of. Let's call it 'M'

This number, M can be broken down as the sum of numbers as the numbers get smaller and approach zero

$$M = A_{0}^{q_{0}} + A_{1}^{q_{1}} + A_{2}^{q_{2}} + A_{3}^{q_{3}} + \ldots + A_{j}^{q_{j}} \enspace eq. 1$$

Visually it would be like this, where the blocks gets smaller and smaller and you can add up all blocks to get M
<p align="center">
  <img width="200" height="200" src="https://github.com/y00sh/HP-35-Logarithm/assets/90585099/8df22aa2-b181-409c-b5ea-15c1198f5f5d">
</p>

For our algorithm to decrease the block size each iteration $` A_{j} = (1 + 2^{-j}) `$

$$M = (1 + 2^{-1})^{q_{0}} + (1 + 2^{-2})^{q_{1}} + (1 + 2^{-3})^{q_{2}} + (1 + 2^{-4})^{q_{3}} + \ldots + (1 + 2^{-j})^{q_{j}}$$

log product rule says $` \log(M \times N)=\log(M)+log(N) `$ but since $`(1 + 2^{-j}) \approx 1, \log(1)=\log(1 + 2^{-j})`$

$$\log(M \times 1^{j})= \log(M)+\log(A_{0}^{q_{0}})+\log(A_{1}^{q_{1}})+\log(A_{2}^{q_{2}})+\log(A_{3}^{q_{3}})+\ldots+\log(A_{j}^{q_{j}})$$

log exponent rule says $` \log(A_{j}^{q_{j}}=q_{j}\log(A_{j} `$

$$\log(M \times 1^{j})= \log(M)+ q_{0}\log(A_{0})+q_{1}\log(A_{1})+q_{2}\log(A_{2}^{q_{3}})+q_{j}\log(A_{3})+\ldots+q_{j}\log(A_{j})$$

To save on computation we can precompute a list for all the values of $`A[j]`$ and all the values of $`\log(A[j])`$





1. First, the constants are defined:

   - $` r = M \times A_{0}^{q_{0}} \times A_{1}^{q_{1}} \times A_{2}^{q_{2}} \times A_{3}^{q_{3}} \times \ldots \times A_{j}^{q_{j}} `$
   - $` (A[j] = 1 + 2^{-j}\) `$ j in the range from 0 to 33 
   - \(\ln(A[j]) = \ln(1 + 2^{-j})\)

2. Then the function `divide(M, j)` is defined to find the largest integer \(q[j]\) such that \(M \geq A[j]^{q[j]}\), and it returns the new value of \(M = M/A[j]^{q[j]}\) and \(q[j]\).

3. The function `compute_ln(M)` computes the natural logarithm of \(M\) by reducing \(M\) to the interval (1, 2] and then applying the series expansion. The steps are as follows:

   - If \(M < 1\), then \(M\) is replaced by \(1/M\) and a sign factor is introduced as \(-1\). Otherwise, the sign factor is \(1\).
   - The next step is to reduce \(M\) to the interval (1, 2]. This is done by repeatedly dividing \(M\) by 2 until it falls within the desired interval. The number of divisions, \(n\), is counted. 

     This step is represented by:
     \[M = \frac{M}{2^n}\]

   - Then the algorithm computes the \(q[j]\) values and the final value of \(I\) using the `divide` function described above.

   - The final approximation for \(\ln(M)\) is given by:
     
     \[\ln(M) = \pm (n\ln(2) + \sum_{j=0}^{33} q[j] \ln(A[j]))\]
     
     
     
     y = b^x = 2^{\log_2(b^x)}
     y = b^x = 2^{x\log_2(b)}



