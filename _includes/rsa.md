### 1. 根据学号最后一位选择p, q

{% include rsa/pq_select.html %}

### 2. 根据学号输出下一个素数作为RSA私钥e

{% include rsa/next_prime.html %}

### 3. 根据RSA私钥求出公钥

求解e关于模n的逆元d即可。

{% include rsa/cal_inverse.html %}

### 4. 使用PPT给出的简单OAEP填充法对姓名全拼进行填充

编码过程：先对姓名的全拼编码（ASCII或UTF-8, 因为都是英文字符，所以编码后的结果都是一样的）后的字节序列**前面补0**直至大小为1024 bit(128字节)。然后选取一个1024bit大小的随机数r和填充后的姓名全拼进行两次的掩码（G和H是掩码生成函数MGF）。过程如下图所示。

![简单OAEP填充法编码](/img/simple_oeap_encode.png){:height="50%" width="50%"}

解码过程是编码过程的逆过程。我们对2048bit的待解码数据**等分成两个1024bit长的字节序列X和Y**，然后进行下图的掩码操作。
最终我们需要的解码结果为下图的黄色方框。

![简单OAEP填充法解码](/img/simple_oeap_decode.png){:height="50%" width="50%"}

G和H的实现我们使用下图的方式进行，这里直接摘抄了PPT的内容：

![简单MGF实现](/img/simple_mgf.png){:height="50%" width="50%"}

其中，`H'`为哈希函数SHA-256，因此，G和H的输出就是四次SHA-256迭代、拼接，然后再将前两个字节置0并输出。为了便于展示，上图的`H'(m||0)`中的0**其实用到了四个字节**，即实际上是`H'(m||0x00000000)`。同理，`H(m||1)`其实是`H(m||0x00000001)`，`H(m||2)`其实是`H(m||0x00000002)`，`H(m||3)`其实是`H(m||0x00000003)`。

下面给出了示例程序，可以看到，无论输入和随机数`r`怎么变化，输出的结果中**前两个字节始终为0**。

{% include rsa/simple_oaep.html %}

### 5. 基于简单OAEP填充的RSA加密/解密

对姓名全拼进行了OAEP填充后，我们就可以开始进行RSA的加密和解密操作了。下面是示例程序，注意输入的待加密数据不用自己事先进行OAEP填充。

{% include rsa/simple_rsa.html %}

### 补充1：RFC8017的EME-OAEP填充法

有兴趣的同学可以进一步实现RFC8017给出的EME-OAEP填充方法，具体实现文档为：[https://datatracker.ietf.org/doc/html/rfc8017#section-7.1.1](https://datatracker.ietf.org/doc/html/rfc8017#section-7.1.1)。

文档给出了相当形象的OAEP示意图。这里简单解释以下，EME-OAEP有两个输入：消息`M`和标签`label`（可为空）。我们将EME-OAEP输出的长度表示为`k`，OAEP所选用的哈希函数（用于对标签`label`计算散列值）输出的长度为`hLen`，消息`M`的长度为`mLen`。因此，算法的流程为：

```
EME-OAEP encoding:

    a.  If the label L is not provided, let L be the empty string. Let lHash = Hash(L), an octet string of length hLen (see the note below).

    b.  Generate a padding string PS consisting of k - mLen - 2hLen - 2 zero octets.  The length of PS may be zero.

    c.  Concatenate lHash, PS, a single octet with hexadecimal value 0x01, and the message M to form a data block DB of length k - hLen - 1 octets as

                    DB = lHash || PS || 0x01 || M.

    d.  Generate a random octet string seed of length hLen.

    e.  Let dbMask = MGF(seed, k - hLen - 1).

    f.  Let maskedDB = DB \xor dbMask.

    g.  Let seedMask = MGF(maskedDB, hLen).

    h.  Let maskedSeed = seed \xor seedMask.

    i.  Concatenate a single octet with hexadecimal value 0x00, maskedSeed, and maskedDB to form an encoded message EM of length k octets as
                    EM = 0x00 || maskedSeed || maskedDB.
```

这里给出了RFC8017的示意图。其中`lHash`为标签`label`的Hash值，`PS`为全0的填充字节，`seed`为一个随机选取的种子，`MGF`则为掩码生成函数。

```
      _________________________________________________________________

                                +----------+------+--+-------+
                           DB = |  lHash   |  PS  |01|   M   |
                                +----------+------+--+-------+
                                               |
                     +----------+              |
                     |   seed   |              |
                     +----------+              |
                           |                   |
                           |-------> MGF ---> xor
                           |                   |
                  +--+     V                   |
                  |00|    xor <----- MGF <-----|
                  +--+     |                   |
                    |      |                   |
                    V      V                   V
                  +--+----------+----------------------------+
            EM =  |00|maskedSeed|          maskedDB          |
                  +--+----------+----------------------------+
      _________________________________________________________________
```

有关MGF的实现我们在PPT上给出了，这里不再赘述。因此，EME-OAEP需要用到两个Hash函数：一个用于对`label`计算散列值，一个用于MGF的底层散列函数。在本次实验中为了方便，上述两个Hash函数都采用SHA-256。

{% include rsa/eme_oaep.html %}

### 补充2：RFC8017的RSAES-OAEP加密/解密

根据上述的EME-OAEP，我们就可以实现了实用的RSA加密方案——RFC8017的RSAES-OAEP，下面给出了实例程序。

{% include rsa/rsaes_oaep.html %}

