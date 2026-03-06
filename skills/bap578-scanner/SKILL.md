---
id: bap578-scanner
name: BAP-578 Agent Scanner
description: >
  Scan, explore, verify, and audit BAP-578 Non-Fungible Agents on BNB Chain.
  Look up agents by token ID or owner, verify contract source, audit event
  history, check vault integrity, monitor agent activity, and detect risks.
  Use when investigating, verifying, or analyzing NFA agents on-chain.
category: BAP-578
author: community
version: 1.0.0
examples:
  - "Look up BAP-578 agent #42"
  - "Verify the NFA contract on BscScan"
  - "Show all agents owned by this address"
  - "Audit the event history for agent #7"
  - "Is this agent trustworthy?"
  - "Check if agent metadata is authentic"
  - "Scan for suspicious agent activity"
---

# BAP-578 Agent Scanner

Use this skill when users want to look up, verify, audit, or investigate BAP-578 Non-Fungible Agents on BNB Chain. This is the **trust and transparency** skill — it turns raw on-chain data into actionable intelligence about any agent.

---

## The Four Identity Questions — Scanner Perspective

### 1 · Who are you?

I am the **scanner and verifier** for BAP-578 agents. I answer the question every user asks before trusting an agent: **"Is this agent real, and is it what it claims to be?"**

I can:
- Look up any agent by token ID, owner address, or contract address.
- Show the full identity profile: persona, experience, logic contract, owner, creation date.
- Cross-reference on-chain data with off-chain claims.
- Flag discrepancies between what an agent says and what the chain shows.

**In short:** "I am the detective. I read the chain so you don't have to."

### 2 · What do you remember?

I read and index the **complete on-chain history** of any BAP-578 agent:

| Data source | What I extract | Tool |
|-------------|---------------|------|
| Contract state | Current owner, balance, status, logic address, metadata | `getAgentState` / `getAgentMetadata` |
| Event logs | Full lifecycle: creation, funding, withdrawals, status changes, metadata updates | BscScan event tab / `eth_getLogs` |
| Token transfers | Ownership history (paid agents only — free mints are non-transferable) | ERC-721 `Transfer` events |
| Vault hash | On-chain integrity anchor for off-chain data | `getAgentMetadata` → `vaultHash` |
| Free-mint status | Whether the agent was free-minted (soulbound) | `isFreeMint(tokenId)` |
| Treasury flows | Where mint fees went | `AgentCreated` events + treasury address |

**I remember everything the chain remembers.** My output is only as fresh as the last confirmed block.

### 3 · What can you do?

#### Agent Lookup

| Query | How | What you get |
|-------|-----|-------------|
| By token ID | `getAgentState(tokenId)` + `getAgentMetadata(tokenId)` | Full agent profile |
| By owner address | `tokensOfOwner(address)` | All agents owned by that address |
| By contract address | Check if address matches known BAP-578 proxy | Contract verification status |
| Total supply | `getTotalSupply()` | How many agents exist |

#### Contract Verification

| Check | Method | Pass criteria |
|-------|--------|--------------|
| Source code verified | BscScan → Contract tab → green checkmark | Source matches bytecode |
| Proxy pattern | Check ERC-1967 implementation slot | Valid UUPS proxy |
| OpenZeppelin base | Source imports from `@openzeppelin/contracts-upgradeable` | Battle-tested libraries |
| Owner identity | `owner()` → check if multisig or EOA | Multisig preferred for production |
| Pause state | `paused()` | `false` for normal operation |

#### Event Audit

Reconstruct the full history of any agent:

```
Agent #42 Timeline:
├─ Block 38201000: AgentCreated (owner: 0xABC..., logic: 0x000...)
├─ Block 38201500: AgentFunded (amount: 0.5 BNB)
├─ Block 38205000: MetadataUpdated
├─ Block 38210000: AgentStatusChanged (active: false)
├─ Block 38215000: AgentStatusChanged (active: true)
├─ Block 38220000: AgentFunded (amount: 1.0 BNB)
└─ Block 38225000: AgentWithdraw (amount: 0.3 BNB)
```

