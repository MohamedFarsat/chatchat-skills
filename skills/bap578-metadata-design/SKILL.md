---
id: bap578-metadata-design
name: BAP-578 Metadata & Persona Design
description: >
  Design agent personas, experiences, vault schemas, voice profiles, and
  metadata URIs for BAP-578 Non-Fungible Agents. Covers the AgentMetadata
  struct fields, JSON persona format, vault architecture, IPFS storage,
  and metadata best practices. Use when creating or customizing NFA agent
  identities.
category: BAP-578
author: community
version: 1.0.0
examples:
  - "Design a persona for my BAP-578 agent"
  - "What should I put in the AgentMetadata fields?"
  - "How to structure the vault for an NFA agent"
  - "Create a financial advisor agent persona"
  - "What format should the persona JSON be?"
  - "How to store agent data on IPFS"
---

# BAP-578 Metadata & Persona Design

Use this skill when designing, creating, or customizing the identity and data layer for BAP-578 Non-Fungible Agents. This covers every field in the `AgentMetadata` struct, persona JSON schemas, vault architecture, voice profiles, animation resources, and IPFS storage patterns.

---

## The Four Identity Questions — Metadata Perspective

### 1 · Who are you?

I am the **identity designer** for BAP-578 agents. I answer the most fundamental question an agent faces: **"Who are you, exactly?"**

The `AgentMetadata` struct is the agent's soul — six fields that define its character, knowledge, voice, appearance, and extended memory:

```solidity
struct AgentMetadata {
    string persona;       // WHO the agent is (character, style, tone)
    string experience;    // WHAT the agent knows (role, expertise)
    string voiceHash;     // HOW the agent sounds (voice profile)
    string animationURI;  // HOW the agent looks (visual identity)
    string vaultURI;      // WHERE the agent's extended memory lives
    bytes32 vaultHash;    // PROOF that the memory is authentic
}
```

**In short:** "I design who the agent is, what it knows, how it presents itself, and where its deeper memory lives."

### 2 · What do you remember?

I define and structure **what an agent remembers** across two layers:

| Layer | Fields | Storage | Mutable |
|-------|--------|---------|---------|
| On-chain identity | `persona`, `experience`, `voiceHash` | Contract state | Yes (by owner via `updateAgentMetadata`) |
| On-chain references | `animationURI`, `vaultURI`, `vaultHash` | Contract state | Yes (by owner) |
| Off-chain profile | tokenURI JSON document | IPFS / Arweave | Yes (URI update) |
| Off-chain vault | Extended data at `vaultURI` | IPFS / Arweave / custom | Yes (content + hash update) |

**Memory architecture:**

```
Agent #42 Memory Map
├── ON-CHAIN (immutable history via events)
│   ├── persona: {"traits": ["analytical"], "style": "formal"}
│   ├── experience: "Financial advisor specializing in DeFi"
│   ├── voiceHash: "voice_pro_financial_001"
│   ├── animationURI: "ipfs://QmAnimation..."
│   ├── vaultURI: "ipfs://QmVault..."
│   └── vaultHash: 0xabc123...
│
└── OFF-CHAIN (referenced, hash-verified)
    ├── tokenURI JSON → full profile, image, attributes
    └── vaultURI → conversation history, learned preferences,
                   extended knowledge base, user interactions
```

### 3 · What can you do?

I design metadata for any agent archetype. Here's how each field works:

#### `persona` — Character Definition

JSON-encoded string defining the agent's personality. Recommended schema:

```json
{
  "traits": ["helpful", "analytical", "patient"],
  "style": "professional yet approachable",
  "tone": "warm and engaging",
  "language": "en",
  "greeting": "Hello! I'm your financial advisor agent.",
  "boundaries": ["no medical advice", "no legal counsel"],
  "quirks": ["uses analogies from nature", "ends sessions with a summary"]
}
```

**Persona templates by archetype:**

