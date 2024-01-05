## Adding Org3 to the test network

You can use the `addOrg3.sh` script to add another organization to the Fabric test network. The `addOrg3.sh` script generates the Org3 crypto material, creates an Org3 organization definition, and adds Org3 to a channel on the test network.

You first need to run `./network.sh up createChannel` in the `test-network` directory before you can run the `addOrg3.sh` script.

```
./network.sh up createChannel
cd addOrg3
./addOrg3.sh up
```

If you used `network.sh` to create a channel other than the default `mychannel`, you need pass that name to the `addorg3.sh` script.

```
./network.sh up createChannel -c channel1
cd addOrg3
./addOrg3.sh up -c channel1
```

You can also re-run the `addOrg3.sh` script to add Org3 to additional channels.

```
cd ..
./network.sh createChannel -c channel2
cd addOrg3
./addOrg3.sh up -c channel2
```

For more information, use `./addOrg3.sh -h` to see the `addOrg3.sh` help text.

# Adding Org3 to the test network standalone

> [**官网文档**](https://hyperledger-fabric.readthedocs.io/en/release-2.5/channel_update_tutorial.html#)

**1、Orderer1**

```shell
scp -r channel-artifacts/ organizations/ peer0.org3.example.com:/data/fabric/fabric-peer3/test-network
```

**2、Peer3**

```shell
cd /data/fabric/fabric-peer3/test-network/addOrg3
./addOrg3.sh up -c carbonchain
```

**3、Set environment**

```shell
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org3MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org3.example.com/users/Admin@org3.example.com/msp
export CORE_PEER_ADDRESS=localhost:11051
```

**4、打包 Basic 链码**

```shell
peer lifecycle chaincode package basic.tar.gz --path ../asset-transfer-basic/chaincode-go/ --lang golang --label basic_1
scp -r basic.tar.gz peer0.org1.example.com:/data/fabric/fabric-peer1/test-network
scp -r basic.tar.gz peer0.org2.example.com:/data/fabric/fabric-peer2/test-network
```

**5、安装链代码包**

```shell
[root@peer3 test-network]# peer lifecycle chaincode install basic.tar.gz
2024-01-04 16:48:00.634 CST 0001 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Installed remotely: response:<status:200 payload:"\nHbasic_1:ef2394600055b69053a488d0ea2ac66bd544e93bb1b272a68d8860df5ac82c8c\022\007basic_1" >
2024-01-04 16:48:00.634 CST 0002 INFO [cli.lifecycle.chaincode] submitInstallProposal -> Chaincode code package identifier: basic_1:ef2394600055b69053a488d0ea2ac66bd544e93bb1b272a68d8860df5ac82c8c
```

**6、批准 Basic 的链码定义为 Org3**

> Org3 需要审议和 Org1 和 Org2 一样的链码定义并提交到通道中。为了调用链码，Org3 需要包含定义标识在链码定义中。你可以通过 peer 查询来找链码标识

```shell
[root@peer3 test-network]# peer lifecycle chaincode queryinstalled
Installed chaincodes on peer:
Package ID: basic_1:ef2394600055b69053a488d0ea2ac66bd544e93bb1b272a68d8860df5ac82c8c, Label: basic_1
```

**7、 以后会用到 package ID,所以将其放入环境变量中，注意更换成你自己的 Packge ID**

```shell
[root@peer3 test-network]# export CC_PACKAGE_ID=basic_1:ef2394600055b69053a488d0ea2ac66bd544e93bb1b272a68d8860df5ac82c8c
```

**8、批准 Org3 基本链码的定义**

```shell
# use the --package-id flag to provide the package identifier(使用--package-id标志提供包标识符)
# use the --init-required flag to require the execution of an initialization function before other chaincode functions can be called(使用--init-required标志要求在调用其他链码函数之前执行初始化函数)
[root@peer3 test-network]# peer lifecycle chaincode approveformyorg -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" --channelID carbonchain --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1
```

**9、peer1**

```shelll
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org1MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org1.example.com/users/Admin@org1.example.com/msp
export CORE_PEER_ADDRESS=localhost:7051
peer lifecycle chaincode install basic.tar.gz
export CC_PACKAGE_ID=basic_1:ef2394600055b69053a488d0ea2ac66bd544e93bb1b272a68d8860df5ac82c8c
peer lifecycle chaincode approveformyorg -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" --channelID carbonchain --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1
```

**10、peer2**

```shell
export PATH=${PWD}/../bin:$PATH
export FABRIC_CFG_PATH=$PWD/../config/
export CORE_PEER_TLS_ENABLED=true
export CORE_PEER_LOCALMSPID="Org2MSP"
export CORE_PEER_TLS_ROOTCERT_FILE=${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt
export CORE_PEER_MSPCONFIGPATH=${PWD}/organizations/peerOrganizations/org2.example.com/users/Admin@org2.example.com/msp
export CORE_PEER_ADDRESS=localhost:9051
peer lifecycle chaincode install basic.tar.gz
export CC_PACKAGE_ID=basic_1:ef2394600055b69053a488d0ea2ac66bd544e93bb1b272a68d8860df5ac82c8c
peer lifecycle chaincode approveformyorg -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" --channelID carbonchain --name basic --version 1.0 --package-id $CC_PACKAGE_ID --sequence 1
```

**11、peer3**

> 将链码定义提交到通道，并完成实例化过程。确保在执行该命令时，指定了正确的链码名称、版本和序列号，并等待链码在通道上成功实例化。

```shell
peer lifecycle chaincode commit -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" --channelID carbonchain --name basic --version 1.0 --sequence 1 --peerAddresses peer0.org1.example.com:7051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org1.example.com/peers/peer0.org1.example.com/tls/ca.crt" --peerAddresses peer0.org2.example.com:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt"
```

**12、检查您批准的链码定义是否已提交到通道**

```shell
[root@peer3 test-network]# peer lifecycle chaincode querycommitted --channelID carbonchain --name basic --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem"
Committed chaincode definition for chaincode 'basic' on channel 'carbonchain':(在通道'carbonchain'上提交链码'basic'的链码定义:)
Version: 1.0, Sequence: 1, Endorsement Plugin: escc, Validation Plugin: vscc, Approvals: [Org1MSP: true, Org2MSP: true, Org3MSP: true]
```

**13、Org3 在批准提交给通道的链码定义后可以使用基本链码。**

> 链码定义使用默认的`背书(endorsement )策略`，这需要通道上的大多数组织对交易进行赞同。这意味着如果一个组织被添加到通道或从通道中删除，背书策略将自动更新。我们之前需要 Org1 和 Org2 的认可（二选二）。现在我们需要 Org1、Org2 和 Org3 中的两个组织的认可（3 个中的 2 个）。
> 查询账本确保链码已经在 Org3 节点运行。注意我们此时需要链码容器启动

```shell
用一些示例资产填充分类帐。我们将从Org2对等体和新的Org3对等体获得endorsements，以使endorsement策略得到满足:
[root@peer3 test-network]# peer chaincode invoke -o orderer.example.com:7050 --ordererTLSHostnameOverride orderer.example.com --tls --cafile "${PWD}/organizations/ordererOrganizations/example.com/orderers/orderer.example.com/msp/tlscacerts/tlsca.example.com-cert.pem" -C carbonchain  -n basic --peerAddresses peer0.org2.example.com:9051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org2.example.com/peers/peer0.org2.example.com/tls/ca.crt" --peerAddresses peer0.org3.example.com:11051 --tlsRootCertFiles "${PWD}/organizations/peerOrganizations/org3.example.com/peers/peer0.org3.example.com/tls/ca.crt" -c '{"function":"InitLedger","Args":[]}'
2024-01-05 17:17:44.331 CST 0001 INFO [chaincodeCmd] chaincodeInvokeOrQuery -> Chaincode invoke successful. result: status:200
```

**14、您可以查询链码以确保 Org3 对等方已提交数据:**

```shell
peer chaincode query -C carbonchain -n basic -c '{"Args":["GetAllAssets"]}'
```

您应该看到作为响应添加到分类帐的初始资产列表。
