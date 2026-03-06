---
id: bap578
name: BAP-578 Non-Fungible Agents
description: >
  End-to-end BAP-578 Non-Fungible Agents skill for BNB Chain. Explains agent
  identity, on-chain memory, capabilities, and trust verification. Covers
  build, test, deploy, interact, and upgrade workflows using the reference
  implementation (ERC-721, UUPS upgradeable, structured metadata, agent fund
  management). Use when asked about BAP-578, NFA, agent NFTs, or agent
  identity on BNB Chain.
category: BAP-578
author: community
version: 1.0.0
examples:
  - "What is BAP-578?"
  - "Explain Non-Fungible Agents"
  - "Build and test the BAP-578 contract"
  - "Deploy BAP-578 to BSC testnet"
  - "How can I trust an agent NFT?"
  - "What can a BAP-578 agent do?"
  - "How does agent memory work on-chain?"
---

# BAP-578 Non-Fungible Agents (NFA)

Use this skill whenever users ask about BAP-578 identity agents, Non-Fungible Agents (NFA), agent NFTs on BNB Chain, or when they want hands-on help building, testing, deploying, or interacting with the BAP-578 reference codebase.

---

## When to use this skill

- Explaining BAP-578 in plain language **and** contract-accurate terms.
- Answering the **four identity questions** (see below).
- Running real engineering workflows: install → compile → test → deploy → verify → interact.
- Translating between spec-level intent (what BAP-578 aims to achieve) and code-level behavior (what the repository actually implements today).

## Trigger patterns

Activate this skill if user asks for:

- "Build BAP-578", "run BAP-578", "deploy BAP-578", "test BAP-578"
- "Explain BAP-578", "what is NFA", "how do agent NFTs work"
- "How to trust agent NFT", "verify contract", "check metadata integrity"
- "How many free mints", "how fund/withdraw", "how upgrade"
- "Who is this agent", "what does this agent remember"

---

## The Four Identity Questions

Every BAP-578 agent should be able to answer these four questions. This is the philosophical and practical core of the standard.

### 1 · Who are you?

A **BAP-578 Non-Fungible Agent** is an ERC-721 tokenized agent identity living on BNB Chain. It is **not** just a profile picture or collectible — it is a self-sovereign identity anchor for an AI agent.

**Identity anchor** = the combination of:

| Component | Where it lives | Purpose |
|-----------|---------------|---------|
| Token ID | On-chain (`_tokenIdCounter`) | Unique numeric identifier |
| Owner address | On-chain (ERC-721 `ownerOf`) | Who controls the agent |
| Metadata URI | On-chain (`tokenURI`) | Pointer to off-chain JSON |
| Structured metadata | On-chain (`AgentMetadata` struct) | Rich agent personality and data |
| Logic address | On-chain (`AgentState.logicAddress`) | Optional bound behavior contract |

**Structured on-chain metadata** — the `AgentMetadata` struct:

```solidity
struct AgentMetadata {
    string persona;       // JSON-encoded character traits, style, tone
    string experience;    // Short summary of the agent's role/expertise
    string voiceHash;     // Reference ID to a stored audio profile
    string animationURI;  // URI to a video or animation resource
    string vaultURI;      // URI to extended off-chain data storage (vault)
    bytes32 vaultHash;    // Keccak-256 hash of vault contents for integrity
}
```

**What makes it an "agent" and not just an NFT:**

- It carries **persona** and **experience** — the agent's character and knowledge domain.
- It can hold **BNB funds** in its own balance (`AgentState.balance`).
- It can bind to a **logic contract** (`logicAddress`) that defines autonomous behavior.
- It has a **lifecycle** (`active` / inactive status).
- Its off-chain data can be **verified** via `vaultHash`.

**In short:** "I am token #{id} on BNB Chain, owned by {address}, with persona {persona}, expertise in {experience}, and I can act autonomously through my logic contract."

---

### 2 · What do you remember?

Agent memory in BAP-578 exists across two layers:

#### On-chain memory (trustable, immutable history)

Stored in contract state and queryable any time:

| Data | Storage | How to read |
|------|---------|-------------|
| Owner | ERC-721 mapping | `ownerOf(tokenId)` |
| Active status | `AgentState.active` | `getAgentState(tokenId)` |
| BNB balance | `AgentState.balance` | `getAgentState(tokenId)` |
| Logic contract | `AgentState.logicAddress` | `getAgentState(tokenId)` |
| Creation time | `AgentState.createdAt` | `getAgentState(tokenId)` |
| Persona, experience, voice, animation, vault | `AgentMetadata` | `getAgentMetadata(tokenId)` |
| Token URI | ERC-721 URI storage | `tokenURI(tokenId)` |
| Free-mint status | `isFreeMint[tokenId]` | `isFreeMint(tokenId)` |

