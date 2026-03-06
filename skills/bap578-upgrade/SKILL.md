---
id: bap578-upgrade
name: BAP-578 Contract Upgrade
description: >
  Safely upgrade BAP-578 Non-Fungible Agents using the UUPS proxy pattern.
  Covers writing V2+ implementations, storage layout safety, upgrade testing,
  deployment, state preservation verification, rollback planning, and
  governance best practices. Use when upgrading or planning upgrades to a
  deployed BAP-578 contract.
category: BAP-578
author: community
version: 1.0.0
examples:
  - "How to upgrade the BAP-578 contract"
  - "Write a V2 implementation for NFA"
  - "Is it safe to upgrade the proxy?"
  - "Add a new feature to BAP-578 via upgrade"
  - "How to preserve state during upgrade"
  - "What is the UUPS upgrade pattern?"
---

# BAP-578 Contract Upgrade

Use this skill when planning, writing, testing, or executing a UUPS upgrade for a deployed BAP-578 Non-Fungible Agents contract. Upgrades allow adding features, fixing bugs, and evolving the contract — but they're irreversible and high-risk. This skill ensures you do it safely.

---

## The Four Identity Questions — Upgrade Perspective

### 1 · Who are you?

I am the **evolution mechanism** for BAP-578. I answer the question: **"How does an agent contract grow without losing its history?"**

BAP-578 uses the **UUPS (Universal Upgradeable Proxy Standard)** pattern from OpenZeppelin:

```
User calls ──► Proxy Contract (permanent address)
                    │
                    │  delegatecall
                    ▼
               Implementation V1  ←── current logic
               Implementation V2  ←── after upgrade (new logic, same state)
```

- The **proxy** holds all state (agents, metadata, balances). Its address never changes.
- The **implementation** holds the logic. It can be swapped by the contract owner.
- After upgrade, all existing agents, balances, and metadata are preserved.

**In short:** "I let the contract evolve. New features, same agents, same address, same trust."

### 2 · What do you remember?

Upgrades preserve **all existing state** because state lives in the proxy, not the implementation:

| Preserved | Where | Verified by |
|-----------|-------|-------------|
| All agent states | Proxy storage | V2 test: `agentStateBefore == agentStateAfter` |
| All agent metadata | Proxy storage | V2 test: persona, experience unchanged |
| Token ownership | Proxy storage | V2 test: `ownerOf`, `balanceOf` unchanged |
| Total supply | Proxy storage | V2 test: `totalSupply` unchanged |
| Treasury address | Proxy storage | V2 test: `treasuryAddress` unchanged |
| Contract owner | Proxy storage | V2 test: `owner()` unchanged |
| Free mint tracking | Proxy storage | V2 test: `freeMintsClaimed`, `bonusFreeMints` unchanged |
| Agent balances | Proxy storage | V2 test: ETH/BNB balances unchanged |

The existing test suite (`BAP578.test.js` → "UUPS Upgrade" section) confirms state preservation.

### 3 · What can you do?

#### Writing a V2 Implementation

**Rules for safe V2 contracts:**

1. **Inherit from the current version** — `contract BAP578V2 is BAP578`
2. **Only APPEND new state variables** — never insert, reorder, or remove existing ones
3. **Never change existing function signatures** — add new functions, don't modify old ones
4. **Use a reinitializer if needed** — `reinitializer(2)` for V2 initialization logic
5. **Keep `_authorizeUpgrade` override** — must remain `onlyOwner`

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.28;

import "./BAP578.sol";

