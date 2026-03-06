---
id: bap578-testing
name: BAP-578 Testing
description: >
  Comprehensive testing strategies for BAP-578 Non-Fungible Agents. Covers
  unit tests, integration tests, upgrade tests, gas profiling, edge cases,
  and test-driven development patterns using Hardhat and Chai. Use when
  writing, running, or debugging tests for BAP-578 contracts.
category: BAP-578
author: community
version: 1.0.0
examples:
  - "How to test BAP-578 contracts"
  - "Write tests for the createAgent function"
  - "What edge cases should I test?"
  - "Run BAP-578 test suite"
  - "Test the free mint logic"
  - "Debug a failing BAP-578 test"
---

# BAP-578 Testing

Use this skill when writing, running, debugging, or expanding the test suite for BAP-578 Non-Fungible Agents. Covers every testable aspect of the contract with patterns, examples, and a complete edge-case checklist.

---

## The Four Identity Questions — Testing Perspective

### 1 · Who are you?

I am the **quality gate** for BAP-578. I answer: **"Does this contract do what it claims, and nothing it shouldn't?"**

The existing test suite (`test/BAP578.test.js`) covers 25+ test cases across 7 categories. I help you understand, run, extend, and debug these tests.

**In short:** "I prove the contract works. If I pass, ship it. If I fail, fix it."

### 2 · What do you remember?

I track **every behavior the contract should exhibit**:

| Test category | Cases | What's verified |
|---------------|-------|----------------|
| Deployment | 3 | Name, symbol, treasury, owner correctly set |
| Free Mints | 4 | 3 free per user, payment after, self-only, non-transferable |
| Agent Creation | 3 | Free mint, paid mint, extended metadata storage |
| Agent Management | 7 | Status, funding, withdrawal, logic address, metadata, pause, ownership |
| View Functions | 4 | State query, tokens of owner, total supply, free mints remaining |
| Admin Functions | 5 | Pause, treasury, owner-only guard, bonus mints, emergency withdraw |
| UUPS Upgrade | 2 | State preservation, owner-only upgrade |

### 3 · What can you do?

#### Running Tests

```bash
cd non-fungible-agents-BAP-578

# Run all tests
npm test

# Run with gas reporting
REPORT_GAS=true npm test

# Run specific test file
npx hardhat test test/BAP578.test.js

# Run with coverage
npm run coverage

# Run a single test (grep pattern)
npx hardhat test --grep "Should allow first 3 mints for free"
```

#### Test Structure

Every test follows this pattern:

```javascript
describe("Category", function () {
    beforeEach(async function () {
        // Deploy fresh contract + set up test state
    });

    it("Should [expected behavior]", async function () {
        // Arrange: set up specific state
        // Act: call the function
        // Assert: check results
    });
});
```

#### Helper Function

```javascript
function createAgentMetadata(overrides = {}) {
    return {
        persona: overrides.persona || '{"traits": "friendly", "style": "casual"}',
        experience: overrides.experience || "AI Assistant specialized in blockchain",
        voiceHash: overrides.voiceHash || "voice_001",
        animationURI: overrides.animationURI || "ipfs://animation1",
        vaultURI: overrides.vaultURI || "ipfs://vault1",
        vaultHash: overrides.vaultHash || ethers.utils.formatBytes32String("vault1"),
    };
}
```

#### Edge Cases Checklist

**Minting:**
- [ ] First 3 mints are free (no value required)
- [ ] 4th mint requires exactly 0.01 BNB
- [ ] Free mints can only be to self (`to == msg.sender`)
- [ ] Paid mints can be to any address
- [ ] Free-minted tokens are non-transferable
- [ ] Free-minted tokens can be burned (with zero balance)
- [ ] Mint to zero address reverts
- [ ] Mint with wrong fee amount reverts
- [ ] Mint when paused reverts
- [ ] Token IDs increment correctly (1, 2, 3...)
- [ ] AgentCreated event emitted with correct args
- [ ] Treasury receives fee on paid mint
- [ ] Metadata stored correctly for all 6 fields

