# Near 区块链

## 概述

本笔记的目的是为 Near 区块链的 Langchain 文档加载器提供功能测试的手段。

最初，该加载器支持：

* 从 NFT 智能合约加载文档（NEP-171 和 NEP-177）

* Near Mainnnet、Near Testnet（默认为 mainnet）

* Mintbase 的 Graph API

如果社区发现该加载器有价值，可以进行扩展，具体来说：

* 可以添加额外的 API（例如与交易相关的 API）

此文档加载器需要：

* 一个免费的 [Mintbase API Key](https://docs.mintbase.xyz/dev/mintbase-graph/)

输出采用以下格式：

- pageContent= 单个 NFT

- metadata={'source': 'nft.yearofchef.near', 'blockchain': 'mainnet', 'tokenId': '1846'}

## 将 NFT 加载到文档加载器中

```python
# 从 https://docs.mintbase.xyz/dev/mintbase-graph/ 获取 MINTBASE_API_KEY
mintbaseApiKey = "..."
```

### 选项 1：以太坊主网（默认 BlockchainType）

```python
from MintbaseLoader import MintbaseDocumentLoader
contractAddress = "nft.yearofchef.near"  # Year of chef 合约地址
blockchainLoader = MintbaseDocumentLoader(
    contract_address=contractAddress, blockchain_type="mainnet", api_key="omni-site"
)
nfts = blockchainLoader.load()
print(nfts[:1])
for doc in blockchainLoader.lazy_load():
    print()
    print(type(doc))
    print(doc)
```
