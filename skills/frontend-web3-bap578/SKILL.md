---
id: frontend-web3-bap578
name: Frontend Web3 for BAP-578
description: >
  Build a Next.js frontend to interact with the BAP-578 Non-Fungible Agents
  contract on BNB Chain. Covers wallet connection (wagmi + RainbowKit + viem),
  BNB Chain setup, contract ABI integration, React hooks for minting, funding,
  withdrawing, metadata management, and agent identity display. Use when
  building a Web3 UI for BAP-578 or integrating NFA into an existing Next.js app.
category: BAP-578
author: community
version: 1.0.0
examples:
  - "Add Web3 support for BAP-578 to my Next.js app"
  - "Build a mint page for Non-Fungible Agents"
  - "Connect wallet to BNB Chain and read agent data"
  - "Create a dashboard to manage my NFA agents"
  - "How do I call createAgent from the frontend?"
  - "Set up wagmi and viem for BSC"
---

# Frontend Web3 for BAP-578

Use this skill when building a Next.js / React frontend that interacts with the BAP-578 Non-Fungible Agents smart contract on BNB Chain. This covers everything from installing Web3 dependencies to building production-ready UI components for minting, funding, managing, and displaying agent identities.

---

## The Four Identity Questions — Frontend Perspective

Every BAP-578 skill must answer these four questions. Here they are answered from the **frontend integration** angle.

### 1 · Who are you?

I am the **frontend Web3 layer** for BAP-578 Non-Fungible Agents. I connect a Next.js application to the BAP-578 smart contract on BNB Chain using wagmi, viem, and RainbowKit. My job is to let users:

- **Connect their wallet** (MetaMask, WalletConnect, Coinbase Wallet) to BNB Chain.
- **See their agent identity** — token ID, owner, persona, experience, logic contract — rendered in a UI.
- **Interact with their agents** — mint, fund, withdraw, toggle status, update metadata — all through React components.

Without me, the agent identity lives only as raw on-chain data. I make it **visible, interactive, and human-readable**.

### 2 · What do you remember?

I read and display everything the BAP-578 contract remembers:

| What | How (frontend) | Source |
|------|---------------|--------|
| Owner & token ID | `useAgentState(tokenId)` → `ownerOf` | On-chain |
| Active/inactive status | `useAgentState(tokenId)` → `active` | On-chain |
| BNB balance | `useAgentState(tokenId)` → `balance` | On-chain |
| Persona & experience | `useAgentMetadata(tokenId)` → `AgentMetadata` struct | On-chain |
| Vault content | Fetch `vaultURI`, verify with `vaultHash` via `keccak256` | Off-chain (hash-anchored) |
| Creation time | `useAgentState(tokenId)` → `createdAt` | On-chain |
| Event history | `useWatchContractEvent` + historical log queries | On-chain events |
| Free-mint status | `useIsFreeMint(tokenId)` | On-chain |

**Trust rule:** I display off-chain vault data only after client-side hash verification using `verifyVaultIntegrity()`. Unverified data is labeled as such.

### 3 · What can you do?

Through the frontend, users can perform every BAP-578 action:

| Action | Hook | UX |
|--------|------|----|
| Connect wallet to BNB Chain | RainbowKit `<ConnectButton>` | One-click modal |
| Mint a new agent (free or paid) | `useCreateAgent()` | Form with persona, experience, URI |
| Fund an agent with BNB | `useFundAgent()` | Amount input + send |
| Withdraw BNB from agent | `useWithdrawFromAgent()` | Amount input + confirm |
| Toggle agent active/inactive | `useSetAgentStatus()` | Toggle switch |
| Update persona, experience, vault | `useUpdateAgentMetadata()` | Edit form |
| View all owned agents | `useMyAgents()` | Dashboard grid |
| View agent identity card | `useAgentState()` + `useAgentMetadata()` | Four-question identity card |
| Verify vault integrity | `verifyVaultIntegrity()` | Hash comparison display |
| Watch real-time events | `useWatchAgentCreated()` / `useWatchAgentFunded()` | Live notifications |

### 4 · How can I trust it?

Frontend trust is built on **transparency and verification**:

- **All reads come from the chain** — `useReadContract` calls the deployed proxy directly via RPC. No backend intermediary.
- **All writes go through the user's wallet** — the user signs every transaction. The frontend never holds private keys.
- **Vault integrity is verified client-side** — `keccak256(fetchedContent) === onChainVaultHash`. If it doesn't match, the UI shows "unverified".
- **Contract ABI is open** — the ABI in `config/bap578.ts` maps 1:1 to the verified source on BscScan. Anyone can audit.
- **Event history is immutable** — event watchers read from chain logs, not from any database.
- **Open source** — all hooks, components, and config are visible in the codebase. No hidden logic.

