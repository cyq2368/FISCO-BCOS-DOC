# 控制台(console)

标签：``console`` ``控制台`` ``命令行交互工具``

---------

```eval_rst
.. important::
    - ``控制台`` 只支持FISCO BCOS 3.0+版本，基于 `Java SDK <../sdk/java_sdk/index.html>`_ 实现。
    - 可通过命令 ``./start.sh --version`` 查看当前控制台版本
```

[控制台](https://github.com/FISCO-BCOS/console)是FISCO BCOS 3.0重要的交互式客户端工具，它通过[Java SDK](../sdk/java_sdk/index.md)与区块链节点建立连接，实现对区块链节点数据的读写访问请求。控制台拥有丰富的命令，包括查询区块链状态、管理区块链节点、部署并调用合约等。此外，控制台提供一个合约编译工具，用户可以方便快捷的将Solidity和Liquid合约文件编译为Java合约文件。

Liquid编译请参考：FIXME:

## 1. 控制台配置与运行

```eval_rst
.. important::
    前置条件：搭建FISCO BCOS区块链，请参考 `搭建第一个区块链网络 <../installation.html>`_
    建链工具参考：`开发部署工具 <../manual/build_chain.html>`_ 或 `运维部署工具 <../enterprise_tools/index.html>`_。
```

### 1.1 获取控制台

FIXME: 更新控制台URL

```shell
cd ~ && mkdir -p fisco && cd fisco
# 获取控制台
curl -#LO https://github.com/FISCO-BCOS/console/releases/download/v3.0.0/download_console.sh && bash download_console.sh
```

```eval_rst
.. note::
    - 如果因为网络问题导致长时间无法下载，请尝试 `curl -#LO https://gitee.com/FISCO-BCOS/console/raw/master/tools/download_console.sh && bash download_console.sh`
```

目录结构如下：

```shell
│── apps # 控制台jar包目录
│   └── console.jar
├── lib # 相关依赖的jar包目录
├── conf
│   ├── clog.ini    # c sdk日志配置文件
│   ├── config-example.toml # 配置文件
│   └── log4j.properties # 日志配置文件
├── contracts # 合约所在目录
│   ├── console # 控制台部署合约时编译的合约abi, bin，java文件目录
│   ├── sdk     # sol2java.sh脚本编译的合约abi, bin，java文件目录
│   ├── liquid  # Liquid合约存放目录
│   └── solidity    # solidity合约存放目录
│       └── HelloWorld.sol # 普通合约：HelloWorld合约，可部署和调用
│       └── KVTableTest.sol # 使用KV存储接口的合约：KVTableTest合约，可部署和调用
│       └── KVTable.sol # 提供KV存储操作的接口合约    
│-- start.sh # 控制台启动脚本
│-- get_account.sh # 账户生成脚本
│-- get_gm_account.sh # 账户生成脚本，国密版
│-- contract2java.sh # Solidity/Liquid合约文件编译为java合约文件的开发工具脚本
```

**注意：默认下载的控制台内置`0.6.10`版本的`solidity`编译器，用户需要编译`0.5`或者`0.6`版本的合约时，可以通过下列命令获取内置对应编译器版本的控制台**

```shell
# 0.4.25
curl -#LO https://github.com/FISCO-BCOS/console/releases/download/v3.0.0/download_console.sh && bash download_console.sh -v 0.4
# 0.5
curl -#LO https://github.com/FISCO-BCOS/console/releases/download/v3.0.0/download_console.sh && bash download_console.sh -v 0.5
```

```eval_rst
.. note::
    - 如果因为网络问题导致长时间无法下载，0.5版本请尝试命令： `curl -#LO https://gitee.com/FISCO-BCOS/console/raw/v2.7.1/tools/download_console.sh && bash download_console.sh -v 0.5` 
    0.6版本请尝试命令： `curl -#LO https://gitee.com/FISCO-BCOS/console/raw/v2.7.1/tools/download_console.sh && bash download_console.sh -v 0.6`

```

### 1.2 配置控制台

- 区块链节点和证书的配置：
  - 将节点sdk目录下的`ca.crt`、`sdk.crt`和`sdk.key`文件拷贝到`conf`目录下。
  - 将`conf`目录下的`config-example.toml`文件重命名为`config.toml`文件。配置`config.toml`文件，其中添加注释的内容根据区块链节点配置做相应修改。

配置示例文件如下：

```toml
[cryptoMaterial]

certPath = "conf"                           # The certification path  
useSMCrypto = "false"                       # RPC SM crypto type

# The following configurations take the certPath by default if commented
# caCert = "conf/ca.crt"                    # CA cert file path
# sslCert = "conf/sdk.crt"                  # SSL cert file path
# sslKey = "conf/sdk.key"                   # SSL key file path

# The following configurations take the sm certPath by default if commented
# caCert = "conf/sm_ca.crt"                    # SM CA cert file path
# sslCert = "conf/sm_sdk.crt"                  # SM SSL cert file path
# sslKey = "conf/sm_sdk.key"                    # SM SSL key file path
# enSslCert = "conf/sm_ensdk.crt"               # SM encryption cert file path
# enSslKey = "conf/sm_ensdk.key"                # SM ssl cert file path

[network]
messageTimeout = "10000"
defaultGroup="group"                            # Console default group to connect
peers=["127.0.0.1:20200", "127.0.0.1:20201"]    # The peer list to connect

[account]
authCheck = "false"
keyStoreDir = "account"         # The directory to load/store the account file, default is "account"
# accountFilePath = ""          # The account file path (default load from the path specified by the keyStoreDir)
accountFileFormat = "pem"       # The storage format of account file (Default is "pem", "p12" as an option)

# accountAddress = ""           # The transactions sending account address
                                # Default is a randomly generated account
                                # The randomly generated account is stored in the path specified by the keyStoreDir

# password = ""                 # The password used to load the account file

[threadPool]
# threadPoolSize = "16"         # The size of the thread pool to process message callback
                                            # Default is the number of cpu cores
```

配置项详细说明[参考这里](../sdk/java_sdk/configuration.md)。

```eval_rst
.. important::

    控制台说明
    
    - 当控制台配置文件在一个群组内配置多个节点连接时，由于群组内的某些节点在操作过程中可能退出群组，因此控制台轮询节点查询时，其返回信息可能不一致，属于正常现象。建议使用控制台时，配置一个节点或者保证配置的节点始终在群组中，这样在同步时间内查询的群组内信息保持一致。
```

#### 1.2.1 Java合约生成工具

控制台提供一个专门的生成Java合约工具，方便开发者将Solidity和Liquid合约文件编译为Java合约文件。

当前合约生成工具支持Solidity的自动编译并生成Java文件、支持指定Liquid编译后的WASM文件以及ABI文件生成Java文件。

**Solidity合约使用**

```shell
$ bash contract2java.sh solidity -h 
usage: contract2java.sh <solidity|liquid> [OPTIONS...]
 -h,--help
 -l,--libraries <arg>   [Optional] Set library address information built
                        into the solidity contract
                        eg:
                        --libraries lib1:lib1_address lib2:lib2_address
 -o,--output <arg>      [Optional] The file path of the generated java
                        code, default is contracts/sdk/java/
 -p,--package <arg>     [Optional] The package name of the generated java
                        code, default is com
 -s,--sol <arg>         [Optional] The solidity file path or the solidity
                        directory path, default is contracts/solidity/
```

参数详细：
- `package`: 生成`Java`文件的包名。
- `sol`: (可选)`solidity`文件的路径，支持文件路径和目录路径两种方式，参数为目录时将目录下所有的`solidity`文件进行编译转换。默认目录为`contracts/solidity`。
- `output`: (可选)生成`Java`文件的目录，默认生成在`contracts/sdk/java`目录。 

**Liquid合约使用**

```shell
$ bash contract2java.sh liquid -h
usage: contract2java.sh <solidity|liquid> [OPTIONS...]
 -a,--abi <arg>       [Required] The ABI file path of Liquid contract.
 -b,--bin <arg>       [Required] The binary file path of Liquid contract.
 -h,--help
 -o,--output <arg>    [Optional] The file path of the generated java code,
                      default is contracts/sdk/java/
 -p,--package <arg>   [Optional] The package name of the generated java
                      code, default is com
 -s,--sm-bin <arg>    [Required] The SM binary file path of Liquid
                      contract.
```

参数详细：

- `abi `：（必选）Liquid合约`ABI`文件的路径，在使用`cargo liquid build`命令之后生成在target文件夹中。
- `bin`：（必选）Liquid合约`wasm bin`文件的路径，在使用`cargo liquid build`命令之后生成在target文件夹中。
- `package`：（可选）生成`Java`文件的包名，默认为`org`。
- `sm-bin`：（必选）Liquid合约`wasm sm bin`文件的路径，在使用`cargo liquid build -g`命令之后生成在target文件夹中。

**使用**

```shell
$ cd ~/fisco/console

# 生成Solidity合约的Java代码
$ bash contract2java.sh solidity -p org.com.fisco

# 生成Liquid合约的Java代码
$ bash contract2java.sh liquid -p org.com.fisco -b ./contracts/liquid/asset_test/asset_test.wasm -a ./contracts/liquid/asset_test/asset_test.abi -s ./contracts/liquid/asset_test/asset_test_sm.wasm 
```

运行成功之后，将会在`console/contracts/sdk`目录生成java、abi和bin目录，如下所示。

```shell
|-- abi # 编译生成的abi目录，存放solidity合约编译的abi文件
|   |-- HelloWorld.abi
|   |-- KVTable.abi
|   |-- KVTableTest.abi
|-- bin # 编译生成的bin目录，存放solidity合约编译的bin文件
|   |-- HelloWorld.bin
|   |-- KVTable.bin
|   |-- KVTableTest.bin
|-- java  # 存放编译的包路径及Java合约文件
|   |-- org
|       |-- com
|           |-- fisco
|               |-- HelloWorld.java # Solidity编译的HelloWorld Java文件
|               |-- KVTable.java    # Solidity编译的KV存储接口合约 Java文件
|               |-- KVTableTest.java  # Solidity编译的KVTableTest Java文件
|               |-- AssetTest.java  # Liquid生成的AssetTest文件
```

Java目录下生成了`org/com/fisco/`包路径目录。包路径目录下将会生成Java合约文件`HelloWorld.java`、`KVTableTest.java`、`KVTable.java`和`AssetTest.java`。其中`HelloWorld.java`、`KVTableTest.java`和`AssetTest.java`是Java应用所需要的Java合约文件。

### 1.3. 启动控制台

在节点正在运行的情况下，启动控制台：

```shell
$ ./start.sh
# 输出下述信息表明启动成功
=====================================================================================
Welcome to FISCO BCOS console(3.0.0)!
Type 'help' or 'h' for help. Type 'quit' or 'q' to quit console.
 ________ ______  ______   ______   ______       _______   ______   ______   ______
|        |      \/      \ /      \ /      \     |       \ /      \ /      \ /      \
| $$$$$$$$\$$$$$|  $$$$$$|  $$$$$$|  $$$$$$\    | $$$$$$$|  $$$$$$|  $$$$$$|  $$$$$$\
| $$__     | $$ | $$___\$| $$   \$| $$  | $$    | $$__/ $| $$   \$| $$  | $| $$___\$$
| $$  \    | $$  \$$    \| $$     | $$  | $$    | $$    $| $$     | $$  | $$\$$    \
| $$$$$    | $$  _\$$$$$$| $$   __| $$  | $$    | $$$$$$$| $$   __| $$  | $$_\$$$$$$\
| $$      _| $$_|  \__| $| $$__/  | $$__/ $$    | $$__/ $| $$__/  | $$__/ $|  \__| $$
| $$     |   $$ \\$$    $$\$$    $$\$$    $$    | $$    $$\$$    $$\$$    $$\$$    $$
 \$$      \$$$$$$ \$$$$$$  \$$$$$$  \$$$$$$      \$$$$$$$  \$$$$$$  \$$$$$$  \$$$$$$

=====================================================================================
```

### 1.4 启动脚本说明

#### 1.4.1 查看当前控制台版本：

```shell
./start.sh --version
console version: 3.0.0
```

#### 1.4.2 账户使用方式

##### 1.4.2.1 控制台加载私钥

FIXME: 账户管理文档

控制台提供账户生成脚本get_account.sh(脚本用法请参考[账户管理文档](../manual/account.md)，生成的的账户文件在accounts目录下，控制台加载的账户文件必须放置在该目录下。
控制台启动方式有如下几种：

```shell
./start.sh
./start.sh group
./start.sh group -pem pemName
./start.sh group -p12 p12Name
```

##### 1.4.2.2 默认启动

使用控制台配置文件指定的默认群组号启动。

```shell
./start.sh
```

**注意**: 控制台启动未指定私钥账户时，会尝试从`account`目录下加载一个可用的私钥账户用于发送交易，加载失败则会创建一个新的`PEM`格式的账户文件，将其保存在`account`目录下。

##### 1.4.2.3 指定群组名启动

使用命令行指定的群组名启动。

```shell
./start.sh group2
```

##### 1.4.2.4 使用PEM格式私钥文件启动

- 使用指定的pem文件的账户启动，输入参数：群组号、-pem、pem文件路径

```shell
./start.sh group -pem accounts/0xebb824a1122e587b17701ed2e512d8638dfb9c88.pem
```

##### 1.4.2.5 使用PKCS12格式私钥文件启动

- 使用指定的p12文件的账户，需要输入密码，输入参数：群组号、-p12、p12文件路径

```shell
./start.sh group -p12 accounts/0x5ef4df1b156bc9f077ee992a283c2dbb0bf045c0.p12
Enter Export Password:
```

**注意：**
控制台启动时加载p12文件出现下面报错：

```shell
exception unwrapping private key - java.security.InvalidKeyException: Illegal key size
```

可能是Java版本的原因，参考解决方案：[https://stackoverflow.com/questions/3862800/invalidkeyexception-illegal-key-size](https://stackoverflow.com/questions/3862800/invalidkeyexception-illegal-key-size)

### 1.5 控制台命令结构

控制台命令由两部分组成，即指令和指令相关的参数：
- **指令**: 指令是执行的操作命令，包括查询区块链相关信息，部署合约和调用合约的指令等，其中部分指令调用JSON-RPC接口，因此与JSON-RPC接口同名。
**使用提示： 指令可以使用tab键补全，并且支持按上下键显示历史输入指令。**

- **指令相关的参数**: 指令调用接口需要的参数，指令与参数以及参数与参数之间均用空格分隔，与JSON-RPC接口同名命令的输入参数和获取信息字段的详细解释参考[JSON-RPC API](../api.md)。

### 1.6 常用命令

#### 合约相关命令

- 利用[CNS](../design/features/cns_contract_name_service.md)部署和调用合约(**推荐**)
  - 部署合约: [deployByCNS](./console.html#deploybycns)
  - 调用合约: [callByCNS](./console.html#callbycns)
  - 查询CNS部署合约信息: [queryCNS](./console.html#querycns)
- 普通部署和调用合约
  - 部署合约: [deploy](./console.html#deploy)
  - 调用合约: [call](./console.html#call)

#### 其他命令

- 查询区块高度：[getBlockNumber](./console.html#getblocknumber)
- 查询共识节点列表：[getSealerList](./console.html#getsealerlist)
- 查询交易回执信息: [getTransactionReceipt](./console.html#gettransactionreceipt)
- 切换群组: [switch](./console.html#switch)

### 快捷键

- `Ctrl+A`：光标移动到行首
- `Ctrl+D`：退出控制台
- `Ctrl+E`：光标移动到行尾
- `Ctrl+R`：搜索输入的历史命令
- &uarr;：向前浏览历史命令
- &darr;：向后浏览历史命令

### 控制台响应

当发起一个控制台命令时，控制台会获取命令执行的结果，并且在终端展示执行结果，执行结果分为2类：

- **正确结果:** 命令返回正确的执行结果，以字符串或是json的形式返回。
- **错误结果:** 命令返回错误的执行结果，以字符串或是json的形式返回。
  - 控制台的命令调用JSON-RPC接口时，错误码[参考这里](../api.html#rpc)。
  - 控制台的命令调用Precompiled Service接口时，错误码[参考这里](../api.html#precompiled-service-api)。

## 2. 控制台命令列表

控制台命令包含

### 2.1 控制台基础命令

#### 2.1.1 help

输入help或者h，查看控制台所有的命令。

```shell
[group]: /> help
* help([-h, -help, --h, --H, --help, -H, h])  Provide help information
* addObserver                               Add an observer node
* addSealer                                 Add a sealer node
* call                                      Call a contract by a function and parameters
* callByCNS                                 Call a contract by a function and parameters by CNS
* cd                                        Change dir to given path.
* clearNodeName                             Clear default node name to empty.
* deploy                                    Deploy a contract on blockchain
* deployByCNS                               Deploy a contract on blockchain by CNS
* quit([quit, q, exit])                     Quit console
* getBlockByHash                            Query information about a block by hash
* getBlockByNumber                          Query information about a block by number
* getBlockHeaderByHash                      Query information about a block header by hash
* getBlockHeaderByNumber                    Query information about a block header by block number
* getBlockNumber                            Query the number of most recent block
* getCode                                   Query code at a given address
* getConsensusStatus                        Query consensus status
* getCurrentAccount                         Get the current account info
* getDeployLog                              Query the log of deployed contracts
* getGroupInfo                              Query the current group information.
* getGroupInfoList                          Get all groups info
* getGroupList                              List all group list
* getGroupNodeInfo                          Get group node info
* getGroupPeers                             List all group peers
* getNodeName                               Get default node name in this client.
* getObserverList                           Query nodeId list for observer nodes.
* getPbftView                               Query the pbft view of node
* getPeers                                  Query peers currently connected to the client
* getPendingTxSize                          Query pending transactions size
* getSealerList                             Query nodeId list for sealer nodes
* getSyncStatus                             Query sync status
* getSystemConfigByKey                      Query a system config value by key
* getTotalTransactionCount                  Query total transaction count
* getTransactionByHash                      Query information about a transaction requested by transaction hash
* getTransactionByHashWithProof             Query the transaction and transaction proof by transaction hash
* getTransactionReceipt                     Query the receipt of a transaction by transaction hash
* getTransactionReceiptByHashWithProof      Query the receipt and transaction receipt proof by transaction hash
* listAbi                                   List functions and events info of the contract.
* listAccount                               List the current saved account list
* listDeployContractAddress                 List the contractAddress for the specified contract
* loadAccount                               Load account for the transaction signature
* ls                                        List resources in given path.
* mkdir                                     Create dir in given path.
* newAccount                                Create account
* pwd                                       Show absolute path of working directory name
* queryCNS                                  Query CNS information by contract name and contract version
* registerCNS                               RegisterCNS information for the given contract
* removeNode                                Remove a node
* switch([s])                               Switch to a specific group by name
* setConsensusWeight                        Set consensus weight for the specified node
* setNodeName                               Set default node name to send request.
* setSystemConfigByKey                      Set a system config value by key
---------------------------------------------------------------------------------------------
```

**注：**

- help显示每条命令的含义是：命令 命令功能描述
- 查看具体命令的使用介绍说明，输入命令 -h或\--help查看。例如：

```shell
[group]: /> getBlockByNumber -h
Query information about a block by block number.
Usage: 
getBlockByNumber blockNumber [boolean]
* blockNumber -- Integer of a block number, from 0 to 2147483647.
* boolean -- (optional) If true it returns the full transaction objects, if false only the hashes of the transactions.
```

#### 2.1.2 switch

运行switch或者s，切换到指定群组。群组号显示在命令提示符前面。

```shell
[group]: />  switch group2
Switched to group2.

[group2]>
```

#### 2.1.3 quit

运行quit、q或exit，退出控制台。

```shell
quit
```

### 2.2 账户相关命令

#### 2.2.1 newAccount

创建新的发送交易的账户，默认会以`PEM`格式将账户保存在`account`目录下。

```shell
# 控制台连接非国密区块链时，账户文件自动保存在`account/ecdsa`目录
# 控制台连接国密区块链时，账户文件自动保存在`accout/gm`目录下
[group]: />  newAccount
AccountPath: account/ecdsa/0x6fad87071f790c3234108f41b76bb99874a6d813.pem
newAccount: 0x6fad87071f790c3234108f41b76bb99874a6d813
AccountType: ecdsa

$ ls -al account/ecdsa/0x6fad87071f790c3234108f41b76bb99874a6d813.pem
$ -rw-r--r--  1 octopus  staff  258  9 30 16:34 account/ecdsa/0x6fad87071f790c3234108f41b76bb99874a6d813.pem
```

#### 2.2.2 loadAccount

加载`PEM`或者`P12`格式的私钥文件，加载的私钥可以用于发送交易签名。
参数：

- 私钥文件路径: 支持相对路径、绝对路径和默认路径三种方式。用户账户地址时，默认从`config.toml`的账户配置选项`keyStoreDir`加载账户，`keyStoreDir`配置项请参考[这里](../sdk/java_sdk/configuration.html#id9)。

- 账户格式: 可选，加载的账户文件类型，支持`pem`与`p12`，默认为`pem`。

```shell
[group]: />  loadAccount 0x6fad87071f790c3234108f41b76bb99874a6d813
Load account 0x6fad87071f790c3234108f41b76bb99874a6d813 success!
```

#### 2.2.3 listAccount

查看当前加载的所有账户信息

```shell
[group]: />  listAccount
0x6fad87071f790c3234108f41b76bb99874a6d813(current account) <=
0x726d9f31cf44debf80b08a7e759fa98b360b0736
```

**注意：带有`<=`后缀标记的为当前用于发送交易的私钥账户，可以使用`loadAccount`进行切换**

#### 2.2.4 getCurrentAccount

获取当前账户地址。

```shell
[group]: />  getCurrentAccount
0x6fad87071f790c3234108f41b76bb99874a6d813
```

### 2.3 区块链状态查询命令

#### 2.3.1 getBlockNumber

运行getBlockNumber，查看区块高度。

```shell
[group]: />  getBlockNumber
90
```

#### 2.3.2 getSyncStatus

运行getSyncStatus，查看同步状态。

```shell
[group]: /> getSyncStatus 
SyncStatusInfo{
    isSyncing='false',
    protocolId='null',
    genesisHash='e1cedcd9d09d47a6ffac4e621b7d852e84363c20151d0ee2df63837ed23318d9',
    nodeId='44c3c0d914d7a3818923f9f45927724bddeeb25df92b93f1242c32b63f726935d6742b51cd40d2c828b52ed6cde94f4d6fb4b3bfdc0689cfcddf7425eafdae85',
    blockNumber='6',
    latestHash='ac0e663c2e99dd733553afe66c41ebf03b1ab2a48a40d72d700705ea860c8290',
    knownHighestNumber='6',
    txPoolSize='null',
    peers=[
        PeersInfo{
            nodeId='bb21228b0762433ea6e4cb185e1c54aeb83cd964ec0e831f8732cb2522795bb569d58215dfbeb7d3fc474fdce33dc9a793d4f0e86ce69834eddc707b48915824',
            genesisHash='e1cedcd9d09d47a6ffac4e621b7d852e84363c20151d0ee2df63837ed23318d9',
            blockNumber='6',
            latestHash='ac0e663c2e99dd733553afe66c41ebf03b1ab2a48a40d72d700705ea860c8290'
        },
        PeersInfo{
            nodeId='c1de42fc9e6798142fdbeddc05018b548b848155a8527f0ffc75eb93d0ae51ebd8074c86b6bdc0f4161dcad7cab9455a4eebf146ac5b08cb23c33c8eef756b7c',
            genesisHash='e1cedcd9d09d47a6ffac4e621b7d852e84363c20151d0ee2df63837ed23318d9',
            blockNumber='6',
            latestHash='ac0e663c2e99dd733553afe66c41ebf03b1ab2a48a40d72d700705ea860c8290'
        },
        PeersInfo{
            nodeId='f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366',
            genesisHash='e1cedcd9d09d47a6ffac4e621b7d852e84363c20151d0ee2df63837ed23318d9',
            blockNumber='6',
            latestHash='ac0e663c2e99dd733553afe66c41ebf03b1ab2a48a40d72d700705ea860c8290'
        }
    ],
    knownLatestHash='ac0e663c2e99dd733553afe66c41ebf03b1ab2a48a40d72d700705ea860c8290'
}
```

#### 2.3.3 getPeers

运行getPeers，查看节点的peers。

```shell
[group]: /> getPeers 
PeersInfo{
    p2pNodeID='3082010a0282010100ced891dad17f04738a2da965264e1387c707b351a0a4e360341066bb2893635fd4e2717ba203aed68d2733a633ecc097bd70fd0c34f31cf82c1772c60a2b252f5498d41a6b7c7d26bad930e38dacee2889fef0e7311f76d4c098c80c8096dc8842d95135e27523aa3cfb1910eb2021bfe8c82f73508e419427f79508ca0d5693e1a913a3749d0496c6fcfc43b69eca7d3bd14a9d6e079737af7cf9ad0e78a40ddb6e3081ef88c9928b3864141933d360dada8a197b45cfebb475efba05387cbf54bfb8357681bdc27c1b48b7ab460dc01b1cb8d5ff8256ad36f1e8c1e4dce63d8758bb08a9f9829e366ef5233997b5ff46ec064e46136a8770addc927bbbc81f0203010001',
    endPoint='0.0.0.0:30300',
    groupNodeIDInfo=[
        NodeIDInfo{
            group='group',
            nodeIDList=[
                44c3c0d914d7a3818923f9f45927724bddeeb25df92b93f1242c32b63f726935d6742b51cd40d2c828b52ed6cde94f4d6fb4b3bfdc0689cfcddf7425eafdae85
            ]
        }
    ],
    peers=[
        PeerInfo{
            p2pNodeID='3082010a0282010100d1f4618de0e7a6867e04e28749f688c02c763e570bebe9d9b4f1dd02446eb970f7eeb2926ff5dce55c3ef8d2736b7edee54e1a171a44b555f129ed9f9658b442c98b5837f2651717e2420720ac98d74177abfbe772c6a478600b29aa370423934111e0d03b2ccadf59a5f7978383b3f672ad5ef122a4df600d40ebed1ae226ac1dc727619baf5658d7866876e06b482857fa109b99e159688ba2a53d3e57c055f55e35613fbfe0e1058e8ce161ed740ee4ea03c7d933a171ef4da8238027c4bca966c85d43d04dfd186bae563c1d6790335c0f96daa9d33272598ab1e2a60cfb22e7cf70bc10ed0a0757d0ef0add741cb8604d8707a8fd80d21f2786f9cdb3670203010001',
            endPoint='127.0.0.1:55686',
            groupNodeIDInfo=[
                NodeIDInfo{
                    group='group',
                    nodeIDList=[
                        f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366
                    ]
                }
            ]
        },
        PeerInfo{
            p2pNodeID='3082010a0282010100d686574a794a3e27aac75eab7f84b3b46cc2bd19afc088bf6eb5702d2faf09116b7c374646a06793c71aa62ea43de6dda7f1d2711da81b1e289132d390b75c501d37aeee5a55d081d4c32b6c53e103547876d19f2f8fe8ca336bbd4a74ee6101861832380e65f6638fc7c52451993fe74084c36692c82c6ad6f958dbbe9d6a9e4a43753c6c5678830c2fc293fb1b2f1ac58484f9088e1ac70d8074ca64a6a6d68b09846f7437453ff0a9e27f4ba39442186d1a5cef1391f627e8f2bdc963c72ad31ee3a1a970db553fdd0a9ed6e0de365479deff5a086417e64ff4b9da29fd1debcdddcd0abb6928bc5c64ac40f1c83690696f1304104718aa4b31fcd06bf2150203010001',
            endPoint='127.0.0.1:55690',
            groupNodeIDInfo=[
                NodeIDInfo{
                    group='group',
                    nodeIDList=[
                        c1de42fc9e6798142fdbeddc05018b548b848155a8527f0ffc75eb93d0ae51ebd8074c86b6bdc0f4161dcad7cab9455a4eebf146ac5b08cb23c33c8eef756b7c
                    ]
                }
            ]
        },
        PeerInfo{
            p2pNodeID='3082010a0282010100c0944e4235c287adb423816074286057a6d07fd03cc17e106bbffda1061d9872a2446378ff802e67eae602db415a2fca82569a554150c7ed70db0438076e545856f7d48356d2ab9cb2c9e3ec1cf3150fef31568056c19f92e568648346226c27b7f5c211a8557ef62ffe17ee55462d88d0f634a61490f4b86f7a4cd6568107bc8a08d2536abbee07c490f477b1c2f680db616663d386f5cdca93edbc895ef899b52518b7cf599001e5d9c7466e289a1a457a6514c479bb9287317a14cf171fc53874b4e368a737f920c1e4d26b835cc6503bb197ab1894db0ecc0383f2230e308a528aa1dae39cc00a2768f636c0573a91112f64550df823fd26e86d953701030203010001',
            endPoint='127.0.0.1:55694',
            groupNodeIDInfo=[
                NodeIDInfo{
                    group='group',
                    nodeIDList=[
                        bb21228b0762433ea6e4cb185e1c54aeb83cd964ec0e831f8732cb2522795bb569d58215dfbeb7d3fc474fdce33dc9a793d4f0e86ce69834eddc707b48915824
                    ]
                }
            ]
        }
    ]
}
```

#### 2.3.4 getBlockHeaderByHash

运行getBlockHeaderByHash，根据区块哈希查询区块头信息。
参数：

- 区块哈希：0x开头的区块哈希值

```shell
[group]: /> getBlockHeaderByHash 0xfde78118e4b387b5c7ba74c19ddbd59992c5956145c5cc86265ee43814c2a320
{
    transactions=null,
    number='1',
    hash='0xfde78118e4b387b5c7ba74c19ddbd59992c5956145c5cc86265ee43814c2a320',
    logsBloom='null',
    transactionsRoot='0xf09ac805845ab8a2f63367d89804fc8e438c1f0988fc13edb2f053adbe6764ec',
    receiptRoot='0x169c46acddead6923067c057172dff458ac3f9e56008bcf2a138d605797e75c8',
    stateRoot='0x6fc1ca9cc8c937ff208aac3f9d262389fb33a16d5f9ca0027ab596cd65667aff',
    sealer='2',
    sealerList=[
        0x44c3c0d914d7a3818923f9f45927724bddeeb25df92b93f1242c32b63f726935d6742b51cd40d2c828b52ed6cde94f4d6fb4b3bfdc0689cfcddf7425eafdae85,
        0xbb21228b0762433ea6e4cb185e1c54aeb83cd964ec0e831f8732cb2522795bb569d58215dfbeb7d3fc474fdce33dc9a793d4f0e86ce69834eddc707b48915824,
        0xc1de42fc9e6798142fdbeddc05018b548b848155a8527f0ffc75eb93d0ae51ebd8074c86b6bdc0f4161dcad7cab9455a4eebf146ac5b08cb23c33c8eef756b7c,
        0xf39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366
    ],
    extraData=0x,
    gasUsed='3',
    timestamp='1638624581393',
    signatureList=[
        {
            index='0',
            signature='0x341ce6b7a7724eb432a914a4e7a154343a3116d457e05329725299f144cf430454a38e64d969c454fe7f1e864ca13a6a7e5d1a01b42bb1afc152471bb9ee3ea401'
        },
        {
            index='2',
            signature='0x2787c0be632b20abfe7f6f1cbf031eee0aa57b6e5e2cc242f3e19bafc455f847450df7acb897958e0ef5b54e1dde1b084127933bf8736b09621b7376d273d5da01'
        },
        {
            index='3',
            signature='0xcc04713e6c6dbbe8c204e9c33493dcbbd94731c56afaa4cd1fd22d14b5bd9a4a53b48d907956743d48c22fe352f361d9d48b7f9ccbb8c7e054b1c5db1bdab40400'
        }
    ]
}
```

#### 2.3.5 getBlockHeaderByNumber

运行getBlockHeaderByNumber，根据区块高度查询区块头信息。
参数：

- 区块高度

```shell
[group]: /> getBlockHeaderByNumber 1
{
    transactions=null,
    number='1',
    hash='0xfde78118e4b387b5c7ba74c19ddbd59992c5956145c5cc86265ee43814c2a320',
    logsBloom='null',
    transactionsRoot='0xf09ac805845ab8a2f63367d89804fc8e438c1f0988fc13edb2f053adbe6764ec',
    receiptRoot='0x169c46acddead6923067c057172dff458ac3f9e56008bcf2a138d605797e75c8',
    stateRoot='0x6fc1ca9cc8c937ff208aac3f9d262389fb33a16d5f9ca0027ab596cd65667aff',
    sealer='2',
    sealerList=[
        0x44c3c0d914d7a3818923f9f45927724bddeeb25df92b93f1242c32b63f726935d6742b51cd40d2c828b52ed6cde94f4d6fb4b3bfdc0689cfcddf7425eafdae85,
        0xbb21228b0762433ea6e4cb185e1c54aeb83cd964ec0e831f8732cb2522795bb569d58215dfbeb7d3fc474fdce33dc9a793d4f0e86ce69834eddc707b48915824,
        0xc1de42fc9e6798142fdbeddc05018b548b848155a8527f0ffc75eb93d0ae51ebd8074c86b6bdc0f4161dcad7cab9455a4eebf146ac5b08cb23c33c8eef756b7c,
        0xf39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366
    ],
    extraData=0x,
    gasUsed='3',
    timestamp='1638624581393',
    signatureList=[
        {
            index='0',
            signature='0x341ce6b7a7724eb432a914a4e7a154343a3116d457e05329725299f144cf430454a38e64d969c454fe7f1e864ca13a6a7e5d1a01b42bb1afc152471bb9ee3ea401'
        },
        {
            index='2',
            signature='0x2787c0be632b20abfe7f6f1cbf031eee0aa57b6e5e2cc242f3e19bafc455f847450df7acb897958e0ef5b54e1dde1b084127933bf8736b09621b7376d273d5da01'
        },
        {
            index='3',
            signature='0xcc04713e6c6dbbe8c204e9c33493dcbbd94731c56afaa4cd1fd22d14b5bd9a4a53b48d907956743d48c22fe352f361d9d48b7f9ccbb8c7e054b1c5db1bdab40400'
        }
    ]
}
```

#### 2.3.6 getBlockByHash

运行getBlockByHash，根据区块哈希查询区块信息。
参数：

- 区块哈希：0x开头的区块哈希值。
- 交易标志：默认false，区块中的交易只显示交易哈希，设置为true，显示交易具体信息。
```shell
[group]: />  getBlockByHash 0xf6afbcc3ec9eb4ac2c2829c2607e95ea0fa1be914ca1157436b2d3c5f1842855
{
    "extraData":[

    ],
    "gasLimit":"0x0",
    "gasUsed":"0x0",
    "hash":"0xf6afbcc3ec9eb4ac2c2829c2607e95ea0fa1be914ca1157436b2d3c5f1842855",
    "logsBloom":"0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
    "number":"0x1",
    "parentHash":"0xeccad5274949b9d25996f7a96b89c0ac5c099eb9b72cc00d65bc6ef09f7bd10b",
    "sealer":"0x0",
    "sealerList":[
        "0471101bcf033cd9e0cbd6eef76c144e6eff90a7a0b1847b5976f8ba32b2516c0528338060a4599fc5e3bafee188bca8ccc529fbd92a760ef57ec9a14e9e4278",
        "2b08375e6f876241b2a1d495cd560bd8e43265f57dc9ed07254616ea88e371dfa6d40d9a702eadfd5e025180f9d966a67f861da214dd36237b58d72aaec2e108",
        "cf93054cf524f51c9fe4e9a76a50218aaa7a2ca6e58f6f5634f9c2884d2e972486c7fe1d244d4b49c6148c1cb524bcc1c99ee838bb9dd77eb42f557687310ebd",
        "ed1c85b815164b31e895d3f4fc0b6e3f0a0622561ec58a10cc8f3757a73621292d88072bf853ac52f0a9a9bbb10a54bdeef03c3a8a42885fe2467b9d13da9dec"
    ],
    "stateRoot":"0x9711819153f7397ec66a78b02624f70a343b49c60bc2f21a77b977b0ed91cef9",
    "timestamp":"0x1692f119c84",
    "transactions":[
        "0xa14638d47cc679cf6eeb7f36a6d2a30ea56cb8dcf0938719ff45023a7a8edb5d"
    ],
    "transactionsRoot":"0x516787f85980a86fd04b0e9ce82a1a75950db866e8cdf543c2cae3e4a51d91b7"
}
[group]: />  getBlockByHash 0xf6afbcc3ec9eb4ac2c2829c2607e95ea0fa1be914ca1157436b2d3c5f1842855 true
{
    "extraData":[

    ],
    "gasLimit":"0x0",
    "gasUsed":"0x0",
    "hash":"0xf6afbcc3ec9eb4ac2c2829c2607e95ea0fa1be914ca1157436b2d3c5f1842855",
    "logsBloom":"0x00000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
    "number":"0x1",
    "parentHash":"0xeccad5274949b9d25996f7a96b89c0ac5c099eb9b72cc00d65bc6ef09f7bd10b",
    "sealer":"0x0",
    "sealerList":[
        "0471101bcf033cd9e0cbd6eef76c144e6eff90a7a0b1847b5976f8ba32b2516c0528338060a4599fc5e3bafee188bca8ccc529fbd92a760ef57ec9a14e9e4278",
        "2b08375e6f876241b2a1d495cd560bd8e43265f57dc9ed07254616ea88e371dfa6d40d9a702eadfd5e025180f9d966a67f861da214dd36237b58d72aaec2e108",
        "cf93054cf524f51c9fe4e9a76a50218aaa7a2ca6e58f6f5634f9c2884d2e972486c7fe1d244d4b49c6148c1cb524bcc1c99ee838bb9dd77eb42f557687310ebd",
        "ed1c85b815164b31e895d3f4fc0b6e3f0a0622561ec58a10cc8f3757a73621292d88072bf853ac52f0a9a9bbb10a54bdeef03c3a8a42885fe2467b9d13da9dec"
    ],
    "stateRoot":"0x9711819153f7397ec66a78b02624f70a343b49c60bc2f21a77b977b0ed91cef9",
    "timestamp":"0x1692f119c84",
    "transactions":[
        {
            "blockHash":"0xf6afbcc3ec9eb4ac2c2829c2607e95ea0fa1be914ca1157436b2d3c5f1842855",
            "blockNumber":"0x1",
            "from":"0x7234c32327795e4e612164e3442cfae0d445b9ad",
            "gas":"0x1c9c380",
            "gasPrice":"0x1",
            "hash":"0xa14638d47cc679cf6eeb7f36a6d2a30ea56cb8dcf0938719ff45023a7a8edb5d",
            "input":"0x608060405234801561001057600080fd5b506040805190810160405280600d81526020017f48656c6c6f2c20576f726c6421000000000000000000000000000000000000008152506000908051906020019061005c929190610062565b50610107565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f106100a357805160ff19168380011785556100d1565b828001600101855582156100d1579182015b828111156100d05782518255916020019190600101906100b5565b5b5090506100de91906100e2565b5090565b61010491905b808211156101005760008160009055506001016100e8565b5090565b90565b6102d7806101166000396000f30060806040526004361061004c576000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff1680634ed3885e146100515780636d4ce63c146100ba575b600080fd5b34801561005d57600080fd5b506100b8600480360381019080803590602001908201803590602001908080601f016020809104026020016040519081016040528093929190818152602001838380828437820191505050505050919291929050505061014a565b005b3480156100c657600080fd5b506100cf610164565b6040518080602001828103825283818151815260200191508051906020019080838360005b8381101561010f5780820151818401526020810190506100f4565b50505050905090810190601f16801561013c5780820380516001836020036101000a031916815260200191505b509250505060405180910390f35b8060009080519060200190610160929190610206565b5050565b606060008054600181600116156101000203166002900480601f0160208091040260200160405190810160405280929190818152602001828054600181600116156101000203166002900480156101fc5780601f106101d1576101008083540402835291602001916101fc565b820191906000526020600020905b8154815290600101906020018083116101df57829003601f168201915b5050505050905090565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061024757805160ff1916838001178555610275565b82800160010185558215610275579182015b82811115610274578251825591602001919060010190610259565b5b5090506102829190610286565b5090565b6102a891905b808211156102a457600081600090555060010161028c565b5090565b905600a165627a7a72305820fd74886bedbe51a7f3d834162de4d9af7f276c70133e04fd6776b5fbdd3653000029",
            "nonce":"0x3443a1391c9c29f751e8350304efb310850b8afbaa7738f5e89ddfce79b1d6",
            "to":null,
            "transactionIndex":"0x0",
            "value":"0x0"
        }
    ],
    "transactionsRoot":"0x516787f85980a86fd04b0e9ce82a1a75950db866e8cdf543c2cae3e4a51d91b7"
}
```


#### 2.3.7 getBlockByNumber

运行getBlockByNumber，根据区块高度查询区块信息。
参数：

- 区块高度：十进制整数。
- 交易标志：默认false，区块中的交易只显示交易哈希，设置为true，显示交易具体信息。

```shell
[group]: /> getBlockByNumber 1 
{
    transactions=[
        {
            version='0',
            from='0x3977d248ce98f3affa78a800c4f234434355aa77',
            hash='0x459e0bbe907bc1fb34367a150a3485921e5ce3d49c6b044e8ebb7171f8081241',
            input='0x2800efc0000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000806262323132323862303736323433336561366534636231383565316335346165623833636439363465633065383331663837333263623235323237393562623536396435383231356466626562376433666334373466646365333364633961373933643466306538366365363938333465646463373037623438393135383234',
            nonce='1244654254262255984079741307028596561526481167594459835423352512079777563980',
            to='0x0000000000000000000000000000000000001003',
            blockLimit='500',
            chainId='chain',
            groupID='group',
            transactionProof='null',
            signature=0x10617472b24161960284e1812f2becefbac5f8a4f3973ad5f600aff10a8935e345e9a8a99406f9b51bcaa2537055c76d24b0864a4fdd64b6c0c9dbfc5056680a00
        }
    ],
    number='1',
    hash='0xfde78118e4b387b5c7ba74c19ddbd59992c5956145c5cc86265ee43814c2a320',
    logsBloom='null',
    transactionsRoot='0xf09ac805845ab8a2f63367d89804fc8e438c1f0988fc13edb2f053adbe6764ec',
    receiptRoot='0x169c46acddead6923067c057172dff458ac3f9e56008bcf2a138d605797e75c8',
    stateRoot='0x6fc1ca9cc8c937ff208aac3f9d262389fb33a16d5f9ca0027ab596cd65667aff',
    sealer='2',
    sealerList=[
        0x44c3c0d914d7a3818923f9f45927724bddeeb25df92b93f1242c32b63f726935d6742b51cd40d2c828b52ed6cde94f4d6fb4b3bfdc0689cfcddf7425eafdae85,
        0xbb21228b0762433ea6e4cb185e1c54aeb83cd964ec0e831f8732cb2522795bb569d58215dfbeb7d3fc474fdce33dc9a793d4f0e86ce69834eddc707b48915824,
        0xc1de42fc9e6798142fdbeddc05018b548b848155a8527f0ffc75eb93d0ae51ebd8074c86b6bdc0f4161dcad7cab9455a4eebf146ac5b08cb23c33c8eef756b7c,
        0xf39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366
    ],
    extraData=0x,
    gasUsed='3',
    timestamp='1638624581393',
    signatureList=[
        {
            index='0',
            signature='0x341ce6b7a7724eb432a914a4e7a154343a3116d457e05329725299f144cf430454a38e64d969c454fe7f1e864ca13a6a7e5d1a01b42bb1afc152471bb9ee3ea401'
        },
        {
            index='2',
            signature='0x2787c0be632b20abfe7f6f1cbf031eee0aa57b6e5e2cc242f3e19bafc455f847450df7acb897958e0ef5b54e1dde1b084127933bf8736b09621b7376d273d5da01'
        },
        {
            index='3',
            signature='0xcc04713e6c6dbbe8c204e9c33493dcbbd94731c56afaa4cd1fd22d14b5bd9a4a53b48d907956743d48c22fe352f361d9d48b7f9ccbb8c7e054b1c5db1bdab40400'
        }
    ]
}
```

#### 2.3.8 getBlockHashByNumber

运行getBlockHashByNumber，通过区块高度获得区块哈希。
参数：

- 区块高度：十进制整数。

```shell
[group]: />  getBlockHashByNumber 1
0xf6afbcc3ec9eb4ac2c2829c2607e95ea0fa1be914ca1157436b2d3c5f1842855
```

#### 2.3.9 getTransactionByHash

运行getTransactionByHash，通过交易哈希查询交易信息。
参数：

- 交易哈希：0x开头的交易哈希值。

```shell
[group]: /> getTransactionByHash 0x459e0bbe907bc1fb34367a150a3485921e5ce3d49c6b044e8ebb7171f8081241
{
    version='0',
    from='0x3977d248ce98f3affa78a800c4f234434355aa77',
    hash='0x459e0bbe907bc1fb34367a150a3485921e5ce3d49c6b044e8ebb7171f8081241',
    input='0x2800efc0000000000000000000000000000000000000000000000000000000000000002000000000000000000000000000000000000000000000000000000000000000806262323132323862303736323433336561366534636231383565316335346165623833636439363465633065383331663837333263623235323237393562623536396435383231356466626562376433666334373466646365333364633961373933643466306538366365363938333465646463373037623438393135383234',
    nonce='1244654254262255984079741307028596561526481167594459835423352512079777563980',
    to='0x0000000000000000000000000000000000001003',
    blockLimit='500',
    chainId='chain',
    groupID='group',
    transactionProof='null',
    signature=0x10617472b24161960284e1812f2becefbac5f8a4f3973ad5f600aff10a8935e345e9a8a99406f9b51bcaa2537055c76d24b0864a4fdd64b6c0c9dbfc5056680a00
}
```

#### 2.3.10 getTransactionReceipt

运行getTransactionReceipt，通过交易哈希查询交易回执。
参数：

- 交易哈希：0x开头的交易哈希值。

```shell
[group]: />  getTransactionReceipt 0x1dfc67c51f5cc93b033fc80e5e9feb049c575a58b863483aa4d04f530a2c87d5
{
    "blockHash":"0xe4e1293837013f547ad7f443a8ff20a4e32a060b9cac56c41462255603548b7b",
    "blockNumber":"0x8",
    "contractAddress":"0x0000000000000000000000000000000000000000",
    "from":"0xf0d2115e52b0533e367447f700bfbf2ed35ff6fc",
    "gasUsed":"0x94f5",
    "input":"0xebf3b24f0000000000000000000000000000000000000000000000000000000000000060000000000000000000000000000000000000000000000000000000000000000100000000000000000000000000000000000000000000000000000000000000a00000000000000000000000000000000000000000000000000000000000000005667275697400000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000056170706c65000000000000000000000000000000000000000000000000000000",
    "logs":[
        {
            "address":"0x42fc572759fd568bd590f46011784be2a2d53f0c",
            "data":"0x0000000000000000000000000000000000000000000000000000000000000001",
            "topics":[
                "0xc57b01fa77f41df77eaab79a0e2623fab2e7ae3e9530d9b1cab225ad65f2b7ce"
            ]
        }
    ],
    "logsBloom":"0x00000000000000800000000000000000000000000000000000000000000000000000000000000000020000000000000000000000000000000000000000800000000000000000000000000000000002000000000000000002000000000000000000000000000000000000000000000000000000000000000000000000000000000001000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000",
    "output":"0x0000000000000000000000000000000000000000000000000000000000000001",
    "status":"0x0",
    "to":"0x42fc572759fd568bd590f46011784be2a2d53f0c",
    "transactionHash":"0x1dfc67c51f5cc93b033fc80e5e9feb049c575a58b863483aa4d04f530a2c87d5",
    "transactionIndex":"0x0"
}
```
#### 2.3.11 getPendingTxSize
运行getPendingTxSize，查询等待处理的交易数量（交易池中的交易数量）。
```shell
[group]: />  getPendingTxSize
0
```

#### 2.3.12 getTotalTransactionCount

运行getTotalTransactionCount，查询当前块高和累计交易数（从块高为0开始）。

```shell
[group]: />  getTotalTransactionCount
{
    "blockNumber":1,
    "txSum":1,
    "failedTxSum":0
}
```

### 2.4 共识操作命令

#### 2.4.1 getSealerList

运行getSealerList，查看共识节点列表。

```shell
[group]: />  getSealerList
[
    Sealer{
        nodeID='44c3c0d914d7a3818923f9f45927724bddeeb25df92b93f1242c32b63f726935d6742b51cd40d2c828b52ed6cde94f4d6fb4b3bfdc0689cfcddf7425eafdae85',
        weight=1
    },
    Sealer{
        nodeID='f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366',
        weight=1
    },
    Sealer{
        nodeID='c1de42fc9e6798142fdbeddc05018b548b848155a8527f0ffc75eb93d0ae51ebd8074c86b6bdc0f4161dcad7cab9455a4eebf146ac5b08cb23c33c8eef756b7c',
        weight=1
    },
    Sealer{
        nodeID='bb21228b0762433ea6e4cb185e1c54aeb83cd964ec0e831f8732cb2522795bb569d58215dfbeb7d3fc474fdce33dc9a793d4f0e86ce69834eddc707b48915824',
        weight=1
    }
]
```

#### 2.4.2 getObserverList

运行getObserverList，查看观察节点列表。

```shell
[group]: />  getObserverList
[  
    bb21228b0762433ea6e4cb185e1c54aeb83cd964ec0e831f8732cb2522795bb569d58215dfbeb7d3fc474fdce33dc9a793d4f0e86ce69834eddc707b48915824
]
```
#### 2.4.3 getPbftView

运行getPbftView，查看pbft视图。

```shell
[group]: />  getPbftView
2730
```

#### 2.4.4 getConsensusStatus

运行getConsensusStatus，查看共识状态。

```shell
[group]: /> getConsensusStatus 
ConsensusStatusInfo{
    nodeID='44c3c0d914d7a3818923f9f45927724bddeeb25df92b93f1242c32b63f726935d6742b51cd40d2c828b52ed6cde94f4d6fb4b3bfdc0689cfcddf7425eafdae85',
    index=0,
    leaderIndex=0,
    consensusNodesNum=3,
    maxFaultyQuorum=0,
    minRequiredQuorum=3,
    isConsensusNode=true,
    blockNumber=1,
    hash='fde78118e4b387b5c7ba74c19ddbd59992c5956145c5cc86265ee43814c2a320',
    timeout=false,
    changeCycle=0,
    view=1,
    connectedNodeList=4,
    consensusNodeInfos=[
        ConsensusNodeInfo{
            nodeID='44c3c0d914d7a3818923f9f45927724bddeeb25df92b93f1242c32b63f726935d6742b51cd40d2c828b52ed6cde94f4d6fb4b3bfdc0689cfcddf7425eafdae85',
            weight='1',
            index='0'
        },
        ConsensusNodeInfo{
            nodeID='c1de42fc9e6798142fdbeddc05018b548b848155a8527f0ffc75eb93d0ae51ebd8074c86b6bdc0f4161dcad7cab9455a4eebf146ac5b08cb23c33c8eef756b7c',
            weight='1',
            index='1'
        },
        ConsensusNodeInfo{
            nodeID='f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366',
            weight='1',
            index='2'
        }
    ]
}

```

#### 2.4.5 addSealer

运行addSealer，将节点添加为共识节点。
参数：

- 节点nodeId
- 节点权重

```shell
[group]: /> addSealer bb21228b0762433ea6e4cb185e1c54aeb83cd964ec0e831f8732cb2522795bb569d58215dfbeb7d3fc474fdce33dc9a793d4f0e86ce69834eddc707b48915824 2
{
    "code":0,
    "msg":"Success"
}
```

#### 2.4.6 addObserver

运行addObserver，将节点添加为观察节点。
参数：

- 节点nodeId

```shell
[group]: />  addObserver ea2ca519148cafc3e92c8d9a8572b41ea2f62d0d19e99273ee18cccd34ab50079b4ec82fe5f4ae51bd95dd788811c97153ece8c05eac7a5ae34c96454c4d3123
{
    "code":1,
    "msg":"success"
}
```

#### 2.4.7 removeNode

运行removeNode，节点退出。通过addSealer命令可以将退出的节点添加为共识节点，通过addObserver命令将节点添加为观察节点。
参数：

- 节点nodeId

```shell
[group]: />  removeNode ea2ca519148cafc3e92c8d9a8572b41ea2f62d0d19e99273ee18cccd34ab50079b4ec82fe5f4ae51bd95dd788811c97153ece8c05eac7a5ae34c96454c4d3123
{
    "code":1,
    "msg":"success"
}
```

#### 2.4.8 setConsensusWeight

运行setConsensusWeight，设置某一个特定节点的共识权重。

```shell
[group]: /> setConsensusWeight 44c3c0d914d7a3818923f9f45927724bddeeb25df92b93f1242c32b63f726935d6742b51cd40d2c828b52ed6cde94f4d6fb4b3bfdc0689cfcddf7425eafdae85 2
{
    "code":0,
    "msg":"Success"
}
```

### 2.5 群组查询命令

#### 2.5.1 getGroupPeers

运行getGroupPeers，查看节点所在group的共识节点和观察节点列表。

```shell
[group]: /> getGroupPeers 
peer0: 44c3c0d914d7a3818923f9f45927724bddeeb25df92b93f1242c32b63f726935d6742b51cd40d2c828b52ed6cde94f4d6fb4b3bfdc0689cfcddf7425eafdae85
peer1: bb21228b0762433ea6e4cb185e1c54aeb83cd964ec0e831f8732cb2522795bb569d58215dfbeb7d3fc474fdce33dc9a793d4f0e86ce69834eddc707b48915824
peer2: c1de42fc9e6798142fdbeddc05018b548b848155a8527f0ffc75eb93d0ae51ebd8074c86b6bdc0f4161dcad7cab9455a4eebf146ac5b08cb23c33c8eef756b7c
peer3: f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366
```

#### 2.5.2 getGroupInfo

运行getGroupInfo，查看节点所在group的详细信息。

```shell
[group]: /> getGroupInfo
{
    "chainID":"chain",
    "groupID":"group",
    "genesisConfig":{
        "consensusType":"pbft",
        "blockTxCountLimit":1000,
        "txGasLimit":300000000,
        "consensusLeaderPeriod":1,
        "sealerList":[
            {
                "nodeID":"44c3c0d914d7a3818923f9f45927724bddeeb25df92b93f1242c32b63f726935d6742b51cd40d2c828b52ed6cde94f4d6fb4b3bfdc0689cfcddf7425eafdae85",
                "weight":1
            },
            {
                "nodeID":"f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366",
                "weight":1
            },
            {
                "nodeID":"c1de42fc9e6798142fdbeddc05018b548b848155a8527f0ffc75eb93d0ae51ebd8074c86b6bdc0f4161dcad7cab9455a4eebf146ac5b08cb23c33c8eef756b7c",
                "weight":1
            },
            {
                "nodeID":"bb21228b0762433ea6e4cb185e1c54aeb83cd964ec0e831f8732cb2522795bb569d58215dfbeb7d3fc474fdce33dc9a793d4f0e86ce69834eddc707b48915824",
                "weight":1
            }
        ]
    },
    "nodeList":[
        {
            "type":0,
            "iniConfig":{
                "binaryInfo":{
                    "version":"3.0.0-rc1",
                    "gitCommitHash":"92ac16b4d2da511cbba33031e58369cac240547a",
                    "platform":"Darwin/appleclang",
                    "buildTime":"20211203 10:43:32"
                },
                "chainID":"chain",
                "groupID":"group",
                "smCryptoType":false,
                "nodeID":"f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366",
                "nodeName":"f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366",
                "rpcServiceName":"",
                "gatewayServiceName":"",
                "isWasm":false
            },
            "name":"f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366",
            "serviceInfoList":null
        }
    ]
}
```

#### 2.5.3 getGroupList

运行getGroupList，查看群组列表：

```shell
[group]: /> getGroupList
group0: group
```

#### 2.5.4 getGroupInfoList

运行getGroupList，查看所有的群组信息列表：

```shell
[group]: /> getGroupInfoList 
[
    {
        "chainID":"chain",
        "groupID":"group",
        "genesisConfig":{
            "consensusType":"pbft",
            "blockTxCountLimit":1000,
            "txGasLimit":300000000,
            "consensusLeaderPeriod":1,
            "sealerList":[
                {
                    "nodeID":"44c3c0d914d7a3818923f9f45927724bddeeb25df92b93f1242c32b63f726935d6742b51cd40d2c828b52ed6cde94f4d6fb4b3bfdc0689cfcddf7425eafdae85",
                    "weight":1
                },
                {
                    "nodeID":"f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366",
                    "weight":1
                },
                {
                    "nodeID":"c1de42fc9e6798142fdbeddc05018b548b848155a8527f0ffc75eb93d0ae51ebd8074c86b6bdc0f4161dcad7cab9455a4eebf146ac5b08cb23c33c8eef756b7c",
                    "weight":1
                },
                {
                    "nodeID":"bb21228b0762433ea6e4cb185e1c54aeb83cd964ec0e831f8732cb2522795bb569d58215dfbeb7d3fc474fdce33dc9a793d4f0e86ce69834eddc707b48915824",
                    "weight":1
                }
            ]
        },
        "nodeList":[
            {
                "type":0,
                "iniConfig":{
                    "binaryInfo":{
                        "version":"3.0.0-rc1",
                        "gitCommitHash":"92ac16b4d2da511cbba33031e58369cac240547a",
                        "platform":"Darwin/appleclang",
                        "buildTime":"20211203 10:43:32"
                    },
                    "chainID":"chain",
                    "groupID":"group",
                    "smCryptoType":false,
                    "nodeID":"f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366",
                    "nodeName":"f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366",
                    "rpcServiceName":"",
                    "gatewayServiceName":"",
                    "isWasm":false
                },
                "name":"f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366",
                "serviceInfoList":null
            }
        ]
    }
]
```

#### 2.5.5 getGroupNodeInfo

运行getGroupNodeInfo命令，获取当前群组内某一个节点的信息：

```shell
[group]: /> getGroupNodeInfo f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366
{
    "type":0,
    "iniConfig":{
        "binaryInfo":{
            "version":"3.0.0-rc1",
            "gitCommitHash":"92ac16b4d2da511cbba33031e58369cac240547a",
            "platform":"Darwin/appleclang",
            "buildTime":"20211203 10:43:32"
        },
        "chainID":"chain",
        "groupID":"group",
        "smCryptoType":false,
        "nodeID":"f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366",
        "nodeName":"f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366",
        "rpcServiceName":"",
        "gatewayServiceName":"",
        "isWasm":false
    },
    "name":"f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366",
    "serviceInfoList":null
}

[group]: /> getGroupNodeInfo f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366
{
    "type":0,
    "iniConfig":{
        "binaryInfo":{
            "version":"3.0.0-rc1",
            "gitCommitHash":"92ac16b4d2da511cbba33031e58369cac240547a",
            "platform":"Darwin/appleclang",
            "buildTime":"20211203 10:43:32"
        },
        "chainID":"chain",
        "groupID":"group",
        "smCryptoType":false,
        "nodeID":"f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366",
        "nodeName":"f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366",
        "rpcServiceName":"",
        "gatewayServiceName":"",
        "isWasm":false
    },
    "name":"f39b21b4832976591085b73a8550442e76dc2ae657adb799ff123001a553be60293b1059e97c472e49bb02b71384f05501f149905015707a2fe08979742c1366",
    "serviceInfoList":null
}
```

### 2.6 合约操作命令

#### 2.6.1 deploy

部署合约。(默认提供HelloWorld合约和TableTest.sol进行示例使用)
参数：

- 合约路径：合约文件的路径，支持相对路径、绝对路径和默认路径三种方式。用户输入为文件名时，从默认目录获取文件，默认目录为: `contracts/solidity`，比如：HelloWorld。

```shell
# 部署HelloWorld合约，默认路径
[group]: />  deploy HelloWorld
contract address:0xc0ce097a5757e2b6e189aa70c7d55770ace47767

# 部署HelloWorld合约，相对路径
[group]: />  deploy contracts/solidity/HelloWorld.sol
contract address:0xd653139b9abffc3fe07573e7bacdfd35210b5576

# 部署HelloWorld合约，绝对路径
[group]: />  deploy ~/fisco/console/contracts/solidity/HelloWorld.sol
contract address:0x85517d3070309a89357c829e4b9e2d23ee01d881
```

**注：**

- 部署用户编写的合约，可以将solidity合约文件放到控制台根目录的`contracts/solidity/`目录下，然后进行部署即可。按tab键可以搜索`contracts/solidity/`目录下的合约名称。
- 若需要部署的合约引用了其他合约或library库，引用格式为`import "./XXX.sol";`。其相关引入的合约和library库均放在`contracts/solidity/`目录。
- 如果合约引用了library库，library库文件的名称必须以`Lib`字符串开始，以便于区分是普通合约与library库文件。library库文件不能单独部署和调用。

#### 2.6.2 call

运行call，调用合约。
参数：

- 合约路径：合约文件的路径，支持相对路径、绝对路径和默认路径三种方式。用户输入为文件名时，从默认目录获取文件，默认目录为: `contracts/solidity`。
- 合约地址: 部署合约获取的地址。
- 合约接口名：调用的合约接口名。
- 参数：由合约接口参数决定。**参数由空格分隔；数组参数需要加上中括号，比如[1,2,3]，数组中是字符串或字节类型，加双引号，例如[“alice”,”bob”]，注意数组参数中不要有空格；布尔类型为true或者false。**

```shell
# 调用HelloWorld的get接口获取name字符串
[group]: />  call HelloWorld 0x175b16a1299c7af3e2e49b97e68a44734257a35e get
---------------------------------------------------------------------------------------------
Return code: 0
description: transaction executed successfully
Return message: Success
---------------------------------------------------------------------------------------------
Return values:
[
    "Hello,World!"
]
---------------------------------------------------------------------------------------------

# 调用HelloWorld的set接口设置name字符串
[group]: />  call HelloWorld 0x175b16a1299c7af3e2e49b97e68a44734257a35e set "Hello, FISCO BCOS"
transaction hash: 0x54b7bc73e3b57f684a6b49d2fad41bd8decac55ce021d24a1f298269e56f1ce1
---------------------------------------------------------------------------------------------
transaction status: 0x0
description: transaction executed successfully
---------------------------------------------------------------------------------------------
Output
Receipt message: Success
Return message: Success
---------------------------------------------------------------------------------------------
Event logs
Event: {}

# 调用HelloWorld的get接口获取name字符串，检查设置是否生效
[group]: />  call HelloWorld 0x175b16a1299c7af3e2e49b97e68a44734257a35e get
---------------------------------------------------------------------------------------------
Return code: 0
description: transaction executed successfully
Return message: Success
---------------------------------------------------------------------------------------------
Return values:
[
    "Hello,FISCO BCOS"
]
---------------------------------------------------------------------------------------------


# 调用TableTest的insert接口插入记录，字段为name, item_id, item_name
[group]: />  call TableTest 0x5f248ad7e917cddc5a4d408cf18169d19c0990e5 insert "fruit" 1 "apple"
transaction hash: 0x64bfab495dc1f50c58d219b331df5a47577aa8afc16be926260238a9b0ec0fbb
---------------------------------------------------------------------------------------------
transaction status: 0x0
description: transaction executed successfully
---------------------------------------------------------------------------------------------
Output
Receipt message: Success
Return message: Success
---------------------------------------------------------------------------------------------
Event logs
Event: {"InsertResult":[1]}

# 调用TableTest的select接口查询记录
[group]: />  [group]: />  call TableTest 0x5f248ad7e917cddc5a4d408cf18169d19c0990e5 select "fruit"
---------------------------------------------------------------------------------------------
Return code: 0
description: transaction executed successfully
Return message: Success
---------------------------------------------------------------------------------------------
Return values:
[
    [
        "fruit"
    ],
    [
        1
    ],
    [
        "apple"
    ]
]
---------------------------------------------------------------------------------------------
```

**注：** TableTest.sol合约代码[参考这里](../manual/smart_contract.html#solidity)。

#### 2.6.3 getCode

运行getCode，根据合约地址查询合约二进制代码。
参数：

- 合约地址：0x的合约地址(部署合约可以获得合约地址)。

```shell
[group]: />  getCode 0x97b8c19598fd781aaeb53a1957ef9c8acc59f705
0x60606040526000357c0100000000000000000000000000000000000000000000000000000000900463ffffffff16806366c99139146100465780636d4ce63c14610066575bfe5b341561004e57fe5b610064600480803590602001909190505061008c565b005b341561006e57fe5b61007661028c565b6040518082815260200191505060405180910390f35b8060006001015410806100aa57506002600101548160026001015401105b156100b457610289565b806000600101540360006001018190555080600260010160008282540192505081905550600480548060010182816100ec919061029a565b916000526020600020906004020160005b608060405190810160405280604060405190810160405280600881526020017f32303137303431330000000000000000000000000000000000000000000000008152508152602001600060000160009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff168152602001600260000160009054906101000a900473ffffffffffffffffffffffffffffffffffffffff1673ffffffffffffffffffffffffffffffffffffffff16815260200185815250909190915060008201518160000190805190602001906101ec9291906102cc565b5060208201518160010160006101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff16021790555060408201518160020160006101000a81548173ffffffffffffffffffffffffffffffffffffffff021916908373ffffffffffffffffffffffffffffffffffffffff160217905550606082015181600301555050505b50565b600060026001015490505b90565b8154818355818115116102c7576004028160040283600052602060002091820191016102c6919061034c565b5b505050565b828054600181600116156101000203166002900490600052602060002090601f016020900481019282601f1061030d57805160ff191683800117855561033b565b8280016001018555821561033b579182015b8281111561033a57825182559160200191906001019061031f565b5b50905061034891906103d2565b5090565b6103cf91905b808211156103cb57600060008201600061036c91906103f7565b6001820160006101000a81549073ffffffffffffffffffffffffffffffffffffffff02191690556002820160006101000a81549073ffffffffffffffffffffffffffffffffffffffff0219169055600382016000905550600401610352565b5090565b90565b6103f491905b808211156103f05760008160009055506001016103d8565b5090565b90565b50805460018160011615610100020316600290046000825580601f1061041d575061043c565b601f01602090049060005260206000209081019061043b91906103d2565b5b505600a165627a7a723058203c1f95b4a803493db0120df571d9f5155734152548a532412f2f9fa2dbe1ac730029
```

#### 2.6.4 listAbi

显示合约接口和Event列表
参数：

- 合约路径：合约文件的路径，支持相对路径、绝对路径和默认路径三种方式。用户输入为文件名时，从默认目录获取文件，默认目录为: `contracts/solidity`，比如：TableTest。
- 合约名：(可选)合约名称，默认情况下使用合约文件名作为合约名参数

```shell
[group]: />  listAbi TableTest
Method list:
 name                |    constant  |    methodId    |    signature
  --------------------------------------------------------------
 remove              |    false     |    0x0fe1160f  |    remove(string,int256)
 update              |    false     |    0x49cc36b5  |    update(string,int256,string)
 select              |    true      |    0x5b325d78  |    select(string)
 insert              |    false     |    0xe020d464  |    insert(string,int256,string)

Event list:
 name                |   topic                                                                   signature
  --------------------------------------------------------------
 remove              |   0x0fe1160f9655e87c29e76aca1cab34fb2a644d375da7a900c7076bad17cad26b  |   remove(string,int256)
 update              |   0x49cc36b56a9320d20b2d9a1938a972c849191bceb97500bfd38fa8a590dac73a  |   update(string,int256,string)
 select              |   0x5b325d7821528d3b52d0cc7a83e1ecef0438f763796770201020ac8b8813ac0a  |   select(string)
 insert              |   0xe020d464e502c11b54a7e37e568c78f0fcd360213eb5f4ac0a25a17733fc19f7  |   insert(string,int256,string)
```

#### 2.6.5 getDeployLog

运行getDeployLog，查询群组内**由当前控制台**部署合约的日志信息。日志信息包括部署合约的时间，群组ID，合约名称和合约地址。参数：

- 日志行数，可选，根据输入的期望值返回最新的日志信息，当实际条数少于期望值时，按照实际数量返回。当期望值未给出时，默认按照20条返回最新的日志信息。

```shell
[group]: />  getDeployLog 2

2019-05-26 08:37:03  [group]  HelloWorld            0xc0ce097a5757e2b6e189aa70c7d55770ace47767
2019-05-26 08:37:45  [group]  TableTest             0xd653139b9abffc3fe07573e7bacdfd35210b5576

[group]: />  getDeployLog 1

2019-05-26 08:37:45  [group]  TableTest             0xd653139b9abffc3fe07573e7bacdfd35210b5576
```

**注：** 如果要查看所有的部署合约日志信息，请查看`console`目录下的`deploylog.txt`文件。该文件只存储最近10000条部署合约的日志记录。

#### 2.6.6 deployByCNS

运行deployByCNS，采用[CNS](../design/features/cns_contract_name_service.md)部署合约。用CNS部署的合约，可用合约名直接调用。
参数：

- 合约路径：合约文件的路径，支持相对路径、绝对路径和默认路径三种方式。用户输入为文件名时，从默认目录获取文件，默认目录为: `contracts/solidity`。
- 合约版本号：部署的合约版本号。
```shell
# 部署HelloWorld合约1.0版
[group]: />  deployByCNS HelloWorld 1.0
transaction hash: 0xb994d8e510f147815bdf9838fda542e553c2fe981177ee7a97a0686b9619cfbb
contract address: 0xf485b9ccfffa668f4d7fac37c81c0cd63408188c
{
    "code":1,
    "msg":"Success"
}

# 部署HelloWorld合约2.0版
[group]: />  deployByCNS HelloWorld 2.0
transaction hash: 0x3132f73e5af72fce44c2e07185a04f554ac06ddd94119dcf798980695b0890a0
contract address: 0xf68a1aabfad336847e109c33ca471b192c93c0a9
{
    "code":1,
    "msg":"Success"
}

# 部署TableTest合约
[group]: />  deployByCNS TableTest 1.0
transaction hash: 0x288ccc6e166e43f5cd3bd563e00af48988e641196aaea57a59f0ab256e23c84e
contract address: 0x0fe221339e50c39aaefddfc3a9a26b4aeff23c63
{
    "code":1,
    "msg":"Success"
}
```

**注：**
- 部署用户编写的合约，可以将solidity合约文件放到控制台根目录的`contracts/solidity/`目录下，然后进行部署即可。按tab键可以搜索`contracts/solidity/`目录下的合约名称。
- 若需要部署的合约引用了其他合约或library库，引用格式为`import "./XXX.sol";`。其相关引入的合约和library库均放在`contracts/solidity/`目录。
- 如果合约引用了library库，library库文件的名称必须以`Lib`字符串开始，以便于区分是普通合约与library库文件。library库文件不能单独部署和调用。


#### 2.6.7 queryCNS

运行queryCNS，根据合约名称和合约版本号（可选参数）查询CNS表记录信息（合约名和合约地址的映射）。
参数：

- 合约名称：部署的合约名称。
- 合约版本号：(可选)部署的合约版本号。

```shell
[group]: />  queryCNS HelloWorld
---------------------------------------------------------------------------------------------
|                   version                   |                   address                   |
|                     1.0                     | 0xf485b9ccfffa668f4d7fac37c81c0cd63408188c  |
|                     2.0                     | 0xf68a1aabfad336847e109c33ca471b192c93c0a9  |
---------------------------------------------------------------------------------------------

[group]: />  queryCNS HelloWorld 1.0
---------------------------------------------------------------------------------------------
|                   version                   |                   address                   |
|                     1.0                     | 0xf485b9ccfffa668f4d7fac37c81c0cd63408188c  |
---------------------------------------------------------------------------------------------
```

#### 2.6.8 callByCNS
运行callByCNS，采用CNS调用合约，即用合约名直接调用合约。
参数：

- 合约名称与合约版本号：合约名称与版本号用英文冒号分隔，例如`HelloWorld:1.0`。当省略合约版本号时，例如`HelloWorld`，则调用最新版本的合约。
- 合约接口名：调用的合约接口名。
- 参数：由合约接口参数决定。**参数由空格分隔，其中字符串、字节类型参数需要加上双引号；数组参数需要加上中括号，比如[1,2,3]，数组中是字符串或字节类型，加双引号，例如["alice","bob"]；布尔类型为true或者false。**

```shell
# 调用HelloWorld合约1.0版，通过set接口设置name字符串
[group]: />  callByCNS HelloWorld:1.0 set "Hello,CNS"
transaction hash: 0x7ad2f34a13bbc2272d2d52fa46915e6f03136a6960d84a23478bee3c37e76ad6
---------------------------------------------------------------------------------------------
transaction status: 0x0
description: transaction executed successfully
---------------------------------------------------------------------------------------------
Output
Receipt message: Success
Return message: Success
---------------------------------------------------------------------------------------------
Event logs
Event: {}

# 调用HelloWorld合约2.0版，通过set接口设置name字符串
[group]: />  callByCNS HelloWorld:2.0 set "Hello,CNS2"
transaction hash: 0x41f8decbe137905824321da6c28a19d957e902cfad1e8078d797bc649c078d3e
---------------------------------------------------------------------------------------------
transaction status: 0x0
description: transaction executed successfully
---------------------------------------------------------------------------------------------
Output
Receipt message: Success
Return message: Success
---------------------------------------------------------------------------------------------
Event logs
Event: {}

# 调用HelloWorld合约1.0版，通过get接口获取name字符串
[group]: />  callByCNS HelloWorld:1.0 get
---------------------------------------------------------------------------------------------
Return code: 0
description: transaction executed successfully
Return message: Success
---------------------------------------------------------------------------------------------
Return values:
[
    "Hello,CNS"
]
---------------------------------------------------------------------------------------------

# 调用HelloWorld合约最新版(即2.0版)，通过get接口获取name字符串
[group]: />  callByCNS HelloWorld get
---------------------------------------------------------------------------------------------
Return code: 0
description: transaction executed successfully
Return message: Success
---------------------------------------------------------------------------------------------
Return values:
[
    "Hello,CNS2"
]
---------------------------------------------------------------------------------------------
```

#### 2.6.9 registerCNS

注册合约至CNS。

参数：
- 合约路径: 合约文件的路径，支持相对路径、绝对路径和默认路径三种方式。用户输入为文件名时，从默认目录获取文件，默认目录为: `contracts/solidity`。
- 合约地址: 注册合约地址
- 合约版本号: 注册合约版本号

```shell
[group]: />  registerCNS HelloWorld 0xf19a7ec01f0b1adb16a033f0a30fb321ec6edcbf v1.0.0
{
    "code":1,
    "msg":"success"
}
```

#### 2.6.10 listDeployContractAddress

列出指定合约名部署的所有合约地址，
列出部署指定合约产生的合约地址列表，参数：

- contractNameOrPath: 合约名或合约绝对路径，用于指定合约;
- recordNumber: 显示的合约地址列表长度，默认为20

```shell
# 获取部署HelloWorld合约产生的合约地址列表
[group]: />  listDeployContractAddress HelloWorld
0xe157185434183b276b9e5af7d0315a9829171281  2020-10-13 15:35:29
0x136b042e1fc480b03e1e5b075cbdfa52f5851a23  2020-10-13 15:35:22
0xd7d0b221bc984a20aa6b0fc098dad89888378e3a  2020-10-13 15:34:14
0x0fe221339e50c39aaefddfc3a9a26b4aeff23c63  2020-10-13 15:16:20
0x5f248ad7e917cddc5a4d408cf18169d19c0990e5  2020-10-13 12:28:23
0xf027fd91a51bd4844f17600c01e943058fc27482  2020-10-12 18:28:50
0x682b51f3c7f9a605fa2026b09fd2635a6fa562d9  2020-10-11 23:22:28
0xf68a1aabfad336847e109c33ca471b192c93c0a9  2020-10-11 22:53:07
0xf485b9ccfffa668f4d7fac37c81c0cd63408188c  2020-10-11 22:52:26
0x175b16a1299c7af3e2e49b97e68a44734257a35e  2020-10-11 22:49:25
0x7c6dc94e4e146cb13eb03dc98d2b96ac79ef5e67  2020-10-11 22:46:35


# 限制显示的合约地址列表长度为2
[group]: />  listDeployContractAddress HelloWorld 2
0xe157185434183b276b9e5af7d0315a9829171281  2020-10-13 15:35:29
0x136b042e1fc480b03e1e5b075cbdfa52f5851a23  2020-10-13 15:35:22
```

### 2.7 BFS操作命令

BFS为FISCO BCOS 3.0新的特性，详细设计请参考：FIXME

#### 2.7.1 cd

与Linux的cd命令类似，可以切换当前所在的路径，支持绝对路径、相对路径。

```shell
[group]: /> cd /tables

[group]: /tables>

[group]: /> cd ./tables/test

[group]: /tables/test>

[group]: /tables/test> cd ../../

[group]: />
```

#### 2.7.2 ls

与Linux的ls命令相似，可以查看当前路径下的资源，如果是目录，则展示出该目录下所有资源；如果是合约，则展示该合约的元信息。

ls参数为0时，展示当前文件夹，ls 参数为1时，支持绝对路径和相对路径。

```shell
[group]: /> ls
apps usr sys tables

[group]: /> ls apps
Hello

[group]: /> ls ./apps/Hello
name: Hello, type: contract
```

#### 2.7.3 mkdir

与Linux的mkdir命令相似，在某个文件夹下创建新的目录，支持绝对路径和相对路径。

```shell
[group]: /> mkdir test

[group]: /> ls
/ apps usr sys tables test

[group]: /> mkdir ./test/test

[group]: /> ls ./test
test
```

#### 2.7.4 pwd

与Linux的pwd命令相似，没有参数，展示当前路径。

```shell
[group]: /apps/Hello/BFS>  pwd
/apps/Hello/BFS
```

### 2.8 系统设置操作命令

#### 2.7.1 setSystemConfigByKey

运行setSystemConfigByKey，以键值对方式设置系统参数。目前设置的系统参数支持`tx_count_limit`,`consensus_leader_period`。这些系统参数的键名可以通过tab键补全：

* tx_count_limit：区块最大打包交易数
* consensus_leader_period：交易执行允许消耗的最大gas数

参数：

- key
- value

```shell
[group]: />  setSystemConfigByKey tx_count_limit 100
{
    "code":1,
    "msg":"success"
}
```

#### 2.7.2 getSystemConfigByKey

运行getSystemConfigByKey，根据键查询系统参数的值。
参数：

- key

```shell
[group]: />  getSystemConfigByKey tx_count_limit
100
```

### 2.9 权限操作命令

权限操作只在节点开启权限模式，且设置初始化治理委员地址才可使用。

##### getCommitteeInfo 获取委员会信息

在初始化时，将会部署一个治理委员，该治理委员的地址信息在 build_chain.sh时自动生成或者指定。初始化只有一个委员，并且委员的权重为1

```shell
[group]: /> getCommitteeInfo
CommitteeInfo{
    governorList=[
        GovernorInfo{
            governorAddress='0xc49fd7d7648ef4f4774d642d905db2e66f5089a8',
            weight=1
        }
    ],
    participatesRate=0,
    winRate=0
}
```

##### getProposalInfo 获取特定提案的信息

获取某个特定的提案信息，提案ID必须存在，否则会报错。

`proposalType` 和  `status` 可以看到提案的类型和状态

proposalType分为以下几种：

- setWeight：当治理委员发起updateGovernorProposal 提案时会生成
- setRate：setRateProposal 提案会生成
- setDeployAuthType：setDeployAuthTypeProposal 提案会生成
- modifyDeployAuth：openDeployAuthProposal 和closeDeployAuthProposal 提案会生成
- resetAdmin：resetAdminProposal 提案会生成
- unknown：这个类型出现时，有可能是有bug

status分为以下几种：

- notEnoughVotes：提案正常，还未收集到足够的投票
- success ：提案成功
- failed：提案失败
- revoke：提案被撤回
- unknown：这个类型出现时，有可能是有bug

```shell
[group]: /> getProposalInfo 1
ProposalInfo{
    resourceId='0xc49fd7d7648ef4f4774d642d905db2e66f5089a8',
    proposer='0xc49fd7d7648ef4f4774d642d905db2e66f5089a8',
    proposalType=setWeight,
    blockNumberInterval=604801,
    status=success,
    agreeVoters=[
        0xc49fd7d7648ef4f4774d642d905db2e66f5089a8
    ],
    againstVoters=[

    ]
}
```

##### getDeployAuth 获取当前全局部署的权限策略

权限策略分为：

- 无权限，所有人都可以部署
- 黑名单，在黑名单上的用户不可以部署
- 白名单，只有在白名单的用户可以部署

```shell
[group]: /> getDeployAuth
There is no deploy strategy, everyone can deploy contracts.
```

#### 2.9.1 治理委员会命令

治理委员会专用命令，必须要有治理委员会的Governors中的账户才可以调用

如果治理委员只有一个，且提案是该委员发起的，那么这个提案一定能成功

##### updateGovernorProposal 更新治理委员会成员提案

如果是新加治理委员，新增地址和权重即可。

如果是删除治理委员，将一个治理委员的权重设置为0 即可

```shell
[group]: /> updateGovernorProposal 0xc49fd7d7648ef4f4774d642d905db2e66f5089a8 321
Update governor proposal created, id is: 2

[group]: /> getProposalInfo 2
ProposalInfo{
    resourceId='0xc49fd7d7648ef4f4774d642d905db2e66f5089a8',
    proposer='0xc49fd7d7648ef4f4774d642d905db2e66f5089a8',
    proposalType=setWeight,
    blockNumberInterval=604802,
    status=success,
    agreeVoters=[
        0xc49fd7d7648ef4f4774d642d905db2e66f5089a8
    ],
    againstVoters=[

    ]
}

```

##### setRateProposal 设置委员会提案阈值的提案

设置提案阈值，提案阈值分为参与阈值和权重阈值。

阈值在这里是以百分比为单位进行使用的。

```shell
[group]: /> setRateProposal 1 1
Set rate proposal created, id is: 3

[group]: /> getProposalInfo 3
ProposalInfo{
    resourceId='0x0000000000000000000000000000000000010001',
    proposer='0xc49fd7d7648ef4f4774d642d905db2e66f5089a8',
    proposalType=setRate,
    blockNumberInterval=604803,
    status=success,
    agreeVoters=[
        0xc49fd7d7648ef4f4774d642d905db2e66f5089a8
    ],
    againstVoters=[

    ]
}
```

##### setDeployAuthTypeProposal 设置全局部署策略的提案

设置部署的ACL策略，只支持 white_list 和 black_list 两种策略

```shell
[group]: /> setDeployAuthTypeProposal white_list
Set deploy auth type proposal created, id is: 4

[group]: /> getDeployAuth
Deploy strategy is White List Access.
```

##### openDeployAuthProposal 开启某个管理员部署权限的提案

```shell
[group]: /> deploy HelloWorld
deploy contract for HelloWorld failed!
return message: Permission denied
return code:18
Return values:null

[group]: /> openDeployAuthProposal 0xc49fd7d7648ef4f4774d642d905db2e66f5089a8
Open deploy auth proposal created, id is: 5

[group]: /> deploy HelloWorld
transaction hash: 0x4708ba16df0df78db5472cffdd5a11c45977b6dcba047abfc346a89179e6a030
contract address: 0x6546c3571F17858Ea45575e7C6457dad03E53DbB
currentAccount: 0xc49fd7d7648ef4f4774d642d905db2e66f5089a8
```

##### closeDeployAuthProposal 关闭某个管理员部署权限的提案

```shell
[group]: /> deploy HelloWorld
transaction hash: 0x4708ba16df0df78db5472cffdd5a11c45977b6dcba047abfc346a89179e6a030
contract address: 0x6546c3571F17858Ea45575e7C6457dad03E53DbB
currentAccount: 0xc49fd7d7648ef4f4774d642d905db2e66f5089a8

[group]: /> closeDeployAuthProposal 0xc49fd7d7648ef4f4774d642d905db2e66f5089a8
Close deploy auth proposal created, id is: 6

[group]: /> deploy HelloWorld
deploy contract for HelloWorld failed!
return message: Permission denied
return code:18
Return values:null
```

##### resetAdminProposal 重置某个合约的管理员的提案

```shell
[group]: /> resetAdminProposal 0xc49fd7d7648ef4f4774d642d905db2e66f5089a8 0x6546c3571F17858Ea45575e7C6457dad03E53DbB
Reset contract admin proposal created, id is: 7

[group]: /> getContractAdmin 0x6546c3571F17858Ea45575e7C6457dad03E53DbB
Admin for contract 0x6546c3571F17858Ea45575e7C6457dad03E53DbB is: 0xc49fd7d7648ef4f4774d642d905db2e66f5089a8
```

##### revokeProposal 撤销还未成功的提案

##### voteProposal 投票提案

#### 2.9.2 管理员专用命令

**特别注意：合约的接口权限控制目前只能对写方法进行控制。**

##### setMethodAuth 管理员设置方法的权限策略

```shell
[group]: /> setMethodAuth 0xd24180cc0feF2f3E545de4F9AAFc09345cD08903 "set(string)" white_list
Set method auth type success, resultCode is: 0

[group]: /> call HelloWorld 0xd24180cc0feF2f3E545de4F9AAFc09345cD08903 set 123
call for HelloWorld failed, contractAddress: 0xd24180cc0feF2f3E545de4F9AAFc09345cD08903
{
    "code":18,
    "msg":"Permission denied"
}
description: Permission denied, please refer to https://fisco-bcos-documentation.readthedocs.io/zh_CN/latest/docs/api.html#id73
```

##### openMethodAuth	管理员开启用户可以访问合约的某个方法的权限

```shell
[group]: /> openMethodAuth  0x0102e8B6fC8cdF9626fDdC1C3Ea8C1E79b3FCE94 "set(string)" 0xc49fd7d7648ef4f4774d642d905db2e66f5089a8
Open success, resultCode is: 0
```

##### closeMethodAuth	管理员关闭用户可以访问合约的某个方法的权限

```shell
[group]: /> closeMethodAuth  0x0102e8B6fC8cdF9626fDdC1C3Ea8C1E79b3FCE94 "set(string)" 0xc49fd7d7648ef4f4774d642d905db2e66f5089a8
Close success, resultCode is: 0
```
