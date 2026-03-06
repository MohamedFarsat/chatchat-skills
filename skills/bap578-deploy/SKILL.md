---
id: bap578-deploy
name: BAP-578 Deployment
description: >
  Deploy BAP-578 Non-Fungible Agents to BNB Chain (testnet and mainnet).
  Covers environment setup, Hardhat configuration, proxy deployment via
  UUPS, BscScan verification, post-deployment checks, and operational
  runbook. Use when deploying, verifying, or configuring a BAP-578 contract.
category: BAP-578
author: community
version: 1.0.0
examples:
  - "Deploy BAP-578 to BSC testnet"
  - "How to deploy the NFA contract"
  - "Verify BAP-578 on BscScan"
  - "Set up Hardhat for BNB Chain"
  - "What's the deployment checklist?"
  - "Configure the treasury after deployment"
---

# BAP-578 Deployment

Use this skill when deploying BAP-578 Non-Fungible Agents to BNB Chain testnet or mainnet. This covers every step from environment setup to post-deployment verification.

---

## The Four Identity Questions — Deployment Perspective

### 1 · Who are you?

I am the **launch operator** for BAP-578. I answer: **"How do I get this contract live on BNB Chain, correctly and safely?"**

Deployment is irreversible — a mistake means redeploying at cost. I ensure every step is verified before execution.

**In short:** "I put agents on-chain. I make sure it's done right the first time."

### 2 · What do you remember?

I track **deployment state** across environments:

| Info | Where stored | Example |
|------|-------------|---------|
| Proxy address | `deployments/{network}_deployment.json` | `0xABC...` |
| Implementation address | Same file | `0xDEF...` |
| Treasury address | Constructor arg + `treasuryAddress()` | `0x123...` |
| Deploy block | Same file | `38201000` |
| Deployer address | Transaction sender | `0x789...` |
| Network | Hardhat config | `bscTestnet` / `bsc` |
| Verification status | BscScan | ✅ Verified |

### 3 · What can you do?

#### Prerequisites

```bash
cd non-fungible-agents-BAP-578

# Install dependencies
npm install

# Copy environment file
cp .env.example .env
```

Edit `.env`:
```env
# Required for deployment
PRIVATE_KEY=0xYourPrivateKey
BSC_TESTNET_RPC=https://data-seed-prebsc-1-s1.binance.org:8545/
BSC_MAINNET_RPC=https://bsc-dataseed.binance.org/
BSCSCAN_API_KEY=YourBscScanApiKey

# Treasury address (receives mint fees)
TREASURY_ADDRESS=0xYourTreasuryAddress
```

#### Step-by-Step Deployment

**1. Compile contracts**
```bash
npm run compile
# or: npx hardhat compile
```

**2. Run full test suite**
```bash
npm test
# ALL tests must pass before deployment
```

**3. Deploy to testnet**
```bash
npx hardhat run scripts/deploy.js --network bscTestnet
```

Expected output:
```
Deploying BAP578 to bscTestnet...
Proxy deployed to: 0xABC...
Implementation deployed to: 0xDEF...
Treasury set to: 0x123...
Deployment saved to deployments/bscTestnet_deployment.json
```

**4. Verify on BscScan**
```bash
npx hardhat verify --network bscTestnet IMPLEMENTATION_ADDRESS
```

For proxy verification, use BscScan's "Is this a proxy?" feature:
1. Go to `https://testnet.bscscan.com/address/{PROXY_ADDRESS}`
2. Click "More Options" → "Is this a proxy?"
3. Click "Verify" — BscScan auto-detects the implementation

**5. Post-deployment checks**
```bash
# Interactive check script
npx hardhat run scripts/interact.js --network bscTestnet
```