**In short:** "Trust the frontend because it's a transparent window into the chain — it reads, displays, and signs. It never stores keys or fabricates data."

---

## When to use this skill

- Adding Web3 wallet connection to an existing Next.js app for BAP-578.
- Building a mint page, agent dashboard, or agent profile viewer for NFA.
- Integrating BAP-578 contract reads/writes into React components.
- Setting up wagmi, viem, and RainbowKit for BNB Chain.

## When NOT to use this skill

- For smart contract development or Solidity — use the `bap578` skill instead.
- For non-BAP-578 Web3 frontends — use the `blockchain-developer` or `wagmi-development` skill.

## Trigger patterns

- "Add Web3 to my app for BAP-578"
- "Build a mint page for NFA"
- "Connect wallet to BSC"
- "Read agent metadata from the frontend"
- "Create agent dashboard UI"
- "Set up wagmi for BNB Chain"

---

## Step 1 — Install Web3 Dependencies

```bash
# From your Next.js web directory
npm install wagmi viem @tanstack/react-query @rainbow-me/rainbowkit
```

| Package | Purpose |
|---------|---------|
| `viem` | Low-level TypeScript client for EVM chains (reads, writes, encoding) |
| `wagmi` | React hooks for wallet connection, contract reads/writes |
| `@tanstack/react-query` | Async state management (required by wagmi) |
| `@rainbow-me/rainbowkit` | Polished wallet connection modal (MetaMask, WalletConnect, etc.) |

---

## Step 2 — BNB Chain Configuration

Create `config/web3.ts`:

```typescript
import { http, createConfig } from "wagmi";
import { bsc, bscTestnet } from "wagmi/chains";
import { getDefaultConfig } from "@rainbow-me/rainbowkit";

// Use RainbowKit's getDefaultConfig for quick setup
export const wagmiConfig = getDefaultConfig({
  appName: "ChatChat NFA",
  // Get a free project ID at https://cloud.walletconnect.com
  projectId: process.env.NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID || "",
  chains: [bsc, bscTestnet],
  transports: {
    [bsc.id]: http("https://bsc-dataseed.binance.org/"),
    [bscTestnet.id]: http(
      "https://data-seed-prebsc-1-s1.binance.org:8545/"
    ),
  },
  ssr: true, // Required for Next.js
});
```

Add to `.env.local`:
```env
NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID=your_walletconnect_project_id
NEXT_PUBLIC_BAP578_PROXY_ADDRESS=0xYourDeployedProxyAddress
NEXT_PUBLIC_CHAIN_ID=97
```

---

## Step 3 — Web3 Provider Setup

Create `providers/Web3Provider.tsx`:

```tsx
"use client";

import { WagmiProvider } from "wagmi";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { RainbowKitProvider, darkTheme } from "@rainbow-me/rainbowkit";
import "@rainbow-me/rainbowkit/styles.css";
import { wagmiConfig } from "@/config/web3";

const queryClient = new QueryClient();

export function Web3Provider({ children }: { children: React.ReactNode }) {
  return (
    <WagmiProvider config={wagmiConfig}>
      <QueryClientProvider client={queryClient}>
        <RainbowKitProvider
          theme={darkTheme({
            accentColor: "#F0B90B", // BNB yellow
            borderRadius: "medium",
          })}
        >
          {children}
        </RainbowKitProvider>
      </QueryClientProvider>
    </WagmiProvider>
  );
}
```

Add to your root `layout.tsx` or `providers.tsx`:

```tsx
import { Web3Provider } from "@/providers/Web3Provider";

// Wrap your app
<Web3Provider>
  {children}
</Web3Provider>
```

---

## Step 4 — Contract ABI and Address

Create `config/bap578.ts`:

