---
title:  "선형대수 기초"
# categories: tech
tags: Linear Algebra
---

### 참고 도서
- Linear Algebra and its application 5th (Davic C.Lay)

### 선형 방정식 (Linear Equation)
- (Lay Ch1.1에서 다룬다)
- 변수 x1, ..., xn의 선형 방정식은 다음과 같은 형식을 가지는 방정식을 뜻한다. 이때 $a_i,b$는 상수라고 하자.
$$a_1x_1 + a_2x_2 + ... + a_nx_n = b$$
- 선형방정식은 아래와 같이 벡터의 내적(inner product) 형태로도 표현이 가능하다.
$$\begin{bmatrix} a_1 & a_2 & ... &a_n\end{bmatrix}\begin{bmatrix} x_1 \\ x_2 \\ ... \\x_n\end{bmatrix} = a^Tx = b$$
- 선형계(Linear system)은 하나 이상의 선형 방정식을 가리킨다.
$$a_{11}x_1 + a_{12}x_2 + ... + a_{1n}x_n = b_1$$
$$a_{21}x_1 + a_{22}x_2 + ... + a_{2n}x_n = b_2$$
- 선형계는 행렬과 컬럼벡터의 곱으로 표현이 가능하다.
$$\begin{bmatrix} a_{11} & a_{12} & ... &a_{1n}\\a_{21} & a_{22} & ... &a_{2n}\end{bmatrix}\begin{bmatrix} x_1 \\ x_2 \\ ... \\x_n\end{bmatrix} = Ax = b$$


### 벡터 방정식 (Vector Equations)
- (Lay Ch1.3에서 다룬다)
- 벡터 방정식은 $\bold{a_i}, \bold{b}$가 컬럼 벡터(column vector, 간단하게 벡터라고 함)인 방정식을 말한다.
$$x_1\bold{a_1} + x_2\bold{a_2} + ... + x_n\bold{a_n} = \bold{b}$$
- 이때 $\bold{b}$는 $\bold{a_1,...,a_n}$의 선형 결합 (Linear combination)으로 생성되었다고 한다.
- $v_1,..., v_p$가 $\reals^n$에 있을 때, $v_1,..., v_p$의 모든 선형 결합의 집합을 **스팬(Span)** 이라고 한다. 
<br>![](../assets/images/span.png)

### 선형 독립 (Linear Independence)
- (Lay Ch1.7에서 다룬다)
- 정의:\
$x_1\bold{v_1} + x_2\bold{v_2} + ... + x_p\bold{v_p} = 0$ (이 식을 Homogeneous라고도 부름)가 $\begin{bmatrix} 0\\0\\0\end{bmatrix}$과 같은 Trivial solution (영벡터의 해)만 가진다면 $\bold{v_1,v_2,...,v_p}$는 선형 독립이라고 한다.
- Non-trivial solution (영벡터 이외의 해)를 가진다면 선형 의존 (Linear Dependence)이라고 한다.

### 부분 공간 (Subspace)
- (Lay Ch4.1에서 다룬다)
- 벡터의 선형 결합에 닫혀있는 부분집합을 의미한다. 예를 들어 $\bold{u_1, u_2}$의 부분 공간은 {$cu_1 + du_2$} 가 된다.

### 기저 벡터 (Basis Vector)
- (Lay Ch4.3에서 다룬다)
- $\bold{B} = \{\bold{b_1,...,b_p}\}$ 가 다음과 같은 성질을 만족한다면 $\bold{B}$를 기적 벡터라고 한다:
  - $\bold{B}$는 선형 독립 집합이다.
  - Span{$\bold{b_1,...,b_p}$}이 완전한 Span을 이룬다.
