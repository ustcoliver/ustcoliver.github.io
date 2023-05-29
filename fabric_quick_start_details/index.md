# Hyperledger Fabric Quick Start 内容详解

>本文将结合`test-network`中的`network.sh`及`scripts`文件夹中脚本源码，介绍整个流程
>
>`network.sh`等相关脚本能实现很多功能，包括使用Fabric-ca创建证书，使用其他数据库，等等，
>
>目前本文档只介绍默认功能，其他功能自行阅读源码和文档学习

## 启动网络

{{< mermaid >}}
flowchart LR;

    subgraph createOrgs
    A1[为org1生成证书] --> A2[为org2生成证书] 
    A2 --> A3[为排序节点orderer生成证书]
    A3 --> A4[ccp-generate.sh]
    end
    subgraph createConsortium
    B[创建联盟<br>生成创世区块]
    end
    subgraph startContainers
    C1[docker compose up <br>启动容器]
    end

    createOrgs --> createConsortium --> startContainers
    

{{< /mermaid >}}

### createOrgs 创建组织
**org1 配置文件**
```yaml
# organizations/cryptogen/crypto-config-org1.yaml
PeerOrgs:
  - Name: Org1
    Domain: org1.example.com
    EnableNodeOUs: true
    Template:
    # 对应一个节点 organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com
      Count: 1 
      SANS:
        - localhost
    Users:
    #对应一个账户 organizations/peerorganizations/org1.examples.com/users/User1@org1.example.com
      Count: 1 
```
org2与之相比仅改了`Domain`为`org2.example.com`，不再赘述

`cryptogen generate --config=./organizations/cryptogen/crypto-config-org1.yaml --output="organizations"`
`cryptogen generate --config=./organizations/cryptogen/crypto-config-org2.yaml --output="organizations"`
上述两条命令为org1和org2在`organizations/peerOrganizations/`文件夹中生成了两个组织所需的证书文件

**排序节点orderer 配置文件**
```yaml
OrdererOrgs:
  - Name: Orderer
    Domain: example.com
    EnableNodeOUs: true
    Specs:
      - Hostname: orderer
        SANS:
          - localhost
```
`cryptogen generate --config=./organizations/cryptogen/crypto-config-orderer.yaml --output="organizations"`
为排序节点在`organizations/ordererOrganizations`生成了证书文件
### createConsortium 创建联盟
创建联盟，根据配置文件生成创世区块
`  configtxgen -profile TwoOrgsOrdererGenesis -channelID system-channel -outputBlock ./system-genesis-block/genesis.block`

**配置文件** `configtx/configtx.yaml`
``` yaml
# configtx.yaml 文件很长，此为关键部分
Profiles:

    TwoOrgsOrdererGenesis:
        <<: *ChannelDefaults
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *Org1
                    - *Org2
```

### 启动容器
`docker-compose -f docker/docker-compose-test-net.yaml` 
使用docker-compose启动所有容器

