---
id: bap578-security-audit
name: BAP-578 Security Audit
description: >
  Security audit checklist, vulnerability assessment, and threat modeling for
  BAP-578 Non-Fungible Agents on BNB Chain. Covers reentrancy, access control,
  upgrade safety, fund management risks, logic contract threats, and off-chain
  data integrity. Use when auditing, reviewing, or hardening a BAP-578 deployment.
category: BAP-578
author: community
version: 1.0.0
examples:
  - "Audit the BAP-578 contract for vulnerabilities"
  - "Is the NFA contract secure?"
  - "What are the attack vectors for BAP-578?"
  - "Review access control in the agent contract"
  - "Check for reentrancy in BAP-578"
  - "How safe is the upgrade mechanism?"
---

# BAP-578 Security Audit

Use this skill when auditing, reviewing, or hardening a BAP-578 Non-Fungible Agents deployment. This covers vulnerability assessment, threat modeling, access control review, and a comprehensive audit checklist grounded in the actual contract code.

---

## The Four Identity Questions — Security Perspective

### 1 · Who are you?

I am the **security auditor** for BAP-578. I answer the question that protects users and funds: **"What can go wrong, and has it been prevented?"**

I examine every function, modifier, state transition, and external interaction in the BAP-578 contract to identify:
- Vulnerabilities (reentrancy, overflow, access bypass)
- Misconfigurations (wrong treasury, unverified logic contracts)
- Operational risks (key compromise, upgrade abuse, pause misuse)
- Off-chain trust gaps (vault tampering, URI hijacking)

**In short:** "I am the adversary's mirror. I think like an attacker so you don't get attacked."

### 2 · What do you remember?

I maintain a **security knowledge base** for BAP-578:

| Category | What I track |
|----------|-------------|
| Known patterns | OpenZeppelin security patterns used in the contract |
| Attack vectors | Reentrancy, front-running, access control bypass, upgrade hijack |
| Contract guards | `nonReentrant`, `onlyOwner`, `onlyTokenOwner`, `whenNotPaused` |
| Historical exploits | Common ERC-721 and UUPS proxy vulnerabilities from the ecosystem |
| Audit findings | Results from reviewing this specific codebase |
| Dependency versions | OpenZeppelin 4.9.6, Solidity 0.8.28, Hardhat 2.24.2 |

### 3 · What can you do?

#### Threat Model

```
┌────────────────────────────────────────────────┐
│           BAP-578 Threat Surface               │
├────────────────────────────────────────────────┤
│                                                │
│  EXTERNAL ATTACKERS                            │
│  ├─ Reentrancy on withdrawFromAgent            │
│  ├─ Front-running createAgent                  │
│  ├─ Malicious logic contract binding           │
│  ├─ Free-mint farming via multiple wallets     │
│  └─ Vault content spoofing                     │
│                                                │
│  COMPROMISED OWNER KEY                         │
│  ├─ Pause contract indefinitely                │
│  ├─ Emergency withdraw all contract funds      │
│  ├─ Redirect treasury to attacker              │
│  ├─ Push malicious upgrade                     │
│  └─ Grant unlimited free mints                 │
│                                                │
│  LOGIC CONTRACT RISKS                          │
│  ├─ Bound contract is malicious/backdoored     │
│  ├─ Bound contract self-destructs              │
│  └─ Logic address points to upgradeable proxy  │
│                                                │
│  OFF-CHAIN RISKS                               │
│  ├─ Vault URI points to compromised server     │
│  ├─ Metadata URI content changes without       │
│  │  on-chain update                            │
│  └─ IPFS content unpinned / unavailable        │
│                                                │
└────────────────────────────────────────────────┘
```

#### Vulnerability Checklist

**Reentrancy**

| Function | Reentrancy risk | Mitigation in code | Status |
|----------|----------------|-------------------|--------|
| `createAgent` | Low (external call to treasury) | `nonReentrant` modifier | ✅ Protected |
| `fundAgent` | None (no external call) | N/A | ✅ Safe |
| `withdrawFromAgent` | High (sends BNB via `.call`) | `nonReentrant` + Checks-Effects-Interactions | ✅ Protected |
| `emergencyWithdraw` | Medium (sends BNB via `.call`) | `onlyOwner` (no `nonReentrant`) | ⚠️ Relies on owner trust only |

**Access Control**

| Function | Required role | Modifier | Bypass risk |
|----------|-------------|----------|-------------|
| `createAgent` | Anyone | `whenNotPaused` | Low — gated by fee or free mint logic |
| `fundAgent` | Anyone | `whenNotPaused` | Low — anyone can fund any agent |
| `withdrawFromAgent` | Token owner | `onlyTokenOwner` | None — checked against `ownerOf` |
| `setAgentStatus` | Token owner | `onlyTokenOwner` | None |
| `setLogicAddress` | Token owner | `onlyTokenOwner` | None |
| `updateAgentMetadata` | Token owner | `onlyTokenOwner` | None |
| `grantAdditionalFreeMints` | Contract owner | `onlyOwner` | Key compromise risk |
| `setTreasury` | Contract owner | `onlyOwner` | Key compromise risk |
| `setPaused` | Contract owner | `onlyOwner` | Key compromise risk |
| `emergencyWithdraw` | Contract owner | `onlyOwner` | Key compromise risk |
| `_authorizeUpgrade` | Contract owner | `onlyOwner` | Key compromise risk |

**Fund Safety**