**Event log** — permanent, indexed history of everything that happened:

| Event | What it records |
|-------|----------------|
| `AgentCreated(tokenId, owner, logicAddress, metadataURI)` | Birth of the agent |
| `AgentFunded(tokenId, amount)` | BNB deposited |
| `AgentWithdraw(tokenId, amount)` | BNB withdrawn |
| `AgentStatusChanged(tokenId, active)` | Lifecycle toggle |
| `LogicAddressUpdated(tokenId, newLogicAddress)` | Behavior contract changed |
| `MetadataUpdated(tokenId)` | Persona or data updated |
| `TreasuryUpdated(newTreasury)` | Admin treasury change |
| `ContractPaused(paused)` | Emergency pause toggle |
| `FreeMintGranted(user, amount)` | Bonus mints granted |

#### Off-chain memory (referenced, hash-verifiable)

- **`vaultURI`** points to an extended data store (e.g., IPFS, Arweave) that can hold conversation history, learned preferences, extended knowledge, or any blob.
- **`vaultHash`** (bytes32) anchors integrity — compare `keccak256(offChainPayload)` against the stored hash. If they match, the off-chain data is authentic.
- **`animationURI`** and **`voiceHash`** reference media and voice profiles stored off-chain.

**Trust rule:** If data is not verifiable from chain state, events, or hash comparison, label it clearly as **unverified**.

**In short:** "I remember everything recorded on-chain (state + events) and can reference off-chain vaults whose integrity I can prove via vaultHash."

---

### 3 · What can you do?

The BAP-578 contract exposes these capabilities, organized by role:

#### Any user (public)

| Function | What it does | Cost |
|----------|-------------|------|
| `createAgent(to, logicAddress, metadataURI, metadata)` | Mint a new agent NFT | Free for first 3 per user, then 0.01 BNB |
| `fundAgent(tokenId)` | Send BNB to any agent's balance | `msg.value` (the BNB sent) |

#### Token owner only (`onlyTokenOwner`)

| Function | What it does |
|----------|-------------|
| `withdrawFromAgent(tokenId, amount)` | Withdraw BNB from your agent |
| `setAgentStatus(tokenId, active)` | Activate or deactivate your agent |
| `setLogicAddress(tokenId, newLogicAddress)` | Bind a logic contract (must be contract, not EOA) |
| `updateAgentMetadata(tokenId, newURI, newMetadata)` | Update persona, experience, voice, vault, etc. |

#### Contract owner only (`onlyOwner` — admin)

| Function | What it does |
|----------|-------------|
| `grantAdditionalFreeMints(user, amount)` | Give bonus free mints beyond the default 3 |
| `setFreeMintsPerUser(amount)` | Change the global default free mints |
| `setTreasury(newTreasury)` | Update where mint fees are sent |
| `setPaused(pausedState)` | Emergency pause / unpause all minting and funding |
| `emergencyWithdraw()` | Drain contract balance to owner (emergency only) |

#### Read-only (view functions)

| Function | Returns |
|----------|---------|
| `getAgentState(tokenId)` | balance, active, logicAddress, createdAt, owner |
| `getAgentMetadata(tokenId)` | AgentMetadata struct + metadataURI |
| `tokensOfOwner(address)` | Array of token IDs owned by that address |
| `getTotalSupply()` | Total agents minted |
| `getFreeMints(user)` | Remaining free mints for a user |

#### Special behaviors

- **Free-minted tokens are non-transferable** — `_beforeTokenTransfer` blocks transfers for tokens where `isFreeMint[tokenId] == true` (except mint and burn).
- **Burning requires zero balance** — you must withdraw all BNB before burning an agent.
- **Direct ETH/BNB sends are rejected** — the `receive()` function reverts with "Use fundAgent() instead".
- **Logic address validation** — `logicAddress` must be `address(0)` or a deployed contract (code length > 0). EOAs are rejected.

**In short:** "I can be minted, funded, managed, upgraded, and queried. My owner controls my lifecycle, funds, and metadata. Admins can pause me in emergencies."

---

### 4 · How can I trust it?

Trust in a BAP-578 agent is built through multiple verification layers:

#### Contract verification

1. **Source code on BscScan** — After deployment, verify source code so anyone can read and audit it.
   ```bash
   npm run verify:testnet   # or verify:mainnet
   ```
2. **Match the proxy** — The contract uses UUPS proxy pattern. Verify both proxy and implementation addresses on BscScan.
3. **OpenZeppelin base** — The contract inherits from battle-tested OpenZeppelin libraries (ERC721, ReentrancyGuard, Ownable, UUPS).