**docker-compose 容器配置文件**
内有4个容器`orderer.example.com, peer0.org1.example.com, peer0.org2.example.com，cli` 以及三个docker volume 数据卷，每个容器都设置了相关的环境变量，路径映射，端口映射等
``` yaml
# Copyright IBM Corp. All Rights Reserved.
#
# SPDX-License-Identifier: Apache-2.0
#

version: '2'

volumes:
  orderer.example.com:
  peer0.org1.example.com:
  peer0.org2.example.com:

networks:
  test:
    name: fabric_test

services:

  orderer.example.com:
    container_name: orderer.example.com
    image: hyperledger/fabric-orderer:latest
    environment:
      - FABRIC_LOGGING_SPEC=INFO
      - ORDERER_GENERAL_LISTENADDRESS=0.0.0.0
      - ORDERER_GENERAL_LISTENPORT=7050
      - ORDERER_GENERAL_GENESISMETHOD=file
      - ORDERER_GENERAL_GENESISFILE=/var/hyperledger/orderer/orderer.genesis.block
      - ORDERER_GENERAL_LOCALMSPID=OrdererMSP
      - ORDERER_GENERAL_LOCALMSPDIR=/var/hyperledger/orderer/msp
      - ORDERER_OPERATIONS_LISTENADDRESS=0.0.0.0:17050
      # enabled TLS
      - ORDERER_GENERAL_TLS_ENABLED=true
      - ORDERER_GENERAL_TLS_PRIVATEKEY=/var/hyperledger/orderer/tls/server.key
      - ORDERER_GENERAL_TLS_CERTIFICATE=/var/hyperledger/orderer/tls/server.crt
      - ORDERER_GENERAL_TLS_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
      - ORDERER_KAFKA_TOPIC_REPLICATIONFACTOR=1
      - ORDERER_KAFKA_VERBOSE=true
      - ORDERER_GENERAL_CLUSTER_CLIENTCERTIFICATE=/var/hyperledger/orderer/tls/server.crt
      - ORDERER_GENERAL_CLUSTER_CLIENTPRIVATEKEY=/var/hyperledger/orderer/tls/server.key
      - ORDERER_GENERAL_CLUSTER_ROOTCAS=[/var/hyperledger/orderer/tls/ca.crt]
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric
    command: orderer
    volumes:
        - ../system-genesis-block/genesis.block:/var/hyperledger/orderer/orderer.genesis.block
        - ../organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp:/var/hyperledger/orderer/msp
        - ../organizations/ordererOrganizations/example.com/orderers/orderer.example.com/tls/:/var/hyperledger/orderer/tls
        - orderer.example.com:/var/hyperledger/production/orderer
    ports:
      - 7050:7050
      - 17050:17050
    networks:
      - test

  peer0.org1.example.com:
    container_name: peer0.org1.example.com
    image: hyperledger/fabric-peer:latest
    environment:
      #Generic peer variables
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=fabric_test
      - FABRIC_LOGGING_SPEC=INFO
      #- FABRIC_LOGGING_SPEC=DEBUG
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
      # Peer specific variabes
      - CORE_PEER_ID=peer0.org1.example.com
      - CORE_PEER_ADDRESS=peer0.org1.example.com:7051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:7051
      - CORE_PEER_CHAINCODEADDRESS=peer0.org1.example.com:7052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:7052
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org1.example.com:7051
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org1.example.com:7051
      - CORE_PEER_LOCALMSPID=Org1MSP
      - CORE_OPERATIONS_LISTENADDRESS=0.0.0.0:17051
    volumes:
        - /var/run/docker.sock:/host/var/run/docker.sock
        - ../organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/msp:/etc/hyperledger/fabric/msp
        - ../organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls:/etc/hyperledger/fabric/tls
        - peer0.org1.example.com:/var/hyperledger/production
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 7051:7051
      - 17051:17051
    networks:
      - test

  peer0.org2.example.com:
    container_name: peer0.org2.example.com
    image: hyperledger/fabric-peer:latest
    environment:
      #Generic peer variables
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - CORE_VM_DOCKER_HOSTCONFIG_NETWORKMODE=fabric_test
      - FABRIC_LOGGING_SPEC=INFO
      #- FABRIC_LOGGING_SPEC=DEBUG
      - CORE_PEER_TLS_ENABLED=true
      - CORE_PEER_PROFILE_ENABLED=true
      - CORE_PEER_TLS_CERT_FILE=/etc/hyperledger/fabric/tls/server.crt
      - CORE_PEER_TLS_KEY_FILE=/etc/hyperledger/fabric/tls/server.key
      - CORE_PEER_TLS_ROOTCERT_FILE=/etc/hyperledger/fabric/tls/ca.crt
      # Peer specific variabes
      - CORE_PEER_ID=peer0.org2.example.com
      - CORE_PEER_ADDRESS=peer0.org2.example.com:9051
      - CORE_PEER_LISTENADDRESS=0.0.0.0:9051
      - CORE_PEER_CHAINCODEADDRESS=peer0.org2.example.com:9052
      - CORE_PEER_CHAINCODELISTENADDRESS=0.0.0.0:9052
      - CORE_PEER_GOSSIP_EXTERNALENDPOINT=peer0.org2.example.com:9051
      - CORE_PEER_GOSSIP_BOOTSTRAP=peer0.org2.example.com:9051
      - CORE_PEER_LOCALMSPID=Org2MSP
      - CORE_OPERATIONS_LISTENADDRESS=0.0.0.0:19051
    volumes:
        - /var/run/docker.sock:/host/var/run/docker.sock
        - ../organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/msp:/etc/hyperledger/fabric/msp
        - ../organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls:/etc/hyperledger/fabric/tls
        - peer0.org2.example.com:/var/hyperledger/production
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: peer node start
    ports:
      - 9051:9051
      - 19051:19051
    networks:
      - test

  cli:
    container_name: cli
    image: hyperledger/fabric-tools:latest
    tty: true
    stdin_open: true
    environment:
      - GOPATH=/opt/gopath
      - CORE_VM_ENDPOINT=unix:///host/var/run/docker.sock
      - FABRIC_LOGGING_SPEC=INFO
      #- FABRIC_LOGGING_SPEC=DEBUG
    working_dir: /opt/gopath/src/github.com/hyperledger/fabric/peer
    command: /bin/bash
    volumes:
        - /var/run/:/host/var/run/
        - ../organizations:/opt/gopath/src/github.com/hyperledger/fabric/peer/organizations
        - ../scripts:/opt/gopath/src/github.com/hyperledger/fabric/peer/scripts/
    depends_on:
      - peer0.org1.example.com
      - peer0.org2.example.com
    networks:
      - test

```