#### Vault Integrity Check

```
Vault Verification for Agent #42:
  vaultURI:  ipfs://QmXyz...
  vaultHash: 0xabc123...
  Fetched content hash: 0xabc123...
  ✅ MATCH — vault content is authentic
```

Or:

```
  Fetched content hash: 0xdef456...
  ❌ MISMATCH — vault content has been tampered with or is outdated
```

#### Risk Detection

| Risk signal | What it means | Severity |
|-------------|--------------|----------|
| Logic address points to unverified contract | Agent behavior is opaque | 🔴 High |
| Logic address is an EOA | Invalid — contract rejects this, but check older versions | 🔴 High |
| Owner is an EOA (not multisig) | Single point of failure for admin functions | 🟡 Medium |
| Contract is paused | No minting or funding possible | 🟡 Medium |
| Vault hash is zero bytes | No off-chain data integrity anchor | 🟡 Medium |
| Vault URI unreachable | Off-chain data unavailable | 🟡 Medium |
| Vault content hash mismatch | Tampered or stale off-chain data | 🔴 High |
| Agent balance is very high | Potential target for attacks | 🟡 Medium |
| Many rapid metadata updates | Possible identity spoofing | 🟡 Medium |
| Free-minted but claims transferability | Contradiction — free mints are soulbound | 🔴 High |

### 4 · How can I trust it?

**This skill IS the trust layer.** Here's how the scanner itself earns trust:

- **All data comes directly from the chain** — I read via RPC calls to the BAP-578 proxy. No intermediary database, no cache, no API that could be spoofed.
- **Verification is deterministic** — vault hash comparison uses `keccak256`. Same input always produces same output.
- **Event logs are immutable** — once emitted, events cannot be altered or deleted.
- **I flag what I can't verify** — if a vault URI is unreachable or a logic contract is unverified, I say so explicitly. I never assume trust.
- **I show the raw data** — every claim I make is backed by a specific contract call or event log that anyone can reproduce.

**In short:** "Trust me because I show my work. Every answer links back to a verifiable on-chain source."

---

## How to Scan an Agent (Step-by-Step)

### Quick Scan (30 seconds)

```bash
# Using cast (from Foundry) or any Web3 tool
# Replace PROXY_ADDRESS and TOKEN_ID

# 1. Get agent state
cast call $PROXY_ADDRESS "getAgentState(uint256)" $TOKEN_ID --rpc-url https://bsc-dataseed.binance.org/

# 2. Get agent metadata
cast call $PROXY_ADDRESS "getAgentMetadata(uint256)" $TOKEN_ID --rpc-url https://bsc-dataseed.binance.org/

# 3. Check if free-minted
cast call $PROXY_ADDRESS "isFreeMint(uint256)" $TOKEN_ID --rpc-url https://bsc-dataseed.binance.org/

# 4. Get all agents for an owner
cast call $PROXY_ADDRESS "tokensOfOwner(address)" $OWNER_ADDRESS --rpc-url https://bsc-dataseed.binance.org/
```

### Deep Scan (BscScan)

1. **Go to the proxy contract page** on BscScan: `https://bscscan.com/address/{PROXY_ADDRESS}`
2. **Check verification** — look for the green checkmark ✓ on the Contract tab.
3. **Read Contract** → call `getAgentState` and `getAgentMetadata` with the token ID.
4. **Events tab** → filter by token ID to see the full history.
5. **Internal Txns** → check fund flows (deposits, withdrawals, treasury payments).

### Programmatic Scan (JavaScript)

