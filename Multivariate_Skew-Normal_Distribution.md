# Multivariate Skew-normal Distribution

## Definition and Analysis

- Definition
  
  > $p(z) = 2 \phi (z;\Omega) \Phi(\alpha^T z)$

- Shape parameter: $\alpha$ (the actual shape is regulated in a more complex way)
- cumulant generating function

  > $K(t)=log M(t)=\frac{1}{2} t^T \Omega t + log(2\Phi(\delta^Tt))$

  where,

  $\delta = \frac{1}{\sqrt(1+\alpha^T\Omega\alpha)}\Omega \alpha$

  - Mean vector: $\mu_z = (2/\pi)^{1/2} \delta$
  - Variance Matrix: $var\{Z\} = \Omega - \mu_z \mu_z^T$
- Proposition 1:

  Suppose that

  $$
  \left(
      \begin{aligned}
          X_0 \\
          X
      \end{aligned}
  \right) \sim N_{k+1}(0, \Omega^*), 
  \qquad \Omega^* =  \left(
      \begin{aligned}
          1 \quad \delta^T \\
          \delta \quad \Omega
      \end{aligned}
  \right)

  $$

  Where $X_0$ is a *scalar component* and $\Omega^*$ is a correlation matrix. Then

  $$
  = \{
      \begin{aligned}
          X  &\quad if X_0 > 0\\
          -X &\quad otherwise
      \end{aligned}

  $$

  is $SN_{k}(\Omega, \alpha)$ where

  $$
  \alpha = \frac{1}{(1-\delta^T\Omega^{-1}\delta)^{1/2}}\Omega^{-1}\delta

  $$

## Linear and Quadratic forms

- Marginal Distributions:

  The marginal distribution of a subset of the components of $Z$ is still a skew-normal variate.

  - Proposition 2:
- Linear Transforms

  - Proposition 3:

    $$
    A^TZ \sim SN_k(A^T \Omega A, A^{-1} \alpha)

    $$
  - Proposition 4

    For a variable $Z \sim SN_k(\Omega, \alpha)$ , there exists a linear transform $Z^*= A^{*}Z$, such that $Z^* \sim SN_k(I_k, \alpha^*)$, where *at most one* component of $\alpha^{*}$ is zero.

    This plays a similar role as the one which converts a multivariate normal variable into a spherical form.

    There is also an inverted process. It is possible to span the whole class $SN_k(\Omega, \alpha)$ starting from $Z^{*}$, and applying suitable linear transformations. The density of $Z^{*}$ is of the form

    $$
    prod_{i=1}^{k} \phi(u_i)\Phi(\alpha_m^*u_m)
    $$

    where

    $$
    pha^*_m = (\alpha^T \Omega \alpha)^{1/2}
    $$

    is the only non-zero component of $\alpha^{*}$
  - Proposition 5 (along with Proposition 6 examine the issue of independence)

    Let $Z \sim SN_{k}(\Omega, \alpha)$ and $A$ is as in Proposition 3, and the linear transform

    $$
    = A^TZ = 
        \left(
        \begin{aligned}
            Y_1 \\
            \vdots\\
            Y_h
        \end{aligned}
        \right)
        =  \left(
        \begin{aligned}
            A_1^T \\
            \vdots \\
            A_h^T
        \end{aligned}
        \right)Z

    $$

    where the matrices $A_1, \dots , A_h$ have $m_1, \dots, m_h$ columns, respectively. Then

    $$
    i \sim SN_{m_i}(\Omega_{Y_i}, \alpha_{Y_i})

    $$

    where

    $$
    mega_{Y_i}=A_i^T \Omega A_i, \qquad \alpha_{Y_i} = \frac{(A_i^T \Omega A_i)^{-1}A_i^T \Omega \alpha}{(1+\alpha^T (\Omega-\Omega A_i(A_i^T \Omega A_i)^{-1}A_i^{T} \Omega)\alpha)^{1/2}}

    $$

    Since $\Phi(u+v) \neq \Phi(u)\Phi(v)$, it follows that **at most one** of the $Y_i$ above can be a 'proper' skew-normal variate, while others are all regular normal variates, if mutual independence holds.
  - Proposition 6
    If $Z \sim SN_{k}(\Omega, \alpha)$, and $A^T \Omega A$ is a *positive definite correlation* matrix, then the variables $(Y_1,\dots,Y_h)$ defined in Proposition 5 are independent iff the following conditions holds simultaneously:

    (a) $A_i^T \Omega A_j = 0$ for $i \neq j$
    (b) $A_i^T \Omega \alpha \neq 0$ for *at most one* $i$

    Remark:

    - if (a) holds and from Proposition 3, we can have a joint distribution of $Y$ is $SN_k(\Omega_Y,\alpha_Y)$ where,

    $$
    mega_Y = diag(A_1^T \Omega A_1,\dots,A_h^T \Omega A_h),$$ 

    $$\alpha_Y = (A^T \Omega A)^{-1} A^T \Omega \alpha 
        = \left(
        \begin{aligned}
             (A_1^T \Omega A_1)^{-1}&A_1^T \Omega \alpha\\
            &\vdots\\
             (A_h^T \Omega A_h)^{-1}&A_h^T \Omega \alpha
        \end{aligned}
        \right)

    $$
- Quadratic forms:

  - Square of 1D SN random variate $\sim \chi_1^2$. This property carries on in the multivariate case since $Z^T \Omega^{-1} Z \sim \chi_k^2$.
  - Proposition 7:

    If $Z \sim SN_k(\Omega, \alpha)$, and $B$ is a symmetric positive semi-definite $k \times k$ matrix of rank $p$ such that $B \Omega B=B$, then $Z^T B Z \sim \chi_p^2$
  - Corollary 8
  - Proposition 9: Mutually independent feature.
  - Proposition 10 (Fisher-Cochran):

    If $Z \sim SN_k(I_k, \alpha)$ and $B_1, \dots, B_h$ are symmetric $k \times k$ matrices of rank $p_1,\dots, p_h$, respectively, such that $\sum B_i = I_k$ and $B_i \alpha \neq 0$ for *at most one* choices of $i$, then the quadratic forms $Z^T B_i Z$ are independent $\chi_{p_i}^2$ iff $\sum p_i = k$

## Culmulants and indices

- Index of skewness takes the form. **(Don't know where this $k$ comes from)**

  $$
  amma_{1,k}=(\frac{4 - \pi}{2})^2(\mu_z^T \Sigma^{-1} \mu_z)^3

  $$

  where $\Sigma = \Omega - \mu_z \mu_z^T$
- One can rewrite

  $$
  u_z^T \Sigma^{-1} \mu_z = \frac{\mu_z^T \Omega^{-1} \mu_z}{1 - \mu_z^T \Omega^{-1} \mu_z}

  $$

  Which allows easier examination of the range of $\mu_z^T \Sigma^{-1} \mu_z$, by considering the range of $\delta^T \Omega^{-1} \delta$, based on the relation btw $\mu_z$ and $\delta$ showed above.

  $$
  elta^T \Omega^{-1} \delta = \frac{\alpha^T \Omega \alpha}{1 + \alpha^T \Omega \alpha} = \frac{a}{1+a}$$  

  where $a$ is the square of  $\alpha_m^*$, defined above in **propositon 4**. Since $a$ spans $[0, \infty)$, then

  $$\mu_z^T \Sigma^{-1} \mu_z = \frac{2a}{\pi + (\pi - 2)a} \in [0,2/(\pi-2))

  $$

## Location and Scale Parameters

- We write

  $$
  Y = \xi + \omega Z
  $$

  where

  $$
  \xi = (\xi_1,\dots,\xi_k)^T, \quad \omega = diag(\omega_1,\dots,\omega_k)

  $$

  are location and scale parameters respectively.

  Density function of $Y$ is

  $$
  2 \phi_k(y-\xi;\Omega)\Phi(\alpha^T\omega^{-1}(y-\xi))
  $$ 

  where 
  $$ 
  \Omega = \omega \Omega_z \omega
  $$

  is a covariance matrix **(This part not finished)**

## Statistical Issues in the scalar case (Mainly Maximal Likelihood Estimation)

- Direct parameters
  - Sigularity problem of the information matrix at $\alpha=0$