### ccp-generate.sh

此步骤貌似是与`Fabric Node SDK`有关，由于此部分没有使用任何sdk，所以在network.sh中将此处跳过也能完成上述功能。

我并不确定，有待后续研究发现

## 创建通道
{{<mermaid>}}
flowchart LR;
  subgraph createChannel
  A1[createChannelTx] --> A2[createChannel]
  end
  subgraph joinChannel
  B1[org1 join channel] --> B2[org2 join channel] --> B3[set anchor peer for org1] --> B4[set archor peer for org2]
  end
  createChannel --> joinChannel

{{</mermaid>}}

此步骤通过调用`script/createChannel.sh`并传入`CHANNEL_NAME,DELAY,MAX_RETRY,VERBOSE`参数实现

主要步骤包括
{{<mermaid>}}
flowchart LR;
    A[createChannelTx] --> B[createChannel] --> C[joinChannel] --> D[setAnchorPeer]
{{</mermaid>}}

### createChannelTx
`configtxgen -profile TwoOrgsChannel -outputCreateChannelTx ./channel-artifacts/${CHANNEL_NAME}.tx -channelID $CHANNEL_NAME
`
其配置文件仍然为`configtx/configtx.yaml`
``` yaml
# 第一部分 TwoOrgsOrdererGenesis 在创建联盟，生成创世区块时使用
# 第二部分 TwoOrgsChannel 在此处用于创建包含org1与org2的通道
Profiles:

    TwoOrgsOrdererGenesis:
        <<: *ChannelDefaults
        Orderer:
            <<: *OrdererDefaults
            Organizations:
                - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Consortiums:
            SampleConsortium:
                Organizations:
                    - *Org1
                    - *Org2
    TwoOrgsChannel:
        Consortium: SampleConsortium
        <<: *ChannelDefaults
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1
                - *Org2
            Capabilities:
                <<: *ApplicationCapabilities
```

