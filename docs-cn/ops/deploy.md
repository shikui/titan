# Titan 的安装与部署

## 部署 TiKV

Titan 是运行在 TiKV 之上的 redis 协议层，将 TiKV 作为其数据持久化的底层，所以需要先部署 TiKV。

TiKV的部署参阅：https://pingcap.com/docs/op-guide/ansible-deployment/ （使用ansible部署）

也可以手工部署 TiKV：https://www.pingcap.com/docs/tikv/deploy-tikv-using-binary/

## 部署 Titan

### 编译

```
go get github.com/meitu/titan
cd $GOPATH/src/github.com/meitu/titan
make 
```

### 编辑配置文件

编辑 conf/titan.toml 设置 pd-addrs

```
pd-addrs="tikv://your-pd-addrs:port"
```

### 启用多租户

多个应用可以使用不同的 namespace 实现数据隔离。

首先，设置 conf/titan.toml 中的 auth。

```
auth = "YOUR_SERVER_KEY"
```

然后使用 ./tools/token/token 生成一个用于客户端验证的 token。

```
cd $GOPATH/src/github.com/meitu/titan/tools/token
go build main.go -o titan-gen-client-token
./titan-gen-client-token -key YOUR_SERVER_KEY -namespace bbs
```

这样就得到了一个用于客户端验证的 token，比如：bbs-1543999615-1-7a50221d92e69d63e1b443

### 运行 Titan

```
cd $GOPATH/src/github.com/meitu/titan
./titan
```

## 测试

```
redis-cli -p 7369
```

如果启用了多租户，要使用服务器端生成的 token 进行验证。

```
redis-cli -p 7369 -a bbs-1543999615-1-7a50221d92e69d63e1b443
```