#### Access control boundaries

| Action | Who can do it | Guard |
|--------|--------------|-------|
| Mint | Anyone | `whenNotPaused` |
| Manage agent (status, withdraw, metadata, logic) | Token owner only | `onlyTokenOwner` modifier |
| Admin (pause, treasury, free mints, emergency) | Contract owner only | `onlyOwner` modifier |
| Upgrade contract | Contract owner only | `_authorizeUpgrade` override |

#### Security patterns in the code

- **Checks-Effects-Interactions** — state is updated before external calls in `withdrawFromAgent`.
- **Reentrancy guard** — `nonReentrant` on `createAgent` and `withdrawFromAgent`.
- **Logic address validation** — only `address(0)` or actual contracts accepted (no EOAs).
- **Pause mechanism** — `whenNotPaused` on `createAgent` and `fundAgent`.
- **Transfer restriction** — free-minted tokens cannot be transferred (soulbound-like).

#### Off-chain data integrity

- Compare `keccak256(fetchedVaultContent)` with the on-chain `vaultHash`. Match = authentic.
- If `vaultHash` is `bytes32(0)` or the off-chain content is unavailable, treat vault data as unverified.

#### Event audit trail

- Every critical action emits an event. Anyone can reconstruct the full history of an agent by scanning events by `tokenId`.
- Events are indexed (`indexed tokenId`, `indexed owner`) for efficient querying.

#### Upgrade transparency

- UUPS pattern means only the owner can authorize upgrades.
- The V2 mock (`BAP578V2Mock.sol`) demonstrates that state is preserved across upgrades.
- Recommend: always test upgrades on testnet first and announce to community.

**In short:** "Trust me by verifying my source on BscScan, checking access controls, auditing events, confirming vaultHash integrity, and monitoring upgrade history."

---

## Contract Architecture

```
BAP578 (UUPS Upgradeable Proxy)
├── ERC721Upgradeable          — Core NFT standard
├── ERC721EnumerableUpgradeable — Token enumeration (tokensOfOwner)
├── ERC721URIStorageUpgradeable — Per-token metadata URI
├── ReentrancyGuardUpgradeable  — Protects against reentrancy
├── OwnableUpgradeable          — Admin access control
└── UUPSUpgradeable             — Upgrade authorization
```

**Key constants:**
- `MINT_FEE` = 0.01 BNB (after free mints exhausted)
- `freeMintsPerUser` = 3 (configurable by owner)

**Networks:**
- BSC Testnet: Chain ID 97
- BSC Mainnet: Chain ID 56
- Local: Hardhat Network

---

## Build, Test, Deploy, and Interact

### Prerequisites

- Node.js v22+ recommended
- npm available
- For testnet/mainnet: funded wallet + BSCScan API key

### Step 1 — Install and compile

```bash
cd non-fungible-agents-BAP-578
npm install
npm run compile
```

### Step 2 — Run tests

```bash
npm test
```

The test suite (`test/BAP578.test.js`) covers:
- Deployment and initialization
- Free mints (3 per user, non-transferable, self-only)
- Paid mints (0.01 BNB to treasury)
- Agent management (status, funding, withdrawal, logic address, metadata)
- View functions (state, metadata, tokens of owner, supply, free mints)
- Admin functions (pause, treasury, bonus mints, emergency withdraw)
- UUPS upgrade (state preservation, V2 functionality, owner-only guard)

If tests fail, report the exact error and failing test name before proposing fixes.

### Step 3 — Configure environment

```bash
cp .env.example .env
```

Edit `.env`:
```env
DEPLOYER_PRIVATE_KEY=your_64_char_hex_key_no_0x_prefix
TESTNET_RPC_URL=https://data-seed-prebsc-1-s1.binance.org:8545/
MAINNET_RPC_URL=https://bsc-dataseed.binance.org/
BSCSCAN_API_KEY=your_bscscan_api_key
```

### Step 4 — Deploy

```bash
# Local (Hardhat network)
npm run deploy

# BSC Testnet
npm run deploy:testnet

# BSC Mainnet (only when explicitly requested)
npm run deploy:mainnet
```

The deploy script:
1. Deploys a UUPS proxy with implementation.
2. Initializes with name "Non-Fungible Agents", symbol "NFA", and a treasury address.
3. Saves deployment info to `./deployments/{network}_deployment.json`.

### Step 5 — Verify on BscScan

```bash
npm run verify:testnet    # automatic
npm run verify:manual:testnet  # if auto fails
```

### Step 6 — Interact

```bash
npm run interact:testnet
```