### createChannel
此处脚本源码为：
`MAX_RETRY=5, DELAY=3`
``` shell
createChannel() {
	setGlobals 1
	# Poll in case the raft leader is not set yet
	local rc=1
	local COUNTER=1
	while [ $rc -ne 0 -a $COUNTER -lt $MAX_RETRY ] ; do
		sleep $DELAY
		set -x
		peer channel create -o localhost:7050 -c $CHANNEL_NAME --ordererTLSHostnameOverride orderer.example.com -f ./channel-artifacts/${CHANNEL_NAME}.tx --outputBlock $BLOCKFILE --tls --cafile $ORDERER_CA >&log.txt
		res=$?
		{ set +x; } 2>/dev/null
		let rc=$res
		COUNTER=$(expr $COUNTER + 1)
	done
	cat log.txt
	verifyResult $res "Channel creation failed"
}
```
每3秒尝试一次，共尝试5次，执行创建通道操作，参数`-o localhost:7050`指定ip和端口， `-c $CHANNEL_NAME`指定通道名，`--ordererTLSHostnameOverride orderer.example.com`覆写orderer排序节点名为`orderer.example.com`, 其后是输出设置

### joinChannel

```shell
joinChannel() {
  FABRIC_CFG_PATH=$PWD/../config/
  ORG=$1
  setGlobals $ORG
	local rc=1
	local COUNTER=1
	## Sometimes Join takes time, hence retry
	while [ $rc -ne 0 -a $COUNTER -lt $MAX_RETRY ] ; do
    sleep $DELAY
    set -x
    peer channel join -b $BLOCKFILE >&log.txt
    res=$?
    { set +x; } 2>/dev/null
		let rc=$res
		COUNTER=$(expr $COUNTER + 1)
	done
	cat log.txt
	verifyResult $res "After $MAX_RETRY attempts, peer0.org${ORG} has failed to join channel '$CHANNEL_NAME' "
}
```
其中`setGlobals`函数设置org1,org2和org3的环境变量，包括`CORE_PEER_LOCALMSPID,CORE_PEER_TLS_ROOTCERT_FILE,CORE_PEER_MSPCONFIGPATH,CORE_PEER_ADDRESS`
随后多次尝试加入channel

### setAnchorPeer
此时`script/createChannel.sh`调用`script/setAnchorPeer.sh`

执行了两个函数`createAnchorPeerUpdate`,`updateAnchorPeer`

看不太懂，先不写了

```shell
createAnchorPeerUpdate() {
  infoln "Fetching channel config for channel $CHANNEL_NAME"
  fetchChannelConfig $ORG $CHANNEL_NAME ${CORE_PEER_LOCALMSPID}config.json

  infoln "Generating anchor peer update transaction for Org${ORG} on channel $CHANNEL_NAME"

  if [ $ORG -eq 1 ]; then
    HOST="peer0.org1.example.com"
    PORT=7051
  elif [ $ORG -eq 2 ]; then
    HOST="peer0.org2.example.com"
    PORT=9051
  elif [ $ORG -eq 3 ]; then
    HOST="peer0.org3.example.com"
    PORT=11051
  else
    errorln "Org${ORG} unknown"
  fi

  set -x
  # Modify the configuration to append the anchor peer 
  jq '.channel_group.groups.Application.groups.'${CORE_PEER_LOCALMSPID}'.values += {"AnchorPeers":{"mod_policy": "Admins","value":{"anchor_peers": [{"host": "'$HOST'","port": '$PORT'}]},"version": "0"}}' ${CORE_PEER_LOCALMSPID}config.json > ${CORE_PEER_LOCALMSPID}modified_config.json
  { set +x; } 2>/dev/null

  # Compute a config update, based on the differences between 
  # {orgmsp}config.json and {orgmsp}modified_config.json, write
  # it as a transaction to {orgmsp}anchors.tx
  createConfigUpdate ${CHANNEL_NAME} ${CORE_PEER_LOCALMSPID}config.json ${CORE_PEER_LOCALMSPID}modified_config.json ${CORE_PEER_LOCALMSPID}anchors.tx
}

updateAnchorPeer() {
  peer channel update -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com -c $CHANNEL_NAME -f ${CORE_PEER_LOCALMSPID}anchors.tx --tls --cafile $ORDERER_CA >&log.txt
  res=$?
  cat log.txt
  verifyResult $res "Anchor peer update failed"
  successln "Anchor peer set for org '$CORE_PEER_LOCALMSPID' on channel '$CHANNEL_NAME'"
}
```

