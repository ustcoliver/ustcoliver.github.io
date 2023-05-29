# FISCO BCOS 区块链开发概述

<!--more-->

通过之前的比赛以及后来区块链电子投票平台的搭建，目前我们小组对FISCO BCOS 区块链的开发和测试的流程还算熟悉，这里写成一个文档，帮助后续同学快速学习。

* [FISCO BCOS 官方文档](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/)
 
* [FISCO BCOS github 源码](https://github.com/FISCO-BCOS/FISCO-BCOS)

> * 本文只涉及在ubuntu 上如何搭建和设置，对于windows或其他发行版linux，请自行阅读官方文档学习。
> * 目前环境基于Ubuntu 21.10搭建， 22.04把openssl大版本从1.x更新到了3.x，如果需要运行fisco bcos， 需要修改脚本跳过openssl的版本检查，可能还会存在未知问题。

## 区块链搭建


### First Blockchain
[参考链接](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/installation.html)

**创建并进入文件夹**

```bash
mkdir fisco && cd fisco
```

**下载官方脚本**
```bash
curl -#LO https://github.com/FISCO-BCOS/FISCO-BCOS/releases/download/v2.9.0/build_chain.sh && chmod u+x build_chain.sh
```
**搭建单群组4节点区块链**
```bash
bash build_chain.sh -l 127.0.0.1:4 -p 30300,20200,8545
```
其中，-l 指定了节点ip`127.0.0.1`和节点个数`4`，可以自行修改. -p 执行了节点各种通信协议的起始端口，第一个节点端口为`30300, 20200, 8545`, 第二个为`30301, 20201, 8546`, 以此类推

关于`build_chain.sh`更加细致的介绍可以参考[官方文档](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/manual/build_chain.html#id2)

**启动区块链**
```bash
bash ./node/127.0.0.1/start_all.sh
```
相应的，文件夹中还有`stop_all.sh`用来关闭区块链节点，每个node文件夹中有单独的启动和关闭脚本

**检查是否成功启动**
```bash
ps -ef | grep -v grep | grep fisco-bcos
```
**检查日志输出**
```bash
tail -f nodes/127.0.0.1/node0/log/log*  | grep connected
```

### docker启动区块链
在原本命令上加`-d`选项，即可使用docker启动4个容器作为节点。 
```shell
bash build_chain.sh -d -l 127.0.0.1:4 -p 30300,20200,8545
```
同样使用`nodes/127.0.0.1/start_all.sh`启动所有节点
```shell
$ ./nodes/127.0.0.1/start_all.sh
try to start node0
try to start node1
try to start node2
try to start node3
e417cc673df4907dee792eb793ab1bd6e9c49d062c5deb3346d75be13f4d57ba
f90f7aaa0dccabb855fb7fdb695d58f9c292252ec7d85e5f36ff4be3a9620309
ab529fe551e84d65ff963c6d109a6a7d4ef4684e1f8b5e73496f907a6f5d70b5
46c4db857296d6dddded3e9aba8328a27b2d6ec3a92255861297c4b426c29daa
 node0 start successfully
 node3 start successfully
 node1 start successfully
 node2 start successfully
```
`docker ps `命令查看节点情况
```shell
$ docker ps
CONTAINER ID   IMAGE                       COMMAND                  CREATED         STATUS         PORTS     NAMES
f90f7aaa0dcc   fiscoorg/fiscobcos:v2.8.0   "/usr/local/bin/fisc…"   5 seconds ago   Up 4 seconds             homeswtfishbonechainnodes127.0.0.1node3
ab529fe551e8   fiscoorg/fiscobcos:v2.8.0   "/usr/local/bin/fisc…"   5 seconds ago   Up 4 seconds             homeswtfishbonechainnodes127.0.0.1node0
46c4db857296   fiscoorg/fiscobcos:v2.8.0   "/usr/local/bin/fisc…"   5 seconds ago   Up 3 seconds             homeswtfishbonechainnodes127.0.0.1node2
e417cc673df4   fiscoorg/fiscobcos:v2.8.0   "/usr/local/bin/fisc…"   5 seconds ago   Up 4 seconds             homeswtfishbonechainnodes127.0.0.1node1
```

### 多机多联盟区块链
[官方文档](https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/enterprise_tools/index.html)已经很详细了，这里不做赘述

## Python-SDK
[参考链接](https://github.com/FISCO-BCOS/python-sdk)

由于ubuntu 21.10自带python版本能够满足其需求，所以这里不再对如何处理python低版本赘述
### 准备

**安装依赖**
```bash
sudo apt install -y zlib1g-dev libffi6 libffi-dev wget git
```
**拉取源码**
```bash
git clone https://github.com/FISCO-BCOS/python-sdk
```
**安装依赖**
```bash
cd python-sdk
pip install -r requirement.txt
```
**初始化配置**
```bash
bash init_env.sh -i
```
此脚本执行操作如下：
* copy client_config.py.template > client_config.py
* 安装solc 编译器

### 配置channel通信协议
此时需要修改config_client.py中的内容

**配置channel信息**
python sdk 使用fisco bcos 称之为`channel`的通信协议与节点进行通信，即创建区块链时所用的`20200`端口
```
channel_host = "127.0.0.1"
channel_port = 20200
```
想要配置其他节点，则可以根据`fisco/nodes/127.0.0.1/nodex/config.ini`中的内容进行修改

**配置证书**
```bash
cp ~/fisco/nodes/127.0.0.1/sdk/* ./bin
```
将区块链的相关sdk证书复制到bin文件夹中

**验证**

```bash
./console.py getNodeVersion
```
如果返回以下类似结果，则可认为配置成功
```json
INFO >> user input : ['getNodeVersion']

INFO >> BcosClient:  channel 127.0.0.1:20200,groupid :1,crypto type:ECDSA,ssl type:ECDSA
INFO : getNodeVersion
     : {
    "Build Time": "20210830 12:52:15",
    "Build Type": "Linux/clang/Release",
    "Chain Id": "1",
    "FISCO-BCOS Version": "2.8.0",
    "Git Branch": "HEAD",
    "Git Commit Hash": "30fb38ac5692468058abf6aa12869d2ae776c275",
    "Supported Version": "2.8.0"
}
```

### 调用示例
```bash
# 查看SDK使用方法
./console.py usage

# 获取节点版本
./console.py getNodeVersion
```
**部署HelloWorld合约**
```shell
./console.py deploy HelloWorld
```
这里编译并部署了位于python-sdk中`contracts/HelloWorld.sol`的文件

**调用HelloWorld合约**
```shell
./console.py call HelloWorld 0x2d1c577e41809453c50e7e5c3f57d06f3cdd90ce get 
```
这里`0x2d1c577e41809453c50e7e5c3f57d06f3cdd90ce`为部署成功后返回的合约地址，此处仅为举例。`get`用来调用合约被修饰为`view`的函数
```shell
./console.py sendtx HelloWorld 0x2d1c577e41809453c50e7e5c3f57d06f3cdd90ce set "Hello, FISCO"
```
set为HelloWorld合约中的函数，”FISCO BCOS“为为set传入的参数

### blockchain.py
我们在参加比赛时，**陈鲁同**同学基于python-sdk源码写了`blockchain.py`,为我们执行一系列合约调用提供了支持
```python
# 代码主体部分
from enum import unique
import sys
import os
import json
from typing import Dict, Any, List, Union
import traceback
import re

from models import Contract, db

from client.bcosclient import BcosClient
from client.bcoserror import BcosError, BcosException
from client.datatype_parser import DatatypeParser
from client.common.compiler import Compiler
from eth_utils import to_checksum_address
from eth_account.account import Account
from client.signer_impl import Signer_ECDSA, Signer_Impl
from client_config import client_config
from __init__ import app
from itertools import count


# 生成区块链账户 写入.keystore文件存储信息
def create_account(name, password=""):
    ac = Account.create(password)
    kf = Account.encrypt(ac.privateKey, password)
    keyfile = "{}/{}.keystore".format(client_config.account_keyfile_path, name)

    with open(keyfile, "w") as dump_f:
        json.dump(kf, dump_f)
        dump_f.close()

    return signin_account(name, password)


# 返回用户区块链地址
def signin_account(username, password) -> Union[Signer_Impl, None]:
    try:
        signer = Signer_ECDSA.from_key_file(
            f"{client_config.account_keyfile_path}/{username}.keystore", password
        )
    except:
        return None
    return signer


def compile_and_abis(compile: bool = True):
    """
    Compiles all contracts and generates abi and bin files
    """
    global abis, dp
    for c in contract_list:
        if compile:
            Compiler.compile_file(f"contracts/{c}.sol", output_path="contracts")
        # data_parser = DatatypeParser()
        # data_parser.load_abi_file(f"contracts/{c}.abi")
        # abis[c] = data_parser.contract_abi
        # dp[c] = data_parser


# 部署合约返回合约地址
def deploy_contract(
    contract, compile: bool = False, signer: Signer_Impl = None, fn_args=None
):
    """
    Args:
        contract: the contract's name, e.g.: "EngineerList"
        compile (bool): compile or not
    Returns:
        the contract address
    """
    if compile and (
        os.path.isfile(client_config.solc_path)
        or os.path.isfile(client_config.solcjs_path)
    ):
        Compiler.compile_file(f"contracts/{contract}.sol", output_path="contracts")

    data_parser = DatatypeParser()
    data_parser.load_abi_file(f"contracts/{contract}.abi")

    client = BcosClient()
    try:
        with open(f"contracts/{contract}.bin", "r") as load_f:
            contract_bin = load_f.read()
            load_f.close()
        result = client.deploy(
            contract_bin,
            contract_abi=data_parser.contract_abi,
            fn_args=fn_args,
            from_account_signer=signer,
        )
        addr = result["contractAddress"]
    except BcosError as e:
        traceback.print_exc()
        return None
    except BcosException as e:
        traceback.print_exc()
        return None
    except Exception as e:
        traceback.print_exc()
        return None
    finally:
        client.finish()
    app.logger.info(f"deploy contract {contract} at {addr}")
    return addr


# 调用合约
def call_contract(
    contract_addr: str,
    contract_name: str,
    fn_name: str,
    args: List = None,
    signer: Signer_Impl = None,
    gasPrice=30000000,
):
    global abis, dp
    client = BcosClient()

    # contract_abi = abis[contract_name]
    data_parser: DatatypeParser = DatatypeParser()
    data_parser.load_abi_file(f"contracts/{contract_name}.abi")
    contract_abi = data_parser.contract_abi

    receipt = client.sendRawTransactionGetReceipt(
        to_address=contract_addr,
        contract_abi=contract_abi,
        fn_name=fn_name,
        args=args,
        from_account_signer=signer,
        gasPrice=gasPrice,
    )

    if receipt["status"] != "0x0":
        msg = receipt.get("statusMsg", "")
        app.logger.warn(
            f"call contract {contract_name} at {contract_addr}. {fn_name} ({args}) error:{msg}"
        )
        app.logger.warn(f"receipt: {receipt}")
        print(msg)
        raise Exception(f"contract error: {msg}")
    txhash = receipt["transactionHash"]
    txresponse = client.getTransactionByHash(txhash)
    inputresult = data_parser.parse_transaction_input(txresponse["input"])
    outputresult = data_parser.parse_receipt_output(
        inputresult["name"], receipt["output"]
    )
    client.finish()
    app.logger.info(
        f"call contract {contract_name} at {contract_addr}. {fn_name} ({args}) -> {outputresult}"
    )
    return outputresult


def call_contract2(
    contract_addr: str,
    contract_name: str,
    fn_name: str,
    args: List = None,
    signer: Signer_Impl = None,
):
    client = BcosClient()
    if signer is not None:
        client.default_from_account_signer = signer

    data_parser: DatatypeParser = DatatypeParser()
    data_parser.load_abi_file(f"contracts/{contract_name}.abi")
    contract_abi = data_parser.contract_abi

    ret = client.call(contract_addr, contract_abi, fn_name, args)
    app.logger.info(
        f"call contract {contract_name} at {contract_addr}. {fn_name} ({args}) -> {ret}"
    )
    client.finish()
    return ret
``` 

## WeBASE 后台部署
WeBASE屏蔽了区块链底层的复杂度，降低区块链使用的门槛，大幅提高区块链应用的开发效率，包含节点前置、节点管理、交易链路，数据导出，Web管理平台等子系统。用户可以根据业务所需，选择子系统进行部署，可以进一步体验丰富交互的体验、可视化智能合约开发环境IDE。
### JAVA 安装
```shell
# 安装openjdk 11
sudo apt install openjdk-11-jdk
# 验证安装
java -version
# 设置JAVA_HOME, 
#如果使用bash，则在~/.bashrc文件最后加入此行，
#如果是zsh，则在~/.zshrc中加入。 目前实验室虚拟机都使用的是zsh
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
```

### MySQL 安装
这里使用docker容器做为mysql服务器，mariadb-client-core作为客户端，
```shell
docker pull mysql:5.6
sudo apt install mariadb-client-core-10.5
pip3 install PyMySQL
```

### WeBASE部署
```shell
wget https://osp-1257653870.cos.ap-guangzhou.myqcloud.com/WeBASE/releases/download/v1.5.3/webase-deploy.zip
unzip webase-deploy.zip && cd webase-deploy
# 创建mysql服务相关文件夹，启动容器
mkdir sqlConf sqlData sqlLogs
docker run -p 3306:3306 --name mysql56 -v $PWD/sqlConf:/etc/mysql/conf.d -v $PWD/sqlLogs:/logs -v $PWD/sqlData:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d mysql:5.6
```
> 此处还需要修改一部分源码，跳过一个webase启动前的检查。 
> 
> 将`webase-deploy/comm/check.py`中第47行`checkExitedChainInfo()`注释掉

### 修改配置文件

根据前文对区块链和mysql的配置，在这里修改`common.properties`文件
```python
# Mysql database configuration of WeBASE-Node-Manager
mysql.ip=127.0.0.1
mysql.port=3306
mysql.user=root
mysql.password=123456
mysql.database=webasenodemanager

# Mysql database configuration of WeBASE-Sign
sign.mysql.ip=127.0.0.1
sign.mysql.port=3306
sign.mysql.user=root
sign.mysql.password=123456
sign.mysql.database=webasesign

# 由于与IPFS端口冲突，所以将5001-5004改为5100-5104
# WeBASE-Web service port 
web.port=5100

# WeBASE-Node-Manager service port
mgr.port=5101

# WeBASE-Front service port
front.port=5102

# WeBASE-Sign service port
sign.port=5104

# 由于使用已有区块链节点，故设置一下参数
if.exist.fisco=yes
# fisco 的文件夹路径，根据自己情况设置
fisco.dir=~/fisco/nodes/127.0.0.1
```
### Usage
```shell
# 安装所有组件
python deploy.py installAll
# 由于我们在上面设置了fisco 路径， 所以startAll与stopAll可以帮助我们启动和关闭区块链
# 启动WeBASE和节点，
python deploy.py startAll
# 关闭WeBASE和区块链
python deploy.py stopAll
```

## Hyperledger Caliper 压力测试工具
Caliper是Hyperledger旗下做基准测试benchmark的工具，能够给Hyperledger Fabric, Ethereum, FISCO BCOS 进行性能测试

### 准备工作
```shell
mkdir benchmarks && cd benchmarks
nvm use 8
nvm init 
npm install --only=prod @hyperledger/caliper-cli@0.2.0
npx caliper bind --caliper-bind-sut fisco-bcos --caliper-bind-sdk latest
git clone https://github.com/vita-dounai/caliper-benchmarks.git
```
### 修改源码
caliper对于FISCO BCOS的支持还存在一些问题，有些代码没有合并，所以需要我们手动修改源码
1. 根据[PR #647](https://github.com/hyperledger/caliper/pull/647/files)，修改以下三个文件
    * `benchmarks/node_modules/@hyperledger/caliper-fisco-bcos/lib/channelPromise.js`
    * `benchmarks/node_modules/@hyperledger/caliper-fisco-bcos/lib/fiscoBcos.js `
    * `benchmarks/node_modules/@hyperledger/caliper-fisco-bcos/lib/web3lib/web3sync.js`
2. 根据[PR #677](https://github.com/hyperledger/caliper/pull/677/files),修改文件
    * `benchmarks/node_modules/@hyperledger/caliper-fisco-bcos/lib/fiscoBcos.js `
3. 根据[issue #1721](https://github.com/FISCO-BCOS/FISCO-BCOS/issues/1721),
    在`benchmarks/node_modules/@hyperledger/caliper-fisco-bcos/package.json`文件中的`dependencies`添加一项`"secp256k1": "^3.8.0"`,随后执行`npm -i`安装

### 修改配置文件

```json
{
    "caliper": {
        "blockchain": "fisco-bcos"
    },
    "fisco-bcos": {
        "config": {
            //  文件会自带，这里没必要修改。之前还傻乎乎的自己重新生成了新账户 
            "privateKey": "${accountPrivkey}", 
            "account": "${newAccount}"
        },
        // 此处填写所有节点的IP和端口信息
        "network": {
            "nodes": [
                {
                    "ip": "127.0.0.1",
                    "rpcPort": "8545",
                    "channelPort": "20200"
                },
                {
                    "ip": "127.0.0.1",
                    "rpcPort": "8546",
                    "channelPort": "20201"
                },
                {
                    "ip": "127.0.0.1",
                    "rpcPort": "8547",
                    "channelPort": "20202"
                },
                {
                    "ip": "127.0.0.1",
                    "rpcPort": "8548",
                    "channelPort": "20203"
                }
            ],
            // 这里很坑，推荐使用完整路径，
            "authentication": {
                "key": "/home/ubuntu/fisco/nodes/127.0.0.1/sdk/node.key",
                "cert": "/home/ubuntu/fisco/nodes/127.0.0.1/sdk/node.crt",
                "ca": "/home/ubuntu/fisco/nodes/127.0.0.1/sdk/ca.crt"
            },
            "groupID": 1,
            "timeout": 100000
        }
    }{以下省略}
}
```
> 注1：如果自己使用python sdk 生成了新账户，返回的新地址和账户都带`0x`前缀，填入时，记得地址无需改动,但是私钥要删除`0x`前缀

> 注2： `node.key` 以及`node.crt` 文件不在 `nodes/127.0.0.1/sdk` 文件夹中，可以选择 
> 1. 将某个节点的这两个文件复制到sdk文件夹，
> 2. 使用sdk中的`sdk.key`, `sdk.crt`文件

**简单测试**
```shell
npx caliper benchmark run --caliper-workspace caliper-benchmarks --caliper-benchconfig benchmarks/samples/fisco-bcos/helloworld/config.yaml  --caliper-networkconfig networks/fisco-bcos/4nodes1group/fisco-bcos.json
```

### 例子
这里根据前面helloworld，写一个如何增加节点测试tps的例子
> 1. Caliper只能读取docker启动的fisco 节点在压力测试时的cpu 内存占用信息，对于进程启动的fisco，只能返回tps。如果不在意cpu，内存占用，可以直接使用进程启动，否则还是推荐docker启动。
> 2. 压力测试会发送大量交易，可能会导致WeBASE后台出现bug，无法读取交易细节。 压力测试时推荐新建区块链系统进行测试，测试完成后删除


<!-- **启动区块链**
```
bash build_chain.sh -d -l 127.0.0.1:4 -p 30300, 20200, 8545
``` -->

**参数设置**
在benchmarks文件夹下：
```shell
# 创建新文件夹，内保存新区块链信息和benchmark相关参数设置
mkdir test && cd test
# 复制建链脚本
cp ~/fiso/build_chain.sh .
# 此文件见保存benchmark参数
#创建一个4节点区块链, 输出到4nodes文件夹下, 
bash build_chain.sh -d -l 127.0.0.1:4 -p 30300,20200,8545 -o 4nodes
cd 4nodes
mkdir benchmarks && cd benchmarks
# 复制config.yaml, get.js set.js 到此处
cp ~/benchmarks/caliper-benchmarks/benchmarks/samples/fisco-bcos/helloworld/* .
# 复制记录网络参数的json到此处
cp ~/benchmarks/caliper-benchmarks/fisco-bcos/4nodes1group/fisco-bcos.json ./network.json
```

**修改config.yaml**
```yaml
test:
  # name 和description会出现在最终报告文件中，根据自己情况修改
  name: Hello World
  description: This is a helloworld benchmark of FISCO BCOS for caliper
  clients:
    type: local
    number: 1
  rounds:
  - label: get
    description: Test performance of getting name
    txNumber:
    # 发送总交易数
    - 10000
    rateControl:
    - type: fixed-rate
      opts:
      # tps设置
        tps: 1000
    # 调用路径，这里修改为当前路径
    callback: benchmarks/4nodes/benchmarks/get.js
  - label: set
    description: Test performance of setting name
    txNumber:
    - 10000
    rateControl:
    - type: fixed-rate
      opts:
        tps: 1000
    # 修改路径
    callback: benchmarks/4nodes/benchmarks/set.js
monitor:
  type:
  # 节点启动方式，docker启动和process进程启动，这里使用docker启动
  - docker
  - process
  # 填写docker 容器名
  docker:
    name:
    - node0
    - node1
    - node2
    - node3
  process:
    - command: node
      arguments: fiscoBcosClientWorker.js
      multiOutput: avg
  interval: 0.5

```



