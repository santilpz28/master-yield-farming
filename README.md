# Yield Farming with `abi.encodePacked`

> A multi-pool yield farming protocol that demonstrates advanced uses of `abi.encodePacked` for compact encoding of pool, user, and transaction data — built with Foundry.

[![Solidity](https://img.shields.io/badge/Solidity-0.8.x-blue)](https://soliditylang.org)
[![Foundry](https://img.shields.io/badge/Built%20with-Foundry-red)](https://book.getfoundry.sh)
[![Tests](https://img.shields.io/badge/Tests-14%2F14%20passing-brightgreen)](#tests)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

---

## 🎯 What it does

A yield-farming protocol where users stake ERC-20 tokens into reward pools and earn rewards over time. Beyond the standard stake/withdraw/claim flow, this project uses `abi.encodePacked` for:

- **Pool identifiers** — compact, deterministic IDs derived from token + params
- **User hashes** — unique per-pool-per-user identifiers
- **Encoded pool data** — abi-encoded payload for off-chain consumers
- **Optimized storage** — fewer SSTOREs by packing related data

---

## 🏗 Architecture

```
┌──────────────────────────────────┐
│       YieldFarmingPool           │
├──────────────────────────────────┤
│  Pools: mapping(bytes32 => Pool) │
│    ├─ token (ERC-20)             │
│    ├─ rewardRate (per second)    │
│    ├─ totalStaked                │
│    ├─ accRewardPerShare          │
│    └─ lastUpdateTime             │
│                                  │
│  Users: mapping(hash => UserInfo)│
│    ├─ amount staked              │
│    ├─ rewardDebt                 │
│    └─ pending (computed)         │
└──────────────────────────────────┘
```

The `bytes32` pool ID is generated via `abi.encodePacked(token, salt, params)`, and the user hash via `abi.encodePacked(poolId, user)`. Both are deterministic and gas-efficient.

---

## 📂 Project structure

```
src/
├── YieldFarmingPool.sol    # 337 LOC — main farming logic
└── MockToken.sol           # 31 LOC — test ERC-20
test/
└── YieldFarmingPool.t.sol  # 14 tests
```

---

## 🔑 Key functions

| Function | Purpose |
|---|---|
| `createPool(token, rewardRate)` | Owner creates a new reward pool (returns `poolId`) |
| `stake(poolId, amount)` | User stakes tokens; updates reward accounting |
| `withdraw(poolId, amount)` | User withdraws staked tokens |
| `claimRewards(poolId)` | User claims accumulated rewards |
| `updatePoolRewardRate(poolId, newRate)` | Owner adjusts reward rate |
| `pendingRewards(poolId, user)` | View: pending reward for a user |
| `getPoolEncodedData(poolId)` | View: `abi.encodePacked` pool data for off-chain |
| `getUserHash(poolId, user)` | View: per-pool-per-user hash |
| `getActivePoolsCount()` / `getActivePools()` | View: enumeration |
| `emergencyWithdraw(token, amount)` | Owner recovery (post-incident) |

---

## 🛡 Security considerations

- **ReentrancyGuard** on `stake`, `withdraw`, `claimRewards`
- **Reward accounting** uses the `MasterChef`-style `accRewardPerShare` pattern to avoid per-user iteration on global state updates
- **Safe reward transfer** — uses a try/catch / pull-payment pattern to avoid blocking the entire pool on a single failing transfer
- **Owner-only** market parameters

> ⚠️ This is a **Master's project**, not production. The educational focus is on `abi.encodePacked` patterns and MasterChef-style reward math. Real yield farms add: time-locked rewards, emission schedules, halving, multi-token pools, slippage protection on reward tokens, etc.

---

## 🧪 Tests (14 passing)

```bash
bash install.sh
forge build
forge test -vv
```

Coverage highlights:

- **Pool creation** — success, owner-only, rate bounds
- **Staking** — first deposit, multiple deposits, balance accounting
- **Withdrawal** — partial, full, reverts on insufficient
- **Reward claiming** — accuracy of pending math across time
- **Pool update** — owner-only, event emission, accounting consistency
- **Edge cases** — zero stake, multiple users, reward distribution fairness

---

## 📚 Concepts demonstrated

- `abi.encodePacked` for compact, gas-efficient identifiers
- MasterChef-style `accRewardPerShare` reward distribution
- Multi-pool management
- Off-chain queryability via encoded data
- Foundry test patterns for time-dependent logic (using `vm.warp`)

---

## 📜 License

MIT — see [LICENSE](LICENSE)

---

**Author:** Santiago López Castaño · [@santilpz28](https://github.com/santilpz28)
Built as part of the **Master in Blockchain Development** (2026) — Advanced Solidity module, `abi.encodePacked` workshop.
