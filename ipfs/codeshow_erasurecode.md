
# 纠删码技术

## 目录

<!-- TOC depthfrom:2 -->
- [纠删码技术](#纠删码技术)
	- [目录](#目录)
	- [纠删码相关代码](#纠删码相关代码)
	- [原型验证](#原型验证)
	- [参考资料](#参考资料)

<!-- /TOC -->

## 纠删码相关代码

```bash
(base) ➜  codeshow git:(main) ✗ ls  /Users/blockchain/code/minio/cmd/erasure* 
/Users/blockchain/code/minio/cmd/erasure-bucket.go
/Users/blockchain/code/minio/cmd/erasure-coding.go
/Users/blockchain/code/minio/cmd/erasure-common.go
/Users/blockchain/code/minio/cmd/erasure-decode.go
/Users/blockchain/code/minio/cmd/erasure-decode_test.go
/Users/blockchain/code/minio/cmd/erasure-encode.go
/Users/blockchain/code/minio/cmd/erasure-encode_test.go
/Users/blockchain/code/minio/cmd/erasure-errors.go
/Users/blockchain/code/minio/cmd/erasure-heal_test.go
/Users/blockchain/code/minio/cmd/erasure-healing-common.go
/Users/blockchain/code/minio/cmd/erasure-healing-common_test.go
/Users/blockchain/code/minio/cmd/erasure-healing.go
/Users/blockchain/code/minio/cmd/erasure-healing_test.go
/Users/blockchain/code/minio/cmd/erasure-metadata-utils.go
/Users/blockchain/code/minio/cmd/erasure-metadata-utils_test.go
/Users/blockchain/code/minio/cmd/erasure-metadata.go
/Users/blockchain/code/minio/cmd/erasure-metadata_test.go
/Users/blockchain/code/minio/cmd/erasure-multipart.go
/Users/blockchain/code/minio/cmd/erasure-object.go
/Users/blockchain/code/minio/cmd/erasure-object_test.go
/Users/blockchain/code/minio/cmd/erasure-server-pool-decom.go
/Users/blockchain/code/minio/cmd/erasure-server-pool-decom_gen.go
/Users/blockchain/code/minio/cmd/erasure-server-pool-decom_gen_test.go
/Users/blockchain/code/minio/cmd/erasure-server-pool-decom_test.go
/Users/blockchain/code/minio/cmd/erasure-server-pool.go
/Users/blockchain/code/minio/cmd/erasure-sets.go
/Users/blockchain/code/minio/cmd/erasure-sets_test.go
/Users/blockchain/code/minio/cmd/erasure-utils.go
/Users/blockchain/code/minio/cmd/erasure.go # 定义ER对象
/Users/blockchain/code/minio/cmd/erasure_test.go # 测试代码
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
// 定义reedsolomon.Encoder编码
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
https://github.com/minio/minio/tree/master/docs/erasure
https://github.com/klauspost/reedsolomon
https://bitbucket.org/kmgreen2/pyeclib/src/master/
https://bitbucket.org/tsg-/liberasurecode/src/master/
https://github.com/randombit/botan.git
https://github.com/randombit/fecpp(不再维护)