contract BAP578V2 is BAP578 {
    // ═══ NEW STATE VARIABLES (append only!) ═══
    uint256 public royaltyBasisPoints;        // New: royalty percentage
    mapping(uint256 => string) public agentTags; // New: tagging system

    // ═══ NEW EVENTS ═══
    event RoyaltyUpdated(uint256 basisPoints);
    event AgentTagged(uint256 indexed tokenId, string tag);

    // ═══ REINITIALIZER (runs once on upgrade) ═══
    function initializeV2(uint256 _royaltyBps) public reinitializer(2) {
        royaltyBasisPoints = _royaltyBps;
    }

    // ═══ NEW FUNCTIONS ═══
    function setRoyalty(uint256 basisPoints) external onlyOwner {
        require(basisPoints <= 1000, "Max 10%");
        royaltyBasisPoints = basisPoints;
        emit RoyaltyUpdated(basisPoints);
    }

    function tagAgent(uint256 tokenId, string memory tag) external onlyTokenOwner(tokenId) {
        agentTags[tokenId] = tag;
        emit AgentTagged(tokenId, tag);
    }

    function version() external pure returns (string memory) {
        return "v2";
    }
}
```

#### Storage Layout Safety

```
BAP578 Storage Layout (DO NOT MODIFY):
═══════════════════════════════════════
Slot 0-N:   ERC721Upgradeable state
Slot N+1:   ERC721EnumerableUpgradeable state
Slot N+2:   ERC721URIStorageUpgradeable state
Slot N+3:   ReentrancyGuardUpgradeable state
Slot N+4:   OwnableUpgradeable state
Slot N+5:   _tokenIdCounter
Slot N+6:   agentStates mapping
Slot N+7:   agentMetadata mapping
Slot N+8:   freeMintsPerUser
Slot N+9:   freeMintsClaimed mapping
Slot N+10:  isFreeMint mapping
Slot N+11:  bonusFreeMints mapping
Slot N+12:  treasuryAddress
Slot N+13:  paused
─────────────────────────────────────
V2 ADDITIONS (append here):
Slot N+14:  royaltyBasisPoints  ← NEW
Slot N+15:  agentTags mapping   ← NEW
```

**Critical rules:**
- ✅ Add new variables AFTER all existing ones
- ❌ Never insert between existing variables
- ❌ Never remove or rename existing variables
- ❌ Never change types of existing variables
- ✅ Use `__gap` arrays for future-proofing (optional but recommended)

#### Upgrade Testing

**Write upgrade tests BEFORE deploying:**

```javascript
describe("BAP578 V2 Upgrade", function () {
    it("Should upgrade and preserve all state", async function () {
        // 1. Deploy V1 and create agents
        const BAP578 = await ethers.getContractFactory("BAP578");
        const proxy = await upgrades.deployProxy(BAP578,
            ["Non-Fungible Agents", "NFA", treasury.address],
            { initializer: "initialize", kind: "uups" }
        );
        
        // Create agent on V1
        const metadata = createAgentMetadata();
        await proxy.connect(addr1).createAgent(
            addr1.address, ethers.constants.AddressZero,
            "ipfs://test", metadata
        );
        await proxy.connect(addr1).fundAgent(1, { value: parseEther("0.5") });
        
        // Record V1 state
        const supplyBefore = await proxy.totalSupply();
        const ownerBefore = await proxy.owner();
        const treasuryBefore = await proxy.treasuryAddress();
        const stateBefore = await proxy.getAgentState(1);
        const [metaBefore] = await proxy.getAgentMetadata(1);
        
        // 2. Upgrade to V2
        const BAP578V2 = await ethers.getContractFactory("BAP578V2");
        const upgraded = await upgrades.upgradeProxy(proxy.address, BAP578V2);
        
        // 3. Verify ALL state preserved
        expect(await upgraded.totalSupply()).to.equal(supplyBefore);
        expect(await upgraded.owner()).to.equal(ownerBefore);
        expect(await upgraded.treasuryAddress()).to.equal(treasuryBefore);
        
        const stateAfter = await upgraded.getAgentState(1);
        expect(stateAfter.balance).to.equal(stateBefore.balance);
        expect(stateAfter.active).to.equal(stateBefore.active);
        expect(stateAfter.owner).to.equal(stateBefore.owner);
        
        const [metaAfter] = await upgraded.getAgentMetadata(1);
        expect(metaAfter.persona).to.equal(metaBefore.persona);
        expect(metaAfter.experience).to.equal(metaBefore.experience);
        
        // 4. Verify V2 functions work
        expect(await upgraded.version()).to.equal("v2");
        
        // 5. Verify V1 functions still work
        await proxy.connect(addr2).createAgent(
            addr2.address, ethers.constants.AddressZero,
            "ipfs://test2", metadata
        );
        expect(await upgraded.totalSupply()).to.equal(supplyBefore.add(1));
    });
    
    it("Should only allow owner to upgrade", async function () {
        const BAP578V2 = await ethers.getContractFactory("BAP578V2", addr1);
        await expect(
            upgrades.upgradeProxy(proxy.address, BAP578V2)
        ).to.be.revertedWith("Ownable: caller is not the owner");
    });
});
```

#### Upgrade Execution

```bash
# 1. Run ALL tests (V1 + V2)
npm test