## 部署链码

``` shell
./network.sh deployCC -ccn basic -ccp ../asset-transfer-basic/chaincode-go -ccl go
```

此处通过运行`script/deployCC.sh`实现，此脚本提供了部署`go, java, nodejs`语言的链码的功能，这里只介绍`go chaincode`

{{<mermaid>}}
flowchart LR
  subgraph Prepare-Install
  A1[go mod vendor<br>下载依赖库源码到vendor文件夹] 
  A2[packageChaincode<br>打包chaincode]
  A3[installChaincode<br>在org1与org2上安装chaincode]
  A1 --> A2 --> A3
  end

  subgraph Query-Approve
  B1[queryInstalled<br>查询在org1是否安装成功]
  B2[approveForMyOrg<br>approve the definition for org1]
  B3[checkCommitReadiness<br>查询chaincode定义 期望org1同意 org2没有]
  B4[approveForMyOrg<br>approve the definition for org2]
  B5[checkCommitReadiness<br>查询chaincode定义 期望两组织都同意]
  B1 --> B2 --> B3 --> B4 --> B5 
  end

  subgraph Commit-Query
  C1[commitChaincodeDefinition]
  C2[queryCommitted]
  C1-->C2
  end
  Prepare-Install --> Query-Approve --> Commit-Query


{{</mermaid>}}

### 下载依赖库源码

```shell
  pushd $CC_SRC_PATH # 切换目录到`asset-transfer-basic`
  GO111MODULE=on go mod vendor
  popd
```
此目录中`go.mod`中只有一个`github.com/hyperledger/fabric-sdk-go`的依赖，使用此命令将依赖源码下载到`asset-transfer-basic/vendor`目录中

### packageChaincode:打包链码
```shell
peer lifecycle chaincode package basic.tar.gz --path ../asset-transfer-basic/chaincode-go --lang golang --label basic_1.0
```

### installChaincode:安装链码 
设置好环境变量后使用以下命令在org1和org2上安装链码
```
peer lifecycle chaincode install basic.tar.gz
```
### queryInstalled:查询链码安装
设置环境变量后，以org1或org2身份查询
```
peer lifecycle chaincode queryinstalled
```
### appreveForMyOrg:approve链码
```shell
peer lifecycle chaincode approveformyorg -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /home/ubuntu/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --channelID mychannel --name basic --version 1.0 --package-id basic_1.0:3cfcf67978d6b3f7c5e0375660c995b21db19c4330946079afc3925ad7306881 --sequence 1
```
### checkCommitReadiness:查询链码approve
```shell
peer lifecycle chaincode checkcommitreadiness --channelID mychannel --name basic --version 1.0 --sequence 1 --output json
```

### commitChaincodeDefinition: 提交链码定义

```shell
peer lifecycle chaincode commit -o localhost:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile /home/ubuntu/fabric-samples/test-network/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem --channelID mychannel --name basic --peerAddresses localhost:7051 --tlsRootCertFiles /home/ubuntu/fabric-samples/test-network/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt --peerAddresses localhost:9051 --tlsRootCertFiles /home/ubuntu/fabric-samples/test-network/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt --version 1.0 --sequence 1
```
### queryCommited:查询提交
```shell
peer lifecycle chaincode querycommitted --channelID mychannel --name basic
```


## 调用链码

这部分没什么好解释的