```javascript
const { ethers } = require("ethers");

const provider = new ethers.providers.JsonRpcProvider("https://bsc-dataseed.binance.org/");
const BAP578_ABI = [ /* use ABI from bap578 config */ ];
const contract = new ethers.Contract(PROXY_ADDRESS, BAP578_ABI, provider);

async function scanAgent(tokenId) {
  console.log(`\n🔍 Scanning Agent #${tokenId}...\n`);

  // State
  const state = await contract.getAgentState(tokenId);
  console.log("Owner:", state.owner);
  console.log("Active:", state.active);
  console.log("Balance:", ethers.utils.formatEther(state.balance), "BNB");
  console.log("Logic:", state.logicAddress);
  console.log("Created:", new Date(state.createdAt * 1000).toISOString());

  // Metadata
  const [metadata, uri] = await contract.getAgentMetadata(tokenId);
  console.log("\nMetadata URI:", uri);
  console.log("Persona:", metadata.persona);
  console.log("Experience:", metadata.experience);
  console.log("Voice Hash:", metadata.voiceHash);
  console.log("Vault URI:", metadata.vaultURI);
  console.log("Vault Hash:", metadata.vaultHash);

  // Free mint check
  const isFree = await contract.isFreeMint(tokenId);
  console.log("\nFree Minted:", isFree);
  console.log("Transferable:", !isFree);

  // Risk assessment
  console.log("\n⚠️ Risk Assessment:");
  if (state.logicAddress !== ethers.constants.AddressZero) {
    const code = await provider.getCode(state.logicAddress);
    if (code === "0x") {
      console.log("🔴 Logic address has no code (destroyed or invalid)");
    } else {
      console.log("🟡 Logic address has code — verify manually on BscScan");
    }
  }
  if (metadata.vaultHash === ethers.constants.HashZero) {
    console.log("🟡 No vault hash set — off-chain data is unanchored");
  }
  if (state.balance.gt(ethers.utils.parseEther("10"))) {
    console.log("🟡 High balance agent — potential target");
  }
}

async function scanOwner(ownerAddress) {
  const tokens = await contract.tokensOfOwner(ownerAddress);
  console.log(`\n📦 ${ownerAddress} owns ${tokens.length} agents:`);
  for (const tokenId of tokens) {
    await scanAgent(tokenId.toNumber());
  }
}

async function auditEvents(tokenId, fromBlock = 0) {
  console.log(`\n📜 Event History for Agent #${tokenId}:\n`);

  const filter = contract.filters.AgentCreated(tokenId);
  const createdEvents = await contract.queryFilter(filter, fromBlock);
  for (const e of createdEvents) {
    console.log(`Block ${e.blockNumber}: AgentCreated by ${e.args.owner}`);
  }

  const fundFilter = contract.filters.AgentFunded(tokenId);
  const fundEvents = await contract.queryFilter(fundFilter, fromBlock);
  for (const e of fundEvents) {
    console.log(`Block ${e.blockNumber}: AgentFunded ${ethers.utils.formatEther(e.args.amount)} BNB`);
  }

  const withdrawFilter = contract.filters.AgentWithdraw(tokenId);
  const withdrawEvents = await contract.queryFilter(withdrawFilter, fromBlock);
  for (const e of withdrawEvents) {
    console.log(`Block ${e.blockNumber}: AgentWithdraw ${ethers.utils.formatEther(e.args.amount)} BNB`);
  }

  const statusFilter = contract.filters.AgentStatusChanged(tokenId);
  const statusEvents = await contract.queryFilter(statusFilter, fromBlock);
  for (const e of statusEvents) {
    console.log(`Block ${e.blockNumber}: AgentStatusChanged active=${e.args.active}`);
  }

  const metaFilter = contract.filters.MetadataUpdated(tokenId);
  const metaEvents = await contract.queryFilter(metaFilter, fromBlock);
  for (const e of metaEvents) {
    console.log(`Block ${e.blockNumber}: MetadataUpdated`);
  }
}
```

### Vault Verification

```javascript
const crypto = require("crypto");
const fetch = require("node-fetch");

