# Hyperledger Fabric Quick Start


<!--more-->

本文主要讲解`Fabric 2.2`区块链的简单搭建和使用, 帮助快速了解Fabric

[官方中文文档](https://hyperledger-fabric.readthedocs.io/zh_CN/release-2.2/)

> Fabric 的长期支持版本为`1.4`和`2.2`，目前`1.4`已经过期，今后所有开发都基于`2.2`版本，本文使用`v2.2.6`
## 准备

安装开始前，确保自己的ubuntu系统中有`git, curl, docker, docker-compose`, 具体安装方法可见[Ubuntu 基础软件安装与配置](http://192.168.5.25/posts/basic_install/)

## 安装

``` shell
curl -sSL https://raw.githubusercontent.com/hyperledger/fabric/v2.2.6/scripts/bootstrap.sh | bash
```
此脚本会帮你
* 克隆`fabric-samples`到当前文件夹，切换到`v2.2.6`
* 下载相应版本的二进制文件和config文件到`fabric-samples/bin`，`fabric-samples/config`文件中，
* 拉取相应版本的docker image,

> 此过程快慢取决于机器到镜像服务器的带宽，可能时间会比较长

## Hello World!


切换目录
``` shell
cd fabric-samples/test-network
```
关闭之前所有fabric相关的docker 容器，网络，volume， 第一次运行还会pull一个`busybox:latex`镜像
```
./network.sh down
```

### 启动网络
``` shell
./network.sh up
```
此命令创建一个由两个对等节点和一个排序节点组成的Fabric网络。


### 创建通道

``` shell
./network.sh createChannel
```
运行成功日志：
``` shell
Channel 'mychannel' joined
```

### 启动链码

``` shell
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
```

### 调用链码
设置环境变量
```shell
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
```

**初始化账本**

```shell
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"InitLedger","Args":[]}'
```


**查询账本**

```shell
peer chaincode query -C mychannel -n basic -c '{"Args":["GetAllAssets"]}'
```
运行成功输出
```
[{"ID":"asset1","color":"blue","size":5,"owner":"Tomoko","appraisedValue":300},
{"ID":"asset2","color":"red","size":5,"owner":"Brad","appraisedValue":400},
{"ID":"asset3","color":"green","size":10,"owner":"Jin Soo","appraisedValue":500},
{"ID":"asset4","color":"yellow","size":10,"owner":"Max","appraisedValue":600},
{"ID":"asset5","color":"black","size":15,"owner":"Adriana","appraisedValue":700},
{"ID":"asset6","color":"white","size":15,"owner":"Michel","appraisedValue":800}]
```
**调用链码，更改所有者**
```shell
peer chaincode invoke -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile ${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem -C mychannel -n basic --peerAddresses localhost:7051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles ${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt -c '{"function":"TransferAsset","Args":["asset6","Christopher"]}'
```
**查询账本**

这里切换到org2的身份去查询账本

```shell
# Environment variables for Org2

export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
```
查询asset6
```shell
peer chaincode query -C mychannel -n basic -c '{"Args":["ReadAsset","asset6"]}'
```
查询结果
```
{"ID":"asset6","color":"white","size":15,"owner":"Christopher","appraisedValue":800}
```
由此可见，owner由前面的`Michel`变成了`CHristopher`

### 关闭网络
至此，hello world部分功能测试结束，关闭网络即可
```shell
./network.sh down
```

