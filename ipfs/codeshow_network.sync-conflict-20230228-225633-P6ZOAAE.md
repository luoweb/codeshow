# 

## 网络配置和节点发现配置：


### 网络节点配置：bootstrap:
1. 节点监听(address),网络组网主要涉及swarm地址列表，支持tcp,udp,quic等协议
```json
"Addresses": {
    "API": "/ip4/127.0.0.1/tcp/5001",
    "Announce": [],
    "AppendAnnounce": [],
    "Gateway": "/ip4/127.0.0.1/tcp/8081",
    "NoAnnounce": [],
    "Swarm": [
        "/ip4/0.0.0.0/tcp/4001",
        "/ip6/::/tcp/4001",
        "/ip4/0.0.0.0/udp/4001/quic",
        "/ip4/0.0.0.0/udp/4001/quic-v1",
        "/ip4/0.0.0.0/udp/4001/quic-v1/webtransport",
        "/ip6/::/udp/4001/quic",
        "/ip6/::/udp/4001/quic-v1",
        "/ip6/::/udp/4001/quic-v1/webtransport"
    ]
},
```
监听启动日志分析如下：
```bash
Initializing daemon...
Kubo version: 0.18.1
Repo version: 13
System version: amd64/darwin
Golang version: go1.19.1
Swarm listening on /ip4/127.0.0.1/tcp/4002
Swarm listening on /ip4/127.0.0.1/udp/4002/quic
Swarm listening on /ip4/127.0.0.1/udp/4002/quic-v1
Swarm listening on /ip4/127.0.0.1/udp/4002/quic-v1/webtransport/certhash/uEiA56Z2R6q-a3ksTbqZvmPPzgF8PY7nuGsdGagWc0liWyg/certhash/uEiAXIGEEC12l-muxGessTQL82MLJaNyn__oxOiEIZHC4fQ
Swarm listening on /ip4/192.168.108.18/tcp/4002
Swarm listening on /ip4/192.168.108.18/udp/4002/quic
Swarm listening on /ip4/192.168.108.18/udp/4002/quic-v1
Swarm listening on /ip4/192.168.108.18/udp/4002/quic-v1/webtransport/certhash/uEiA56Z2R6q-a3ksTbqZvmPPzgF8PY7nuGsdGagWc0liWyg/certhash/uEiAXIGEEC12l-muxGessTQL82MLJaNyn__oxOiEIZHC4fQ
Swarm listening on /ip4/192.168.64.1/tcp/4002
Swarm listening on /ip4/192.168.64.1/udp/4002/quic
Swarm listening on /ip4/192.168.64.1/udp/4002/quic-v1
Swarm listening on /ip4/192.168.64.1/udp/4002/quic-v1/webtransport/certhash/uEiA56Z2R6q-a3ksTbqZvmPPzgF8PY7nuGsdGagWc0liWyg/certhash/uEiAXIGEEC12l-muxGessTQL82MLJaNyn__oxOiEIZHC4fQ
Swarm listening on /ip6/::1/tcp/4002
Swarm listening on /ip6/::1/udp/4002/quic
Swarm listening on /ip6/::1/udp/4002/quic-v1
Swarm listening on /ip6/::1/udp/4002/quic-v1/webtransport/certhash/uEiA56Z2R6q-a3ksTbqZvmPPzgF8PY7nuGsdGagWc0liWyg/certhash/uEiAXIGEEC12l-muxGessTQL82MLJaNyn__oxOiEIZHC4fQ
Swarm listening on /p2p-circuit
```
> 最后启动日志监听了/p2p-circuit这个地址，如果2个节点处于两个不同内网环境，由于存在NAT设备，NAT设备可能是对称型，对称型的NAT设备是没有办法穿透的，所以IPFS提供了relay的方式解决不同内网环境下节点的连接问题，上面提到的监听/p2p-circuit地址则是为了解决该问题，对于2个处于不同内网环境不能直接连接的节点，通过配置relay节点中转从而建立连接。