```typescript
// BAP-578 Contract Address (deployed proxy)
export const BAP578_ADDRESS = process.env
  .NEXT_PUBLIC_BAP578_PROXY_ADDRESS as `0x${string}`;

// AgentMetadata struct type (mirrors Solidity)
export interface AgentMetadata {
  persona: string;       // JSON-encoded character traits
  experience: string;    // Agent role/expertise summary
  voiceHash: string;     // Voice profile reference
  animationURI: string;  // Animation/video URI
  vaultURI: string;      // Extended data vault URI
  vaultHash: `0x${string}`; // bytes32 vault content hash
}

// AgentState type (from getAgentState return)
export interface AgentState {
  balance: bigint;
  active: boolean;
  logicAddress: `0x${string}`;
  createdAt: bigint;
  owner: `0x${string}`;
}

// Minimal ABI covering all user-facing functions
// Generate the full ABI from: npx hardhat compile (then artifacts/contracts/BAP578.sol/BAP578.json)
export const BAP578_ABI = [
  // === WRITE FUNCTIONS ===
  {
    name: "createAgent",
    type: "function",
    stateMutability: "payable",
    inputs: [
      { name: "to", type: "address" },
      { name: "logicAddress", type: "address" },
      { name: "metadataURI", type: "string" },
      {
        name: "extendedMetadata",
        type: "tuple",
        components: [
          { name: "persona", type: "string" },
          { name: "experience", type: "string" },
          { name: "voiceHash", type: "string" },
          { name: "animationURI", type: "string" },
          { name: "vaultURI", type: "string" },
          { name: "vaultHash", type: "bytes32" },
        ],
      },
    ],
    outputs: [{ name: "", type: "uint256" }],
  },
  {
    name: "fundAgent",
    type: "function",
    stateMutability: "payable",
    inputs: [{ name: "tokenId", type: "uint256" }],
    outputs: [],
  },
  {
    name: "withdrawFromAgent",
    type: "function",
    stateMutability: "nonpayable",
    inputs: [
      { name: "tokenId", type: "uint256" },
      { name: "amount", type: "uint256" },
    ],
    outputs: [],
  },
  {
    name: "setAgentStatus",
    type: "function",
    stateMutability: "nonpayable",
    inputs: [
      { name: "tokenId", type: "uint256" },
      { name: "active", type: "bool" },
    ],
    outputs: [],
  },
  {
    name: "setLogicAddress",
    type: "function",
    stateMutability: "nonpayable",
    inputs: [
      { name: "tokenId", type: "uint256" },
      { name: "newLogicAddress", type: "address" },
    ],
    outputs: [],
  },
  {
    name: "updateAgentMetadata",
    type: "function",
    stateMutability: "nonpayable",
    inputs: [
      { name: "tokenId", type: "uint256" },
      { name: "newMetadataURI", type: "string" },
      {
        name: "newExtendedMetadata",
        type: "tuple",
        components: [
          { name: "persona", type: "string" },
          { name: "experience", type: "string" },
          { name: "voiceHash", type: "string" },
          { name: "animationURI", type: "string" },
          { name: "vaultURI", type: "string" },
          { name: "vaultHash", type: "bytes32" },
        ],
      },
    ],
    outputs: [],
  },
  // === READ FUNCTIONS ===
  {
    name: "getAgentState",
    type: "function",
    stateMutability: "view",
    inputs: [{ name: "tokenId", type: "uint256" }],
    outputs: [
      { name: "balance", type: "uint256" },
      { name: "active", type: "bool" },
      { name: "logicAddress", type: "address" },
      { name: "createdAt", type: "uint256" },
      { name: "owner", type: "address" },
    ],
  },
  {
    name: "getAgentMetadata",
    type: "function",
    stateMutability: "view",
    inputs: [{ name: "tokenId", type: "uint256" }],
    outputs: [
      {
        name: "metadata",
        type: "tuple",
        components: [
          { name: "persona", type: "string" },
          { name: "experience", type: "string" },
          { name: "voiceHash", type: "string" },
          { name: "animationURI", type: "string" },
          { name: "vaultURI", type: "string" },
          { name: "vaultHash", type: "bytes32" },
        ],
      },
      { name: "metadataURI", type: "string" },
    ],
  },
  {
    name: "tokensOfOwner",
    type: "function",
    stateMutability: "view",
    inputs: [{ name: "account", type: "address" }],
    outputs: [{ name: "", type: "uint256[]" }],
  },
  {
    name: "getTotalSupply",
    type: "function",
    stateMutability: "view",
    inputs: [],
    outputs: [{ name: "", type: "uint256" }],
  },
  {
    name: "getFreeMints",
    type: "function",
    stateMutability: "view",
    inputs: [{ name: "user", type: "address" }],
    outputs: [{ name: "", type: "uint256" }],
  },
  {
    name: "MINT_FEE",
    type: "function",
    stateMutability: "view",
    inputs: [],
    outputs: [{ name: "", type: "uint256" }],
  },
  {
    name: "ownerOf",
    type: "function",
    stateMutability: "view",
    inputs: [{ name: "tokenId", type: "uint256" }],
    outputs: [{ name: "", type: "address" }],
  },
  {
    name: "balanceOf",
    type: "function",
    stateMutability: "view",
    inputs: [{ name: "owner", type: "address" }],
    outputs: [{ name: "", type: "uint256" }],
  },
  {
    name: "tokenURI",
    type: "function",
    stateMutability: "view",
    inputs: [{ name: "tokenId", type: "uint256" }],
    outputs: [{ name: "", type: "string" }],
  },
  {
    name: "isFreeMint",
    type: "function",
    stateMutability: "view",
    inputs: [{ name: "tokenId", type: "uint256" }],
    outputs: [{ name: "", type: "bool" }],
  },
  // === EVENTS ===
  {
    name: "AgentCreated",
    type: "event",
    inputs: [
      { name: "tokenId", type: "uint256", indexed: true },
      { name: "owner", type: "address", indexed: true },
      { name: "logicAddress", type: "address", indexed: false },
      { name: "metadataURI", type: "string", indexed: false },
    ],
  },
  {
    name: "AgentFunded",
    type: "event",
    inputs: [
      { name: "tokenId", type: "uint256", indexed: true },
      { name: "amount", type: "uint256", indexed: false },
    ],
  },
  {
    name: "AgentWithdraw",
    type: "event",
    inputs: [
      { name: "tokenId", type: "uint256", indexed: true },
      { name: "amount", type: "uint256", indexed: false },
    ],
  },
  {
    name: "MetadataUpdated",
    type: "event",
    inputs: [
      { name: "tokenId", type: "uint256", indexed: true },
    ],
  },
  {
    name: "AgentStatusChanged",
    type: "event",
    inputs: [
      { name: "tokenId", type: "uint256", indexed: true },
      { name: "active", type: "bool", indexed: false },
    ],
  },
] as const;
```