| Archetype | Traits | Style | Tone |
|-----------|--------|-------|------|
| Financial advisor | analytical, precise, cautious | formal | confident |
| Creative writer | imaginative, expressive, playful | casual | enthusiastic |
| Code reviewer | meticulous, direct, constructive | technical | neutral |
| Customer support | empathetic, patient, solution-oriented | friendly | warm |
| Research analyst | thorough, objective, detail-oriented | academic | measured |
| Personal coach | motivating, supportive, challenging | conversational | energetic |
| Translator | accurate, culturally aware, adaptive | clear | neutral |
| Game master | dramatic, creative, fair | narrative | theatrical |

#### `experience` — Role Summary

Short plain-text string (not JSON). Should be a one-line elevator pitch:

```
"Financial advisor specializing in DeFi yield strategies and risk management"
"Senior code reviewer for TypeScript, Rust, and Solidity projects"
"Creative writing coach with expertise in science fiction world-building"
"Multilingual customer support agent for SaaS products (EN/ES/FR/DE)"
```

**Best practices:**
- Keep under 200 characters.
- Lead with the role, then specialization.
- Include domain keywords for discoverability.

#### `voiceHash` — Audio Profile

Reference ID pointing to a stored voice profile (TTS model, voice clone, etc.):

```
"voice_default"           — Platform default
"voice_pro_financial_001" — Professional financial tone
"voice_casual_creative"   — Casual creative tone
"elevenlabs_abc123"       — ElevenLabs voice clone ID
"custom_v2_deepvoice"     — Custom trained model
```

**Note:** The voiceHash is a reference, not the audio itself. The consuming application resolves it.

#### `animationURI` — Visual Identity

URI pointing to the agent's visual representation:

```
"ipfs://QmAnimation..."         — IPFS-hosted animation/video
"ar://txid123"                  — Arweave permanent storage
"https://cdn.nfa.xyz/agent42/"  — CDN-hosted asset
""                              — No animation (empty string)
```

Supported formats (by convention): MP4, WebM, Lottie JSON, GIF, GLTF (3D).

#### `vaultURI` — Extended Memory

URI pointing to the agent's extended data vault:

```
"ipfs://QmVaultData..."     — IPFS-hosted vault
"ar://vaultTxId"            — Arweave permanent vault
""                          — No vault (empty string)
```

**Vault content schema (recommended):**

```json
{
  "version": "1.0",
  "agentId": 42,
  "createdAt": "2026-01-15T14:30:00Z",
  "updatedAt": "2026-03-01T10:00:00Z",
  "knowledge": {
    "domain": "DeFi yield optimization",
    "sources": ["Aave docs", "Compound whitepaper", "Uniswap V3 research"],
    "lastTrainingDate": "2026-02-15"
  },
  "conversations": {
    "totalSessions": 150,
    "summaries": [
      {"date": "2026-02-28", "topic": "yield farming on Aave V3", "outcome": "strategy recommended"}
    ]
  },
  "preferences": {
    "riskTolerance": "medium",
    "preferredProtocols": ["Aave", "Compound", "Curve"],
    "communicationStyle": "detailed with charts"
  },
  "customData": {}
}
```

#### `vaultHash` — Integrity Anchor

`bytes32` keccak256 hash of the vault content. Computed as:

```javascript
// JavaScript (ethers.js)
const vaultHash = ethers.utils.keccak256(
  ethers.utils.toUtf8Bytes(JSON.stringify(vaultContent))
);

// JavaScript (viem)
import { keccak256, toBytes } from "viem";
const vaultHash = keccak256(toBytes(JSON.stringify(vaultContent)));
```

**Rules:**
- Update `vaultHash` every time vault content changes.
- If no vault exists, use `bytes32(0)` (64 zero hex chars).
- Verifier fetches `vaultURI`, hashes content, compares to on-chain `vaultHash`.

#### tokenURI JSON — ERC-721 Metadata

The `tokenURI` returned by `tokenURI(tokenId)` should point to a standard ERC-721 JSON:

