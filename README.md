# HP-35-Logarithm
Logarithm function from HP-35 calculator in Vyper

The method for approximating the natural logarithm using a series expansion in this code can be expressed with the following mathematical equations:

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