> **Tip:** For the full ABI, compile the contract with `npx hardhat compile` and copy from `artifacts/contracts/BAP578.sol/BAP578.json` → `abi` field.

---

## Step 5 — React Hooks for BAP-578

Create `hooks/useBap578.ts`:

```typescript
"use client";

import { useAccount, useReadContract, useWriteContract, useWaitForTransactionReceipt } from "wagmi";
import { parseEther, zeroAddress, encodeAbiParameters, keccak256, toBytes } from "viem";
import { BAP578_ABI, BAP578_ADDRESS, type AgentMetadata } from "@/config/bap578";

// ─── Read Hooks ───────────────────────────────────────────

/** Get remaining free mints for the connected user */
export function useFreeMints() {
  const { address } = useAccount();
  return useReadContract({
    address: BAP578_ADDRESS,
    abi: BAP578_ABI,
    functionName: "getFreeMints",
    args: address ? [address] : undefined,
    query: { enabled: !!address },
  });
}

/** Get all token IDs owned by the connected user */
export function useMyAgents() {
  const { address } = useAccount();
  return useReadContract({
    address: BAP578_ADDRESS,
    abi: BAP578_ABI,
    functionName: "tokensOfOwner",
    args: address ? [address] : undefined,
    query: { enabled: !!address },
  });
}

/** Get agent state for a specific token */
export function useAgentState(tokenId: bigint | undefined) {
  return useReadContract({
    address: BAP578_ADDRESS,
    abi: BAP578_ABI,
    functionName: "getAgentState",
    args: tokenId !== undefined ? [tokenId] : undefined,
    query: { enabled: tokenId !== undefined },
  });
}

/** Get agent metadata for a specific token */
export function useAgentMetadata(tokenId: bigint | undefined) {
  return useReadContract({
    address: BAP578_ADDRESS,
    abi: BAP578_ABI,
    functionName: "getAgentMetadata",
    args: tokenId !== undefined ? [tokenId] : undefined,
    query: { enabled: tokenId !== undefined },
  });
}

/** Get total supply of agents */
export function useTotalSupply() {
  return useReadContract({
    address: BAP578_ADDRESS,
    abi: BAP578_ABI,
    functionName: "getTotalSupply",
  });
}

/** Get the mint fee (0.01 BNB) */
export function useMintFee() {
  return useReadContract({
    address: BAP578_ADDRESS,
    abi: BAP578_ABI,
    functionName: "MINT_FEE",
  });
}

/** Check if a token was free-minted */
export function useIsFreeMint(tokenId: bigint | undefined) {
  return useReadContract({
    address: BAP578_ADDRESS,
    abi: BAP578_ABI,
    functionName: "isFreeMint",
    args: tokenId !== undefined ? [tokenId] : undefined,
    query: { enabled: tokenId !== undefined },
  });
}

// ─── Write Hooks ──────────────────────────────────────────

/** Mint a new agent */
export function useCreateAgent() {
  const { writeContract, data: hash, isPending, error } = useWriteContract();
  const { isLoading: isConfirming, isSuccess } = useWaitForTransactionReceipt({ hash });

  const createAgent = (
    metadataURI: string,
    metadata: AgentMetadata,
    isFree: boolean,
    logicAddress: `0x${string}` = zeroAddress
  ) => {
    const { address } = useAccount();
    writeContract({
      address: BAP578_ADDRESS,
      abi: BAP578_ABI,
      functionName: "createAgent",
      args: [
        address!,       // to (mint to self)
        logicAddress,    // logic contract or zero address
        metadataURI,
        metadata,
      ],
      value: isFree ? 0n : parseEther("0.01"),
    });
  };

  return { createAgent, hash, isPending, isConfirming, isSuccess, error };
}

/** Fund an agent with BNB */
export function useFundAgent() {
  const { writeContract, data: hash, isPending, error } = useWriteContract();
  const { isLoading: isConfirming, isSuccess } = useWaitForTransactionReceipt({ hash });

  const fundAgent = (tokenId: bigint, amountBnb: string) => {
    writeContract({
      address: BAP578_ADDRESS,
      abi: BAP578_ABI,
      functionName: "fundAgent",
      args: [tokenId],
      value: parseEther(amountBnb),
    });
  };

  return { fundAgent, hash, isPending, isConfirming, isSuccess, error };
}

/** Withdraw BNB from an agent */
export function useWithdrawFromAgent() {
  const { writeContract, data: hash, isPending, error } = useWriteContract();
  const { isLoading: isConfirming, isSuccess } = useWaitForTransactionReceipt({ hash });

  const withdraw = (tokenId: bigint, amountBnb: string) => {
    writeContract({
      address: BAP578_ADDRESS,
      abi: BAP578_ABI,
      functionName: "withdrawFromAgent",
      args: [tokenId, parseEther(amountBnb)],
    });
  };

  return { withdraw, hash, isPending, isConfirming, isSuccess, error };
}

/** Toggle agent active status */
export function useSetAgentStatus() {
  const { writeContract, data: hash, isPending, error } = useWriteContract();
  const { isLoading: isConfirming, isSuccess } = useWaitForTransactionReceipt({ hash });

  const setStatus = (tokenId: bigint, active: boolean) => {
    writeContract({
      address: BAP578_ADDRESS,
      abi: BAP578_ABI,
      functionName: "setAgentStatus",
      args: [tokenId, active],
    });
  };

  return { setStatus, hash, isPending, isConfirming, isSuccess, error };
}

/** Update agent metadata */
export function useUpdateAgentMetadata() {
  const { writeContract, data: hash, isPending, error } = useWriteContract();
  const { isLoading: isConfirming, isSuccess } = useWaitForTransactionReceipt({ hash });

  const updateMetadata = (
    tokenId: bigint,
    newMetadataURI: string,
    newMetadata: AgentMetadata
  ) => {
    writeContract({
      address: BAP578_ADDRESS,
      abi: BAP578_ABI,
      functionName: "updateAgentMetadata",
      args: [tokenId, newMetadataURI, newMetadata],
    });
  };

  return { updateMetadata, hash, isPending, isConfirming, isSuccess, error };
}

// ─── Utility ──────────────────────────────────────────────

/** Compute vault hash for off-chain content verification */
export function computeVaultHash(content: string): `0x${string}` {
  return keccak256(toBytes(content));
}
```