**Funding & Withdrawal:**
- [ ] Anyone can fund any agent
- [ ] Fund when paused reverts
- [ ] Fund non-existent token reverts
- [ ] Withdraw full balance succeeds
- [ ] Withdraw partial balance succeeds
- [ ] Withdraw more than balance reverts
- [ ] Only token owner can withdraw
- [ ] Reentrancy on withdraw is blocked
- [ ] BNB actually arrives at recipient
- [ ] Agent balance updates correctly
- [ ] Events emitted correctly

**Agent Management:**
- [ ] Only owner can toggle status
- [ ] Only owner can set logic address
- [ ] Logic address must be contract (not EOA)
- [ ] Zero address is valid for logic (unbind)
- [ ] Only owner can update metadata
- [ ] All 6 metadata fields update correctly
- [ ] Token URI updates correctly
- [ ] Events emitted for each change

**Admin:**
- [ ] Only contract owner can pause
- [ ] Only contract owner can change treasury
- [ ] Treasury cannot be zero address
- [ ] Only contract owner can grant bonus mints
- [ ] Bonus mints stack with default
- [ ] Emergency withdraw sends all balance to owner
- [ ] Emergency withdraw with zero balance reverts
- [ ] Direct BNB send to contract reverts

**Upgrade:**
- [ ] State preserved after upgrade (all fields)
- [ ] V2 functions work after upgrade
- [ ] V1 functions still work after upgrade
- [ ] Only owner can authorize upgrade
- [ ] Token counter continues correctly
- [ ] Agent balances preserved

#### Writing New Tests

**Pattern for testing a new function:**

```javascript
describe("New Feature", function () {
    let nfa, owner, addr1, addr2, treasury;

    beforeEach(async function () {
        [owner, addr1, addr2, treasury] = await ethers.getSigners();
        const BAP578 = await ethers.getContractFactory("BAP578");
        nfa = await upgrades.deployProxy(BAP578,
            ["Non-Fungible Agents", "NFA", treasury.address],
            { initializer: "initialize", kind: "uups" }
        );
    });

    it("Should succeed with valid input", async function () {
        // Happy path
        const tx = await nfa.connect(addr1).someFunction(validArgs);
        await expect(tx).to.emit(nfa, "ExpectedEvent").withArgs(expectedArgs);
        expect(await nfa.someView()).to.equal(expectedValue);
    });

    it("Should revert with invalid input", async function () {
        await expect(
            nfa.connect(addr1).someFunction(invalidArgs)
        ).to.be.revertedWith("Expected error message");
    });

    it("Should enforce access control", async function () {
        await expect(
            nfa.connect(addr2).adminFunction()
        ).to.be.revertedWith("Ownable: caller is not the owner");
    });
});
```

#### Debugging Failed Tests

| Symptom | Likely cause | Fix |
|---------|-------------|-----|
| "Token does not exist" | Token ID doesn't exist yet | Mint first in `beforeEach` |
| "Not token owner" | Wrong signer used | Use `.connect(correctSigner)` |
| "Incorrect fee" | Free mints exhausted, no value sent | Add `{ value: MINT_FEE }` |
| "Contract is paused" | Previous test paused contract | Check `beforeEach` resets state |
| Gas estimation failed | Transaction would revert | Check revert reason in the call |
| Timeout | Slow network or large test | Increase mocha timeout in hardhat.config.js |

### 4 · How can I trust it?

Testing trust comes from **coverage and correctness**:

- **28+ test cases** covering all contract functions.
- **Revert testing** — every invalid path is checked for correct error messages.
- **Event verification** — `to.emit()` confirms events fire with correct args.
- **State checks** — every write function is followed by a read to verify state changed.
- **Access control tests** — every restricted function tested with unauthorized caller.
- **Upgrade tests** — full state preservation verified across V1 → V2.
- **Coverage report** — `npm run coverage` shows line/branch/function coverage percentages.

**In short:** "Trust the tests because they cover happy paths, error paths, access control, events, state changes, and upgrades. Run `npm test` — if all pass, the contract is behaving as designed."

---

## Related Skills

- **`bap578`** — Core contract spec and function reference
- **`bap578-security-audit`** — Security-focused test recommendations
- **`bap578-upgrade`** — Upgrade-specific testing patterns