Interactive CLI menu:
1. Create new agent (first 3 free)
2. View my agents
3. View agent details (state + metadata)
4. Fund an agent
5. Withdraw from agent
6. Update agent status
7. Update logic address
8. Update metadata
9. View contract info
10. Admin functions (grant mints, treasury, pause, emergency)

### Deliverable to user

After any build/deploy/interact workflow, return:
- Commands run and pass/fail status
- Contract addresses (proxy + implementation)
- Transaction hashes
- Next recommended action

---

## Gas Costs Reference

| Action | Approximate Gas | BNB Cost @ 5 gwei |
|--------|----------------|-------------------|
| Deploy contract | ~4,500,000 | ~0.0225 BNB |
| Mint (free) | ~300,000 | ~0.0015 BNB |
| Mint (paid, after free) | ~300,000 | ~0.0015 BNB + 0.01 BNB fee |
| Fund agent | ~50,000 | ~0.00025 BNB |
| Withdraw from agent | ~60,000 | ~0.0003 BNB |
| Update metadata | ~150,000 | ~0.00075 BNB |
| Set agent status | ~30,000 | ~0.00015 BNB |

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| `Unsupported engine` | Install Node.js v22 |
| `Cannot find module` | `rm -rf node_modules package-lock.json && npm install` |
| `insufficient funds for gas` | Add BNB to deployer wallet (need 0.05+ BNB) |
| `Incorrect fee` | You've used all free mints — send exactly 0.01 BNB as `msg.value` |
| `Free mints can only be minted to self` | Use your own address as `to` when free-minting |
| `Free minted tokens are non-transferable` | Free-minted agents cannot be transferred (by design) |
| `Invalid logic address` | Logic address must be a deployed contract, not an EOA |
| `Not token owner` | Only the NFT owner can manage that agent |
| `Contract is paused` | Admin has paused the contract — wait for unpause |
| `Agent balance must be 0` | Withdraw all BNB before burning |
| `Use fundAgent() instead` | Don't send BNB directly to the contract — call `fundAgent(tokenId)` |
| Verification failed | Wait 60s after deploy, check BSCSCAN_API_KEY, try manual verify |

---

## Operational and Security Rules

1. **Testnet first** — always deploy and test on BSC Testnet before mainnet.
2. **Never expose secrets** — do not log or share private keys or `.env` contents.
3. **Logic address is high-risk** — validate the logic contract's code before binding it to an agent. A malicious logic contract could affect agent behavior.
4. **Off-chain data is not trusted by default** — only treat vault data as authentic if `keccak256(content) == vaultHash`.
5. **Free-mint transfer lock** — remind users that free-minted tokens cannot be transferred, only burned (after zero balance).
6. **Upgrade caution** — UUPS upgrades are irreversible. Always test on testnet, preserve state, and communicate changes.

---

## Default Response Behavior

When answering BAP-578 questions:

1. **Start with a direct answer** to the user's question.
2. **Then provide concrete commands** or function calls (copy/paste ready).
3. **Distinguish spec vs implementation** — if the BAP-578 specification intent differs from the current codebase behavior, call out the difference explicitly.
4. **For implementation requests**, use this output format:
   - Goal summary
   - Commands to run
   - Expected outputs and checks
   - Risks or blockers
   - Next step recommendation

---

## Available npm Scripts

| Command | Description |
|---------|-------------|
| `npm test` | Run full test suite |
| `npm run compile` | Compile Solidity contracts |
| `npm run deploy` | Deploy to local Hardhat network |
| `npm run deploy:testnet` | Deploy to BSC Testnet |
| `npm run deploy:mainnet` | Deploy to BSC Mainnet |
| `npm run interact` | Interactive CLI (local) |
| `npm run interact:testnet` | Interactive CLI (testnet) |
| `npm run interact:mainnet` | Interactive CLI (mainnet) |
| `npm run verify:testnet` | Verify on testnet BscScan |
| `npm run verify:mainnet` | Verify on mainnet BscScan |
| `npm run verify:manual:testnet` | Manual verification fallback |
| `npm run clean` | Clean build artifacts |
| `npm run coverage` | Generate test coverage report |
| `npm run lint` | Lint Solidity files |
| `npm run format` | Format all source files |

---

## References

- **BAP-578 Specification:** https://github.com/bnb-chain/BEPs/blob/master/BAPs/BAP-578.md
- **Reference Implementation:** https://github.com/ChatAndBuild/non-fungible-agents-BAP-578
- **BNB Chain:** https://www.bnbchain.org
- **OpenZeppelin Upgradeable Contracts:** https://docs.openzeppelin.com/contracts/4.x/upgradeable
- **Hardhat:** https://hardhat.org