---

## Step 6 — Key UI Components

### 6a. Wallet Connect Button

```tsx
"use client";

import { ConnectButton } from "@rainbow-me/rainbowkit";

export function WalletButton() {
  return (
    <ConnectButton
      chainStatus="icon"
      showBalance={true}
      accountStatus="avatar"
    />
  );
}
```

### 6b. Mint Agent Form

```tsx
"use client";

import { useState } from "react";
import { useAccount } from "wagmi";
import { zeroAddress } from "viem";
import { useCreateAgent, useFreeMints, computeVaultHash } from "@/hooks/useBap578";
import type { AgentMetadata } from "@/config/bap578";

export function MintAgentForm() {
  const { address, isConnected } = useAccount();
  const { data: freeMints } = useFreeMints();
  const { createAgent, isPending, isConfirming, isSuccess, error } = useCreateAgent();

  const [persona, setPersona] = useState('{"traits": ["helpful"], "style": "friendly"}');
  const [experience, setExperience] = useState("AI Assistant");
  const [metadataURI, setMetadataURI] = useState("");

  if (!isConnected) return <p>Connect your wallet to mint an agent.</p>;

  const isFree = freeMints !== undefined && freeMints > 0n;

  const handleMint = () => {
    const metadata: AgentMetadata = {
      persona,
      experience,
      voiceHash: "",
      animationURI: "",
      vaultURI: "",
      vaultHash: "0x0000000000000000000000000000000000000000000000000000000000000000",
    };
    createAgent(metadataURI || "ipfs://placeholder", metadata, isFree);
  };

  return (
    <div className="space-y-4">
      <h2 className="text-xl font-bold">Mint New Agent</h2>

      {isFree && (
        <p className="text-green-500">
          🎁 You have {freeMints?.toString()} free mints remaining!
        </p>
      )}
      {!isFree && <p className="text-yellow-500">💎 Mint fee: 0.01 BNB</p>}

      <input
        placeholder="Metadata URI (e.g., ipfs://...)"
        value={metadataURI}
        onChange={(e) => setMetadataURI(e.target.value)}
        className="w-full p-2 border rounded"
      />
      <input
        placeholder='Persona JSON'
        value={persona}
        onChange={(e) => setPersona(e.target.value)}
        className="w-full p-2 border rounded"
      />
      <input
        placeholder="Experience / Role"
        value={experience}
        onChange={(e) => setExperience(e.target.value)}
        className="w-full p-2 border rounded"
      />

      <button
        onClick={handleMint}
        disabled={isPending || isConfirming}
        className="px-4 py-2 bg-yellow-500 text-black font-bold rounded disabled:opacity-50"
      >
        {isPending
          ? "Confirm in wallet..."
          : isConfirming
            ? "Minting..."
            : isFree
              ? "Mint (Free)"
              : "Mint (0.01 BNB)"}
      </button>

      {isSuccess && <p className="text-green-500">✅ Agent minted successfully!</p>}
      {error && <p className="text-red-500">❌ {error.message}</p>}
    </div>
  );
}
```