| Risk | Assessment | Details |
|------|-----------|---------|
| Agent balance tracking | ✅ Safe | `agentStates[tokenId].balance` updated before external call |
| Treasury transfer on paid mint | ✅ Safe | `require(success)` checks return value |
| Withdrawal pattern | ✅ Safe | CEI pattern: state update → event → external call |
| Emergency withdraw | ⚠️ Risk | Drains entire contract balance, not per-agent — could affect agent balances |
| Direct BNB sends | ✅ Safe | `receive()` reverts |

**Upgrade Safety**

| Risk | Assessment | Details |
|------|-----------|---------|
| Unauthorized upgrade | ✅ Protected | `_authorizeUpgrade` requires `onlyOwner` |
| Storage collision | ⚠️ Manual review needed | New state variables must be appended, never inserted |
| Initialization bypass | ✅ Protected | `_disableInitializers()` in constructor |
| State preservation | ✅ Tested | V2 mock test confirms state preservation |

**Logic Contract Risks**

| Risk | Assessment | Mitigation |
|------|-----------|------------|
| EOA as logic address | ✅ Blocked | `logicAddress.code.length > 0` check |
| Malicious contract | ⚠️ Not audited by BAP-578 | User responsibility to verify logic contract |
| Self-destructed contract | ⚠️ Post-bind risk | Code length check only at bind time, not ongoing |
| Upgradeable proxy as logic | ⚠️ Risk | Logic could change behavior without re-binding |

#### Audit Checklist

```
BAP-578 Security Audit Checklist
═══════════════════════════════════

SMART CONTRACT
  [✅] Compiler version: Solidity 0.8.28 (overflow protection built-in)
  [✅] OpenZeppelin base: v4.9.6 (battle-tested)
  [✅] Reentrancy protection: nonReentrant on createAgent, withdrawFromAgent
  [✅] CEI pattern: withdrawFromAgent updates state before external call
  [✅] Access control: onlyOwner for admin, onlyTokenOwner for agent management
  [✅] Pause mechanism: whenNotPaused on createAgent and fundAgent
  [✅] Zero-address checks: treasury, mint recipient
  [✅] Logic address validation: must be contract or zero address
  [✅] Free-mint non-transferable: _beforeTokenTransfer blocks transfers
  [✅] Burn requires zero balance: _burn checks agentStates[tokenId].balance == 0
  [✅] Direct BNB send rejected: receive() reverts

UPGRADE SAFETY
  [✅] UUPS pattern with _authorizeUpgrade override
  [✅] Initializers disabled in constructor
  [✅] V2 upgrade tested with state preservation
  [⚠️] Storage layout: manually verify no slot collisions on future upgrades

OPERATIONAL SECURITY
  [⚠️] Owner is EOA: recommend multisig for production
  [⚠️] emergencyWithdraw drains all: may desync agent balance tracking
  [⚠️] No timelock on admin functions: owner changes take effect immediately
  [⚠️] No event for setFreeMintsPerUser: state change not easily auditable

OFF-CHAIN SECURITY
  [⚠️] Vault URI mutable: owner can change vaultURI without content guarantee
  [✅] Vault hash anchors integrity: keccak256 verification possible
  [⚠️] IPFS content availability: depends on pinning service
  [⚠️] Metadata URI mutable: tokenURI can be updated by token owner
```

#### Recommendations

| Priority | Finding | Recommendation |
|----------|---------|---------------|
| 🔴 Critical | Owner key is single point of failure | Use Gnosis Safe multisig for contract ownership |
| 🔴 Critical | No timelock on upgrades | Add `TimelockController` before UUPS upgrade calls |
| 🟡 Medium | `emergencyWithdraw` desyncs agent balances | Add per-agent accounting or document clearly |
| 🟡 Medium | No event for `setFreeMintsPerUser` | Add `FreeMintConfigUpdated(uint256)` event |
| 🟡 Medium | Logic contract not re-validated over time | Consider periodic `code.length` check or callback |
| 🟢 Low | Unbounded loop in `tokensOfOwner` | Document gas risk for large holders; consider pagination |
| 🟢 Low | `fundAgent` allows funding non-active agents | Consider gating behind `active` check if desired |

### 4 · How can I trust it?

Trust in the security audit comes from **methodology transparency**:

- **Every finding maps to specific code** — line numbers, function names, modifiers.
- **Severity is rated objectively** — Critical (fund loss), Medium (operational risk), Low (gas/UX).
- **Mitigations are verified** — I check if the code actually implements the claimed protection.
- **Known patterns are referenced** — OpenZeppelin standards, SWC registry, Trail of Bits guidelines.
- **Limitations are stated** — this skill audits the BAP-578 contract only, not bound logic contracts or off-chain services.

**In short:** "Trust the audit because it's reproducible. Every claim maps to code, every risk has a severity, and every limitation is disclosed."

---

## Quick Security Check Commands

```bash
# Run the full test suite (should all pass)
cd non-fungible-agents-BAP-578
npm test

# Check test coverage
npm run coverage

# Lint Solidity for style/security hints
npm run lint

# Static analysis with Slither (if installed)
slither contracts/BAP578.sol --config-file slither.config.json

# Check contract size (must be < 24KB for deployment)
npm run size
```

---

## Related Skills

- **`bap578`** — Core contract spec, build/test/deploy workflows
- **`bap578-scanner`** — Runtime verification and agent-level trust checks
- **`bap578-upgrade`** — Safe upgrade procedures and state migration
