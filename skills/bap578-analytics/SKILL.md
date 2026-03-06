---
id: bap578-analytics
name: BAP-578 Analytics & Indexing
description: >
  Build analytics dashboards, on-chain indexers, and data pipelines for
  BAP-578 Non-Fungible Agents. Track minting activity, agent economics,
  TVL, user growth, engagement metrics, and event history using The Graph,
  custom indexers, or direct RPC queries. Use when analyzing NFA data or
  building monitoring tools.
category: BAP-578
author: community
version: 1.0.0
examples:
  - "How many BAP-578 agents have been minted?"
  - "Build an analytics dashboard for NFA"
  - "Track treasury revenue from agent mints"
  - "Index BAP-578 events with The Graph"
  - "What is the TVL in NFA agents?"
  - "Show minting activity over time"
---

# BAP-578 Analytics & Indexing

Use this skill when building analytics dashboards, indexers, or data pipelines for BAP-578 Non-Fungible Agents. This covers metrics definition, event indexing, query patterns, and dashboard building for monitoring NFA ecosystem health.

---

## The Four Identity Questions — Analytics Perspective

### 1 · Who are you?

I am the **data layer** for BAP-578. I answer: **"What's happening across the NFA ecosystem, and what does it mean?"**

I transform raw on-chain events and state into structured metrics: adoption, revenue, engagement, TVL, and user behavior. I power dashboards, alerts, and business intelligence.

**In short:** "I turn blockchain data into business insights. I watch everything so you can make informed decisions."

### 2 · What do you remember?

I index and aggregate **every on-chain event** the BAP-578 contract emits:

| Event | Data points | Analytics use |
|-------|-------------|--------------|
| `AgentCreated` | tokenId, owner, logicAddress, metadataURI | Mint volume, user acquisition, time series |
| `AgentFunded` | tokenId, amount | Fund inflows, TVL growth, agent popularity |
| `AgentWithdraw` | tokenId, amount | Fund outflows, net TVL change |
| `AgentStatusChanged` | tokenId, active | Active agent count, churn rate |
| `MetadataUpdated` | tokenId | Engagement frequency, personalization rate |
| `LogicAddressUpdated` | tokenId, newLogicAddress | Logic contract adoption |
| `TreasuryUpdated` | newTreasury | Admin activity |
| `ContractPaused` | paused | Operational incidents |
| `FreeMintGranted` | user, amount | Campaign tracking, influencer ROI |
| `Transfer` (ERC-721) | from, to, tokenId | Secondary market activity |

### 3 · What can you do?

#### Key Metrics

| Metric | Formula | Query |
|--------|---------|-------|
| Total agents | `getTotalSupply()` | Direct contract call |
| Active agents | Count where `active == true` | Iterate or index `AgentStatusChanged` |
| Unique owners | Distinct `owner` from `tokensOfOwner` | Index `Transfer` events |
| Revenue (BNB) | Count paid mints × 0.01 | Count `AgentCreated` after free mints exhausted |
| TVL | Sum of all `agentState.balance` | Aggregate `AgentFunded` - `AgentWithdraw` |
| Free-to-paid conversion | Users with >3 agents / total users | Track per-user mint count |
| Engagement rate | `MetadataUpdated` events / total agents | Event count ratio |
| Agent lifespan | Time between creation and last activity | `createdAt` vs latest event |

#### The Graph Subgraph

**`subgraph.yaml`:**

```yaml
specVersion: 0.0.5
schema:
  file: ./schema.graphql
dataSources:
  - kind: ethereum
    name: BAP578
    network: bsc
    source:
      address: "0xYourProxyAddress"
      abi: BAP578
      startBlock: 38000000
    mapping:
      kind: ethereum/events
      apiVersion: 0.0.7
      language: wasm/assemblyscript
      entities:
        - Agent
        - User
        - FundEvent
        - DailyStats
      abis:
        - name: BAP578
          file: ./abis/BAP578.json
      eventHandlers:
        - event: AgentCreated(indexed uint256,indexed address,address,string)
          handler: handleAgentCreated
        - event: AgentFunded(indexed uint256,uint256)
          handler: handleAgentFunded
        - event: AgentWithdraw(indexed uint256,uint256)
          handler: handleAgentWithdraw
        - event: AgentStatusChanged(indexed uint256,bool)
          handler: handleAgentStatusChanged
        - event: MetadataUpdated(indexed uint256)
          handler: handleMetadataUpdated
        - event: Transfer(indexed address,indexed address,indexed uint256)
          handler: handleTransfer
```

**`schema.graphql`:**

