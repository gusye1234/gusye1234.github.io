---
layout: post
title: continuous-time kalman filter
subtitle: chinese
tags: [ note ]
comments: true
---



## continuous-time  filter

考虑:

* continuous process: $\dot{x_t} = Ax_t + B(u_t + \epsilon_t)$
* Measurement:          $y_t = Hx_t + \nu_t$
* 其中, 假设  $\epsilon_t$ 和 $\nu_t$ 是独立的 white noise, 也就是满足
  * $E[\epsilon_t\epsilon_s]= Qδ(t − s) $
  * $E[\nu_t\nu_s] = Rδ(t − s)$
  * $E[\epsilon_t] = 0$
  * $E[\nu_t] = 0$

notion:

* $x_{n+1} \equiv x_{t+\Delta t} \quad x_n \equiv x_t$
* $x_{n+1} = x_n + \dot{x}_n*\Delta t$
* $\tilde{x}_{n+1} = \hat{x_n} + \dot{\tilde{x}}_n*\Delta t$, 称为先验估计, 即对于已经求到的$\hat{x}_n$, 我们直接给出$\tilde{x}_{n+1}$
* $\hat{x}_{n+1} = \hat{x}_n + \dot{\hat{x}}_n*\Delta t$, 称为后验估计, 结合实际的观测$y_{n+1}$, 修正估计
* $\dot{\tilde{x}}_n = A\hat{x}_n + Bu_t$, 输入过程中的噪声无法纳入估计
* $\dot{\hat{x}}_n = \dot{\tilde{x}}_n + K_t(y_{n+1} - \tilde{y}_{n+1})$
* $y_{n+1} = Hx_{n+1} + \nu_{n+1}$
* $\tilde{y}_{n+1} = H\tilde{x}_{n+1}$
* $K_T \equiv K^{\Delta}_t \equiv \Delta t*K_t$
* $\hat{e}_{n+1} \equiv x_{n+1} - \hat{x}_{n+1}$, 后验误差
* $\tilde{e}_{n+1} \equiv x_{n+1} - \tilde{x}_{n+1}$ , 先验误差

*接下来是错误可能出现的地方*:

* $P_{n+1|n} \equiv \frac{E[\tilde{e}_{n+1}\tilde{e}_{n+1}^T]}{\Delta t}$, 真实值与先验估计之间的协方差
* $P_{n+1|n+1} \equiv P_{n+1} \equiv \frac{E[\hat{e}_{n+1}\hat{e}_{n+1}^T]}{\Delta t}$, 真实值与后验估计之间的协方差





接下来按照离散kalman filter 的推导思路进行:

$$
\begin{aligned}
\hat{x}_{n+1} &= \tilde{x}_{n+1} + \Delta t*K_t(y_{n+1} - \tilde{y}) \\
              &= \tilde{x}_{n+1} +  K_T(Hx_{n+1} + \nu_{n+1} -  H\tilde{x}_{n+1}) \\
              &= \tilde{x}_{n+1} + K_TH(x_{n+1} - \tilde{x}_{n+1}) + K\nu_{n+1}

             
\end{aligned}
$$

进而有
$$
\begin{aligned}
\hat{x}_{n+1} - x_{n+1} &= \tilde{x}_{n+1} - x_{n+1} + K_TH(x_{n+1} - \tilde{x}_{n+1}) + K_T\nu_{n+1} \\
-\hat{e}_{n+1} &= -\tilde{e}_{n+1} + K_TH\tilde{e}_{n+1} + K_T\nu_{n+1} \\
\hat{e}_{n+1} &= (1-K_TH)\tilde{e}_{n+1} - K_T\nu_t \\
\end{aligned}
$$