# 2. Deploy on testnet first
npx hardhat run scripts/upgrade-v2.js --network testnet

# 3. Verify new implementation on BscScan
npx hardhat verify --network testnet NEW_IMPLEMENTATION_ADDRESS

# 4. Test on testnet thoroughly
npm run interact:testnet
# → Verify existing agents still work
# → Test new V2 features

# 5. Only then upgrade mainnet
npx hardhat run scripts/upgrade-v2.js --network mainnet
```

**Example upgrade script (`scripts/upgrade-v2.js`):**

```javascript
const { ethers, upgrades } = require("hardhat");
const fs = require("fs");

async function main() {
    const network = hre.network.name;
    const deployment = JSON.parse(
        fs.readFileSync(`./deployments/${network}_deployment.json`, "utf8")
    );
    
    console.log("Upgrading proxy at:", deployment.proxy);
    
    const BAP578V2 = await ethers.getContractFactory("BAP578V2");
    const upgraded = await upgrades.upgradeProxy(deployment.proxy, BAP578V2);
    
    const newImpl = await upgrades.erc1967.getImplementationAddress(upgraded.address);
    console.log("New implementation:", newImpl);
    console.log("Version:", await upgraded.version());
    
    // Save updated deployment info
    deployment.implementationV2 = newImpl;
    deployment.upgradedAt = new Date().toISOString();
    fs.writeFileSync(
        `./deployments/${network}_deployment.json`,
        JSON.stringify(deployment, null, 2)
    );
}

main().catch(console.error);
```

### 4 · How can I trust it?

Upgrade trust is the highest bar in BAP-578:

- **Only the owner can authorize** — `_authorizeUpgrade` is `onlyOwner`. No one else can change the logic.
- **State is preserved** — the V2 test suite proves all agents, balances, metadata, and ownership survive the upgrade.
- **Testnet first** — always deploy and test on BSC Testnet before mainnet. Verify via interact CLI.
- **New implementation is verified** — publish source on BscScan so anyone can audit the changes.
- **Announce to community** — publish what changed, why, and the new implementation address before executing.
- **Consider a timelock** — add `TimelockController` so upgrades have a delay period for community review.

**Trust checklist before mainnet upgrade:**
```
[ ] V2 tests pass (state preservation + new features)
[ ] Storage layout verified (no slot collisions)
[ ] Deployed and tested on testnet
[ ] New implementation verified on BscScan
[ ] Changelog published to community
[ ] Timelock delay completed (if applicable)
[ ] Multisig approval (if owner is multisig)
```

**In short:** "Trust upgrades by verifying tests pass, state is preserved, testnet is clean, source is published, and the community is informed."

---

## Related Skills

- **`bap578`** — Core contract spec and UUPS architecture
- **`bap578-security-audit`** — Auditing upgrade safety and storage layout
- **`bap578-testing`** — Writing comprehensive upgrade test suites
