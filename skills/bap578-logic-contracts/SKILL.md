---
id: bap578-logic-contracts
name: BAP-578 Logic Contracts
description: >
  Build, deploy, and bind logic contracts to BAP-578 Non-Fungible Agents.
  Logic contracts define autonomous agent behavior on-chain — automated
  responses, DeFi actions, scheduled tasks, and cross-agent interactions.
  Covers architecture, Solidity patterns, binding via setLogicAddress,
  security considerations, and example implementations.
category: BAP-578
author: community
version: 1.0.0
examples:
  - "How do logic contracts work in BAP-578?"
  - "Build an autonomous agent with a logic contract"
  - "Bind a logic contract to my NFA agent"
  - "What can a logic contract do for an agent?"
  - "Create a DeFi auto-compounder logic contract"
  - "Is it safe to bind a logic contract?"
---

# BAP-578 Logic Contracts

Use this skill when building, deploying, or binding logic contracts to BAP-578 Non-Fungible Agents. Logic contracts transform agents from passive identities into **autonomous actors** — they define what an agent can do on-chain without human intervention.

---

## The Four Identity Questions — Logic Contract Perspective

### 1 · Who are you?

I am the **behavioral brain** of a BAP-578 agent. While the `AgentMetadata` struct defines who the agent *is* (persona, experience), the logic contract defines what the agent *does*.

The `logicAddress` field in `AgentState` can point to any deployed smart contract on BNB Chain. This contract becomes the agent's autonomous behavior layer:

```
Agent Identity (AgentMetadata)     Agent Behavior (logicAddress)
├── persona (character)            ├── Automated DeFi strategies
├── experience (knowledge)         ├── Scheduled task execution
├── voice (presentation)           ├── Cross-agent messaging
└── vault (memory)                 └── Custom business logic
```

**In short:** "I am the code that makes an agent act. Without me, the agent is a profile. With me, it's an autonomous actor."

### 2 · What do you remember?

The logic contract can store its own state and read the agent's state:

| Memory source | Access method | Example |
|---------------|--------------|---------|
| Agent state | Call `BAP578.getAgentState(tokenId)` | Check balance, active status |
| Agent metadata | Call `BAP578.getAgentMetadata(tokenId)` | Read persona, experience |
| Logic contract state | Own storage variables | Strategy parameters, execution history |
| BNB Chain state | Standard EVM reads | Block number, timestamps, other contracts |
| Events | Emit and index own events | Execution logs, decision records |

### 3 · What can you do?

#### Architecture

```
┌──────────────┐    setLogicAddress()    ┌──────────────────┐
│  BAP-578     │◄────────────────────────│  Agent Owner      │
│  Contract    │                         └──────────────────┘
│              │
│  Agent #42   │    logicAddress ──────► ┌──────────────────┐
│  ├─ state    │                         │  Logic Contract   │
│  ├─ metadata │                         │                  │
│  └─ balance  │                         │  execute()       │
│              │◄─── reads state ────────│  autoCompound()  │
└──────────────┘                         │  rebalance()     │
                                         └──────────────────┘
```

#### Binding Rules

1. **Only the token owner** can set or change the logic address (`onlyTokenOwner` modifier).
2. **Must be a deployed contract** — `logicAddress.code.length > 0`. EOAs are rejected.
3. **Zero address is valid** — `address(0)` unbinds the logic (removes behavior).
4. **No automatic execution** — BAP-578 stores the reference but does not call the logic contract. External triggers (keepers, cron, users) invoke the logic.

```solidity
// Bind a logic contract
await bap578.setLogicAddress(tokenId, logicContractAddress);

// Unbind (remove behavior)
await bap578.setLogicAddress(tokenId, ethers.constants.AddressZero);
```

#### Example Logic Contracts

**1. Simple Greeter — Minimal pattern**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.28;

interface IBAP578 {
    function getAgentState(uint256 tokenId) external view returns (
        uint256 balance, bool active, address logicAddress, uint256 createdAt, address owner
    );
    function getAgentMetadata(uint256 tokenId) external view returns (
        AgentMetadata memory metadata, string memory metadataURI
    );

    struct AgentMetadata {
        string persona;
        string experience;
        string voiceHash;
        string animationURI;
        string vaultURI;
        bytes32 vaultHash;
    }
}

contract AgentGreeter {
    IBAP578 public immutable bap578;

    event Greeted(uint256 indexed tokenId, string message);

    constructor(address _bap578) {
        bap578 = IBAP578(_bap578);
    }

    function greet(uint256 tokenId) external view returns (string memory) {
        (IBAP578.AgentMetadata memory meta, ) = bap578.getAgentMetadata(tokenId);
        return string(abi.encodePacked("Hello from Agent #", _toString(tokenId),
            ". I am: ", meta.experience));
    }

    function _toString(uint256 value) internal pure returns (string memory) {
        if (value == 0) return "0";
        uint256 temp = value;
        uint256 digits;
        while (temp != 0) { digits++; temp /= 10; }
        bytes memory buffer = new bytes(digits);
        while (value != 0) { digits--; buffer[digits] = bytes1(uint8(48 + uint256(value % 10))); value /= 10; }
        return string(buffer);
    }
}
```

**2. Auto-Fund Distributor — Splits received funds**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.28;

contract AgentFundSplitter {
    address public immutable bap578;
    
    struct SplitConfig {
        address[] recipients;
        uint256[] shares; // basis points (10000 = 100%)
    }
    
    mapping(uint256 => SplitConfig) public splits;
    
    event FundsSplit(uint256 indexed tokenId, uint256 totalAmount);
    
    constructor(address _bap578) {
        bap578 = _bap578;
    }
    
    function configureSplit(
        uint256 tokenId,
        address[] calldata recipients,
        uint256[] calldata shares
    ) external {
        // Verify caller is agent owner
        (, , , , address owner) = IBAP578(bap578).getAgentState(tokenId);
        require(msg.sender == owner, "Not agent owner");
        require(recipients.length == shares.length, "Length mismatch");
        
        uint256 totalShares;
        for (uint i = 0; i < shares.length; i++) {
            totalShares += shares[i];
        }
        require(totalShares == 10000, "Shares must total 10000");
        
        splits[tokenId] = SplitConfig(recipients, shares);
    }
    
    function executeSplit(uint256 tokenId) external payable {
        SplitConfig memory config = splits[tokenId];
        require(config.recipients.length > 0, "No split configured");
        
        uint256 total = msg.value;
        for (uint i = 0; i < config.recipients.length; i++) {
            uint256 amount = (total * config.shares[i]) / 10000;
            (bool success, ) = payable(config.recipients[i]).call{value: amount}("");
            require(success, "Transfer failed");
        }
        
        emit FundsSplit(tokenId, total);
    }
}
```

