## 1. 实验参数生成

使用下面的工具生成一个2048bit长的素数`p`, 能被`p-1`整除的256bit长的素数`q`以及`alpha`。

{% include dsa/generate_params.html %}

## 2. 生成用户密钥

从 `{0, ..., q-1}` 选择随机整数 `a`
计算 `beta = (alpha ^ a) mod p`

其中 `a` 是私钥, `alpha` 是公钥。

该过程不难，下面只给出模幂的计算示例(直接搬运DLP示例中的模幂计算)。

{% include dsa/calc_pow_mod.html %}


## 3. 签名

消息`msg`签名流程如下：

- 从 `{1, ..., q-1}` 选择随机整数 `k`

- 计算 `gamma = (a ^ k mod p) mod q`，当出现 `gamma = 0` 状况时重新选择随机数 `k`

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