P2P网络监听代码实现[kubo/p2p/local.go](https://github.com/ipfs/kubo/blob/master/p2p/local.go)
```go
// ForwardLocal creates new P2P stream to a remote listener
func (p2p *P2P) ForwardLocal(ctx context.Context, peer peer.ID, proto protocol.ID, bindAddr ma.Multiaddr) (Listener, error) {
	listener := &localListener{
		ctx:   ctx,
		p2p:   p2p,
		proto: proto,
		peer:  peer,
	}

	maListener, err := manet.Listen(bindAddr)
	if err != nil {
		return nil, err
	}

	listener.listener = maListener
	listener.laddr = maListener.Multiaddr()

	if err := p2p.ListenersLocal.Register(listener); err != nil {
		return nil, err
	}

	go listener.acceptConns()

	return listener, nil
}
```



2. 初始化节点组网配置(bootstrap)：静态寻址
> IPFS节点在启动之前需要配置它的Bootstrap节点，配置文件中相关配置如下图所示，Bootstrap配置中配置了IPFS节点启动时需要连接的所有种子节点列表，这些节点地址列表信息是默认的，如果需要搭建IPFS私有网络可以修改成自己的种子节点列表（Qm开头的字符串是IPFS的节点id）。默认提供的种子节点都是具有公网地址的节点，IPFS节点启动的时候首先连接该种子节点，后续通过该种子节点去发现IPFS网络中更多的节点，从而进行连接，也就是DHT组网阶段。

```json
"Bootstrap": [
    "/dnsaddr/bootstrap.libp2p.io/p2p/QmNnooDu7bfjPFoTZYxMNLWUQJyrVwtbZg5gBMjTezGAJN",
    "/dnsaddr/bootstrap.libp2p.io/p2p/QmQCU2EcMqAqQPR2i9bChDtGNJchTbq5TbXJJ16u19uLTa",
    "/dnsaddr/bootstrap.libp2p.io/p2p/QmbLHAnMoJPWSCR5Zhtx6BHJX9KiKNN6tpvbUcqanj75Nb",
    "/dnsaddr/bootstrap.libp2p.io/p2p/QmcZf59bWwK5XFi76CZX8cbJ4BhTzzA3gU1ZjYZcYW3dwt",
    "/ip4/104.131.131.82/tcp/4001/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ",
    "/ip4/104.131.131.82/udp/4001/quic/p2p/QmaCpDMGvV2BGHeYERUEnRQAwe3N8SzbUtfsmvsqQLuvuJ"
],
```


### 网络节点发现：DHT
IPFS基于libp2p协议和模块的，节点发现用来发现P2P网络中的其它节点及维护节点在线状态，并且根据节点状态调整网络连接，构建稳定的网络拓扑。
> IPFS节点连接种子节点成功以后则去通过DHT去发现其他节点，发现其他节点之后则尝试进行连接，连接成功的节点会加入到该节点的节点列表，以便后续可以直接与该节点通信，考虑到全世界的IPFS节点规模很大，不可能每个节点和其他节点保持长连接，所以对每个节点的连接数量做了限制，一般节点连接数量都在1千以下（IPFS配置文件中可以配置），对于没有连接的节点需要通信的话，可以通过DHT找到该节点地址，然后连接该节点进行通信，这样就构成了大规模的分布式节点网络。


DHT节点发现代码实现[core/coreapi/dht.go](https://github.com/ipfs/kubo/blob/master/core/coreapi/dht.go)



## 参考资料：

1. [IPFS网络组网解析](https://tech.hyperchain.cn/ipfs-4/) 
2. [揭秘IPFS数据交换模块Bitswap](https://mp.weixin.qq.com/s?__biz=Mzg2MDA2NzQwNw==&mid=2247484025&idx=1&sn=691e344491c16a3c3f120de48f386f77&scene=21#wechat_redirect)
3. [更通用的P2P网络协议栈——Libp2p](https://tech.hyperchain.cn/bitxmesh/)
4. [P2P传输的开源库:Libjingle库 综述](https://blog.51cto.com/iteyer/3240049)