```json
{
  "name": "NFA #42 — Financial Advisor",
  "description": "A DeFi-specialized financial advisor agent on BNB Chain.",
  "image": "ipfs://QmAgentAvatar...",
  "external_url": "https://nfa.xyz/agent/42",
  "attributes": [
    {"trait_type": "Experience", "value": "Financial advisor"},
    {"trait_type": "Style", "value": "formal"},
    {"trait_type": "Language", "value": "English"},
    {"trait_type": "Active", "value": "true"},
    {"trait_type": "Free Minted", "value": "false"}
  ],
  "animation_url": "ipfs://QmAnimation..."
}
```

### 4 · How can I trust it?

Metadata trust follows a **verification chain**:

1. **On-chain fields are authoritative** — `persona`, `experience`, `voiceHash` stored directly in contract state.
2. **Off-chain content is hash-anchored** — `vaultHash` proves vault authenticity. If `keccak256(content) !== vaultHash`, the data is untrustworthy.
3. **Metadata URI is mutable** — token owner can change it. Always check the current `tokenURI` from the contract, not cached values.
4. **IPFS is content-addressed** — IPFS URIs (`ipfs://Qm...`) are inherently tamper-proof. The hash IS the content. But availability depends on pinning.
5. **Arweave is permanent** — `ar://` URIs are stored forever on Arweave. Most trustworthy for long-term data.
6. **HTTP URIs are weakest** — `https://` can change content at any time. Avoid for critical data; use `vaultHash` verification.

**Trust hierarchy:**
```
Most trusted → On-chain state (persona, experience in contract)
             → IPFS/Arweave with vaultHash verification
             → IPFS/Arweave without hash verification
Least trusted → HTTP URIs without hash verification
```

**In short:** "Trust metadata by checking on-chain state first, then verifying off-chain content against vaultHash. Never trust HTTP URIs without hash proof."

---

## IPFS Storage Guide

### Upload to IPFS (using Pinata)

```bash
# Install Pinata CLI
npm install -g @pinata/sdk

# Upload vault content
curl -X POST "https://api.pinata.cloud/pinning/pinJSONToIPFS" \
  -H "Authorization: Bearer YOUR_JWT" \
  -H "Content-Type: application/json" \
  -d '{"pinataContent": { "version": "1.0", "knowledge": {...} }}'

# Returns: { "IpfsHash": "QmVault...", "PinSize": 1234, "Timestamp": "..." }
# Use as: vaultURI = "ipfs://QmVault..."
```

### Compute Hash After Upload

```javascript
const content = JSON.stringify(vaultContent);
const vaultHash = ethers.utils.keccak256(ethers.utils.toUtf8Bytes(content));
// Use this vaultHash when calling createAgent or updateAgentMetadata
```

---

## Complete Example — Creating a Financial Advisor Agent

```javascript
const metadata = {
  persona: JSON.stringify({
    traits: ["analytical", "precise", "patient"],
    style: "formal but accessible",
    tone: "confident and reassuring",
    language: "en",
    greeting: "Welcome! I'm your DeFi advisor. How can I help you today?",
    boundaries: ["no tax advice", "no guaranteed returns"],
  }),
  experience: "DeFi yield strategist specializing in Aave, Compound, and Curve",
  voiceHash: "voice_pro_financial_001",
  animationURI: "ipfs://QmFinancialAdvisorAnimation",
  vaultURI: "ipfs://QmFinancialAdvisorVault",
  vaultHash: "0x...", // keccak256 of vault JSON content
};

const metadataURI = "ipfs://QmTokenURIJson"; // ERC-721 JSON with name, image, attributes

// Mint
await contract.createAgent(
  myAddress,
  ethers.constants.AddressZero, // no logic contract
  metadataURI,
  metadata
);
```

---

## Related Skills

- **`bap578`** — Core contract spec and function reference
- **`frontend-web3-bap578`** — React components for displaying and editing metadata
- **`bap578-scanner`** — Verifying metadata integrity on-chain
