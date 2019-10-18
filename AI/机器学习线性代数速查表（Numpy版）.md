- 作者：景略集智

- 链接：https://zhuanlan.zhihu.com/p/48580747

- 来源：知乎

-  

- Python 中的数值计算库 Numpy 提供了很多线性代数函数，在机器学习中非常实用。

- 本文会为大家总结在机器学习实践中处理向量和矩阵所用的重要函数，这是一份速查表，所有的示例都很简短，假设你已经熟悉文中所涉及的运算。

- 在机器学习中使用 Numpy 执行的所有线性代数运算都包含在内，建议收藏备用。

- 速查表共分为 7 部分：

- - 数组
  - 向量
  - 矩阵
  - 矩阵类型
  - 矩阵运算
  - 矩阵分解
  - 统计
  - 数组

- 有很多种方法创建 Numpy 数组。

- 数组

- from numpy import array
  A = array([[1,2,3],[1,2,3],[1,2,3]])

-  

- Empty函数

- from numpy import empty
  A = empty([3,3])

-  

- Zeros函数

- from numpy import zeros
  A = zeros([3,5])

-  

- Ones函数

- from numpy import ones
  A = ones([5, 5])

- - 向量

- 向量就是一列标量。

-  

- 向量加法

- c = a + b

-  

- 向量减法

- c = a - b

-  

- 向量乘法

- c = a * b

-  

- 向量除法

- c = a / b

-  

- 向量点积

- c = a.dot(b)

-  

- 向量-标量乘法

- c = a * 2.2

-  

- 向量范数

- from numpy.linalg import norm
  l2 = norm(v)

- - 矩阵

- 矩阵就是由标量组成的二维数组。

-  

- 矩阵加法

- C = A + B

-  

- 矩阵减法

- C = A - B

-  

- 矩阵乘法（Hadamard积）

- C = A * B

-  

- 矩阵除法

- C = A / B

-  

- 矩阵-矩阵乘法（点积）

- C = A.dot(B)

-  

- 矩阵-向量乘法（点积）

- C = A.dot(b)

-  

- 矩阵-标量乘法

-  

- C = A.dot(2.2)

- - 矩阵类型

- 在更广泛的计算中，常将不同类型的矩阵用作元素。

-  

- 三角矩阵

- \# lower
  from numpy import tril
  lower = tril(M)
  \# upper
  from numpy import triu
  upper = triu(M)

-  

- 对角矩阵

- from numpy import diag
  d = diag(M)

-  

- 单位矩阵

- from numpy import identity
  I = identity(3)

- - 矩阵运算

- 在更广泛的计算中，矩阵运算常会被用作元素。

-  

- 矩阵转置

- B = A.T

-  

- 矩阵求逆

- from numpy.linalg import inv
  B = inv(A)

-  

- 矩阵的迹

- from numpy import trace
  B = trace(A)

-  

- 矩阵行列式

- from numpy.linalg import det
  B = det(A)

-  

- 矩阵秩

- from numpy.linalg import matrix_rank
  r = matrix_rank(A)

- - 矩阵分解

- 矩阵分解就是将其拆解为几个部分，让其它运算更简单、在数值上更稳定。

-  

- LU分解

- from scipy.linalg import lu
  P, L, U = lu(A)

-  

- QR分解

- from numpy.linalg import qr
  Q, R = qr(A, 'complete')

-  

- 特征分解

- from numpy.linalg import eig
  values, vectors = eig(A)

-  

- 奇异值分解

- from scipy.linalg import svd
  U, s, V = svd(A)

- - 统计

- 统计部分总结了向量和矩阵的知识，常被用于组成更广泛的计算。

-  

- 均值

- from numpy import mean
  result = mean(v)

-  

- 方差

- from numpy import var
  result = var(v, ddof=1)

-  

- 标准偏差

- from numpy import std
  result = std(v, ddof=1)

-  

- 协方差矩阵

- from numpy import cov
  sigma = cov(v1, v2)

-  

- 线性最小二乘

- from numpy.linalg import lstsq
  b = lstsq(X, y)