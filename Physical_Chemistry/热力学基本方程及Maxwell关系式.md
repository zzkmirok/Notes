# 热力学基本方程及Maxwell关系式

### 五大状态函数的关系

$U, S, H, A, G$

$H = U+PV$

$A = U - TS$ Helmholtz 定容，定温

$G = H - TS$ Gibbs 定压，定温

封闭系统，只做体积功，可逆

$\implies dU = TdS - PdV$

无论实际过程是否可逆，上述的积分都存在。只是在上述条件下对应热力学第一定律

$dH = dU + PdV +VdP$

$\implies dH = TdS - \cancel{PdV} + \cancel{PdV} +VdP$

$\implies dH = TdS + VdP$

$dA = dU - TdS - SdT$

$\implies dA = -PdV + \cancel{TdS} - \cancel{TdS} - SdT$

$\implies dA = -PdV - SdT$

$dG = dH - TdS - SdT$

$\implies dG = VdP + \cancel{TdS} - \cancel{TdS} -SdT$

$\implies dG = VdP - SdT$

### 热力学基本方程及其推论

$dU = TdS - PdV$

$dH = TdS + VdP$

$dA = -PdV - SdT$

$dG = VdP - SdT$

$\implies$

$T = (\frac{\partial U}{\partial S})_V = (\frac{\partial H}{\partial S})_P$

$P = -(\frac{\partial U}{\partial V})_S = -(\frac{\partial A}{\partial V})_T$

$V = (\frac{\partial H}{\partial P})_S = (\frac{\partial G}{\partial P})_T$

$S = -(\frac{\partial A}{\partial T})_V = (\frac{\partial G}{\partial T})_P$

### Maxwell 关系

由全微分的存在性（混合二阶导数相等）

$(\frac{\partial T}{\partial V})_S = -(\frac{\partial P}{\partial S})_V$

$(\frac{\partial T}{\partial P})_S = (\frac{\partial V}{\partial S})_P$

$(\frac{\partial P}{\partial T})_V = (\frac{\partial S}{\partial V})_T$

$(\frac{\partial V}{\partial T})_P = -(\frac{\partial S}{\partial P})_T$