```graphql
type Agent @entity {
  id: ID!
  tokenId: BigInt!
  owner: User!
  active: Boolean!
  balance: BigDecimal!
  createdAt: BigInt!
  metadataURI: String
  logicAddress: Bytes
  isFreeMint: Boolean!
  fundEvents: [FundEvent!]! @derivedFrom(field: "agent")
  totalFunded: BigDecimal!
  totalWithdrawn: BigDecimal!
  metadataUpdateCount: Int!
  lastActivityAt: BigInt!
}

type User @entity {
  id: ID!
  address: Bytes!
  agents: [Agent!]! @derivedFrom(field: "owner")
  agentCount: Int!
  totalFunded: BigDecimal!
  firstMintAt: BigInt
}

type FundEvent @entity {
  id: ID!
  agent: Agent!
  amount: BigDecimal!
  timestamp: BigInt!
  type: String! # "fund" or "withdraw"
}

type DailyStats @entity {
  id: ID!
  date: String!
  mintCount: Int!
  fundVolume: BigDecimal!
  withdrawVolume: BigDecimal!
  activeAgents: Int!
  uniqueUsers: Int!
  tvl: BigDecimal!
}
```

**Example GraphQL queries:**

```graphql
# Top 10 agents by balance
{
  agents(first: 10, orderBy: balance, orderDirection: desc) {
    tokenId
    owner { address }
    balance
    active
    createdAt
  }
}

# Daily minting activity
{
  dailyStats(first: 30, orderBy: date, orderDirection: desc) {
    date
    mintCount
    fundVolume
    tvl
    uniqueUsers
  }
}

# Power users (most agents)
{
  users(first: 20, orderBy: agentCount, orderDirection: desc) {
    address
    agentCount
    totalFunded
  }
}
```

#### Simple RPC-Based Analytics Script

For quick analytics without The Graph:

```javascript
const { ethers } = require("ethers");

const provider = new ethers.providers.JsonRpcProvider("https://bsc-dataseed.binance.org/");
const contract = new ethers.Contract(PROXY_ADDRESS, BAP578_ABI, provider);

async function getAnalytics() {
    const totalSupply = await contract.getTotalSupply();
    console.log("Total Agents:", totalSupply.toString());

    // Count events
    const fromBlock = DEPLOY_BLOCK;
    const toBlock = "latest";

    const created = await contract.queryFilter(
        contract.filters.AgentCreated(), fromBlock, toBlock
    );
    const funded = await contract.queryFilter(
        contract.filters.AgentFunded(), fromBlock, toBlock
    );
    const withdrawn = await contract.queryFilter(
        contract.filters.AgentWithdraw(), fromBlock, toBlock
    );

    // Revenue calculation
    let totalFundedBNB = ethers.BigNumber.from(0);
    for (const e of funded) {
        totalFundedBNB = totalFundedBNB.add(e.args.amount);
    }

    let totalWithdrawnBNB = ethers.BigNumber.from(0);
    for (const e of withdrawn) {
        totalWithdrawnBNB = totalWithdrawnBNB.add(e.args.amount);
    }

    console.log("Mint Events:", created.length);
    console.log("Fund Events:", funded.length);
    console.log("Total Funded:", ethers.utils.formatEther(totalFundedBNB), "BNB");
    console.log("Total Withdrawn:", ethers.utils.formatEther(totalWithdrawnBNB), "BNB");
    console.log("Net TVL:", ethers.utils.formatEther(
        totalFundedBNB.sub(totalWithdrawnBNB)
    ), "BNB");

    // Unique owners
    const owners = new Set(created.map(e => e.args.owner));
    console.log("Unique Owners:", owners.size);
}
```

#### Dashboard Metrics Template

```
╔═══════════════════════════════════════╗
║      NFA Analytics Dashboard          ║
╠═══════════════════════════════════════╣
║                                       ║
║  SUPPLY        1,234 total agents     ║
║  ACTIVE          987 (80.0%)          ║
║  OWNERS          456 unique           ║
║                                       ║
║  REVENUE                              ║
║  Paid Mints      284 × 0.01 BNB      ║
║  Total Rev       2.84 BNB            ║
║  Treasury        2.10 BNB            ║
║                                       ║
║  TVL                                  ║
║  Total Funded    150.5 BNB            ║
║  Total Withdrawn  45.2 BNB           ║
║  Net TVL         105.3 BNB            ║
║                                       ║
║  ENGAGEMENT (30d)                     ║
║  Metadata Updates  340                ║
║  Fund Events       210                ║
║  Status Changes     89                ║
║                                       ║
║  CONVERSION                           ║
║  Free→Paid Rate   18.4%              ║
║  Avg Agents/User  2.7                ║
║                                       ║
╚═══════════════════════════════════════╝
```

### 4 · How can I trust it?

Analytics trust comes from **data provenance**:

- **All data originates from the chain** — events and state are read directly from the BAP-578 proxy via RPC or subgraph indexer.
- **Events are immutable** — once emitted, event data cannot be altered.
- **Aggregations are reproducible** — anyone can rerun the same queries and get the same results.
- **The Graph is decentralized** — subgraph data is indexed by multiple nodes and verifiable.
- **Metrics formulas are transparent** — every metric definition is documented above.

**In short:** "Trust the analytics because every number traces back to an immutable on-chain event or state read. Reproduce any metric yourself with the same queries."

---

## Related Skills

- **`bap578-scanner`** — Single-agent lookup and verification
- **`bap578-business`** — Business metrics interpretation and strategy
- **`frontend-web3-bap578`** — Displaying analytics in React dashboards
