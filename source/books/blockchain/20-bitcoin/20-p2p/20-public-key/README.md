# 公钥和地址

### 公钥

比特币的公钥是根据私钥计算出来的。

私钥本质上是一个256位整数，记作`k`。根据比特币采用的ECDSA算法，可以推导出两个256位整数，记作`(x, y)`，这两个256位整数即为非压缩格式的公钥。

由于ECC曲线的特点，根据非压缩格式的公钥`(x, y)`的`x`实际上也可推算出`y`，但需要知道`y`的奇偶性，因此，可以根据`(x, y)`推算出`x'`，作为压缩格式的公钥。

压缩格式的公钥实际上只保存`x`这一个256位整数，但需要根据`y`的奇偶性在`x`前面添加`02`或`03`前缀，`y`为偶数时添加`02`，否则添加`03`，这样，得到一个1+32=33字节的压缩格式的公钥数据，记作`x'`。

注意压缩格式的公钥和非压缩格式的公钥是可以互相转换的，但均不可反向推导出私钥。

非压缩格式的公钥目前已很少使用，原因是非压缩格式的公钥签名脚本数据会更长。

我们来看看如何根据私钥推算出公钥：

```x-javascript
const bitcoin = require('bitcoinjs-lib');

let
    wif = 'KwdMAjGmerYanjeui5SHS7JkmpZvVipYvB2LJGU1ZxJwYvP98617',
    ecPair = bitcoin.ECPair.fromWIF(wif); // 导入私钥
// 计算公钥:
let pubKey = ecPair.getPublicKeyBuffer(); // 返回Buffer对象
console.log(pubKey.toString('hex')); // 02或03开头的压缩公钥
```

构造出`ECPair`对象后，即可通过`getPublicKeyBuffer()`以`Buffer`对象返回公钥数据。

### 地址

要特别注意，比特币的地址并不是公钥，而是公钥的哈希，即从公钥能推导出地址，但从地址不能反推公钥，因为哈希函数是单向函数。

以压缩格式的公钥为例，从公钥计算地址的方法是，首先对1+32=33字节的公钥数据进行Hash160（即先计算SHA256，再计算RipeMD160），得到20字节的哈希。然后，添加`0x00`前缀，得到1+20=21字节数据，再计算4字节校验码，拼在一起，总计得到1+20+4=25字节数据：

```ascii
0x00      hash160         check
┌─┬──────────────────────┬─────┐
│1│          20          │  4  │
└─┴──────────────────────┴─────┘
```

对上述25字节数据进行Base58编码，得到总是以`1`开头的字符串，该字符串即为比特币地址，整个过程如下：

![Compressed Address](compressed-address.jpg)

使用JavaScript实现公钥到地址的编码如下：

```x-javascript
const bitcoin = require('bitcoinjs-lib');

let
    publicKey = '02d0de0aaeaefad02b8bdc8a01a1b8b11c696bd3d66a2c5f10780d95b7df42645c',
    ecPair = bitcoin.ECPair.fromPublicKeyBuffer(Buffer.from(publicKey, 'hex')); // 导入公钥
// 计算地址:
let address = ecPair.getAddress();
console.log(address); // 1开头的地址
```

计算地址的时候，不必知道私钥，可以直接从公钥计算地址，即通过`ECPair.fromPublicKeyBuffer`构造一个不带私钥的`ECPair`即可计算出地址。

要注意，对非压缩格式的公钥和压缩格式的公钥进行哈希编码得到的地址，都是以`1`开头的，因此，从地址本身并无法区分出使用的是压缩格式还是非压缩格式的公钥。

以`1`开头的字符串地址即为比特币收款地址，可以安全地公开给任何人。

仅提供地址并不能让其他人得知公钥。通常来说，公开公钥并没有安全风险。实际上，如果某个地址上有对应的资金，要花费该资金，就需要提供公钥。如果某个地址的资金被花费过至少一次，该地址的公钥实际上就公开了。

私钥、公钥以及地址的推导关系如下：

```ascii
┌───────────┐      ┌───────────┐
│Private Key│─────▶│Public Key │
└───────────┘      └───────────┘
      ▲                  │
      │                  │
      ▼                  ▼
┌───────────┐      ┌───────────┐
│    WIF    │      │  Address  │
└───────────┘      └───────────┘
```

### 小结

比特币的公钥是根据私钥由ECDSA算法推算出来的，公钥有压缩和非压缩两种表示方法，可互相转换。

比特币的地址是公钥哈希的编码，并不是公钥本身，通过公钥可推导出地址。

通过地址不可推导出公钥，通过公钥不可推导出私钥。