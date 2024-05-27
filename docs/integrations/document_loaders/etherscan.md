# Etherscan

[Etherscan](https://docs.etherscan.io/) 是领先的以太坊区块链浏览器、搜索、API 和分析平台，用于以太坊这一去中心化智能合约平台。

## 概述

`Etherscan` 加载器使用 `Etherscan API` 在 `以太坊主网` 下载特定账户的交易历史记录。

您需要一个 `Etherscan API 密钥` 才能继续。免费的 API 密钥每秒有 5 次调用配额。

该加载器支持以下六个功能：

- 在以太坊主网下检索特定账户的普通交易
- 在以太坊主网下检索特定账户的内部交易
- 在以太坊主网下检索特定账户的 erc20 交易
- 在以太坊主网下检索特定账户的 erc721 交易
- 在以太坊主网下检索特定账户的 erc1155 交易
- 在以太坊主网下检索特定账户的 wei 以太币余额

如果账户没有相应的交易，加载器将返回一个包含一个文档的列表。文档的内容为''。

您可以向加载器传递不同的过滤器以访问我们上面提到的不同功能：

- "normal_transaction"
- "internal_transaction"
- "erc20_transaction"
- "eth_balance"
- "erc721_transaction"
- "erc1155_transaction"
默认情况下，过滤器设置为 normal_transaction。

如果您有任何问题，可以访问[Etherscan API 文档](https://etherscan.io/tx/0x0ffa32c787b1398f44303f731cb06678e086e4f82ce07cebf75e99bb7c079c77)，或通过 i@inevitable.tech 联系我。

由于 Etherscan 限制，与交易历史记录相关的所有功能最多只能返回 1000 条历史记录。您可以使用以下参数找到所需的交易历史记录：

- offset：默认为 20。一次显示 20 笔交易
- page：默认为 1。这控制分页。
- start_block：默认为 0。交易历史记录从 0 区块开始。
- end_block：默认为 99999999。交易历史记录从 99999999 区块开始
- sort： "desc" 或 "asc"。默认设置为 "desc" 以获取最新交易。

## 设置

```python
%pip install --upgrade --quiet  langchain -q
```

```python
etherscanAPIKey = "..."
```

```python
import os
from langchain_community.document_loaders import EtherscanLoader
```

```python
os.environ["ETHERSCAN_API_KEY"] = etherscanAPIKey
```

## 创建一个 ERC20 交易加载器

```python
account_address = "0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b"
loader = EtherscanLoader(account_address, filter="erc20_transaction")
result = loader.load()
eval(result[0].page_content)
```

```output
{'blockNumber': '13242975',
 'timeStamp': '1631878751',
 'hash': '0x366dda325b1a6570928873665b6b418874a7dedf7fee9426158fa3536b621788',
 'nonce': '28',
 'blockHash': '0x5469dba1b1e1372962cf2be27ab2640701f88c00640c4d26b8cc2ae9ac256fb6',
 'from': '0x2ceee24f8d03fc25648c68c8e6569aa0512f6ac3',
 'contractAddress': '0x2ceee24f8d03fc25648c68c8e6569aa0512f6ac3',
 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b',
 'value': '298131000000000',
 'tokenName': 'ABCHANGE.io',
 'tokenSymbol': 'XCH',
 'tokenDecimal': '9',
 'transactionIndex': '71',
 'gas': '15000000',
 'gasPrice': '48614996176',
 'gasUsed': '5712724',
 'cumulativeGasUsed': '11507920',
 'input': 'deprecated',
 'confirmations': '4492277'}
```

## 创建一个带有自定义参数的普通交易加载器

```python
loader = EtherscanLoader(
    account_address,
    page=2,
    offset=20,
    start_block=10000,
    end_block=8888888888,
    sort="asc",
)
result = loader.load()
result
```

```output
20
```
```output
[Document(page_content="{'blockNumber': '1723771', 'timeStamp': '1466213371', 'hash': '0xe00abf5fa83a4b23ee1cc7f07f9dda04ab5fa5efe358b315df8b76699a83efc4', 'nonce': '3155', 'blockHash': '0xc2c2207bcaf341eed07f984c9a90b3f8e8bdbdbd2ac6562f8c2f5bfa4b51299d', 'transactionIndex': '5', 'from': '0x3763e6e1228bfeab94191c856412d1bb0a8e6996', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b', 'value': '13149213761000000000', 'gas': '90000', 'gasPrice': '22655598156', 'isError': '0', 'txreceipt_status': '', 'input': '0x', 'contractAddress': '', 'cumulativeGasUsed': '126000', 'gasUsed': '21000', 'confirmations': '16011481', 'methodId': '0x', 'functionName': ''}", metadata={'from': '0x3763e6e1228bfeab94191c856412d1bb0a8e6996', 'tx_hash': '0xe00abf5fa83a4b23ee1cc7f07f9dda04ab5fa5efe358b315df8b76699a83efc4', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b'}),
 Document(page_content="{'blockNumber': '1727090', 'timeStamp': '1466262018', 'hash': '0xd5a779346d499aa722f72ffe7cd3c8594a9ddd91eb7e439e8ba92ceb7bc86928', 'nonce': '3267', 'blockHash': '0xc0cff378c3446b9b22d217c2c5f54b1c85b89a632c69c55b76cdffe88d2b9f4d', 'transactionIndex': '20', 'from': '0x3763e6e1228bfeab94191c856412d1bb0a8e6996', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b', 'value': '11521979886000000000', 'gas': '90000', 'gasPrice': '20000000000', 'isError': '0', 'txreceipt_status': '', 'input': '0x', 'contractAddress': '', 'cumulativeGasUsed': '3806725', 'gasUsed': '21000', 'confirmations': '16008162', 'methodId': '0x', 'functionName': ''}", metadata={'from': '0x3763e6e1228bfeab94191c856412d1bb0a8e6996', 'tx_hash': '0xd5a779346d499aa722f72ffe7cd3c8594a9ddd91eb7e439e8ba92ceb7bc86928', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b'}),
 Document(page_content="{'blockNumber': '1730337', 'timeStamp': '1466308222', 'hash': '0xceaffdb3766d2741057d402738eb41e1d1941939d9d438c102fb981fd47a87a4', 'nonce': '3344', 'blockHash': '0x3a52d28b8587d55c621144a161a0ad5c37dd9f7d63b629ab31da04fa410b2cfa', 'transactionIndex': '1', 'from': '0x3763e6e1228bfeab94191c856412d1bb0a8e6996', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b', 'value': '9783400526000000000', 'gas': '90000', 'gasPrice': '20000000000', 'isError': '0', 'txreceipt_status': '', 'input': '0x', 'contractAddress': '', 'cumulativeGasUsed': '60788', 'gasUsed': '21000', 'confirmations': '16004915', 'methodId': '0x', 'functionName': ''}", metadata={'from': '0x3763e6e1228bfeab94191c856412d1bb0a8e6996', 'tx_hash': '0xceaffdb3766d2741057d402738eb41e1d1941939d9d438c102fb981fd47a87a4', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b'}),
 Document(page_content="{'blockNumber': '1733479', 'timeStamp': '1466352351', 'hash': '0x720d79bf78775f82b40280aae5abfc347643c5f6708d4bf4ec24d65cd01c7121', 'nonce': '3367', 'blockHash': '0x9928661e7ae125b3ae0bcf5e076555a3ee44c52ae31bd6864c9c93a6ebb3f43e', 'transactionIndex': '0', 'from': '0x3763e6e1228bfeab94191c856412d1bb0a8e6996', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b', 'value': '1570706444000000000', 'gas': '90000', 'gasPrice': '20000000000', 'isError': '0', 'txreceipt_status': '', 'input': '0x', 'contractAddress': '', 'cumulativeGasUsed': '21000', 'gasUsed': '21000', 'confirmations': '16001773', 'methodId': '0x', 'functionName': ''}", metadata={'from': '0x3763e6e1228bfeab94191c856412d1bb0a8e6996', 'tx_hash': '0x720d79bf78775f82b40280aae5abfc347643c5f6708d4bf4ec24d65cd01c7121', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b'}),
 Document(page_content="{'blockNumber': '1734172', 'timeStamp': '1466362463', 'hash': '0x7a062d25b83bafc9fe6b22bc6f5718bca333908b148676e1ac66c0adeccef647', 'nonce': '1016', 'blockHash': '0x8a8afe2b446713db88218553cfb5dd202422928e5e0bc00475ed2f37d95649de', 'transactionIndex': '4', 'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b', 'value': '6322276709000000000', 'gas': '90000', 'gasPrice': '20000000000', 'isError': '0', 'txreceipt_status': '', 'input': '0x', 'contractAddress': '', 'cumulativeGasUsed': '105333', 'gasUsed': '21000', 'confirmations': '16001080', 'methodId': '0x', 'functionName': ''}", metadata={'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'tx_hash': '0x7a062d25b83bafc9fe6b22bc6f5718bca333908b148676e1ac66c0adeccef647', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b'}),
 Document(page_content="{'blockNumber': '1737276', 'timeStamp': '1466406037', 'hash': '0xa4e89bfaf075abbf48f96700979e6c7e11a776b9040113ba64ef9c29ac62b19b', 'nonce': '1024', 'blockHash': '0xe117cad73752bb485c3bef24556e45b7766b283229180fcabc9711f3524b9f79', 'transactionIndex': '35', 'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b', 'value': '9976891868000000000', 'gas': '90000', 'gasPrice': '20000000000', 'isError': '0', 'txreceipt_status': '', 'input': '0x', 'contractAddress': '', 'cumulativeGasUsed': '3187163', 'gasUsed': '21000', 'confirmations': '15997976', 'methodId': '0x', 'functionName': ''}", metadata={'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'tx_hash': '0xa4e89bfaf075abbf48f96700979e6c7e11a776b9040113ba64ef9c29ac62b19b', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b'}),
 Document(page_content="{'blockNumber': '1740314', 'timeStamp': '1466450262', 'hash': '0x6e1a22dcc6e2c77a9451426fb49e765c3c459dae88350e3ca504f4831ec20e8a', 'nonce': '1051', 'blockHash': '0x588d17842819a81afae3ac6644d8005c12ce55ddb66c8d4c202caa91d4e8fdbe', 'transactionIndex': '6', 'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b', 'value': '8060633765000000000', 'gas': '90000', 'gasPrice': '22926905859', 'isError': '0', 'txreceipt_status': '', 'input': '0x', 'contractAddress': '', 'cumulativeGasUsed': '153077', 'gasUsed': '21000', 'confirmations': '15994938', 'methodId': '0x', 'functionName': ''}", metadata={'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'tx_hash': '0x6e1a22dcc6e2c77a9451426fb49e765c3c459dae88350e3ca504f4831ec20e8a', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b'}),
 Document(page_content="{'blockNumber': '1743384', 'timeStamp': '1466494099', 'hash': '0xdbfcc15f02269fc3ae27f69e344a1ac4e08948b12b76ebdd78a64d8cafd511ef', 'nonce': '1068', 'blockHash': '0x997245108c84250057fda27306b53f9438ad40978a95ca51d8fd7477e73fbaa7', 'transactionIndex': '2', 'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b', 'value': '9541921352000000000', 'gas': '90000', 'gasPrice': '20000000000', 'isError': '0', 'txreceipt_status': '', 'input': '0x', 'contractAddress': '', 'cumulativeGasUsed': '119650', 'gasUsed': '21000', 'confirmations': '15991868', 'methodId': '0x', 'functionName': ''}", metadata={'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'tx_hash': '0xdbfcc15f02269fc3ae27f69e344a1ac4e08948b12b76ebdd78a64d8cafd511ef', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b'}),
 Document(page_content="{'blockNumber': '1746405', 'timeStamp': '1466538123', 'hash': '0xbd4f9602f7fff4b8cc2ab6286efdb85f97fa114a43f6df4e6abc88e85b89e97b', 'nonce': '1092', 'blockHash': '0x3af3966cdaf22e8b112792ee2e0edd21ceb5a0e7bf9d8c168a40cf22deb3690c', 'transactionIndex': '0', 'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b', 'value': '8433783799000000000', 'gas': '90000', 'gasPrice': '25689279306', 'isError': '0', 'txreceipt_status': '', 'input': '0x', 'contractAddress': '', 'cumulativeGasUsed': '21000', 'gasUsed': '21000', 'confirmations': '15988847', 'methodId': '0x', 'functionName': ''}", metadata={'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'tx_hash': '0xbd4f9602f7fff4b8cc2ab6286efdb85f97fa114a43f6df4e6abc88e85b89e97b', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b'}),
 Document(page_content="{'blockNumber': '1749459', 'timeStamp': '1466582044', 'hash': '0x28c327f462cc5013d81c8682c032f014083c6891938a7bdeee85a1c02c3e9ed4', 'nonce': '1096', 'blockHash': '0x5fc5d2a903977b35ce1239975ae23f9157d45d7bd8a8f6205e8ce270000797f9', 'transactionIndex': '1', 'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b', 'value': '10269065805000000000', 'gas': '90000', 'gasPrice': '20000000000', 'isError': '0', 'txreceipt_status': '', 'input': '0x', 'contractAddress': '', 'cumulativeGasUsed': '42000', 'gasUsed': '21000', 'confirmations': '15985793', 'methodId': '0x', 'functionName': ''}", metadata={'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'tx_hash': '0x28c327f462cc5013d81c8682c032f014083c6891938a7bdeee85a1c02c3e9ed4', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b'}),
 Document(page_content="{'blockNumber': '1752614', 'timeStamp': '1466626168', 'hash': '0xc3849e550ca5276d7b3c51fa95ad3ae62c1c164799d33f4388fe60c4e1d4f7d8', 'nonce': '1118', 'blockHash': '0x88ef054b98e47504332609394e15c0a4467f84042396717af6483f0bcd916127', 'transactionIndex': '11', 'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b', 'value': '11325836780000000000', 'gas': '90000', 'gasPrice': '20000000000', 'isError': '0', 'txreceipt_status': '', 'input': '0x', 'contractAddress': '', 'cumulativeGasUsed': '252000', 'gasUsed': '21000', 'confirmations': '15982638', 'methodId': '0x', 'functionName': ''}", metadata={'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'tx_hash': '0xc3849e550ca5276d7b3c51fa95ad3ae62c1c164799d33f4388fe60c4e1d4f7d8', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b'}),
 Document(page_content="{'blockNumber': '1755659', 'timeStamp': '1466669931', 'hash': '0xb9f891b7c3d00fcd64483189890591d2b7b910eda6172e3bf3973c5fd3d5a5ae', 'nonce': '1133', 'blockHash': '0x2983972217a91343860415d1744c2a55246a297c4810908bbd3184785bc9b0c2', 'transactionIndex': '14', 'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b', 'value': '13226475343000000000', 'gas': '90000', 'gasPrice': '20000000000', 'isError': '0', 'txreceipt_status': '', 'input': '0x', 'contractAddress': '', 'cumulativeGasUsed': '2674679', 'gasUsed': '21000', 'confirmations': '15979593', 'methodId': '0x', 'functionName': ''}", metadata={'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'tx_hash': '0xb9f891b7c3d00fcd64483189890591d2b7b910eda6172e3bf3973c5fd3d5a5ae', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b'}),
 Document(page_content="{'blockNumber': '1758709', 'timeStamp': '1466713652', 'hash': '0xd6cce5b184dc7fce85f305ee832df647a9c4640b68e9b79b6f74dc38336d5622', 'nonce': '1147', 'blockHash': '0x1660de1e73067251be0109d267a21ffc7d5bde21719a3664c7045c32e771ecf9', 'transactionIndex': '1', 'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b', 'value': '9758447294000000000', 'gas': '90000', 'gasPrice': '20000000000', 'isError': '0', 'txreceipt_status': '', 'input': '0x', 'contractAddress': '', 'cumulativeGasUsed': '42000', 'gasUsed': '21000', 'confirmations': '15976543', 'methodId': '0x', 'functionName': ''}", metadata={'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'tx_hash': '0xd6cce5b184dc7fce85f305ee832df647a9c4640b68e9b79b6f74dc38336d5622', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b'}),
 Document(page_content="{'blockNumber': '1761783', 'timeStamp': '1466757809', 'hash': '0xd01545872629956867cbd65fdf5e97d0dde1a112c12e76a1bfc92048d37f650f', 'nonce': '1169', 'blockHash': '0x7576961afa4218a3264addd37a41f55c444dd534e9410dbd6f93f7fe20e0363e', 'transactionIndex': '2', 'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b', 'value': '10197126683000000000', 'gas': '90000', 'gasPrice': '20000000000', 'isError': '0', 'txreceipt_status': '', 'input': '0x', 'contractAddress': '', 'cumulativeGasUsed': '63000', 'gasUsed': '21000', 'confirmations': '15973469', 'methodId': '0x', 'functionName': ''}", metadata={'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'tx_hash': '0xd01545872629956867cbd65fdf5e97d0dde1a112c12e76a1bfc92048d37f650f', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b'}),
 Document(page_content="{'blockNumber': '1764895', 'timeStamp': '1466801683', 'hash': '0x620b91b12af7aac75553b47f15742e2825ea38919cfc8082c0666f404a0db28b', 'nonce': '1186', 'blockHash': '0x2e687643becd3c36e0c396a02af0842775e17ccefa0904de5aeca0a9a1aa795e', 'transactionIndex': '7', 'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b', 'value': '8690241462000000000', 'gas': '90000', 'gasPrice': '20000000000', 'isError': '0', 'txreceipt_status': '', 'input': '0x', 'contractAddress': '', 'cumulativeGasUsed': '168000', 'gasUsed': '21000', 'confirmations': '15970357', 'methodId': '0x', 'functionName': ''}", metadata={'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'tx_hash': '0x620b91b12af7aac75553b47f15742e2825ea38919cfc8082c0666f404a0db28b', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b'}),
 Document(page_content="{'blockNumber': '1767936', 'timeStamp': '1466845682', 'hash': '0x758efa27576cd17ebe7b842db4892eac6609e3962a4f9f57b7c84b7b1909512f', 'nonce': '1211', 'blockHash': '0xb01d8fd47b3554a99352ac3e5baf5524f314cfbc4262afcfbea1467b2d682898', 'transactionIndex': '0', 'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b', 'value': '11914401843000000000', 'gas': '90000', 'gasPrice': '20000000000', 'isError': '0', 'txreceipt_status': '', 'input': '0x', 'contractAddress': '', 'cumulativeGasUsed': '21000', 'gasUsed': '21000', 'confirmations': '15967316', 'methodId': '0x', 'functionName': ''}", metadata={'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'tx_hash': '0x758efa27576cd17ebe7b842db4892eac6609e3962a4f9f57b7c84b7b1909512f', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b'}),
 Document(page_content="{'blockNumber': '1770911', 'timeStamp': '1466888890', 'hash': '0x9d84470b54ab44b9074b108a0e506cd8badf30457d221e595bb68d63e926b865', 'nonce': '1212', 'blockHash': '0x79a9de39276132dab8bf00dc3e060f0e8a14f5e16a0ee4e9cc491da31b25fe58', 'transactionIndex': '0', 'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b', 'value': '10918214730000000000', 'gas': '90000', 'gasPrice': '20000000000', 'isError': '0', 'txreceipt_status': '', 'input': '0x', 'contractAddress': '', 'cumulativeGasUsed': '21000', 'gasUsed': '21000', 'confirmations': '15964341', 'methodId': '0x', 'functionName': ''}", metadata={'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'tx_hash': '0x9d84470b54ab44b9074b108a0e506cd8badf30457d221e595bb68d63e926b865', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b'}),
 Document(page_content="{'blockNumber': '1774044', 'timeStamp': '1466932983', 'hash': '0x958d85270b58b80f1ad228f716bbac8dd9da7c5f239e9f30d8edeb5bb9301d20', 'nonce': '1240', 'blockHash': '0x69cee390378c3b886f9543fb3a1cb2fc97621ec155f7884564d4c866348ce539', 'transactionIndex': '2', 'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b', 'value': '9979637283000000000', 'gas': '90000', 'gasPrice': '20000000000', 'isError': '0', 'txreceipt_status': '', 'input': '0x', 'contractAddress': '', 'cumulativeGasUsed': '63000', 'gasUsed': '21000', 'confirmations': '15961208', 'methodId': '0x', 'functionName': ''}", metadata={'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'tx_hash': '0x958d85270b58b80f1ad228f716bbac8dd9da7c5f239e9f30d8edeb5bb9301d20', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b'}),
 Document(page_content="{'blockNumber': '1777057', 'timeStamp': '1466976422', 'hash': '0xe76ca3603d2f4e7134bdd7a1c3fd553025fc0b793f3fd2a75cd206b8049e74ab', 'nonce': '1248', 'blockHash': '0xc7cacda0ac38c99f1b9bccbeee1562a41781d2cfaa357e8c7b4af6a49584b968', 'transactionIndex': '7', 'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b', 'value': '4556173496000000000', 'gas': '90000', 'gasPrice': '20000000000', 'isError': '0', 'txreceipt_status': '', 'input': '0x', 'contractAddress': '', 'cumulativeGasUsed': '168000', 'gasUsed': '21000', 'confirmations': '15958195', 'methodId': '0x', 'functionName': ''}", metadata={'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'tx_hash': '0xe76ca3603d2f4e7134bdd7a1c3fd553025fc0b793f3fd2a75cd206b8049e74ab', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b'}),
 Document(page_content="{'blockNumber': '1780120', 'timeStamp': '1467020353', 'hash': '0xc5ec8cecdc9f5ed55a5b8b0ad79c964fb5c49dc1136b6a49e981616c3e70bbe6', 'nonce': '1266', 'blockHash': '0xfc0e066e5b613239e1a01e6d582e7ab162ceb3ca4f719dfbd1a0c965adcfe1c5', 'transactionIndex': '1', 'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b', 'value': '11890330240000000000', 'gas': '90000', 'gasPrice': '20000000000', 'isError': '0', 'txreceipt_status': '', 'input': '0x', 'contractAddress': '', 'cumulativeGasUsed': '42000', 'gasUsed': '21000', 'confirmations': '15955132', 'methodId': '0x', 'functionName': ''}", metadata={'from': '0x16545fb79dbee1ad3a7f868b7661c023f372d5de', 'tx_hash': '0xc5ec8cecdc9f5ed55a5b8b0ad79c964fb5c49dc1136b6a49e981616c3e70bbe6', 'to': '0x9dd134d14d1e65f84b706d6f205cd5b1cd03a46b'})]
```
### 原文段落

Deep learning models have achieved remarkable success in various fields such as computer vision, natural language processing, and speech recognition. These models are often trained on large-scale datasets to learn complex patterns and relationships in the data. However, training deep learning models requires significant computational resources and can be time-consuming. To address this issue, researchers have proposed techniques to compress and accelerate deep learning models without significantly sacrificing their performance.

One popular approach to model compression is knowledge distillation, where a smaller model, known as the student model, is trained to mimic the behavior of a larger model, known as the teacher model. By transferring knowledge from the teacher model to the student model, the student model can achieve comparable performance to the teacher model while being more compact and computationally efficient.

Another common technique for model compression is quantization, where the weights and activations of the neural network are represented with lower precision to reduce the model size and computational complexity. For example, instead of using 32-bit floating-point numbers to represent weights and activations, quantization uses 8-bit integers or even binary values. This reduction in precision can lead to significant savings in memory and computational costs with minimal impact on model accuracy.

In addition to knowledge distillation and quantization, researchers have also explored other methods such as pruning, which involves removing unimportant weights from the model, and low-rank approximation, which approximates the weight matrices of the neural network with low-rank matrices. These techniques aim to reduce the number of parameters in the model and simplify its structure, leading to faster inference and lower memory footprint.

By applying these model compression techniques, deep learning models can be deployed more efficiently on resource-constrained devices such as mobile phones, IoT devices, and embedded systems. This enables a wide range of applications that benefit from the capabilities of deep learning models while operating within the constraints of limited computational resources.

![Model Compression](https://example.com/model_compression.jpg)

### 参考文献

- [20] Hinton, G., Vinyals, O., & Dean, J. (2015). Distilling the knowledge in a neural network. arXiv preprint arXiv:1503.02531.