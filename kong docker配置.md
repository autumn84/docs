# kong docker配置

## 目标
实现API Gateway相关控制，API Key, 流控等

## 软件
Postgres + Kong   
参考：  
[使用Docker安装Kong - 玩转Kong网关](https://www.jianshu.com/p/8bfc8b88e812)

## 安装及配置

### network配置
```
docker network create kong-net
```

### postgres

```
docker run -d --name kong-database --network kong-net -p 5432:5432 -e "POSTGRES_USER=kong" -e "POSTGRES_DB=kong" postgres:9.6
```

### 初始化kong数据库
```
docker run --rm --network=kong-net -e "KONG_DATABASE=postgres" -e "KONG_PG_HOST=kong-database" -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database"  kong:0.13.1-centos  kong migrations up 
```

### kong启动
```
docker run -d --name kong --network=kong-net -e "KONG_DATABASE=postgres" -e "KONG_PG_HOST=kong-database" -e "KONG_CASSANDRA_CONTACT_POINTS=kong-database" -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" -e "KONG_PROXY_ERROR_LOG=/dev/stderr" -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" -e "KONG_ADMIN_LISTEN=0.0.0.0:8001, 0.0.0.0:8444 ssl" -p 8000:8000 -p 8443:8443 -p 8001:8001 -p 8444:8444 -v ~/kong:/etc/kong kong:0.13.1-centos
```

#### kong https配置
对于https配置，主要在kong.conf文件如下：
```
ssl_cert =  /etc/kong/ssl.crt
ssl_cert_key =  /etc/kong/ssl.key
```
这里ssl.crt和ssl.key是我们申请到的证书和key文件，kong.conf，ssl.crt和ssl.key都放在宿主机上，比方都再放在宿主机~/kong目录下，然后可以通过“-v ~/kong:/etc/kong”挂载到kong容器内即可。Kong默认的https端口是8443，对外暴露时也可以修改成https默认的443。

#### service 
```
curl -i -X POST http://localhost:8001/services -d "name=服务名称" -d "url=后台实际服务地址"
```
#### route
```
curl -i -X POST --url http://localhost:8001/routes/ -d 'protocols[]=https' -d 'paths[]=/relay2api' -d 'service.id=上边配置的服务ID'
```
#### API Key
启动key-auth
```
curl -i -X POST --url http://localhost:8001/services/前边配置的service名称/plugins/ --data 'name=key-auth’
```
配置用户
```
curl -X POST http://localhost:8001/consumers/ --data “username=***" --data “custom_id=***”
```
取得用户的apiKey
```
curl -X POST http://localhost:8001/consumers/用户username/key-auth -d ‘'
```

举例：
```
curl  --data '{"method":"get_accounts","params":{"addresses":["0xd65a23388d5d6f0b1ec52a7fc07a291c132d57ed"], "tokens":[],"allTokens":true},"id":"aaa","jsonrpc":"2.0"}' -H "Content-Type: application/json" -H "apikey:***" -X POST https://lightcone.io/***
```