以上简单推导是离散 kalman filter 的内容, 可参考[blog](https://zhuanlan.zhihu.com/p/48876718).

上一步, 我们得到了先验误差与后验误差的关系, 接下来我们通过求解一个$K_T$, 使得在给定了先验误差的协方差矩阵$P_{n+1|n}$下, 最小化后验误差的协方差矩阵$P_{n+1|n+1}$.



接下来带入我们的定义, (注意, 下面推导的第一步等号右侧本该有四项, 我省略了两项, 因为那两项在下一步的取期望中为零)
$$
\begin{aligned}
\hat{e}_{n+1}\hat{e}_{n+1}^T &= (1-K_TH)\tilde{e}_{n+1}\tilde{e}_{n+1}^T(1-K_TH)^T - K_T\nu_{n+1}\nu_{n+1}^TK_T^T + ... \\
E[\hat{e}_{n+1}\hat{e}_{n+1}^T] &= E[(1-K_TH)\tilde{e}_{n+1}\tilde{e}_{n+1}^T(1-K_TH)^T -K_T\nu_{n+1}\nu_{n+1}^TK_T^T] \\
P_{n+1|n+1}*\Delta t &= (1-K_TH)P_{n+1|n}(1-K_TH)^T * \Delta t - K_T R K_T^T \\
P_{n+1|n+1} &= (1-K_TH)P_{n+1|n}(1-K_TH)^T -  K_T(\frac{R}{\Delta t})K_T^T \quad (1)
\end{aligned}
$$

通过极小化 $P_{n+1|n+1}$ 来实现最优化估计 (原文中的优化目标是最小二乘误差, 即$E(\parallel x_{n+1} - \hat{x}_{n+1} \parallel^2)$, 和极小化协方差矩阵的等效性没有证明, 我感觉是一样的, 原 kalman filter 是极小化协方差矩阵)

有令$\frac{\partial{P_{n+1|n+1}}}{\partial{K_T}} = 0$, 并忽略二阶小量, 可以得到
$$
\begin{aligned}
K_T &= P_{n+1|n}H^T(HP_{n+1|n}H^T + \frac{R}{\Delta t})^{-1} \\
    &= \Delta tP_{n+1|n}H^T(\Delta tHP_{n+1|n}H^T + R)^{-1} \\
    &\cong \Delta tP_{n+1|n}H^TR^{-1}
\end{aligned}
$$
所以, 回顾定义$K_T \equiv K^{\Delta}_t \equiv \Delta t*K_t$, 我们有

$K_t = P_{n+1|n}H^TR^{-1}$

带入$K_T$到(1)中, 可以得到

$ P_{n+1|n+1} = (1-K_TH)P_{n+1|n} \quad (2)$



接下来我们建立先验误差和上一时刻的后验误差的联系
$$
\begin{aligned}
\tilde{e}_{n+1} &= x_{n+1} - \tilde{x_{n+1}} \\
                &= x_n + \dot{x}*\Delta t - \hat{x_n} - \dot{\tilde{x}}_n*\Delta t \\
                &= \hat{e}_n + \Delta t*(\dot{x}_n - \dot{\tilde{x}}_n) \\
                &= \hat{e}_n + \Delta t*(Ax_n + B(u_t + \epsilon_t) - A\hat{x}_n + Bu_t) \\
                &= (1+\Delta tA)\hat{e}_n + \Delta tB\epsilon_t \\
\end{aligned}
$$

进而构建先验误差协方差矩阵$P_{n+1|n}$和上一时刻的后验误差协方差矩阵$P_{n|n}$的联系, 自己推嗷.
$$
P_{n+1|n} = (1+\Delta t A)P_{n|n}(1+\Delta t A)^T + \Delta tBQB^T \quad (3)
$$





又因为我们有$ P_{n+1|n+1} = (1-K_TH)P_{n+1|n} = (1-\Delta t K_tH)P_{n+1|n}$

也就是$ P_{n|n} = (1-\Delta t K_{t-\Delta t}H)P_{n|n-1}$.

带入(3)中, 可以得到:
$$
P_{n+1|n} = (1+\Delta t F)(1-\Delta t K_{t-\Delta t}H)P_{n|n-1}(1+\Delta t F)^T + \Delta tBQB^T \quad
$$
展开并忽略二阶小量得到(自己推嗷):
$$
P_{n+1|n} = P_{n|n-1} + \Delta tAP_{n|n-1} + \Delta tP_{n|n-1}A^T - \Delta t K_{t-\Delta t}HP_{n|n-1} + \Delta tBQB^T \quad (4)
$$
---



接下来讨论一些性质:

* $\Delta t \to 0, K_T \equiv \Delta t*K_t \to 0$
* 由(2)得到, $\Delta t \to 0, P_{n+1|n+1} \to P_{n+1|n}$
* 由(3)得到, $\Delta t \to 0, P_{n+1|n} \to P_{n|n}$
* $K_T \to 0, P_{n+1|n+1} \to P_{n|n}$

所以, 直观的可以感受到
$$
\begin{aligned}
\dot{P}_t &= lim_{\Delta t \to 0}\frac{P_{t+\Delta t} - P_t}{\Delta t} \\
        &= lim_{\Delta t \to 0} \frac{P_{n+1|n} - P_{n|n-1}}{\Delta t}
\end{aligned}
$$

所以, 从(4)可以得到 (自己推嗷):

$$
\begin{aligned}
\dot{P}_t &= AP_t + P_tA^T - K_{t}HP_t + BQB^T \\
          &= AP_t + P_tA^T - P_tH^TR^{-1}HP_t + BQB^T \\
\end{aligned}
$$

对于后验误差协方差稳定的情况, 即$\dot{P}_t = 0$, 可以得到所谓的 **Algebraic Riccati Equation** :
$$
AP_t + P_tA^T - P_tH^TR^{-1}HP_t + BQB^T = 0
$$
对于稳定的一个系统, $t \to \infty$, 应有$P$趋于稳定值, 所以上式也可以记为:
$$
AP_{\infty} + P_{\infty}A^T - P_{\infty}H^TR^{-1}HP_{\infty} + BQB^T = 0
$$
*(paper当中, 这个方程为)*
$$
AP_{\infty} + P_{\infty}A^T - P_{\infty}H^TR^{-1}HP_{\infty} + Q = 0
$$


此方程的解计算$K_{\infty} = P_{\infty}H^TR^{-1}$ 