### 6c. Agent Dashboard (List My Agents)

```tsx
"use client";

import { useMyAgents, useAgentState, useAgentMetadata } from "@/hooks/useBap578";
import { formatEther } from "viem";

function AgentCard({ tokenId }: { tokenId: bigint }) {
  const { data: state } = useAgentState(tokenId);
  const { data: metaResult } = useAgentMetadata(tokenId);

  if (!state || !metaResult) return <div>Loading agent #{tokenId.toString()}...</div>;

  const [balance, active, logicAddress, createdAt, owner] = state as any;
  const [metadata, metadataURI] = metaResult as any;

  let parsedPersona: any = {};
  try {
    parsedPersona = JSON.parse(metadata.persona);
  } catch {
    parsedPersona = { raw: metadata.persona };
  }

  return (
    <div className="border rounded-lg p-4 space-y-2">
      <div className="flex justify-between items-center">
        <h3 className="font-bold text-lg">Agent #{tokenId.toString()}</h3>
        <span className={active ? "text-green-500" : "text-gray-400"}>
          {active ? "● Active" : "○ Inactive"}
        </span>
      </div>
      <p><strong>Experience:</strong> {metadata.experience}</p>
      <p><strong>Balance:</strong> {formatEther(balance)} BNB</p>
      <p><strong>Created:</strong> {new Date(Number(createdAt) * 1000).toLocaleDateString()}</p>
      {metadata.vaultURI && <p><strong>Vault:</strong> {metadata.vaultURI}</p>}
      <p className="text-xs text-gray-500 truncate"><strong>URI:</strong> {metadataURI}</p>
    </div>
  );
}

export function AgentDashboard() {
  const { data: tokenIds, isLoading } = useMyAgents();

  if (isLoading) return <p>Loading your agents...</p>;
  if (!tokenIds || tokenIds.length === 0) return <p>You don't own any agents yet.</p>;

  return (
    <div className="space-y-4">
      <h2 className="text-xl font-bold">My Agents ({tokenIds.length})</h2>
      <div className="grid gap-4 md:grid-cols-2">
        {tokenIds.map((id: bigint) => (
          <AgentCard key={id.toString()} tokenId={id} />
        ))}
      </div>
    </div>
  );
}
```

