
# 纠删码技术

## 目录

<!-- TOC depthfrom:2 -->
- [纠删码技术](#纠删码技术)
	- [目录](#目录)
	- [纠删码简介](#纠删码简介)
	- [纠删码相关代码](#纠删码相关代码)
	- [原型验证](#原型验证)
	- [参考资料](#参考资料)

<!-- /TOC -->

## 纠删码简介


公式： $\sqrt{x^2+y^2}$
IEEE 802.3 119.2.4.6中定义了编码的算法：

区块链数据分块采用多项式表示：原始数据块k,校验块m(2t=m=4)

$$(m_{k-1} x^{k-1} + \cdots + m_1x + m_0) \times x^{2t}$$

并根据校验块的方式进行后补零
$$m_{k-1} x^{n-1} + \cdots + m_1x^{2t+1} + m_0x^{2t} + 0x^{2t-1} + \cdots + 0x +0$$

<!-- $$ \sum_{i=1}^n \frac{1}{i^2}$$ -->
<!-- $$ g(x) = \prod_{j=0}^{2t-1} \frac{1}{i^2}$$ -->

生成多项式g(x)的生成如下：

$$ g(x) = \frac{1}{\vec{a}}\prod_{j=0}^{2t-1} (x - a^j) = \frac{1}{\vec{a}} (g_{2t}x^{2t} + \cdot + g_1x + g_0)$$

数据多项式除以生成多项式g(x)  取余下的多项式为校验多项式 p(x)。
$$m_{k-1} x^{n-1} + \cdots + m_1x^{2t+1} + m_0x^{2t} + p_{2t-1}x^{2t-1} + \cdots + p_1x +p_0$$


<!-- 如果传输过程没有任何错误，那么接收到的编码数据块多项式去除生成多项式  是可以整除、没有余数的，如下图所示： -->
$$m_{k-1} x^{n-1} + \cdots + m_1x^{2t+1} + m_0x^{2t} + p_{2t-1}x^{2t-1} + \cdots + p_1x +p_0 \div \prod_{j=0}^{2t-1} (x - a^j) = \vec{a}$$






## 纠删码相关代码

```bash
(base) ➜  codeshow git:(main) ✗ ls  http://github.com/minio/minio/cmd/erasure* 
http://github.com/minio/minio/cmd/erasure-bucket.go
http://github.com/minio/minio/cmd/erasure-coding.go
http://github.com/minio/minio/cmd/erasure-common.go
http://github.com/minio/minio/cmd/erasure-decode.go
http://github.com/minio/minio/cmd/erasure-decode_test.go
http://github.com/minio/minio/cmd/erasure-encode.go
http://github.com/minio/minio/cmd/erasure-encode_test.go
http://github.com/minio/minio/cmd/erasure-errors.go
http://github.com/minio/minio/cmd/erasure-heal_test.go
http://github.com/minio/minio/cmd/erasure-healing-common.go
http://github.com/minio/minio/cmd/erasure-healing-common_test.go
http://github.com/minio/minio/cmd/erasure-healing.go
http://github.com/minio/minio/cmd/erasure-healing_test.go
http://github.com/minio/minio/cmd/erasure-metadata-utils.go
http://github.com/minio/minio/cmd/erasure-metadata-utils_test.go
http://github.com/minio/minio/cmd/erasure-metadata.go
http://github.com/minio/minio/cmd/erasure-metadata_test.go
http://github.com/minio/minio/cmd/erasure-multipart.go
http://github.com/minio/minio/cmd/erasure-object.go
http://github.com/minio/minio/cmd/erasure-object_test.go
http://github.com/minio/minio/cmd/erasure-server-pool-decom.go
http://github.com/minio/minio/cmd/erasure-server-pool-decom_gen.go
http://github.com/minio/minio/cmd/erasure-server-pool-decom_gen_test.go
http://github.com/minio/minio/cmd/erasure-server-pool-decom_test.go
http://github.com/minio/minio/cmd/erasure-server-pool.go
http://github.com/minio/minio/cmd/erasure-sets.go
http://github.com/minio/minio/cmd/erasure-sets_test.go
http://github.com/minio/minio/cmd/erasure-utils.go
http://github.com/minio/minio/cmd/erasure.go # 定义ER对象
http://github.com/minio/minio/cmd/erasure_test.go # 测试代码
```

github.com/klauspost/reedsolomon
[纠删码入口](../../../../code/minio/cmd/erasure.go)
定义ER对象
```go
// erasureObjects - Implements ER object layer.
type erasureObjects struct {
	GatewayUnsupported

	setDriveCount      int //节点数量
	defaultParityCount int //默认分区

	setIndex  int
	poolIndex int

	// getDisks returns list of storageAPIs.
	getDisks func() []StorageAPI

	// getLockers returns list of remote and local lockers.
	getLockers func() ([]dsync.NetLocker, string)

	// getEndpoints returns list of endpoint strings belonging this set.
	// some may be local and some remote.
	getEndpoints func() []Endpoint

	// Locker mutex map.
	nsMutex *nsLockMap

	// Byte pools used for temporary i/o buffers.
	bp *bpool.BytePoolCap

	// Byte pools used for temporary i/o buffers,
	// legacy objects.
	bpOld *bpool.BytePoolCap

	deletedCleanupSleeper *dynamicSleeper
}

```
[RS纠删码](../../../../code/minio/cmd/erasure-coding.go)
```go
// 定义reedsolomon.Encoder编码，引用"github.com/klauspost/reedsolomon"
// Erasure - erasure encoding details.
type Erasure struct {
	encoder                  func() reedsolomon.Encoder
	dataBlocks, parityBlocks int
	blockSize                int64
}
```
[编码过程](../../../../code/minio/cmd/erasure-encode.go)


[解码过程](../../../../code/minio/cmd/erasure-decode.go)


[元数据处理](../../../../code/minio/cmd/erasure-metadata.go)


[解码过程](../../../../code/minio/cmd/erasure-healing.go)
## 原型验证


## 参考资料
1. minio纠删码(golang)：https://github.com/minio/minio/tree/master/docs/erasure
2. RS纠删码（golang)： https://github.com/klauspost/reedsolomon
3. RS纠删码（java): https://github.com/Backblaze/JavaReedSolomon
4. PyEC纠删码(C)： https://bitbucket.org/kmgreen2/pyeclib/src/master/
5. Jec纠删码(C)：https://bitbucket.org/tsg-/liberasurecode/src/master/
6. (C):https://github.com/randombit/botan.git
7. (C):https://github.com/randombit/fecpp(不再维护)
8. 【编码-纠错码】通信编码中的R-S编码方式：https://blog.csdn.net/rouranling/article/details/125159273