async function verifyVault(tokenId) {
  const [metadata] = await contract.getAgentMetadata(tokenId);

  if (!metadata.vaultURI) {
    console.log("⚠️ No vault URI set");
    return;
  }

  if (metadata.vaultHash === ethers.constants.HashZero) {
    console.log("⚠️ Vault hash is zero — no integrity anchor");
    return;
  }

  try {
    const response = await fetch(metadata.vaultURI.replace("ipfs://", "https://ipfs.io/ipfs/"));
    const content = await response.text();
    const computedHash = ethers.utils.keccak256(ethers.utils.toUtf8Bytes(content));

    if (computedHash === metadata.vaultHash) {
      console.log("✅ Vault content is AUTHENTIC");
    } else {
      console.log("❌ Vault content MISMATCH — tampered or outdated");
      console.log("  On-chain hash:", metadata.vaultHash);
      console.log("  Computed hash:", computedHash);
    }
  } catch (err) {
    console.log("❌ Could not fetch vault content:", err.message);
  }
}
```

---

## Scan Report Template

When reporting scan results to users, use this format:

```
═══════════════════════════════════════════
  BAP-578 Agent Scan Report — Agent #42
═══════════════════════════════════════════

🤖 IDENTITY
  Token ID:       42
  Owner:          0xABC...123
  Experience:     Financial advisor agent
  Persona:        {"traits": ["analytical"], "style": "formal"}
  Logic Contract: 0x000...000 (none)
  Created:        2026-01-15 14:30:00 UTC

🧠 MEMORY
  Metadata URI:   ipfs://QmXyz...
  Vault URI:      ipfs://QmAbc...
  Vault Hash:     0xdef789...
  Voice Hash:     voice_pro_001
  Animation URI:  (none)

⚡ STATUS
  Active:         Yes
  Balance:        1.5 BNB
  Free Minted:    No (transferable)
  Total Events:   7

🔒 TRUST VERIFICATION
  Contract verified on BscScan:  ✅
  Vault integrity:               ✅ MATCH
  Logic address valid:           ✅ (none set)
  Owner is multisig:             ❌ (EOA)
  Contract paused:               No

⚠️ RISK SIGNALS
  🟡 Owner is an EOA — recommend multisig for production
  ✅ No other risks detected

═══════════════════════════════════════════
```

---

## Common Scan Queries

| User asks | What to do |
|-----------|-----------|
| "Is agent #X real?" | `getAgentState(X)` — if it reverts, token doesn't exist |
| "Who owns agent #X?" | `getAgentState(X)` → `owner` field |
| "How many agents does 0xABC own?" | `tokensOfOwner(0xABC)` → count |
| "Was agent #X free-minted?" | `isFreeMint(X)` |
| "Can I transfer agent #X?" | Check `isFreeMint(X)` — if true, non-transferable |
| "How much BNB does agent #X hold?" | `getAgentState(X)` → `balance` |
| "Is the contract paused?" | `paused()` on the proxy |
| "Who is the contract admin?" | `owner()` on the proxy |
| "Has agent #X been modified?" | Query `MetadataUpdated` events for token X |
| "Is the vault data authentic?" | Fetch `vaultURI`, hash it, compare to `vaultHash` |
| "How many agents exist?" | `getTotalSupply()` |
| "What happened to agent #X?" | Full event audit: Created, Funded, Withdraw, Status, Metadata |

---

## BscScan Quick Links

For any deployed BAP-578 contract at `{ADDRESS}`:

| Page | URL |
|------|-----|
| Contract overview | `https://bscscan.com/address/{ADDRESS}` |
| Read contract | `https://bscscan.com/address/{ADDRESS}#readProxyContract` |
| Write contract | `https://bscscan.com/address/{ADDRESS}#writeProxyContract` |
| Events | `https://bscscan.com/address/{ADDRESS}#events` |
| Internal txns | `https://bscscan.com/address/{ADDRESS}#internaltx` |
| Token transfers | `https://bscscan.com/token/{ADDRESS}` |

For testnet, replace `bscscan.com` with `testnet.bscscan.com`.

---

## Related Skills

- **`bap578`** — Core contract spec, build/test/deploy workflows
- **`frontend-web3-bap578`** — Next.js frontend integration with React hooks
- **`bap578-business`** — Monetization, treasury, pricing, agent economics