Or manual checks:
```bash
# Check owner
cast call $PROXY "owner()" --rpc-url $BSC_TESTNET_RPC

# Check treasury
cast call $PROXY "treasuryAddress()" --rpc-url $BSC_TESTNET_RPC

# Check total supply (should be 0)
cast call $PROXY "getTotalSupply()" --rpc-url $BSC_TESTNET_RPC

# Check pause state (should be false)
cast call $PROXY "paused()" --rpc-url $BSC_TESTNET_RPC

# Test mint
cast send $PROXY "createAgent(address,address,string,(string,string,string,string,string,bytes32))" \
  $MY_ADDRESS 0x0000000000000000000000000000000000000000 "ipfs://test" \
  '("test_persona","test_experience","","","",0x0000000000000000000000000000000000000000000000000000000000000000)' \
  --rpc-url $BSC_TESTNET_RPC --private-key $PRIVATE_KEY
```

**6. Deploy to mainnet** (only after testnet is verified)
```bash
npx hardhat run scripts/deploy.js --network bsc
npx hardhat verify --network bsc IMPLEMENTATION_ADDRESS
```

#### Deployment Checklist

```
PRE-DEPLOYMENT
  [ ] All tests pass (npm test)
  [ ] .env configured with correct keys and addresses
  [ ] Treasury address is correct (double-check!)
  [ ] Deployer wallet has enough BNB for gas
      - Testnet: 0.1 tBNB (get from faucet)
      - Mainnet: 0.05 BNB (~$30)
  [ ] Private key is for the intended deployer/owner

DEPLOYMENT
  [ ] Contract compiled without errors
  [ ] Proxy deployed successfully
  [ ] Implementation address recorded
  [ ] Deployment JSON saved

VERIFICATION
  [ ] Implementation verified on BscScan
  [ ] Proxy marked as proxy on BscScan
  [ ] "Read as Proxy" works on BscScan

POST-DEPLOYMENT
  [ ] owner() returns expected address
  [ ] treasuryAddress() returns expected address
  [ ] paused() returns false
  [ ] getTotalSupply() returns 0
  [ ] Test mint succeeds (free mint)
  [ ] Test fund succeeds
  [ ] Test withdraw succeeds
  [ ] Test metadata update succeeds

PRODUCTION HARDENING
  [ ] Transfer ownership to multisig (recommended)
  [ ] Announce contract address to community
  [ ] Update frontend with proxy address
  [ ] Monitor first 10 mints for issues
```

#### Gas Costs

| Operation | Estimated gas | BNB cost (~5 gwei) |
|-----------|-------------|-------------------|
| Deploy proxy + implementation | ~4,500,000 | ~0.0225 BNB |
| Verify on BscScan | N/A (API call) | Free |
| First mint (free) | ~350,000 | ~0.00175 BNB |
| Paid mint | ~380,000 | ~0.0019 BNB |
| Fund agent | ~50,000 | ~0.00025 BNB |
| Withdraw | ~60,000 | ~0.0003 BNB |

#### Testnet BNB Faucet

Get free testnet BNB: [https://www.bnbchain.org/en/testnet-faucet](https://www.bnbchain.org/en/testnet-faucet)

### 4 · How can I trust it?

Deployment trust is built on **verification and transparency**:

- **Source verified on BscScan** — anyone can read the contract code and confirm it matches the repository.
- **Proxy verification** — BscScan links the proxy to its implementation, showing the actual logic.
- **Deployment JSON recorded** — proxy address, implementation, deployer, block number all saved for reference.
- **Post-deployment checks** — a checklist confirms every setting matches expectations before announcing.
- **Testnet first** — always deploy and test on testnet before committing to mainnet.

**In short:** "Trust the deployment because it's verified on BscScan, tested on testnet, and every address and setting is double-checked against a checklist."

---

## Troubleshooting

| Issue | Cause | Fix |
|-------|-------|-----|
| "Insufficient funds" | Not enough BNB for gas | Top up deployer wallet |
| "Nonce too low" | Pending transaction | Wait or speed up in wallet |
| Verification fails | Wrong compiler version | Match `hardhat.config.js` solidity version |
| Proxy not detected | BscScan needs time | Wait 5 minutes, retry "Is this a proxy?" |
| "Already initialized" | Proxy already set up | This is expected — proxy can only initialize once |

---

## Related Skills

- **`bap578`** — Core contract spec and Hardhat configuration
- **`bap578-testing`** — Full test suite to run before deployment
- **`bap578-security-audit`** — Security checks before going live
- **`bap578-upgrade`** — Upgrading after initial deployment
