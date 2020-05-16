# Deep unfolding of the weighted MMSE beamforming algorithm

This GitHub repository complements our paper [[1]](#ourpaper). The user can find the code used to reproduce the plots in the paper.
In our paper we propose the novel application of **deep unfolding** to the weighted minimum mean squre error (WMMSE) in [[2]](#WMMSE_Shi).
The WMMSE algorithm is an iterative algorithm that converges to a local solution of the weigthed sum rate maximization problem subject to a power constraint, which is known to be NP-hard. As noted in [[3]](#WMMSE_E2E), the formulation of the WMMSE algorithm, as described in [[2]](#WMMSE_Shi), is not suitable to be unfolded due to the matrix inversion, the eigenvale decomposition, and bisection search performed at each itearation of the algorithm. Therefore, in our paper [[1]](#ourpaper), we propose an alternative formulation that avoids these operations. Specifically, we replace the method of Lagrangian multipliers with the **projected gradient descent (PGD) approach**. 

In the jupyter notebook "" the implementation in Python 3.6.8 of the WMMSE algorithm in [[2]](#WMMSE_Shi) and the implementation in Python 3.6.8 and Tensorflow 1.13.1 of the unfolded WMMSE algorithm can be found.

## Problem formulation
We consider a multiple-input single-output (MISO) interference downlink channel. The base station has M transmit antennas and sends independent data symbols to N single-antenna users. ![x_i \sim \mathcal{CN}(0,\,1)](https://render.githubusercontent.com/render/math?math=x_i%20%5Csim%20%5Cmathcal%7BCN%7D(0%2C%5C%2C1)) is the transmitted data symbol to user *i* and ![\boldsymbol{h}_i \sim \mathcal{CN}(\boldsymbol{0},\,\boldsymbol{I}_M)](https://render.githubusercontent.com/render/math?math=%5Cboldsymbol%7Bh%7D_i%20%5Csim%20%5Cmathcal%7BCN%7D(%5Cboldsymbol%7B0%7D%2C%5C%2C%5Cboldsymbol%7BI%7D_M)) is the channel between user *i* and the base station.

With linear beamforming, the signal at the receiver of user *i* is 

![y_{i} = {\boldsymbol{h}^{H}_i}\boldsymbol{v}_{i}x_{i} + \sum_{j=1,j \neq i}^{N}{\boldsymbol{h}^{H}_i}{\boldsymbol{v}_{j}x_{j}} + n_{i}](https://render.githubusercontent.com/render/math?math=y_%7Bi%7D%20%3D%20%7B%5Cboldsymbol%7Bh%7D%5E%7BH%7D_i%7D%5Cboldsymbol%7Bv%7D_%7Bi%7Dx_%7Bi%7D%20%2B%20%5Csum_%7Bj%3D1%2Cj%20%5Cneq%20i%7D%5E%7BN%7D%7B%5Cboldsymbol%7Bh%7D%5E%7BH%7D_i%7D%7B%5Cboldsymbol%7Bv%7D_%7Bj%7Dx_%7Bj%7D%7D%20%2B%20n_%7Bi%7D),

where ![\boldsymbol{v}_i \in \mathbb{C}^{M}](https://render.githubusercontent.com/render/math?math=%5Cboldsymbol%7Bv%7D_i%20%5Cin%20%5Cmathbb%7BC%7D%5E%7BM%7D) is the beamformer for user *i* and where ![n_i \sim \mathcal{CN}(0,\,\sigma^{2})](https://render.githubusercontent.com/render/math?math=n_i%20%5Csim%20%5Cmathcal%7BCN%7D(0%2C%5C%2C%5Csigma%5E%7B2%7D)) is independent additive white Gaussian noise with power ![\sigma^2](https://render.githubusercontent.com/render/math?math=%5Csigma%5E2). The estimated data symbol at the receiver is ![\hat{x}_i = u_{i}y_{i}](https://render.githubusercontent.com/render/math?math=%5Chat%7Bx%7D_i%20%3D%20u_%7Bi%7Dy_%7Bi%7D), where ![u_i \in \mathbb{C}](https://render.githubusercontent.com/render/math?math=u_i%20%5Cin%20%5Cmathbb%7BC%7D) is the receiver gain at user *i*. 
We define ![\boldsymbol{H} \triangleq \[\boldsymbol{h}_1,\boldsymbol{h}_2,\ldots,\boldsymbol{h}_N\]^T](https://render.githubusercontent.com/render/math?math=%5Cboldsymbol%7BH%7D%20%5Ctriangleq%20%5B%5Cboldsymbol%7Bh%7D_1%2C%5Cboldsymbol%7Bh%7D_2%2C%5Cldots%2C%5Cboldsymbol%7Bh%7D_N%5D%5ET), ![\boldsymbol{V}\triangleq \[\boldsymbol{v}_1,\boldsymbol{v}_2,\ldots,\boldsymbol{v}_N\]^T](https://render.githubusercontent.com/render/math?math=%5Cboldsymbol%7BV%7D%5Ctriangleq%20%5B%5Cboldsymbol%7Bv%7D_1%2C%5Cboldsymbol%7Bv%7D_2%2C%5Cldots%2C%5Cboldsymbol%7Bv%7D_N%5D%5ET), and ![\boldsymbol{u}  \triangleq \[u_1, u_2, \ldots,u_N\]^T](https://render.githubusercontent.com/render/math?math=%5Cboldsymbol%7Bu%7D%20%20%5Ctriangleq%20%5Bu_1%2C%20u_2%2C%20%5Cldots%2Cu_N%5D%5ET).

We seek to maximize the weighted sum rate (WSR) subject to a total transmit power constraint, i.e.,

![\begin{aligned} \max_{\boldsymbol{V}} \quad & \sum_{i = 1}^{N}{\alpha_i\log_{2}{( 1 + \mathrm{SINR}_i)}} \\ \textrm{s.t.} \quad  & \mathtt{Tr}(\boldsymbol{V}\boldsymbol{V}^H)\leq P, \end{aligned}](https://render.githubusercontent.com/render/math?math=%5Cbegin%7Baligned%7D%20%5Cmax_%7B%5Cboldsymbol%7BV%7D%7D%20%5Cquad%20%26%20%5Csum_%7Bi%20%3D%201%7D%5E%7BN%7D%7B%5Calpha_i%5Clog_%7B2%7D%7B(%201%20%2B%20%5Cmathrm%7BSINR%7D_i)%7D%7D%20%5C%5C%20%5Ctextrm%7Bs.t.%7D%20%5Cquad%20%20%26%20%5Cmathtt%7BTr%7D(%5Cboldsymbol%7BV%7D%5Cboldsymbol%7BV%7D%5EH)%5Cleq%20P%2C%20%5Cend%7Baligned%7D)

where ![\alpha_{i}](https://render.githubusercontent.com/render/math?math=%5Calpha_%7Bi%7D) indicates the user priority (assumed to be known) and where P is the maximum transmit power at the BS. It can be shown this problem is equivalent to the following weighted sum mean square error minimization problem subject to a total transmit power constraint, i.e.,

![\begin{alignat}{2} \min_{\boldsymbol{u}, \boldsymbol{w}, \boldsymbol{V}} \quad & \sum_{i = 1}^{N}{\alpha_i \left(w_{i}e_{i}-\log_2{w_i} \right)}\\ \textrm{s.t.} \quad  & \textrm{Tr}(\boldsymbol{V}\boldsymbol{V}^H)\leq P, \end{alignat}](https://render.githubusercontent.com/render/math?math=%5Cbegin%7Balignat%7D%7B2%7D%20%5Cmin_%7B%5Cboldsymbol%7Bu%7D%2C%20%5Cboldsymbol%7Bw%7D%2C%20%5Cboldsymbol%7BV%7D%7D%20%5Cquad%20%26%20%5Csum_%7Bi%20%3D%201%7D%5E%7BN%7D%7B%5Calpha_i%20%5Cleft(w_%7Bi%7De_%7Bi%7D-%5Clog_2%7Bw_i%7D%20%5Cright)%7D%5C%5C%20%5Ctextrm%7Bs.t.%7D%20%5Cquad%20%20%26%20%5Ctextrm%7BTr%7D(%5Cboldsymbol%7BV%7D%5Cboldsymbol%7BV%7D%5EH)%5Cleq%20P%2C%20%5Cend%7Balignat%7D)

where ![e_i =\mathbb{E}_{x_i,n_i}\{{|\hat{x}_i-x_i|^2}\}](https://render.githubusercontent.com/render/math?math=e_i%20%3D%5Cmathbb%7BE%7D_%7Bx_i%2Cn_i%7D%5C%7B%7B%7C%5Chat%7Bx%7D_i-x_i%7C%5E2%7D%5C%7D) assuming that ![x_{i}](https://render.githubusercontent.com/render/math?math=x_%7Bi%7D) and ![n_{i}](https://render.githubusercontent.com/render/math?math=n_%7Bi%7D) are independent, where ![w_i](https://render.githubusercontent.com/render/math?math=w_i) is the user weight, and where ![\boldsymbol{w} \triangleq \[w_1,w_2,\ldots,w_N\]^T](https://render.githubusercontent.com/render/math?math=%5Cboldsymbol%7Bw%7D%20%5Ctriangleq%20%5Bw_1%2Cw_2%2C%5Cldots%2Cw_N%5D%5ET).

## Proposed unfolded WMMSE algorithm
Algorithm 1 reports the pseudocode of the unfolded WMMSE and Fig 1 depicts the overall neural network architecture.

![](pseudocode.png)

where ![l = 1,...,L](https://render.githubusercontent.com/render/math?math=l%20%3D%201%2C...%2CL) indexes the iterations and ![k = 1,...,K](https://render.githubusercontent.com/render/math?math=k%20%3D%201%2C...%2CK) indexes the PGD steps, where , and ![\Pi_{\mathcal{C}}\{\boldsymbol{V}\} =  \begin{cases}       \boldsymbol{V}, & \text{if}\ \mathtt{Tr}(\boldsymbol{V}\boldsymbol{V}^H)\leq P \\       \frac{\boldsymbol{V}}{\left\lVert \boldsymbol{V} \right\rVert}\sqrt{P}, & \text{otherwise.}     \end{cases} ](https://render.githubusercontent.com/render/math?math=%5CPi_%7B%5Cmathcal%7BC%7D%7D%5C%7B%5Cboldsymbol%7BV%7D%5C%7D%20%3D%20%20%5Cbegin%7Bcases%7D%20%20%20%20%20%20%20%5Cboldsymbol%7BV%7D%2C%20%26%20%5Ctext%7Bif%7D%5C%20%5Cmathtt%7BTr%7D(%5Cboldsymbol%7BV%7D%5Cboldsymbol%7BV%7D%5EH)%5Cleq%20P%20%5C%5C%20%20%20%20%20%20%20%5Cfrac%7B%5Cboldsymbol%7BV%7D%7D%7B%5Cleft%5ClVert%20%5Cboldsymbol%7BV%7D%20%5Cright%5CrVert%7D%5Csqrt%7BP%7D%2C%20%26%20%5Ctext%7Botherwise.%7D%20%20%20%20%20%5Cend%7Bcases%7D%20)

![](unfolded_network.png)
*Fig. 1. The neural network architecture obtained by unfolding $L$ iterations of the WMMSE and $K$ projected gradient descent (PGD) steps per iteration. The subscript $(\cdot)_{l}$ corresponds to the $l^{th}$ layer/iteration and the superscript $(\cdot)^{(k)}$ corresponds to the $k^{th}$ PGD step. The trainable parameters, highlighted in red, are the step sizes of the PGD approach. Each layer of the neural network is given by the update equation of $\boldsymbol{w}$~(\ref{eq:w_opt}), indicated by $\boldsymbol{\Psi}$, by the update equation of $\boldsymbol{u}$~(\ref{eq:u_opt}), indicated by $\boldsymbol{\Omega}$, and by unfolding $K$ PGD steps, as depicted in the gray box. In particular, $\nabla$ and $\Pi_\mathcal{C}$ indicate the gradient and the projection operations in~(\ref{eq:PGD}), respectively. *


## Computation Environment
In order to run the code in this repository the following software packages are needed:
* `Python 3` ( for reference we use Python 3.6.8 ), with the following packages:`numpy`, `tensorflow` (version 1.x - for reference we use version 1.13.1) , `matplotlib`,`copy`,`time`.
* `Jupyter` ( for reference we use version 6.0.3 )


## Reference

<a id='ourpaper'></a> [1] L. Pellaco, M. Bengtsson, J. Jaldén, "Deep unfolding of the weighted MMSE algorithm," submitted to IEEE Transactions of Signal Processing.

<a id='WMMSE_Shi'></a> [2] Q. Shi, M. Razaviyayn, Z. Luo and C. He, "An Iteratively Weighted MMSE Approach to Distributed Sum-Utility Maximization for a MIMO Interfering Broadcast Channel," in IEEE Transactions on Signal Processing, vol. 59, no. 9, pp. 4331-4340, Sept. 2011, doi: 10.1109/TSP.2011.2147784.

<a id='WMMSE_E2E'></a> [3] H. Sun, X. Chen, Q. Shi, M. Hong, X. Fu and N. D. Sidiropoulos, "Learning to Optimize: Training Deep Neural Networks for Interference Management," in IEEE Transactions on Signal Processing, vol. 66, no. 20, pp. 5438-5453, 15 Oct.15, 2018, doi: 10.1109/TSP.2018.2866382.