**3. DeFi Strategy — Auto-compound pattern**

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.28;

interface IYieldProtocol {
    function deposit(uint256 amount) external;
    function withdraw(uint256 amount) external;
    function claimRewards() external returns (uint256);
    function balanceOf(address account) external view returns (uint256);
}

contract AgentDeFiStrategy {
    address public immutable bap578;
    IYieldProtocol public immutable yieldProtocol;
    
    mapping(uint256 => bool) public autoCompoundEnabled;
    
    event StrategyExecuted(uint256 indexed tokenId, string action, uint256 amount);
    
    constructor(address _bap578, address _yieldProtocol) {
        bap578 = _bap578;
        yieldProtocol = IYieldProtocol(_yieldProtocol);
    }
    
    function enableAutoCompound(uint256 tokenId) external {
        (, , , , address owner) = IBAP578(bap578).getAgentState(tokenId);
        require(msg.sender == owner, "Not agent owner");
        autoCompoundEnabled[tokenId] = true;
    }
    
    function executeStrategy(uint256 tokenId) external {
        require(autoCompoundEnabled[tokenId], "Auto-compound not enabled");
        
        // Claim rewards
        uint256 rewards = yieldProtocol.claimRewards();
        
        if (rewards > 0) {
            // Re-deposit rewards (auto-compound)
            yieldProtocol.deposit(rewards);
            emit StrategyExecuted(tokenId, "auto-compound", rewards);
        }
    }
}
```

#### Keeper / Automation Integration

Logic contracts don't execute themselves. Use external triggers:

| Trigger | Service | Best for |
|---------|---------|----------|
| Chainlink Automation | [automation.chain.link](https://automation.chain.link) | Production, reliable |
| Gelato Network | [gelato.network](https://www.gelato.network) | Multi-chain, flexible |
| Custom cron + script | Hardhat task / Node.js cron | Development, simple cases |
| User-initiated | Frontend button | Interactive strategies |

```javascript
// Simple keeper script (Node.js)
const { ethers } = require("ethers");

async function runKeeper() {
    const provider = new ethers.providers.JsonRpcProvider(RPC_URL);
    const wallet = new ethers.Wallet(KEEPER_KEY, provider);
    const logic = new ethers.Contract(LOGIC_ADDRESS, LOGIC_ABI, wallet);
    
    // Execute strategy for agent #42 every hour
    setInterval(async () => {
        try {
            const tx = await logic.executeStrategy(42);
            await tx.wait();
            console.log("Strategy executed for agent #42");
        } catch (err) {
            console.error("Execution failed:", err.message);
        }
    }, 3600000); // 1 hour
}
```

### 4 · How can I trust it?

Logic contract trust requires **extra vigilance** because the logic contract is external to BAP-578:

| Trust check | How | Why |
|-------------|-----|-----|
| Source verified on BscScan | Check contract page for green checkmark | Readable, auditable code |
| No admin backdoors | Review for `selfdestruct`, hidden `onlyOwner` functions | Prevent rug pulls |
| No upgradeable proxy | Check if logic is behind a proxy | Proxy logic can change silently |
| Access control | Verify it checks agent ownership for sensitive actions | Prevent unauthorized execution |
| Fund handling | Check if it handles BNB — audit carefully | High-risk area |
| Events emitted | Verify all actions emit events | Auditability |
| Test coverage | Run logic contract tests | Functional correctness |

**Red flags:**
- 🔴 Logic contract is not verified on BscScan
- 🔴 Logic contract has `selfdestruct`
- 🔴 Logic contract is an upgradeable proxy (behavior can change)
- 🔴 Logic contract has hidden admin functions
- 🟡 Logic contract handles BNB without reentrancy protection
- 🟡 Logic contract doesn't check agent ownership

**In short:** "Trust a logic contract by verifying its source, auditing its functions, checking for backdoors, and confirming it respects agent ownership. BAP-578 validates that it's a contract — you must validate what the contract does."

---

## Deployment Flow

```bash
# 1. Write logic contract in contracts/logic/
# 2. Compile
npm run compile

# 3. Deploy logic contract
npx hardhat run scripts/deploy-logic.js --network testnet

# 4. Verify on BscScan
npx hardhat verify --network testnet LOGIC_ADDRESS CONSTRUCTOR_ARGS

# 5. Bind to agent
# Via interact CLI or frontend:
await bap578.setLogicAddress(tokenId, LOGIC_ADDRESS);
```

---

## Related Skills

- **`bap578`** — Core contract spec and `setLogicAddress` function reference
- **`bap578-security-audit`** — Auditing logic contracts for vulnerabilities
- **`bap578-metadata-design`** — Designing the identity that the logic contract serves
