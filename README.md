c# Uniscam V2 Subgraph

[Uniscam](https://swap.y3d.finance/) is a decentralized protocol for automated token exchange on Ethereum.

This subgraph dynamically tracks any pair created by the Uniscam factory. It tracks of the current state of Uniscam contracts, and contains derived stats for things like historical data and USD prices.

- aggregated data across pairs and tokens,
- data on individual pairs and tokens,
- data on transactions
- data on liquidity providers
- historical data on Uniscam, pairs or tokens, aggregated by day

## Q & A

### 可用命令
```bash
yarn # 安装编译、部署等依赖
yarn codegen # 通过 ABI 生成 typescript 对象，方便写 mapping
yarn build # 编译 subgraph
yarn create-bscgraph # 在 bscgraph 注册名字
yarn deploy # 部署到 bscgraph（必须先注册subgraph名字）
```

### 合约更新了地址？

Subgraph 只依赖于 Unisave 的 Factory 合约。

如果你更更换了 Factory 合约，请**全项目搜索你的原来 Factory 地址，并以新 Factory 地址代替**。

> 注意 Factory 地址必须是区分大小写的Checksumed格式。否则可能拿不到数据。

同时你需要创建两个 BNB 与稳定币（BUSD\USDT\DAI等）的交易对，请按照市价来创建。这个会影响到美元计价的功能模块。

创建后，在 `src/mappings/pricing.ts` 更新 BNB 与稳定币交易对的Pair地址。**注意地址要全小写**。否则可能拿不到数据。

### 合约更新了接口？

你需要把 ABI 放进 `abis`，并运行一下命令：

```bash
yarn codegen # 通过 ABI 生成 typescript 对象，方便写 mapping
```

## 需要本地测试？

请参考 TheGraph 的 Graph-Node 文档 [Quick Start](https://github.com/graphprotocol/graph-node#quick-start)

需要：

* 一个 IPFS 节点，用于上传编译后的 subgraph
* 一个 PostgresSQL 数据库，用于存放 GraphNode 聚合出来的数据


### 运行节点

需要根据文档 Quick Start 配置好环境，然后再启动本地节点进行测试

```bash
cargo run -p graph-node --release -- \
  --postgres-url postgresql://[YOUR_PG_USER_NAME]@[PQ_SQL_PASSWORD]localhost:5432/graph-node \
  --ethereum-rpc mainnet:https://bsc-dataseed.binance.org\
  --ipfs 127.0.0.1:5001
```

## Running Locally

Make sure to update package.json settings to point to your own graph account.

## Queries

Below are a few ways to show how to query the Uniscam-subgraph for data. The queries show most of the information that is queryable, but there are many other filtering options that can be used, just check out the [querying api](https://thegraph.com/docs/graphql-api). These queries can be used locally or in The Graph Explorer playground.

## Key Entity Overviews

#### UniscamFactory

Contains data across all of Uniscam V2. This entity tracks important things like total liquidity (in ETH and USD, see below), all time volume, transaction count, number of pairs and more.

#### Token

Contains data on a specific token. This token specific data is aggregated across all pairs, and is updated whenever there is a transaction involving that token.

#### Pair

Contains data on a specific pair.

#### Transaction

Every transaction on Uniscam is stored. Each transaction contains an array of mints, burns, and swaps that occured within it.

#### Mint, Burn, Swap

These contain specifc information about a transaction. Things like which pair triggered the transaction, amounts, sender, recipient, and more. Each is linked to a parent Transaction entity.

## Example Queries

### Querying Aggregated Uniscam Data

This query fetches aggredated data from all Uniscam pairs and tokens, to give a view into how much activity is happening within the whole protocol.

```graphql
{
  UniscamFactories(first: 1) {
    pairCount
    totalVolumeUSD
    totalLiquidityUSD
  }
}
```