### 6d. Fund Agent Component

```tsx
"use client";

import { useState } from "react";
import { useFundAgent } from "@/hooks/useBap578";

export function FundAgent({ tokenId }: { tokenId: bigint }) {
  const [amount, setAmount] = useState("");
  const { fundAgent, isPending, isConfirming, isSuccess, error } = useFundAgent();

  return (
    <div className="flex gap-2 items-center">
      <input
        type="number"
        step="0.001"
        placeholder="BNB amount"
        value={amount}
        onChange={(e) => setAmount(e.target.value)}
        className="p-2 border rounded w-32"
      />
      <button
        onClick={() => fundAgent(tokenId, amount)}
        disabled={isPending || isConfirming || !amount}
        className="px-3 py-2 bg-blue-500 text-white rounded disabled:opacity-50"
      >
        {isPending ? "Confirm..." : isConfirming ? "Sending..." : "Fund"}
      </button>
      {isSuccess && <span className="text-green-500">✅</span>}
      {error && <span className="text-red-500" title={error.message}>❌</span>}
    </div>
  );
}
```

### 6e. Agent Identity Card (The Four Questions)

```tsx
"use client";

import { useAgentState, useAgentMetadata, useIsFreeMint } from "@/hooks/useBap578";
import { formatEther, zeroAddress } from "viem";

export function AgentIdentityCard({ tokenId }: { tokenId: bigint }) {
  const { data: state } = useAgentState(tokenId);
  const { data: metaResult } = useAgentMetadata(tokenId);
  const { data: isFreeMinted } = useIsFreeMint(tokenId);

  if (!state || !metaResult) return <div>Loading...</div>;

  const [balance, active, logicAddress, createdAt, owner] = state as any;
  const [metadata, metadataURI] = metaResult as any;

  return (
    <div className="border rounded-lg p-6 space-y-4 max-w-lg">
      <h2 className="text-2xl font-bold">Agent #{tokenId.toString()}</h2>

      {/* Who are you? */}
      <section>
        <h3 className="font-semibold text-yellow-500">🤖 Who are you?</h3>
        <p>I am NFA #{tokenId.toString()}, owned by <code className="text-xs">{owner}</code>.</p>
        <p><strong>Experience:</strong> {metadata.experience || "Not set"}</p>
        <p><strong>Persona:</strong> {metadata.persona || "Not set"}</p>
        {logicAddress !== zeroAddress && (
          <p><strong>Logic contract:</strong> <code className="text-xs">{logicAddress}</code></p>
        )}
      </section>

      {/* What do you remember? */}
      <section>
        <h3 className="font-semibold text-blue-500">🧠 What do you remember?</h3>
        <p><strong>Created:</strong> {new Date(Number(createdAt) * 1000).toLocaleString()}</p>
        <p><strong>Status:</strong> {active ? "Active" : "Inactive"}</p>
        {metadata.vaultURI && <p><strong>Vault:</strong> {metadata.vaultURI}</p>}
        {metadata.voiceHash && <p><strong>Voice:</strong> {metadata.voiceHash}</p>}
      </section>

      {/* What can you do? */}
      <section>
        <h3 className="font-semibold text-green-500">⚡ What can you do?</h3>
        <p><strong>Balance:</strong> {formatEther(balance)} BNB</p>
        <p><strong>Logic bound:</strong> {logicAddress !== zeroAddress ? "Yes" : "No (autonomous behavior not set)"}</p>
        <p><strong>Transferable:</strong> {isFreeMinted ? "No (free mint — soulbound)" : "Yes"}</p>
      </section>

      {/* How can I trust it? */}
      <section>
        <h3 className="font-semibold text-purple-500">🔒 How can I trust it?</h3>
        <p><strong>On-chain metadata URI:</strong> <a href={metadataURI} target="_blank" className="underline text-xs">{metadataURI}</a></p>
        <p><strong>Vault hash:</strong> <code className="text-xs break-all">{metadata.vaultHash}</code></p>
        <p className="text-xs text-gray-500">
          Verify by comparing keccak256(vault content) with the on-chain vaultHash.
          Check source code on BscScan for full transparency.
        </p>
      </section>
    </div>
  );
}
```

---

## Step 7 — Vault Hash Verification (Client-Side)

