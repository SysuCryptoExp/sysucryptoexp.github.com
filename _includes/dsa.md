## 1. 实验参数生成

使用下面的工具生成一个2048bit长的素数`p`, 能被`p-1`整除的256bit长的素数`q`以及`alpha`。

{% include dsa/generate_params.html %}

## 2. 生成用户密钥

从 `{0, ..., q-1}` 选择随机整数 `a`
计算 `beta = (alpha ^ a) mod p`

其中 `a` 是私钥, `beta` 是公钥。

该过程不难，下面只给出模幂的计算示例(直接搬运DLP示例中的模幂计算)。

{% include dsa/calc_pow_mod.html %}


## 3. 签名

消息`msg`签名流程如下：

- 从 `{1, ..., q-1}` 选择随机整数 `k` **(每次签名随机数k需要重新选择! 不要固定! 下面的示例只是为了验证输出而允许输入k! )**

- 计算 `gamma = (alpha ^ k mod p) mod q`，当出现 `gamma = 0` 状况时重新选择随机数 `k`

- 计算 `delta = (SHA256(msg) + a * gamma) * k ^ (-1) mod q`，当出现 `delta = 0` 状况时重新选择随机数 `k`

签名为 `(gamma, delta)` 组合。


{% include dsa/sign.html %}

## 4. 验证签名

透过以下步骤可以验证 `(gamma, delta)` 是消息 `msg` 的有效签名：

- 验证 `0 < gamma < q` 且 `0 < delta < q`

- 计算 `w = delta ^ (-1) mod q`

- 计算 `u1 = SHA256(msg) * w mod q`

- 计算 `u2 = gamma * w mod q`

- 计算 `v = ((alpha ^ u1) * (beta ^ u2) mod p) mod q`

只有在 `v = gamma` 时代表签名是有效的。

{% include dsa/verify.html %}


## 5. 为什么每次签名的k都要重新随机选择? 

如果我们每次签名都固定使用一个$$k$$, 攻击者可以轻而易举地破解出私钥 $$a$$ 。攻击的方案是这样的：

假定攻击者已经知道了公共参数 $$q$$ , 以及同一个用户使用固定的 $$k$$ 所签署的两个消息 $$m_1, m_2$$ 及其签名 $$(\gamma_1, \delta_1), (\gamma_2, \delta_2)$$ 。不难得知$$\gamma_1 = \gamma_2$$, 此时攻击者可以通过如下方式得到 $$k$$ 以及用户的私钥 $$a$$：

$$\delta_1 - \delta_2 = (H(m_1) - H(m_2))k^{-1} \mod q$$

等号两边乘以 $$k$$ , 得

$$k(\delta_1 - \delta_2) = (H(m_1) - H(m_2)) \mod q$$

最后可以求出 $$k$$：

$$k = (H(m_1) - H(m_2)) \cdot (\delta_1 - \delta_2) ^ {-1} \mod q$$

有了 $$k$$ , 攻击者可以直接算出私钥 $$a$$：

$$a=(\delta_1 k - H(m_1)) \cdot \gamma_1 ^ {-1} \mod q$$

所以, 随机数 $$k$$ 的产生对于安全性来说是非常重要的, 在实验的过程中, 请务必保证 $$k$$ 一定要在 $$\{1, \cdots, q - 1\}$$的范围内选择, 过小的选择范围依然有很大概率发生碰撞, 导致私钥泄漏。

下面给出一个简单的示例程序来演示攻击, 只要输入公共参数以及两个使用相同的 $$k$$ 产生的签名, 服务器可以很快地破解出私钥。

{% include dsa/hack.html %}