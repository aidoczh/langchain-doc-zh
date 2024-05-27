# 区块链

## 概述

这个笔记本的目的是为了测试 Langchain 区块链文档加载器的功能。

最初，该加载器支持：

*  从 NFT 智能合约 (ERC721 和 ERC1155) 中加载 NFT 文档

*  以太坊主网、以太坊测试网、Polygon 主网、Polygon 测试网 (默认为 eth-mainnet)

*  Alchemy 的 getNFTsForCollection API

如果社区发现这个加载器有价值，它可以进行扩展。具体来说：

*  可以添加额外的 API (例如与交易相关的 API)

这个文档加载器需要：

*  一个免费的 [Alchemy API Key](https://www.alchemy.com/)

输出采用以下格式：

- pageContent= 单个 NFT

- metadata={'source': '0x1a92f7381b9f03921564a437210bb9396471050c', 'blockchain': 'eth-mainnet', 'tokenId': '0x15'})

## 将 NFT 加载到文档加载器中

```python
# 从 https://www.alchemy.com/ 获取 ALCHEMY_API_KEY
alchemyApiKey = "..."
```

### 选项 1: 以太坊主网 (默认区块链类型)

```python
from langchain_community.document_loaders.blockchain import (
    BlockchainDocumentLoader,
    BlockchainType,
)
contractAddress = "0xbc4ca0eda7647a8ab7c2061c2e118a18a936f13d"  # Bored Ape Yacht Club 合约地址
blockchainType = BlockchainType.ETH_MAINNET  # 默认值，可选参数
blockchainLoader = BlockchainDocumentLoader(
    contract_address=contractAddress, api_key=alchemyApiKey
)
nfts = blockchainLoader.load()
nfts[:2]
```

### 选项 2: Polygon 主网

```python
contractAddress = (
    "0x448676ffCd0aDf2D85C1f0565e8dde6924A9A7D9"  # Polygon 主网合约地址
)
blockchainType = BlockchainType.POLYGON_MAINNET
blockchainLoader = BlockchainDocumentLoader(
    contract_address=contractAddress,
    blockchainType=blockchainType,
    api_key=alchemyApiKey,
)
nfts = blockchainLoader.load()
nfts[:2]
```