```typescript
import { keccak256, toBytes } from "viem";

/**
 * Verify off-chain vault content against on-chain hash.
 * @param vaultContent - The raw content fetched from vaultURI
 * @param onChainHash - The vaultHash stored on-chain (bytes32)
 * @returns true if content is authentic
 */
export function verifyVaultIntegrity(
  vaultContent: string,
  onChainHash: `0x${string}`
): boolean {
  const computedHash = keccak256(toBytes(vaultContent));
  return computedHash === onChainHash;
}
```

---

## Step 8 — Listening to Events

```typescript
import { useWatchContractEvent } from "wagmi";
import { BAP578_ABI, BAP578_ADDRESS } from "@/config/bap578";

/** Watch for new agent mints in real-time */
export function useWatchAgentCreated(
  onAgentCreated: (tokenId: bigint, owner: `0x${string}`) => void
) {
  useWatchContractEvent({
    address: BAP578_ADDRESS,
    abi: BAP578_ABI,
    eventName: "AgentCreated",
    onLogs(logs) {
      for (const log of logs) {
        const { tokenId, owner } = log.args as any;
        onAgentCreated(tokenId, owner);
      }
    },
  });
}

/** Watch for agent funding events */
export function useWatchAgentFunded(
  tokenId: bigint,
  onFunded: (amount: bigint) => void
) {
  useWatchContractEvent({
    address: BAP578_ADDRESS,
    abi: BAP578_ABI,
    eventName: "AgentFunded",
    onLogs(logs) {
      for (const log of logs) {
        const args = log.args as any;
        if (args.tokenId === tokenId) {
          onFunded(args.amount);
        }
      }
    },
  });
}
```

---

## File Structure

When complete, the Web3 integration should look like this:

```
web/
├── config/
│   ├── web3.ts              # wagmi + RainbowKit config
│   └── bap578.ts            # ABI, address, types
├── providers/
│   └── Web3Provider.tsx      # WagmiProvider + QueryClient + RainbowKit
├── hooks/
│   └── useBap578.ts         # All read/write hooks
├── components/
│   └── nfa/
│       ├── WalletButton.tsx
│       ├── MintAgentForm.tsx
│       ├── AgentDashboard.tsx
│       ├── AgentIdentityCard.tsx
│       └── FundAgent.tsx
└── utils/
    └── vaultVerify.ts       # Client-side vault hash verification
```

---

## Environment Variables Checklist

| Variable | Required | Description |
|----------|----------|-------------|
| `NEXT_PUBLIC_WALLETCONNECT_PROJECT_ID` | Yes | From [cloud.walletconnect.com](https://cloud.walletconnect.com) |
| `NEXT_PUBLIC_BAP578_PROXY_ADDRESS` | Yes | Deployed proxy contract address |
| `NEXT_PUBLIC_CHAIN_ID` | Optional | Default chain (97 for testnet, 56 for mainnet) |

---

## Common Pitfalls

| Issue | Cause | Fix |
|-------|-------|-----|
| `useReadContract` returns undefined | Wallet not connected or wrong chain | Check `isConnected` and chain ID |
| "Free mints can only be minted to self" | Passing different `to` address during free mint | Always pass connected user's address |
| "Incorrect fee" | Free mints exhausted but no value sent | Check `getFreeMints` first, send 0.01 BNB if zero |
| Hydration mismatch | Wallet state differs server vs client | Use `"use client"` directive, enable `ssr: true` in config |
| "Free minted tokens are non-transferable" | Attempting to transfer a free-minted agent | Free-minted agents are soulbound by design |
| Transaction reverts silently | Contract is paused | Check `paused()` state before writes |
| ABI encoding error | Struct fields missing or wrong type | Ensure all 6 `AgentMetadata` fields are present, `vaultHash` is bytes32 |

---

## Testing the Integration

1. **Start local Hardhat node** (in the BAP-578 repo):
   ```bash
   cd non-fungible-agents-BAP-578
   npx hardhat node
   ```

2. **Deploy locally**:
   ```bash
   npm run deploy:localhost
   ```

3. **Update `.env.local`** with the local proxy address and set chain to localhost.

4. **Start Next.js dev server**:
   ```bash
   cd chatchat-frontend/web
   npm run dev
   ```

5. **Connect MetaMask** to `localhost:8545` (Chain ID 31337) and import a Hardhat test account.

6. **Test flows**: Connect → Mint (free) → View dashboard → Fund → Withdraw → Update metadata.

---

## Related Skills

- **`bap578`** — Smart contract details, spec, build/test/deploy from CLI
- **`blockchain-developer`** — General Web3 development patterns
- **`wagmi-development`** — Advanced wagmi hook patterns
